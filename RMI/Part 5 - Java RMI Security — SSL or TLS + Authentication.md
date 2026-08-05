# Java RMI Security — SSL/TLS + Authentication — Part 5

Plain RMI (Part 1-4 မှာ လုပ်ခဲ့တဲ့ setup) မှာ **encryption မရှိပါ, authentication လည်း built-in မရှိပါ**။ Network ပေါ်က data အားလုံး **plain text** အနေနဲ့ ပြေးနေတာဖြစ်လို့ (username/password, sensitive data) — production မှာ သုံးမယ်ဆိုရင် ၂ ခုကို ထည့်ရပါမယ်:

1. **SSL/TLS** — Network traffic ကို encrypt လုပ်ဖို့ (sniffing/eavesdropping ကာကွယ်ဖို့)
2. **Authentication** — RMI object ကို ဘယ်သူမှမဆို ခေါ်နိုင်တာမဟုတ်ဘဲ login ဝင်ပြီးမှ ခေါ်ခွင့်ရအောင်

## အပိုင်း ၁ — SSL/TLS (Encrypted Communication)

### Step 1 — Keystore & Truststore ဖန်တီးမယ် (Server Computer ပေါ်)

Command line (JDK ပါလာတဲ့ `keytool`) နဲ့ certificate ဖန်တီးရပါမယ်:

```bash
# Server ရဲ့ private key + self-signed certificate ဖန်တီး
keytool -genkeypair -alias serverkey -keyalg RSA -keysize 2048 ^
  -validity 365 -keystore server.keystore -storepass serverpass

# Server ရဲ့ certificate ကို export
keytool -exportcert -alias serverkey -keystore server.keystore ^
  -storepass serverpass -file server.cer

# Client ဘက်မှာ ဒီ certificate ကို "truststore" ထဲ import (Client ကို ကူးပေးရပါမယ်)
keytool -importcert -alias serverkey -keystore client.truststore ^
  -storepass clientpass -file server.cer -noprompt
```

- `server.keystore` — **Server computer** မှာပဲ ထားရမယ် (private key ပါလို့)
- `server.cer` → `client.truststore` — **Client computer** ကို ကူးပို့ရမယ် (public certificate ပါလို့ ဒါက share လုပ်လို့ရတယ်)

### Step 2 — Server ကို SSL Socket Factory နဲ့ Start

```java
// SecureServer.java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import javax.rmi.ssl.SslRMIClientSocketFactory;
import javax.rmi.ssl.SslRMIServerSocketFactory;

public class SecureServer {
    public static void main(String[] args) {
        // Keystore location ကို system property အနေနဲ့ ပေးရမယ်
        System.setProperty("javax.net.ssl.keyStore", "server.keystore");
        System.setProperty("javax.net.ssl.keyStorePassword", "serverpass");

        try {
            SslRMIClientSocketFactory clientFactory = new SslRMIClientSocketFactory();
            SslRMIServerSocketFactory serverFactory = new SslRMIServerSocketFactory();

            // Registry ကိုလည်း SSL factory တွေနဲ့ create
            Registry registry = LocateRegistry.createRegistry(1099, clientFactory, serverFactory);

            // Remote object ကို SSL factory တွေနဲ့ export
            InventoryImpl inventory = new InventoryImpl(clientFactory, serverFactory);
            registry.rebind("InventoryService", inventory);

            System.out.println("Secure Server (SSL) စတင်ပြီးပါပြီ...");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```java
// InventoryImpl ကို SSL export ဖြစ်အောင် constructor ပြင်ရမယ်
public class InventoryImpl extends UnicastRemoteObject implements InventoryInterface {

