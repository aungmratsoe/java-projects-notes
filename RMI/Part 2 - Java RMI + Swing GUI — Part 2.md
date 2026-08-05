# Java RMI + Swing GUI — Part 2

Part 1 က `CalculatorInterface`/`Server`/`Client` (console) ကို လုပ်ပြီးပါပြီ။ ဒီအပိုင်းမှာ **Client side ကို Swing GUI** နဲ့ ချိတ်ပြီး Server computer ပေါ်က method တွေကို button click နဲ့ ခေါ်သုံးပုံကို လက်တွေ့ဥပမာနဲ့ ပြပါမယ်။ (Server side ကတော့ Part 1 အတိုင်း console program အနေနဲ့ ဆက်ထားလို့ရပါတယ် — Server ကို GUI ထည့်စရာမလိုပါ)

## အရေးကြီးဆုံးအချက် — Network Call ကို UI Thread မှာ တိုက်ရိုက်မခေါ်ရ

RMI method ခေါ်တာက **network operation** ဖြစ်လို့ (server နှေးရင်/disconnect ဖြစ်ရင်) အချိန်ကြာနိုင်ပါတယ်။ Swing UI thread (Event Dispatch Thread) ပေါ်မှာ တိုက်ရိုက်ခေါ်လိုက်ရင် **UI ဂడ်ခဲ (freeze)** သွားပါလိမ့်မယ်။ ဒါကြောင့် `SwingWorker` ကို အသုံးပြုပြီး background thread မှာ ခေါ်ရပါမယ်။

## Client Swing GUI — Full Example

```java
// RmiClientGUI.java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.NotBoundException;

public class RmiClientGUI extends JFrame {

    private JTextField serverIpField;
    private JTextField numberAField;
    private JTextField numberBField;
    private JTextField nameField;
    private JLabel resultLabel;
    private JButton connectButton;
    private JButton addButton;
    private JButton greetButton;

    // Server ချိတ်ပြီးရင် ဒီထဲမှာ object သိမ်းထားမယ်
    private CalculatorInterface calculator;

    public RmiClientGUI() {
        setTitle("RMI Calculator Client");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(420, 320);
        setLocationRelativeTo(null);

        JPanel panel = new JPanel(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(6, 6, 6, 6);
        gbc.fill = GridBagConstraints.HORIZONTAL;

        // --- Server IP + Connect ---
        gbc.gridx = 0; gbc.gridy = 0;
        panel.add(new JLabel("Server IP:"), gbc);

        serverIpField = new JTextField("192.168.1.10", 12);
        gbc.gridx = 1;
        panel.add(serverIpField, gbc);

        connectButton = new JButton("Connect");
        gbc.gridx = 2;
        panel.add(connectButton, gbc);

        // --- Add numbers ---
        gbc.gridx = 0; gbc.gridy = 1;
        panel.add(new JLabel("A:"), gbc);
        numberAField = new JTextField(5);
        gbc.gridx = 1;
        panel.add(numberAField, gbc);

        gbc.gridx = 0; gbc.gridy = 2;
        panel.add(new JLabel("B:"), gbc);
        numberBField = new JTextField(5);
        gbc.gridx = 1;
        panel.add(numberBField, gbc);

        addButton = new JButton("Add (Server ပေါ်တွက်မယ်)");
        gbc.gridx = 2; gbc.gridy = 1; gbc.gridheight = 2;
        panel.add(addButton, gbc);
        gbc.gridheight = 1;

        // --- Greeting ---
        gbc.gridx = 0; gbc.gridy = 3;
        panel.add(new JLabel("Name:"), gbc);
        nameField = new JTextField(10);
        gbc.gridx = 1;
        panel.add(nameField, gbc);

        greetButton = new JButton("Greet");
        gbc.gridx = 2;
        panel.add(greetButton, gbc);

        // --- Result ---
        resultLabel = new JLabel("Server ကို ပထမ Connect လုပ်ပါ");
        gbc.gridx = 0; gbc.gridy = 4; gbc.gridwidth = 3;
        panel.add(resultLabel, gbc);

        add(panel);

        // အစမှာ Add/Greet ကို disable ထားမယ် (connect မလုပ်ရသေးလို့)
        addButton.setEnabled(false);
        greetButton.setEnabled(false);

        connectButton.addActionListener(this::onConnect);
        addButton.addActionListener(this::onAdd);
        greetButton.addActionListener(this::onGreet);
    }

    // ===== Connect Button =====
    private void onConnect(ActionEvent e) {
        String ip = serverIpField.getText().trim();
        connectButton.setEnabled(false);
        resultLabel.setText("Connecting...");

        // SwingWorker: background thread မှာ network work, 
        // UI update ကိုတော့ EDT ပြန်ရောက်မှ လုပ်မယ်
        SwingWorker<CalculatorInterface, Void> worker = new SwingWorker<>() {
            @Override
            protected CalculatorInterface doInBackground() throws Exception {
                Registry registry = LocateRegistry.getRegistry(ip, 1099);
                return (CalculatorInterface) registry.lookup("CalculatorService");
            }

            @Override
            protected void done() {
                try {
                    calculator = get(); // exception ရှိရင် ဒီမှာ throw ဖြစ်မယ်
                    resultLabel.setText("Server ချိတ်ဆက်ပြီးပါပြီ ✓");
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

    // ===== Add Button =====
    private void onAdd(ActionEvent e) {
        int a, b;
        try {
            a = Integer.parseInt(numberAField.getText().trim());
            b = Integer.parseInt(numberBField.getText().trim());
        } catch (NumberFormatException ex) {
            resultLabel.setText("နံပါတ်မှန်မှန် ရိုက်ထည့်ပါ");
            return;
        }

        addButton.setEnabled(false);
        resultLabel.setText("Server ကို ခေါ်နေသည်...");

        SwingWorker<Integer, Void> worker = new SwingWorker<>() {
            @Override
            protected Integer doInBackground() throws Exception {
                return calculator.add(a, b);
            }

            @Override
            protected void done() {
                try {
                    int result = get();
                    resultLabel.setText("ရလဒ်: " + result);
                } catch (Exception ex) {
                    resultLabel.setText("Error: Server နဲ့ ဆက်သွယ်မှု ပြတ်တောက်နေပါသည်");
                } finally {
                    addButton.setEnabled(true);
                }
            }
        };
        worker.execute();
    }

    // ===== Greet Button =====
    private void onGreet(ActionEvent e) {
        String name = nameField.getText().trim();
        greetButton.setEnabled(false);

        SwingWorker<String, Void> worker = new SwingWorker<>() {
            @Override
            protected String doInBackground() throws Exception {
                return calculator.getGreeting(name);
            }

            @Override
            protected void done() {
                try {
                    resultLabel.setText(get());
                } catch (Exception ex) {
                    resultLabel.setText("Error: Server နဲ့ ဆက်သွယ်မှု ပြတ်တောက်နေပါသည်");
                } finally {
                    greetButton.setEnabled(true);
                }
            }
        };
        worker.execute();
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new RmiClientGUI().setVisible(true));
    }
}
```

