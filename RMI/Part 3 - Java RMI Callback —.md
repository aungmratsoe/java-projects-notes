# Java RMI Callback — Part 3

ပုံမှန် RMI မှာ **Client က Server ကို ခေါ်** တာပဲ ဖြစ်ပါတယ် (one-way). ဒါပေမယ့် **Server ကနေ Client ကို push notification ပို့ချင်တဲ့အခါ** (ဥပမာ - Stock update ဖြစ်တာ, Chat message ရောက်လာတာ, Order status ပြောင်းတာ) — **Client ကိုယ်တိုင်လည်း RMI server တစ်ခု ဖြစ်ရပါမယ်**။ ဒါကို **Callback pattern** လို့ခေါ်ပါတယ်။

## သဘောတရား

```
Server                                    Client
──────                                    ──────
1. Client register() ခေါ်တဲ့အခါ
   Client ရဲ့ callback object ကို
   list ထဲ သိမ်းထားမယ်
                                     ◄──   register(myCallback)
                                           (Client က ကိုယ့် object ကို
                                            server ဆီ "ဒီကနေခေါ်ပါ" 
                                            ဆိုပြီး ပေးလိုက်တာ)

2. Event တစ်ခုခု ဖြစ်လာရင်
   (ဥပမာ - broadcastMessage())
   register ထားတဲ့ client list ကို
   loop ပြီး callback.notify(...) ခေါ်  ──►  Client ရဲ့ update() method
                                            အလိုအလျောက် run ပြီး
                                            UI ကို update
```

**အဓိက idea**: Client ဟာ Server ကို method ခေါ်ရင်း "ငါ့ကို callback interface object ပါ ထည့်ပေးလိုက်မယ်" ဆိုပြီး ပေးလိုက်တာပါ။ Server က အဲ့ object ကို list ထဲ သိမ်းထားပြီး၊ event ဖြစ်တဲ့အခါ list ထဲက object တွေ အားလုံးကို method ခေါ်ပေးလိုက်တာပါ။

## Step 1 — Callback Interface (Client ဘက်က implement လုပ်မယ့် interface)

```java
// ClientCallback.java — Server နဲ့ Client နှစ်ဖက်စလုံးမှာ ရှိရမယ်
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface ClientCallback extends Remote {
    void notify(String message) throws RemoteException;
}
```

## Step 2 — Main Server Interface ကို ပြင်မယ် (register/unregister ထည့်)

```java
// CalculatorInterface.java (Part 1 ကို ပြင်ဆင်ထားတာ)
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface CalculatorInterface extends Remote {
    int add(int a, int b) throws RemoteException;
    String getGreeting(String name) throws RemoteException;

    // Callback အသစ်ထည့်တာ
    void registerClient(ClientCallback client) throws RemoteException;
    void unregisterClient(ClientCallback client) throws RemoteException;
    void broadcastMessage(String message) throws RemoteException;
}
```

## Step 3 — Server Implementation (Callback list သိမ်းပြီး ခေါ်မယ်)

```java
// CalculatorImpl.java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class CalculatorImpl extends UnicastRemoteObject implements CalculatorInterface {

    // Register ထားတဲ့ client callback တွေ သိမ်းမယ့် list
    // (Thread-safe list သုံးရမယ် — client များစွာက တစ်ပြိုင်နက် register/broadcast လို့)
    private final List<ClientCallback> clients = new CopyOnWriteArrayList<>();

    protected CalculatorImpl() throws RemoteException {
        super();
    }

    @Override
    public int add(int a, int b) throws RemoteException {
        return a + b;
    }

    @Override
    public String getGreeting(String name) throws RemoteException {
        return "မင်္ဂလာပါ " + name;
    }

    @Override
    public void registerClient(ClientCallback client) throws RemoteException {
        clients.add(client);
        System.out.println("Client အသစ် register ဖြစ်သွားပါပြီ (စုစုပေါင်း: " + clients.size() + ")");
    }

    @Override
    public void unregisterClient(ClientCallback client) throws RemoteException {
        clients.remove(client);
        System.out.println("Client တစ်ခု ထွက်သွားပါပြီ (စုစုပေါင်း: " + clients.size() + ")");
    }

    @Override
    public void broadcastMessage(String message) throws RemoteException {
        System.out.println("Broadcasting: " + message + " (client " + clients.size() + " ခုဆီ ပို့မယ်)");

        // Register ထားသမျှ client တွေအားလုံးကို ခေါ်မယ်
        for (ClientCallback client : clients) {
            try {
                client.notify(message);
            } catch (RemoteException ex) {
                // Client တစ်ခု disconnect ဖြစ်နေရင် list ထဲက ဖြုတ်ချမယ်
                System.out.println("Client တစ်ခု connect ပြတ်နေလို့ ဖြုတ်လိုက်ပါတယ်");
                clients.remove(client);
            }
        }
    }
}
```

Server.java ကတော့ Part 1 အတိုင်းပါပဲ (မပြောင်းလဲပါ)။

## Step 4 — Client Callback Implementation

Client ကလည်း **RMI object တစ်ခု ဖြစ်ရပါမယ်** (Server ကနေ ပြန်ခေါ်လို့ရအောင်):

