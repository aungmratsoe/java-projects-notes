# QRCode Project ကို RMI Client-Server ပြောင်းနည်း

သင့် project ရဲ့ package structure ကို ကြည့်တဲ့အခါ ဒါက **typical layered desktop app** ပါ — `ui` က `dao` ကို တိုက်ရိုက်ခေါ်၊ `dao` က `db` ကို တိုက်ရိုက်ခေါ်နေတဲ့ pattern ဖြစ်နိုင်ပါတယ်။ RMI ပြောင်းဖို့ **အဓိက idea** က — `db`/`dao` (database access ရှိသမျှ) ကို **Server computer** ပေါ်ထားပြီး, `ui`/`components` ကို **Client computer** ပေါ်ထားမယ်၊ ကြားထဲမှာ RMI interface layer တစ်ခု ထည့်မယ်။

## Before → After Architecture

```
BEFORE (Single computer, direct calls):
┌─────────────────────────────────┐
│  ui  →  dao  →  db (Database)   │
│  (Server, Client သီးခြားမရှိ)    │
└─────────────────────────────────┘

AFTER (Two computers, RMI):
┌─── Client Computer ───┐        ┌─── Server Computer ───┐
│  ui                   │        │  service (impl)       │
│  components            │  RMI  │  dao                   │
│  (db/dao ကို မခေါ်     │◄─────►│  db (Database ချိတ်)   │
│   တော့ပါ)              │       │                        │
└────────────────────────┘        └────────────────────────┘
        ▲
        │ shared (နှစ်ဖက်စလုံးမှာ ရှိရမယ်)
   model, exceptions, 
   service interfaces (Remote)
```

## Package Structure ပြင်ဆင်ရမယ့်ပုံ

|Package အဟောင်း|ဘယ်ကို ပြောင်းရမလဲ|
|---|---|
|`com.ams.qrcode.model`|**Server + Client နှစ်ဖက်စလုံး** — `Serializable` implement ထည့်ရမယ်|
|`com.ams.qrcode.exceptions`|**Server + Client နှစ်ဖက်စလုံး** — `Serializable` ဖြစ်အောင် စစ်ရမယ်|
|`com.ams.qrcode.dao`|**Server ဘက်ပဲ ကျန်ခဲ့** — ပြောင်းစရာမလို (internal logic အတိုင်း)|
|`com.ams.qrcode.db`|**Server ဘက်ပဲ ကျန်ခဲ့** — Client ကို ကူးစရာ **လုံးဝမလို**|
|`com.ams.qrcode.ui`|**Client ဘက်ပဲ ကျန်ခဲ့** — dao ခေါ်နေတာကို RMI service ခေါ်တာအဖြစ် ပြောင်းရမယ်|
|`com.ams.qrcode.components`|**Client ဘက်ပဲ ကျန်ခဲ့** — ပြောင်းစရာမလို|
|`com.ams.qrcode.utils`|Utility အမျိုးအစားပေါ် မူတည် — DB-related ဆိုရင် Server, UI-related ဆိုရင် Client|
|**`com.ams.qrcode.service`** (အသစ်)|Remote interface — **နှစ်ဖက်စလုံးမှာ ရှိရမယ်**|
|**`com.ams.qrcode.service.impl`** (အသစ်)|Interface ကို implement — **Server ဘက်ပဲ**|
|**`com.ams.qrcode.server`** (အသစ်)|`Server.java` main class — **Server ဘက်ပဲ**|

## လက်တွေ့ ဥပမာ — QRCode DAO တစ်ခုကို RMI ပြောင်းခြင်း

သင့် `dao` package ထဲမှာ ဒီလို class ရှိတယ်လို့ ယူဆမယ်:

```java
// com.ams.qrcode.dao.QRCodeDao (ရှိပြီးသား ဥပမာ)
public interface QRCodeDao {
    void save(QRCode qrCode) throws DaoException;
    QRCode findById(int id) throws DaoException;
    List<QRCode> findAll() throws DaoException;
}
```

### Step 1 — Model ကို Serializable ဖြစ်အောင် ပြင်

```java
// com.ams.qrcode.model.QRCode
public class QRCode implements java.io.Serializable {
    private static final long serialVersionUID = 1L;
    // fields အတိုင်း ရှိသလို ထားလို့ရပါတယ် — Serializable ပဲ ထပ်ထည့်ရမယ်
}
```

