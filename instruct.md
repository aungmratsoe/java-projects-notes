# RMI နဲ့ Client / Server အတွက် အဆင့်လိုက် လမ်းညွှန်ချက်

## 1. စီမံခန့်ခွဲပုံ

Project ရဲ့ လောလောဆယ်ဖွဲ့စည်းပုံ

- UI နဲ့ Login စနစ်: `src/main/java/com/ams/login/ui/`
- Database connection: `src/main/java/com/ams/login/db/DBConnection.java`
- Photo data access: `src/main/java/com/ams/login/dao/PhotoDAO.java`
- Main application launcher: `src/main/java/com/ams/login/Main.java`

ကျွန်တော်တို့ RMI ဖြင့် ခွဲချင်တာ

- `server` side: Database access, login logic, RMI service implementation
- `client` side: Swing UI, user input, remote service ကို ခေါ်သုံးခြင်း
- `shared` package: RMI interface ကို client-server နှစ်ဖက်လုံး share လုပ်

## 2. RMI အခြေခံ ကိုယ်စားလှယ် (Remote interface) ဖန်တီးခြင်း

`src/main/java/com/ams/login/rmi/AuthService.java` ဆိုပြီး interface file တစ်ခု ဖန်တီးပါ။

```java
package com.ams.login.rmi;

import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.List;
import com.ams.login.model.PhotoItem;

public interface AuthService extends Remote {
    boolean login(String username, String password) throws RemoteException;
    List<PhotoItem> getAllPhotos() throws RemoteException;
}
```

နောက်ထပ် service method လိုချင်ရင် ဒီ interface ရဲ့ method တွေကိုပဲ add ပါ။

## 3. Server-side implementation ရေးခြင်း

`src/main/java/com/ams/login/server/AuthServiceImpl.java` ဖိုင်ကို သင့် project ထဲ ဖန်တီးပါ။

```java
package com.ams.login.server;

import com.ams.login.db.DBConnection;
import com.ams.login.model.PhotoItem;
import com.ams.login.rmi.AuthService;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class AuthServiceImpl extends UnicastRemoteObject implements AuthService {
    public AuthServiceImpl() throws RemoteException {
        super();
    }

    @Override
    public boolean login(String username, String password) throws RemoteException {
        // အချက်အလက်စစ်ဆေးရန် DB မှ SQL query ချပြီး စစ်ပါ
        // လက်ရှိ project မှာ username/password ကို hardcode နဲ့စစ်နေပြီး ဖြစ်တယ်
        return "photogallery".equals(username) && "password".equals(password);
    }

    @Override
    public List<PhotoItem> getAllPhotos() throws RemoteException {
        List<PhotoItem> photos = new ArrayList<>();
        String sql = "SELECT id, photo_name, photo_path, created_at FROM photos ORDER BY id DESC";

        try (Connection conn = new DBConnection().getDBConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql);
             ResultSet rs = pstmt.executeQuery()) {

            while (rs.next()) {
                photos.add(new PhotoItem(
                    rs.getInt("id"),
                    rs.getString("photo_name"),
                    rs.getString("photo_path"),
                    rs.getTimestamp("created_at")
                ));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return photos;
    }
}
```

- ဒီ class က `DBConnection` ကို server မှာ သုံးပြီး database ကို ချိတ်ဆက်သည်။
- RMI service မှ client ကို data return ပြန်ပေးသည်။

## 4. Server main app ထည့်ခြင်း

Server ကို run လိုက်တာနဲ့ RMI registry ဖွင့်ပြီး service ကို bind ပြုလုပ်နိုင်မည်။

`src/main/java/com/ams/login/server/ServerMain.java`

```java
package com.ams.login.server;

import com.ams.login.rmi.AuthService;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class ServerMain {
    public static void main(String[] args) {
        try {
            AuthService service = new AuthServiceImpl();
            Registry registry = LocateRegistry.createRegistry(1099);
            registry.rebind("AuthService", service);
            System.out.println("RMI Server started on port 1099");
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```

## 5. Client-side ပြင်ဆင်ခြင်း

`src/main/java/com/ams/login/ui/Login.java` မှာ local login logic ကို အောက်ပါအတိုင်း ပြောင်းပါ။

- RMI server ကို connect
- `AuthService` ကို lookup
- `authService.login(...)` method ကိုခေါ်

