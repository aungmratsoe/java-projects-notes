# Java RMI (Remote Method Invocation) — Part 1

RMI ဆိုတာက Java program တစ်ခုက ကွန်ပျူတာတစ်လုံးမှာ run နေတဲ့ object ရဲ့ method ကို ကွန်ပျူတာနောက်တစ်လုံးက (network ကနေတဆင့်) ခေါ်သုံးနိုင်အောင် ဖန်တီးထားတဲ့ Java-specific technology ဖြစ်ပါတယ်။ REST/SOAP လိုမျိုး HTTP+JSON/XML သုံးတာမဟုတ်ဘဲ Java object serialization ကို တိုက်ရိုက်သုံးလို့ Java-to-Java communication အတွက် အရမ်းမြန်ပြီး ရေးရလွယ်ပါတယ်။

## အခြေခံသဘောတရား

RMI မှာ အဓိကအစိတ်အပိုင်း ၄ ခု ပါဝင်ပါတယ်:

1. **Remote Interface** — Server က ဘယ် method တွေကို client ကနေ ခေါ်လို့ရမလဲဆိုတာ သတ်မှတ်ပေးတဲ့ interface (`java.rmi.Remote` ကို extend လုပ်ရမယ်)
2. **Remote Object (Implementation)** — အဲ့ interface ကို implement လုပ်တဲ့ class (`UnicastRemoteObject` ကို extend)
3. **RMI Registry** — Server object တွေကို name နဲ့ register လုပ်ပြီး client တွေ ရှာတွေ့နိုင်အောင် ထားတဲ့ naming service (`rmiregistry` port 1099 default)
4. **Client** — Registry ကနေ remote object ကို lookup လုပ်ပြီး local method ခေါ်သလိုပဲ ခေါ်သုံးနိုင်တဲ့ program

## လုပ်ငန်းစဉ် (Flow)

```
Server computer                          Client computer
─────────────────                        ─────────────────
1. Remote interface ရေး
2. Implementation class ရေး
3. RMI Registry စတင် (rmiregistry)
4. Object ကို registry မှာ bind        
                                    ──►   5. Registry ကို connect (IP:port)
                                          6. lookup() နဲ့ object ရယူ
                                          7. Method ခေါ်သုံး (local method 
                                             လိုပဲ ခေါ်ရုံပါပဲ)
```

## လက်တွေ့ဥပမာ — Two Computer Setup

**ရည်ရွယ်ချက်**: Server computer ပေါ်က `add(a, b)` နဲ့ `getGreeting(name)` method နှစ်ခုကို Client computer ကနေ ခေါ်သုံးမယ်။

### Step 1 — Remote Interface (နှစ်ဖက်စလုံးမှာ ရှိရမယ်)

```java
// CalculatorInterface.java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface CalculatorInterface extends Remote {
    int add(int a, int b) throws RemoteException;
    String getGreeting(String name) throws RemoteException;
}
```

**⚠️ အရေးကြီးချက်**: Interface file ကို Server နဲ့ Client နှစ်ဖက်စလုံးမှာ (package name အတူတူ) ထားရပါမယ်။ Method တိုင်းက `RemoteException` throw လုပ်ရမယ် — network error ဖြစ်နိုင်လို့ပါ။

### Step 2 — Server Implementation

```java
// CalculatorImpl.java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class CalculatorImpl extends UnicastRemoteObject implements CalculatorInterface {

    protected CalculatorImpl() throws RemoteException {
        super();
    }

    @Override
    public int add(int a, int b) throws RemoteException {
        System.out.println("add() ခေါ်ခံရ: " + a + " + " + b);
        return a + b;
    }

    @Override
    public String getGreeting(String name) throws RemoteException {
        return "မင်္ဂလာပါ " + name + "၊ Server ကနေ ပြန်စကားပြောနေပါတယ်";
    }
}
```

### Step 3 — Server Program (Server Computer ပေါ်မှာ Run)

```java
// Server.java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class Server {
    public static void main(String[] args) {
        try {
        
	        //System.setProperty("java.rmi.server.hostname", "172.16.11.196"); // Server ရဲ့ WiFi IP
            // Registry ကို port 1099 မှာ start (code ထဲကနေတိုက်ရိုက် start)
            Registry registry = LocateRegistry.createRegistry(1099);

            CalculatorImpl calculator = new CalculatorImpl();

            // Object ကို "CalculatorService" ဆိုတဲ့ name နဲ့ bind
            registry.rebind("CalculatorService", calculator);

            System.out.println("Server စတင်ပြီးပါပြီ... Client တွေကို စောင့်နေသည်");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Step 4 — Client Program (Client Computer ပေါ်မှာ Run)

```java
// Client.java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class Client {
    public static void main(String[] args) {
        try {
            // Server ရဲ့ IP address ကို ဒီမှာထည့်ပါ (2 computer ဆိုရင် LAN IP)
            String serverIP = "192.168.1.10";  // Server computer ရဲ့ IP

            Registry registry = LocateRegistry.getRegistry(serverIP, 1099);

            CalculatorInterface calculator =
                (CalculatorInterface) registry.lookup("CalculatorService");

            int result = calculator.add(15, 27);
            System.out.println("ရလဒ်: " + result);

            String greeting = calculator.getGreeting("Aung");
            System.out.println(greeting);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Two Computer Setup — လုပ်ဆောင်ရမယ့် အဆင့်ဆင့်

1. **Server Computer**:
    
    - `CalculatorInterface.java`, `CalculatorImpl.java`, `Server.java` ကို compile
    - Firewall မှာ port **1099** ကို allow ပေးထား (Windows Firewall / Linux ufw)
    - `Server.java` ကို run → "Server စတင်ပြီးပါပြီ" ပြပါလိမ့်မယ်
    - Server IP ကို သိထားရမယ် (Command Prompt မှာ `ipconfig`, Linux မှာ `ifconfig` သို့ `ip addr`)
2. **Client Computer**:
    
    - `CalculatorInterface.java` (interface **အတူတူ**) နဲ့ `Client.java` ကို compile
    - `Client.java` ထဲက `serverIP` variable ကို Server ရဲ့ actual IP address နဲ့ ပြောင်း
    - Run လိုက်ရင် Server ပေါ်က method တွေကို ခေါ်သုံးနိုင်ပါပြီ
3. **သတိထားရမယ့်အချက်များ**:
    
    - Server နဲ့ Client နှစ်ခုစလုံး **WiFi/LAN network တူတူ** ပေါ်ရှိရပါမယ် (သို့မဟုတ် port forwarding လုပ်ရမယ်)
    - `CalculatorImpl.java` ကို client side မှာ **မလိုအပ်ပါ** — interface ပဲလိုတယ်
    - Java version Server/Client နှစ်ဖက် တူသင့်ပါတယ်

## အောက်ပါအချက်တွေကို နောက် Part မှာ ဆက်ရေးပေးနိုင်ပါတယ်

- Swing GUI (JFrame/JButton) ကနေ RMI client ကို ခေါ်သုံးနည်း (calculator ကို GUI နဲ့ ချိတ်ခြင်း)
- Server crash/disconnect ဖြစ်ရင် Swing UI မှာ error handling ဘယ်လိုပြမလဲ
- Callback (Server ကနေ Client ကို ပြန်ခေါ်တဲ့) RMI pattern
- Multiple clients တစ်ပြိုင်နက်တည်း connect လုပ်တဲ့အခါ

ဘယ်အပိုင်းကို ဆက်လေ့လာချင်ပါသလဲ — **Swing GUI integration** လား၊ ဒါမှမဟုတ် **error handling / callback** လား?