### Step 2 — Custom Exception ကို Serializable စစ်

```java
// com.ams.qrcode.exceptions.DaoException
public class DaoException extends Exception implements java.io.Serializable {
    // Exception က Serializable ကို default အဆင့်တင် implement ထားပြီးသားပါ
    // ဒါပေမယ့် fields အားလုံး (cause အပါအဝင်) serializable ဖြစ်ရမယ်ဆိုတာ သတိထား
    public DaoException(String message) { super(message); }
    public DaoException(String message, Throwable cause) { super(message, cause); }
}
```

### Step 3 — Remote Service Interface အသစ် ဖန်တီး

```java
// com.ams.qrcode.service.QRCodeService.java (Server + Client နှစ်ဖက်)
package com.ams.qrcode.service;

import com.ams.qrcode.model.QRCode;
import com.ams.qrcode.exceptions.DaoException;
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.List;

public interface QRCodeService extends Remote {
    void save(QRCode qrCode) throws RemoteException, DaoException;
    QRCode findById(int id) throws RemoteException, DaoException;
    List<QRCode> findAll() throws RemoteException, DaoException;
}
```

**သတိထားရမယ့်အချက်**: RMI method **တိုင်း** မှာ `RemoteException` ကို မဖြစ်မနေ ထပ်ထည့်ရမယ် (network fail ဖြစ်နိုင်လို့) — မူလ `DaoException` ကို ဖျက်စရာမလိုပါ၊ ၂ ခုလုံး ရှိနိုင်ပါတယ်။

### Step 4 — Service Implementation (Server ဘက် — dao ကို wrap လုပ်တာပါ)

```java
// com.ams.qrcode.service.impl.QRCodeServiceImpl.java (Server ဘက်ပဲ)
package com.ams.qrcode.service.impl;

import com.ams.qrcode.service.QRCodeService;
import com.ams.qrcode.dao.QRCodeDao;
import com.ams.qrcode.dao.QRCodeDaoImpl; // ရှိပြီးသား implementation
import com.ams.qrcode.model.QRCode;
import com.ams.qrcode.exceptions.DaoException;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.List;

public class QRCodeServiceImpl extends UnicastRemoteObject implements QRCodeService {

    private final QRCodeDao dao; // ရှိပြီးသား DAO ကို ဒီထဲ ပိုက်ထားတာပါ

    public QRCodeServiceImpl() throws RemoteException {
        super();
        this.dao = new QRCodeDaoImpl(); // မူလ DAO logic ကို အပြောင်းအလဲ မလိုအပ်ပါ
    }

    @Override
    public void save(QRCode qrCode) throws RemoteException, DaoException {
        dao.save(qrCode); // ရှိပြီးသား dao method ကို တိုက်ရိုက် ခေါ်ရုံပါပဲ
    }

    @Override
    public QRCode findById(int id) throws RemoteException, DaoException {
        return dao.findById(id);
    }

    @Override
    public List<QRCode> findAll() throws RemoteException, DaoException {
        return dao.findAll();
    }
}
```

**ဒါကတော့ Pattern ရဲ့ အရေးကြီးဆုံးအချက်ပါ**: သင့် `dao` package ရဲ့ **logic ကို လုံးဝ ပြင်စရာမလိုပါ** — Service class က dao ကို ခေါ်ပြီး "RMI-callable wrapper" အဖြစ်ပဲ ဆောင်ရွက်ပေးတာပါ။

### Step 5 — Server Main Class

```java
// com.ams.qrcode.server.QRCodeServer.java (Server ဘက်ပဲ)
package com.ams.qrcode.server;

import com.ams.qrcode.service.impl.QRCodeServiceImpl;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class QRCodeServer {
    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.createRegistry(1099);
            QRCodeServiceImpl service = new QRCodeServiceImpl();
            registry.rebind("QRCodeService", service);
            System.out.println("QRCode Server စတင်ပြီးပါပြီ...");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Step 6 — UI Layer ကို ပြောင်းခြင်း (Client ဘက်)

သင့် `ui` package ထဲက class တွေမှာ ဒီလို ရှိနေမယ် (before):

```java
// com.ams.qrcode.ui.QRCodeListPanel (BEFORE)
private QRCodeDao dao = new QRCodeDaoImpl(); // တိုက်ရိုက် dao ခေါ်နေတာ