```java
// ClientCallbackImpl.java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import javax.swing.SwingUtilities;

public class ClientCallbackImpl extends UnicastRemoteObject implements ClientCallback {

    private final RmiClientGUI gui; // UI ကို update လုပ်ဖို့ reference

    protected ClientCallbackImpl(RmiClientGUI gui) throws RemoteException {
        super();
        this.gui = gui;
    }

    @Override
    public void notify(String message) throws RemoteException {
        // ⚠️ ဒီ method ကို Server ဘက်က thread က ခေါ်တာ — 
        // Swing UI ကို တိုက်ရိုက် update မလုပ်ရ (EDT မဟုတ်လို့)
        // SwingUtilities.invokeLater() နဲ့ EDT ပေါ်ကို ပြန်ပို့ရမယ်
        SwingUtilities.invokeLater(() -> {
            gui.showNotification(message);
        });
    }
}
```

## Step 5 — Swing Client GUI ကို ချိတ်မယ် (Part 2 ကို ဆက်ထည့်တာ)

```java
// RmiClientGUI.java ထဲမှာ ထပ်ထည့်ရမယ့် အစိတ်အပိုင်းများ

private ClientCallback myCallback;
private JTextArea notificationArea; // Broadcast message တွေ ပြမယ့် area

// Connect လုပ်ပြီးတဲ့အခါ callback ကိုပါ register လုပ်မယ်
private void onConnect(ActionEvent e) {
    String ip = serverIpField.getText().trim();
    connectButton.setEnabled(false);

    SwingWorker<CalculatorInterface, Void> worker = new SwingWorker<>() {
        @Override
        protected CalculatorInterface doInBackground() throws Exception {
            Registry registry = LocateRegistry.getRegistry(ip, 1099);
            CalculatorInterface calc = (CalculatorInterface) registry.lookup("CalculatorService");

            // Client ကိုယ်တိုင် RMI object ဖြစ်အောင် ဖန်တီးပြီး register လုပ်မယ်
            myCallback = new ClientCallbackImpl(RmiClientGUI.this);
            calc.registerClient(myCallback);

            return calc;
        }

        @Override
        protected void done() {
            try {
                calculator = get();
                resultLabel.setText("Server ချိတ်ဆက်ပြီး Callback register လည်း ပြီးပါပြီ ✓");
                addButton.setEnabled(true);
                greetButton.setEnabled(true);
            } catch (Exception ex) {
                resultLabel.setText("Connect မအောင်မြင်ပါ: " + ex.getCause());
            } finally {
                connectButton.setEnabled(true);
            }
        }
    };
    worker.execute();
}

// Server ကနေ callback.notify() ခေါ်လာတဲ့အခါ ဒီ method run မယ်
public void showNotification(String message) {
    notificationArea.append("[Server Notification] " + message + "\n");
}

// Window ပိတ်တဲ့အခါ unregister လုပ်ရမယ် (memory leak ကာကွယ်ဖို့)
@Override
public void dispose() {
    try {
        if (calculator != null && myCallback != null) {
            calculator.unregisterClient(myCallback);
        }
    } catch (RemoteException ex) {
        // Server ပြီးသွားပြီဆိုရင် ignore လုပ်လိုက်ပါ
    }
    super.dispose();
}
```

## Test လုပ်ကြည့်နည်း

1. Server, Client (တစ်ခု သို့ အများကြီး) run ထားပြီး Connect လုပ်ထား
2. Server console ဘက်မှာ manual command line တစ်ခု ထပ်ထည့်ပြီး broadcast ခေါ်ကြည့်နိုင်ပါတယ်:

```java
// Server.java ထဲ main() ရဲ့ အောက်ဆုံးမှာ ထပ်ထည့်
Scanner sc = new Scanner(System.in);
while (true) {
    System.out.print("Broadcast message ရိုက်ထည့်ပါ: ");
    String msg = sc.nextLine();
    calculator.broadcastMessage(msg);
}
```

3. Server console မှာ message ရိုက်ထည့်လိုက်ရင် **Client GUI ရဲ့ notification area ထဲ instant ရောက်လာမှာပါ** — ဒါက real push notification ပါပဲ

## အရေးကြီးတဲ့ အချက်များ

|အချက်|ရှင်းလင်းချက်|
|---|---|
|Client က RMI server လည်း ဖြစ်ရမယ်|`ClientCallbackImpl extends UnicastRemoteObject`|
|`SwingUtilities.invokeLater()` မဖြစ်မနေ လိုအပ်|Server ဘက်က thread က `notify()` ကို ခေါ်တာဖြစ်လို့ Swing UI update ကို EDT ပေါ်ပြန်ပို့ရမယ်|
|`CopyOnWriteArrayList` သုံးရတဲ့ အကြောင်း|Client များစွာက တစ်ပြိုင်နက် register/unregister လုပ်နိုင်လို့ thread-safe list လိုအပ်|
|Disconnected client ကို ဖယ်ရှားခြင်း|`broadcastMessage()` ထဲက try-catch က RemoteException ဖြစ်ရင် list ထဲကနေ auto-remove|
|Firewall|Client side ကလည်း RMI object ဖြစ်တဲ့အတွက် client computer ပေါ်မှာလည်း dynamic port ဖွင့်ပေးထားရပါမယ်|

---

Multiple clients တစ်ပြိုင်နက်တည်း handle လုပ်တာနဲ့ ပတ်သက်ပြီး (thread safety, synchronized data access) ဆက်လေ့လာချင်ပါသလား၊ ဒါမှမဟုတ် ဒီ RMI series ကို ဒီမှာ ရပ်ထားပြီး **နောက် topic** ဆက်လေ့လာချင်ပါသလား?