```java
import com.ams.login.rmi.AuthService;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class Login extends javax.swing.JFrame {
    private AuthService authService;

    public Login() {
        initComponents();
        connectToServer();
    }

    private void connectToServer() {
        try {
            Registry registry = LocateRegistry.getRegistry("192.168.1.10", 1099);
            authService = (AuthService) registry.lookup("AuthService");
        } catch (Exception ex) {
            JOptionPane.showMessageDialog(this, "Server နှင့် ချိတ်ဆက်မရပါ။", "Connection Error", JOptionPane.ERROR_MESSAGE);
            ex.printStackTrace();
        }
    }

    private void login() {
        String username = txtUserName.getText().trim();
        String password = new String(jPass.getPassword());

        try {
            boolean success = authService.login(username, password);
            if (success) {
                // Login အောင်မြင်သောအခါ နောက် screen ထပ်ဖွင့်ရန်
            } else {
                JOptionPane.showMessageDialog(this, "Invalid username or password.", "Login Error", JOptionPane.ERROR_MESSAGE);
            }
        } catch (Exception ex) {
            JOptionPane.showMessageDialog(this, "Server error ဖြစ်နေပါသည်။", "Error", JOptionPane.ERROR_MESSAGE);
            ex.printStackTrace();
        }
    }
}
```

### Client side မှလည်း `PhotoGallery` ကို remote data နဲ့သာ ပြောင်းပါ

`PhotoGallery` class မှာ `PhotoDAO` ကို local direct call ထက် `authService.getAllPhotos()` ကဲ့သို့ remote RMI method တစ်ခုကိုခေါ်သင့်သည်။

## 6. Project ကို client/server သီးခြား build ချရန်

### Server

- `server` jar ထဲမှာ
  - `com.ams.login.rmi.*`
  - `com.ams.login.server.*`
  - `com.ams.login.db.*`
  - `com.ams.login.dao.*` (server side DB logic ရှိလျှင်)
- `ServerMain` ကို run ပါ
- အရေးကြီး: server က `DBConnection` ကိုသာ သုံးရမည်

### Client

- `client` jar ထဲမှာ
  - `com.ams.login.rmi.*`
  - `com.ams.login.ui.*`
  - `com.ams.login.Main.java` or `Login` main method
- `DBConnection` ကို client ထဲ မထားရပါ
- client ကတော့ server IP ကိုသုံးပြီး RMI lookup လုပ်ရမည်

## 7. Network များအတွက် ချိန်ညှိချက်

- Server က အခြားကွန်ပျူတာတွင် run နေပါက `192.168.1.10` (သို့) အခြား IP သတ်မှတ်ပါ
- Port `1099` ကို firewall ထဲ permit ရှိပါစေ
- `rmiregistry` အလိုအလျောက် `ServerMain` က create လုပ်တတ်သည်။ ထူးခြားချိန်မှာ manual သာ run ပေးပါ။

## 8. RMI ကို run ဖို့ နည်းလမ်း

1. Server machine ပေါ်တွင် `ServerMain` run
2. Client machine ပေါ်တွင် client jar run
3. Client က `getRegistry("SERVER_IP", 1099)` အားသုံးပြီး `AuthService` တိုက်ရိုက် lookup
4. Login button တစ်ချက်နဲ့ `authService.login(...)` ကိုခေါ်

## 9. လုပ်ရန်အချက်များ

1. `com.ams.login.rmi.AuthService` ဖန်တီးပါ
2. `com.ams.login.server.AuthServiceImpl` ဖန်တီးပါ
3. `com.ams.login.server.ServerMain` ဖန်တီးပြီး RMI register လုပ်ပါ
4. `src/main/java/com/ams/login/ui/Login.java` ထဲမှ local login logic ကို RMI call ပြောင်းပါ
5. `PhotoGallery` ကို local DB ကို မခေါ်ပဲ server မှ `getAllPhotos()` return method ကိုသုံးပါ
6. `DBConnection` ကို server ကသာသုံးပြီး client မှ database ကိုတိုက်ရိုက်မခေါ်ရဘူး
7. Server address ကို client က သတ်မှတ်ပါ

## 10. အရေးကြီး သတိပေးချက်

- RMI ျဖင့်ချိတ်ဆက်သော်လည်း client က database credentials ကို မပေးသင့်ပါ
- Server က database connection ကိုသာ စီမံရမည်
- Client က UI နှင့် RMI call အတွက်သာ ရှိရမည်
- `AuthService` interface ကို `shared` package အဖြစ် client နှင့် server နှစ်ဖက်လုံးတူညီရမည်

---

အကယ်၍ သင်လိုချင်ပါက ဒီ instruction အရ `AuthService` interface နမူနာ code, `ServerMain` နမူနာ code, `Login` client-side update code ကိုလည်း တိုက်ရိုက်ပြင်ပေးနိုင်ပါတယ်။