private void loadData() {
    try {
        List<QRCode> list = dao.findAll();
        // table ထဲ ထည့်
    } catch (DaoException e) { ... }
}
```

AFTER — `dao` field ကို RMI service reference အဖြစ် ပြောင်း:

```java
// com.ams.qrcode.ui.QRCodeListPanel (AFTER)
import com.ams.qrcode.service.QRCodeService;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

private QRCodeService service; // dao field အစား ဒါကို သုံး

private void connectToServer(String serverIp) throws Exception {
    Registry registry = LocateRegistry.getRegistry(serverIp, 1099);
    service = (QRCodeService) registry.lookup("QRCodeService");
}

private void loadData() {
    // Part 2 မှာ ပြခဲ့တဲ့ SwingWorker pattern သုံးရမယ် (UI freeze မဖြစ်အောင်)
    SwingWorker<List<QRCode>, Void> worker = new SwingWorker<>() {
        @Override
        protected List<QRCode> doInBackground() throws Exception {
            return service.findAll(); // dao.findAll() အစား service.findAll()
        }
        @Override
        protected void done() {
            try {
                List<QRCode> list = get();
                // table ထဲ ထည့်
            } catch (Exception e) { /* error handling */ }
        }
    };
    worker.execute();
}
```

## Dao Class များစွာ ရှိရင် ဘယ်လိုလုပ်မလဲ

`dao` package ထဲမှာ `QRCodeDao` တစ်ခုတည်း မဟုတ်ဘဲ `UserDao`, `SettingsDao` စတာတွေ ထပ်ရှိနိုင်ပါတယ် (packages list ကြည့်ရင် Model များစွာ ရှိနိုင်ပါတယ်)။ Dao class **တစ်ခုချင်းစီအတွက်**:

1. `com.ams.qrcode.service` ထဲမှာ Remote interface တစ်ခု (ဥပမာ `UserService`) ဖန်တီး
2. `com.ams.qrcode.service.impl` ထဲမှာ implementation (ဥပမာ `UserServiceImpl`) ဖန်တီး — dao ကို wrap
3. Server main class ထဲမှာ `registry.rebind("UserService", new UserServiceImpl())` ထပ်ထည့်

**Tip**: Service class အားလုံးကို တစ်ခုတည်းသော "Facade" interface ထဲ ပေါင်းပြီး register လုပ်လို့လည်း ရပါတယ် (service တစ်ခုတည်းက method အားလုံး expose) — Class အရေအတွက် သိပ်များနေရင် ဒါက ပိုစီမံလွယ်ပါတယ်။

## Migration လုပ်ငန်းစဉ် — အကြံပြု အဆင့်ဆင့်

1. **`model`** package ရဲ့ class တွေ အားလုံး `Serializable` ဖြစ်အောင် စစ်/ပြင်
2. **`exceptions`** package ရဲ့ class တွေ Serializable ဖြစ်ကြောင်း သေချာအောင် စစ်
3. **`service`** package အသစ်ဖန်တီးပြီး `dao` interface တစ်ခုချင်းစီအတွက် Remote interface တစ်ခုစီ ရေး
4. **`service.impl`** package ဖန်တီးပြီး existing `dao` implementation ကို wrap
5. **`server`** package ဖန်တီးပြီး Server main class ရေး + register services အားလုံး
6. **`ui`** package ထဲက `dao` field/import တွေအားလုံးကို `service` field/import အဖြစ် လဲ
7. **`db`** package ကို Server side project ထဲကပဲ ချန်ထား — Client ကို ပို့ပေးစရာမလို
8. Client GUI startup ထဲမှာ **login/connect screen** ထည့် (Part 5 က Auth ပါ ပေါင်းချင်ရင်)
9. Maven project ကို **module ၂ ခု** (`qrcode-common`, `qrcode-server`, `qrcode-client`) ခွဲထုတ်ဖို့ စဉ်းစားနိုင် (Part 2 ရဲ့ pom.xml shade plugin approach အတိုင်း jar ၂ ခု ခွဲထုတ်လို့လည်း ရပါတယ်)

---

သင့် project ထဲက **`dao` interface တစ်ခုချင်းစီရဲ့ method list** ကို ပြပေးရင် ကျန်တဲ့ Service interface တွေကို တစ်ခါတည်း template ဆွဲပေးနိုင်ပါတယ်။ ဒါမှမဟုတ် **module ခွဲထုတ်တဲ့ pom.xml structure** (common/server/client) ကို အသေးစိတ် ကူညီရမလား?