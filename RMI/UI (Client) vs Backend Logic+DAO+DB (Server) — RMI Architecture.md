# UI (Client) vs Backend Logic+DAO+DB (Server) — RMI Architecture

ဒါက **layered architecture ကို physically ၂ computer ခွဲထားတာ** ဖြစ်ပြီး RMI နဲ့ အကောင်းဆုံး ကိုက်ညီတဲ့ pattern ပါ။ အောက်မှာ project တစ်ခုလုံးရဲ့ structure ကနေ deploy လုပ်နည်းအထိ အဆင့်ဆင့် ရှင်းပြပါမယ်။

## Package/Project Structure (အကြံပြု)

```
project-root/
├── common/              ← Server + Client နှစ်ဖက်စလုံးမှာ ရှိရမယ်
│   ├── model/           (Entity classes — Serializable)
│   ├── exceptions/      (Custom exceptions — Serializable)
│   └── service/         (Remote interfaces — extends Remote)
│
├── server/              ← Server computer ပေါ်ပဲ run
│   ├── db/              (DB connection: JDBC/Connection Pool)
│   ├── dao/             (SQL queries, CRUD logic)
│   ├── service/impl/    (Remote interface ကို implement, dao ကို ခေါ်)
│   └── ServerMain.java  (Registry start + bind)
│
└── client/              ← Client computer ပေါ်ပဲ run
    ├── ui/               (JFrame, JPanel စသည်)
    ├── components/       (Custom Swing components)
    └── ClientMain.java   (Registry lookup + UI launch)
```

## Layer တစ်ခုချင်းစီ ဘာလုပ်ရမလဲ

|Layer|ရှိရမယ့်နေရာ|တာဝန်|
|---|---|---|
|`model`|Server + Client|Data object (QRCode, User, Product...) — Network ကနေတဆင့် ပို့လို့ရအောင် `Serializable` ဖြစ်ရမယ်|
|`exceptions`|Server + Client|Custom exception (DaoException, ValidationException)|
|`service` (interface)|Server + Client|`Remote` extend လုပ်ထားတဲ့ interface — "API contract" ပါ|
|`db`|**Server ချည်း**|JDBC connection, connection pool — Client ကို **လုံးဝ မထုတ်ရ**|
|`dao`|**Server ချည်း**|SQL query logic — မူလအတိုင်း ပြင်စရာမလို|
|`service.impl`|**Server ချည်း**|`dao` ကို wrap ပြီး `UnicastRemoteObject` extend|
|`ui`|**Client ချည်း**|Swing component — `dao`/`db` ကို **direct import လုံးဝ မလုပ်ရ**|

## Full Flow — QRCode ဥပမာနဲ့ ပြန်ချိတ်ကြည့်ရအောင်

### 1. `common/model/QRCode.java` (Server + Client)

```java
package com.ams.common.model;

public class QRCode implements java.io.Serializable {
    private static final long serialVersionUID = 1L;
    private int id;
    private String content;
    private String createdDate;
    // getters/setters
}
```

### 2. `common/service/QRCodeService.java` (Server + Client) — "Contract"

```java
package com.ams.common.service;

import com.ams.common.model.QRCode;
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.List;

public interface QRCodeService extends Remote {
    void save(QRCode qr) throws RemoteException;
    QRCode findById(int id) throws RemoteException;
    List<QRCode> findAll() throws RemoteException;
    void delete(int id) throws RemoteException;
}
```

### 3. `server/dao/QRCodeDao.java` (Server ချည်း — မူလ logic အတိုင်း)

```java
package com.ams.server.dao;

import com.ams.common.model.QRCode;
import com.ams.server.db.DbConnection;
import java.sql.*;
import java.util.*;

public class QRCodeDao {
    public void save(QRCode qr) throws SQLException {
        try (Connection con = DbConnection.getConnection();
             PreparedStatement ps = con.prepareStatement(
                 "INSERT INTO qrcode(content) VALUES (?)")) {
            ps.setString(1, qr.getContent());
            ps.executeUpdate();
        }
    }

    public List<QRCode> findAll() throws SQLException {
        List<QRCode> list = new ArrayList<>();
        try (Connection con = DbConnection.getConnection();
             Statement st = con.createStatement();
             ResultSet rs = st.executeQuery("SELECT * FROM qrcode")) {
            while (rs.next()) {
                QRCode qr = new QRCode();
                qr.setId(rs.getInt("id"));
                qr.setContent(rs.getString("content"));
                list.add(qr);
            }
        }
        return list;
    }
    // findById(), delete() ... အလားတူ
}
```

### 4. `server/service/impl/QRCodeServiceImpl.java` (Server ချည်း — "Bridge")

```java
package com.ams.server.service.impl;

import com.ams.common.service.QRCodeService;
import com.ams.common.model.QRCode;
import com.ams.server.dao.QRCodeDao;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.sql.SQLException;
import java.util.List;

public class QRCodeServiceImpl extends UnicastRemoteObject implements QRCodeService {

    private final QRCodeDao dao = new QRCodeDao();

    public QRCodeServiceImpl() throws RemoteException {
        super();
    }

    @Override
    public void save(QRCode qr) throws RemoteException {
        try {
            dao.save(qr);
        } catch (SQLException e) {
            throw new RemoteException("Save failed: " + e.getMessage(), e);
        }
    }

    @Override
    public List<QRCode> findAll() throws RemoteException {
        try {
            return dao.findAll();
        } catch (SQLException e) {
            throw new RemoteException("Fetch failed: " + e.getMessage(), e);
        }
    }
    // findById(), delete() ... အလားတူ
}
```