    protected InventoryImpl(RMIClientSocketFactory csf, RMIServerSocketFactory ssf) 
            throws RemoteException {
        super(0, csf, ssf); // port 0 = OS ကို random port ရွေးခိုင်း
    }
    // ... method များ Part 4 အတိုင်း
}
```

### Step 3 — Client ကို Truststore နဲ့ Connect

```java
// SecureClient.java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class SecureClient {
    public static void main(String[] args) {
        // Server ရဲ့ certificate ကို ယုံကြည်ဖို့ truststore ညွှန်းပေးရမယ်
        System.setProperty("javax.net.ssl.trustStore", "client.truststore");
        System.setProperty("javax.net.ssl.trustStorePassword", "clientpass");

        try {
            Registry registry = LocateRegistry.getRegistry("192.168.1.10", 1099);
            InventoryInterface inventory = (InventoryInterface) registry.lookup("InventoryService");

            System.out.println("Stock: " + inventory.getStock("Laptop"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**ဒီနေရာမှာ ဖြစ်ပျက်နေတာက** — Server နဲ့ Client ကြားက data အားလုံး TLS handshake ပြီးမှ encrypted channel ကနေတဆင့် ပို့နေတာပါ။ Client ဘက်မှာ truststore မထားရင် (သို့) certificate မတူရင် connection **fail** ဖြစ်ပါလိမ့်မယ်။

## အပိုင်း ၂ — Authentication (Login/Password)

RMI မှာ built-in login mechanism မရှိလို့ **manual authentication layer** ကိုယ်တိုင်ရေးရပါမယ်။ Pattern အများဆုံးသုံးတာက **Login method က "Session token" ပြန်ပေးပြီး၊ နောက် method တွေခေါ်တိုင်း token ပါတွဲပို့ရတဲ့ pattern** ပါ:

### Step 1 — Login Interface

```java
// AuthInterface.java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface AuthInterface extends Remote {
    // Login အောင်မြင်ရင် session token ပြန်ပေးမယ်, မအောင်မြင်ရင် exception throw
    String login(String username, String password) throws RemoteException, SecurityException;
}
```

### Step 2 — Main Service Interface ကို Token လက်ခံအောင် ပြင်

```java
// InventoryInterface.java
public interface InventoryInterface extends Remote {
    // Method တိုင်းမှာ sessionToken ကို parameter အနေနဲ့ ထည့်ရမယ်
    boolean reduceStock(String sessionToken, String product, int quantity) 
        throws RemoteException, SecurityException;
    int getStock(String sessionToken, String product) throws RemoteException, SecurityException;
}
```

### Step 3 — Server Implementation (Token စစ်ဆေးခြင်း)

```java
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

public class AuthImpl extends UnicastRemoteObject implements AuthInterface {

    // token → username (login state သိမ်းထားရန်)
    private final Map<String, String> activeSessions = new ConcurrentHashMap<>();

    // (Production မှာ password ကို database + bcrypt/hash နဲ့ စစ်ရပါမယ်, ဒါက ဥပမာအတွက်ပါ)
    private final Map<String, String> validUsers = Map.of(
        "admin", "admin123",
        "branch1", "branch1pass"
    );

    protected AuthImpl() throws RemoteException { super(); }

    @Override
    public String login(String username, String password) throws RemoteException {
        String correctPassword = validUsers.get(username);
        if (correctPassword == null || !correctPassword.equals(password)) {
            throw new SecurityException("Username (သို့) Password မှားနေပါသည်");
        }
        String token = UUID.randomUUID().toString();
        activeSessions.put(token, username);
        System.out.println(username + " login ဝင်ပါပြီ (token: " + token + ")");
        return token;
    }

    // ဒီ method ကို InventoryImpl ကနေ token စစ်ဖို့ ခေါ်သုံးမယ်
    public boolean isValidSession(String token) {
        return activeSessions.containsKey(token);
    }
}
```

```java
public class InventoryImpl extends UnicastRemoteObject implements InventoryInterface {

    private final AuthImpl authService; // Token စစ်ဖို့ reference လိုတယ်

    protected InventoryImpl(AuthImpl authService) throws RemoteException {
        super();
        this.authService = authService;
    }

    @Override
    public boolean reduceStock(String sessionToken, String product, int quantity) 
            throws RemoteException {
        // ⚠️ Method တိုင်းရဲ့ အစမှာ token စစ်ရမယ်
        if (!authService.isValidSession(sessionToken)) {
            throw new SecurityException("Login ဝင်ထားခြင်း မရှိပါ / Session သက်တမ်းကုန်နေပါသည်");
        }
        // ... Part 4 က synchronized logic
        synchronized (this) {
            // stock reduce logic
        }
        return true;
    }
}
```

### Step 4 — Client ဘက်က Login ဝင်ပြီးမှ Method ခေါ်ခြင်း

```java
Registry registry = LocateRegistry.getRegistry(serverIP, 1099);

AuthInterface auth = (AuthInterface) registry.lookup("AuthService");
String token;
try {
    token = auth.login("branch1", "branch1pass");
} catch (SecurityException ex) {
    System.out.println("Login မအောင်မြင်ပါ: " + ex.getMessage());
    return;
}

InventoryInterface inventory = (InventoryInterface) registry.lookup("InventoryService");
// Method ခေါ်တိုင်း token ပါတွဲပို့ရမယ်
boolean success = inventory.reduceStock(token, "Laptop", 5);
```

## Swing GUI မှာ ဘယ်လို ချိတ်ရမလဲ (Concept)

- App ဖွင့်ချင်းမှာ **Login dialog** (JDialog) ပြပြီး username/password ယူ
- `auth.login()` ကို SwingWorker နဲ့ background ခေါ်
- Success ဆိုရင် token ကို field variable မှာ သိမ်း၊ Main Swing window ဖွင့်
- Fail ဆိုရင် error message ပြ (login dialog ပြန်ပြ)
- နောက်ပိုင်း method ခေါ်တိုင်း သိမ်းထားတဲ့ token ကို parameter အဖြစ် ပါ့တွဲပို့

## Summary — Security Checklist

|အချက်|အကြောင်းအရာ|
|---|---|
|`SslRMIClientSocketFactory` / `SslRMIServerSocketFactory`|Network traffic ကို encrypt|
|Keystore (Server), Truststore (Client)|Certificate-based trust ချိတ်ဆက်ဖို့|
|Login method + Session token|RMI မှာ built-in auth မရှိလို့ manual ရေးရမယ်|
|Method တိုင်းမှာ token စစ်ခြင်း|Login မဝင်ဘဲ method ခေါ်လို့ မရအောင်|
|Password ကို hash (bcrypt) + DB သုံး|Production မှာ plain text password မသိမ်းရ|
|Session expiry (timeout) ထည့်ခြင်း|Token ကို infinite သက်တမ်းမပေးသင့် — logout/timeout logic ထည့်ရမယ်|

---

RMI series (Part 1-5) — Basic setup → Swing integration → Callback → Thread safety → Security — ပြီးပါပြီ။ ဆက်လက်လေ့လာချင်တာ ရှိပါသလား — **Session timeout logic အသေးစိတ်** လား၊ ဒါမှမဟုတ် Part 1 က ရပ်ထားတဲ့ **gRPC learning** ကို ပြန်ဆက်မလား?