## ဒီ Code ရဲ့ အဓိက Concept များ

|အပိုင်း|ရှင်းလင်းချက်|
|---|---|
|`SwingWorker<T, Void>`|Background thread မှာ RMI call လုပ်ဖို့ — UI ဂడ်ခဲမနေအောင်|
|`doInBackground()`|Network/RMI logic ဒီထဲမှာပဲ ရေးရမယ် (EDT မဟုတ်ဘူး)|
|`done()`|Result ရလာရင် UI ကို update လုပ်တဲ့နေရာ (ဒါက EDT ပြန်ရောက်တယ်)|
|`get()`|`doInBackground()` ရဲ့ result ကို ယူတာ — Exception ရှိရင် ဒီနေရာမှာ ပြန်ပေါ်တယ် (try-catch လိုအပ်)|
|Button disable/enable|Double-click ကို ကာကွယ်ဖို့ + connect မလုပ်ခင် အသုံးမပြုနိုင်အောင်|

## Test လုပ်ကြည့်နည်း

1. Server computer မှာ Part 1 က `Server.java` ကို run ထား
2. Client computer မှာ `RmiClientGUI.java` ကို run
3. GUI ပွင့်လာရင် Server IP ရိုက်ထည့်ပြီး **Connect** နှိပ်
4. "Server ချိတ်ဆက်ပြီးပါပြီ ✓" ပြရင် A, B ဂဏန်းများထည့်ပြီး **Add** နှိပ်ကြည့်ပါ

## Error Handling — Server ပိတ်ထားရင် (သို့) Network ပြတ်ရင်

`done()` ထဲက `catch (Exception ex)` က `RemoteException` (connection lost), `NotBoundException` (service name မှားနေ), `ConnectException` (server မရှိ/port မဖွင့်ထား) စတာတွေကို catch လုပ်ပေးပါလိမ့်မယ်။ Production app ဆိုရင် exception type အလိုက် message ကို ပိုတိကျအောင် ခွဲခြားပြရပါမယ် — ဥပမာ:

```java
catch (Exception ex) {
    Throwable cause = ex.getCause();
    if (cause instanceof java.rmi.ConnectException) {
        resultLabel.setText("Server ကို ဆက်သွယ်လို့ မရပါ (offline ဖြစ်နေသလား စစ်ပါ)");
    } else if (cause instanceof NotBoundException) {
        resultLabel.setText("Service name မှားနေပါသည်");
    } else {
        resultLabel.setText("Error: " + cause);
    }
}
```

---

နောက်တစ်ပိုင်းအနေနဲ့ ဆက်လေ့လာချင်တာ ရှိပါသလား — **Callback (Server ကနေ Client ကို push notification ပို့တဲ့ pattern)** လား၊ ဒါမှမဟုတ် **Multiple clients တစ်ပြိုင်နက်တည်း connect** ဖြစ်တဲ့အခါ Server ဘက်က ဘယ်လိုကိုင်တွယ်ရမလဲဆိုတာလား?