**Pattern**: SQLException (checked, non-serializable-friendly issues) ကို `RemoteException` ထဲ wrap ပြီး client ဆီ ပို့တာက clean နည်းပါ — client ဘက်က `SQLException`, `Connection` classes import လုပ်စရာ လုံးဝမလိုအောင်။

### 5. `server/ServerMain.java`

```java
package com.ams.server;

import com.ams.server.service.impl.QRCodeServiceImpl;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class ServerMain {
    public static void main(String[] args) {
        try {
            System.setProperty("java.rmi.server.hostname", "192.168.1.10");
            Registry registry = LocateRegistry.createRegistry(1099);

            registry.rebind("QRCodeService", new QRCodeServiceImpl());
            // Dao class ထပ်ရှိရင် ဒီနေရာမှာ ထပ်ထည့် register:
            // registry.rebind("UserService", new UserServiceImpl());

            System.out.println("Server ready on port 1099...");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 6. `client/ClientMain.java` + `client/ui/QRCodeListPanel.java`

```java
package com.ams.client;

import com.ams.common.service.QRCodeService;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import javax.swing.SwingUtilities;

public class ClientMain {
    public static void main(String[] args) throws Exception {
        Registry registry = LocateRegistry.getRegistry("192.168.1.10", 1099);
        QRCodeService service = (QRCodeService) registry.lookup("QRCodeService");

        SwingUtilities.invokeLater(() -> new MainFrame(service).setVisible(true));
        // service ကို UI panel များဆီ constructor/setter ကနေတဆင့် ပေးလိုက်တာ
    }
}
```

```java
package com.ams.client.ui;

import com.ams.common.service.QRCodeService;
import com.ams.common.model.QRCode;
import javax.swing.*;
import java.util.List;

public class QRCodeListPanel extends JPanel {

    private final QRCodeService service; // dao field လုံးဝ မရှိတော့ပါ

    public QRCodeListPanel(QRCodeService service) {
        this.service = service;
        loadData();
    }

    private void loadData() {
        SwingWorker<List<QRCode>, Void> worker = new SwingWorker<>() {
            @Override
            protected List<QRCode> doInBackground() throws Exception {
                return service.findAll(); // RMI call
            }
            @Override
            protected void done() {
                try {
                    List<QRCode> list = get();
                    // JTable/JList ထဲ ထည့်
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(QRCodeListPanel.this,
                        "Server ဆက်သွယ်မှု ချို့ယွင်းနေပါသည်: " + ex.getMessage());
                }
            }
        };
        worker.execute();
    }
}
```

## Maven Multi-Module Setup (`common`/`server`/`client`)

3 module ခွဲထားချင်ရင် parent pom.xml:

```xml
<project>
    <groupId>com.ams</groupId>
    <artifactId>qrcode-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>common</module>
        <module>server</module>
        <module>client</module>
    </modules>
    <properties>
        <maven.compiler.release>25</maven.compiler.release>
    </properties>
</project>
```

`server/pom.xml` နဲ့ `client/pom.xml` နှစ်ခုစလုံးက `common` module ကို dependency အနေနဲ့ ထည့်ရမယ်:

```xml
<dependency>
    <groupId>com.ams</groupId>
    <artifactId>common</artifactId>
    <version>1.0.0</version>
</dependency>
```

`db`, `dao` dependency (JDBC driver, connection pool library) ကတော့ `server/pom.xml` ထဲမှာသာ ထည့်ပါ — `client/pom.xml` က DB driver **လုံးဝ မလို**ပါ။

## Checklist — အားလုံးပြီးဆုံးမှ

|Item|Status|
|---|---|
|`model` classes အားလုံး Serializable|☐|
|`service` interface (Remote) — dao method အားလုံးအတွက် ရေးပြီးလား|☐|
|`service.impl` — dao ကို wrap၊ Exception ကို RemoteException ပြောင်းပြီးလား|☐|
|`ServerMain` — Registry create + service အားလုံး rebind|☐|
|`ui` package ထဲက dao/db import အားလုံး ဖယ်ထုတ်ပြီးလား|☐|
|SwingWorker သုံးပြီး method ခေါ်ထားလား (UI freeze မဖြစ်အောင်)|☐|
|`java.rmi.server.hostname` Server ရဲ့ actual IP နဲ့ set ထားလား|☐|
|Firewall — port 1099 + dynamic port (fix ထားရင် အဲ့ port) ဖွင့်ထားလား|☐|

---

Multi-module Maven pom.xml (`common`/`server`/`client` သီးခြား ၃ ခု) အပြည့်အစုံ ရေးပေးရမလား၊ ဒါမှမဟုတ် သင့် project ထဲက dao/model class list ပေးရင် Service interface template တွေ တစ်ခါတည်း generate ပေးရမလား?