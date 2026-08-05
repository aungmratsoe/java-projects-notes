JProgressBar ကို အသေးစိတ် ရှင်းပြပေးပါမယ်။

## JProgressBar ဆိုတာ ဘာလဲ?

Task တစ်ခု ဘယ်လောက် ပြီးစီးပြီလဲဆိုတာကို visual bar နဲ့ ပြပေးတဲ့ **display-only** component ပါ (user input လက်မခံပါဘူး)။ Mode **၂ မျိုး** ရှိပါတယ်:

1. **Determinate** — % တိုင်းတိုင်း တိကျစွာ သိတဲ့အခါ (file 100 items ရှိတယ်ဆိုရင် 1 item ပြီးတိုင်း 1% တိုးတာမျိုး)
2. **Indeterminate** — ဘယ်လောက် ကြာမလဲ မသိတဲ့အခါ (bar ကို animation loop ပြန်ပြန်ပြပေးမယ်)

---

## 1. Basic Setup

```java
JProgressBar progressBar = new JProgressBar(0, 100); // min=0, max=100
progressBar.setValue(0);
progressBar.setStringPainted(true); // "45%" ဆိုပြီး text ပြချင်ရင်
panel.add(progressBar);
```

## 2. Determinate Mode (% သိတဲ့အခါ)

```java
progressBar.setValue(50); // 50% ဖြစ်စေချင်ရင်
```

## 3. Indeterminate Mode (ဘယ်လောက်ကြာမလဲ မသိတဲ့အခါ)

```java
progressBar.setIndeterminate(true); // bar က loop ပြန်ပြန် animate ဖြစ်နေမယ်
progressBar.setStringPainted(false); // indeterminate မှာ % မပြသင့်ဘူး
```

---

## ⚠️ အရေးကြီးဆုံးအချက် — SwingWorker အသုံးပြုရမယ်

Task (file copy, database query, QR code generation) ကို **main thread (Event Dispatch Thread/EDT) ပေါ်မှာ တိုက်ရိုက် run လုပ်လို့ မရပါဘူး** — UI freeze ဖြစ်သွားမှာပါ။ ဒါကြောင့် **`SwingWorker`** ကို background thread အနေနဲ့ သုံးရပါတယ်။

### Full Example — File processing progress (SwingWorker)

```java
JProgressBar progressBar = new JProgressBar(0, 100);
progressBar.setStringPainted(true);
JButton startBtn = new JButton("Start");

startBtn.addActionListener(e -> {
    SwingWorker<Void, Integer> worker = new SwingWorker<>() {
        @Override
        protected Void doInBackground() throws Exception {
            // Background thread ပေါ်မှာ run ဖြစ်မယ့် heavy task
            for (int i = 0; i <= 100; i += 10) {
                Thread.sleep(200); // simulate work (actual task ကို ဒီနေရာမှာ ထည့်ရမယ်)
                publish(i); // progress update ကို EDT ကို ပို့ပေးတယ်
            }
            return null;
        }

        @Override
        protected void process(java.util.List<Integer> chunks) {
            // EDT ပေါ်မှာ run ဖြစ်တယ် - UI update ဒီနေရာမှာသာ လုပ်ရမယ်
            int latest = chunks.get(chunks.size() - 1);
            progressBar.setValue(latest);
        }

        @Override
        protected void done() {
            // Task ပြီးတဲ့အခါ EDT ပေါ်မှာ run ဖြစ်တယ်
            JOptionPane.showMessageDialog(null, "Completed!");
            startBtn.setEnabled(true);
        }
    };

    startBtn.setEnabled(false);
    worker.execute(); // background thread စတင် run
});
```

### SwingWorker Method များ ရှင်းပြချက်

|Method|Run ဖြစ်တဲ့နေရာ|ရည်ရွယ်ချက်|
|---|---|---|
|`doInBackground()`|Background thread|Heavy task (loop, file I/O, DB query) run ဖို့|
|`publish(chunks)`|Background thread ကနေ ခေါ်တယ်|Progress data ကို EDT ဆီ ပို့ဖို့|
|`process(chunks)`|EDT|`publish()` ကနေ လက်ခံပြီး UI (progressBar) update လုပ်ဖို့|
|`done()`|EDT|Task ပြီးတဲ့အခါ run (success/error handle)|

---

## QRCode Project Example (Bulk QR generation အတွက်)

Student list များစွာအတွက် QR code တစ်ပြိုင်နက် generate လုပ်မယ်ဆိုရင် ဒီလို သုံးလို့ရပါတယ်:

```java
JProgressBar progressBar = new JProgressBar(0, students.size());
progressBar.setStringPainted(true);

SwingWorker<Void, Integer> worker = new SwingWorker<>() {
    @Override
    protected Void doInBackground() {
        for (int i = 0; i < students.size(); i++) {
            Student s = students.get(i);
            generateQRCode(s.getStudentId()); // your existing QR logic
            publish(i + 1);
        }
        return null;
    }

    @Override
    protected void process(java.util.List<Integer> chunks) {
        progressBar.setValue(chunks.get(chunks.size() - 1));
    }

    @Override
    protected void done() {
        JOptionPane.showMessageDialog(null, "All QR codes generated!");
    }
};
worker.execute();
```

---

## Indeterminate Example (Duration မသိတဲ့ task — ဥပမာ Database connect)

```java
progressBar.setIndeterminate(true);

SwingWorker<Void, Void> worker = new SwingWorker<>() {
    @Override
    protected Void doInBackground() throws Exception {
        connectToDatabase(); // ဘယ်လောက်ကြာမလဲ မသိ
        return null;
    }

    @Override
    protected void done() {
        progressBar.setIndeterminate(false);
        progressBar.setValue(100);
    }
};
worker.execute();
```

---

**Key point**:

- `Thread.sleep()` / heavy loop ကို **`doInBackground()`** ထဲမှာသာ ထားရမယ်
- **`progressBar.setValue()`** ကို EDT ပေါ်မှာသာ (`process()` ဒါမှမဟုတ် `done()` ထဲမှာသာ) ခေါ်ရမယ် — direct `doInBackground()` ထဲကနေ UI update လုပ်ရင် bug ဖြစ်တတ်ပါတယ်

QRCode project ရဲ့ bulk generate feature အတွက် တိုက်ရိုက် integrate လုပ်ပေးရမလား၊ ဒါမှမဟုတ် `Timer`-based simple version လိုချင်တာလား?

---

JProgressBar ကို **ဘယ်အချိန်/ဘယ်အခြေအနေမှာ သုံးသင့်လဲ** ဆိုတာ ရှင်းပြပေးပါမယ်။

## အခြေခံစည်းမျဉ်း

Task တစ်ခု run ဖို့ **အချိန် အတိုင်းအတာတစ်ခု ကြာမယ်ဆိုရင်** (user ကို ခဏစောင့်ခိုင်းရမယ်ဆိုရင်) progress bar သုံးသင့်ပါတယ်။ ပုံမှန်အားဖြင့် **1 စက္ကန့်ထက် ပိုကြာမယ့် task** တွေအတွက် အသုံးပြုလေ့ရှိပါတယ် (၁ စက္ကန့်ထက် နည်းရင် user မသိလိုက်ဘူး၊ progress bar ပြရင် တောင် flash ဖြတ်သွားတတ်တယ်)။

## ဘယ်အခါမျိုးတွေမှာ သုံးသင့်လဲ

**1. File Operations**

- File upload/download (internet download, cloud sync)
- File copy/move (folder large ဆိုရင်)
- **ဥပမာ**: QRCode project မှာ student list ကို Excel export/import လုပ်တဲ့အခါ

**2. Bulk/Batch Processing**

- Loop ထဲမှာ item အများကြီးကို process လုပ်တဲ့အခါ (student 500 ယောက်ရဲ့ QR code တစ်ပြိုင်နက် generate လုပ်တာမျိုး)
- Database bulk insert/update

**3. Database Operations**

- Large query run တာ (record အများကြီး fetch လုပ်တာ)
- Database backup/restore
- Database connection (initial connect - ဒီအချိန်မှာ Indeterminate mode သုံးလေ့ရှိတယ်)

**4. Network Operations**

- API call (server ကနေ response စောင့်ရင်း)
- RMI method call (server response ကြာနိုင်တဲ့အခါ) — RMI project ရေးနေတာမို့ ဒါကလည်း relevant ဖြစ်နိုင်တယ်

**5. Report/Document Generation**

- PDF report generate လုပ်တာ
- Multiple page document render လုပ်တာ

**6. Installation/Setup Process**

- Application first-time setup, data migration

---

## Determinate vs Indeterminate — ဘယ်အချိန် ဘယ်ဟာသုံးမလဲ

|အခြေအနေ|Mode|ဘာကြောင့်|
|---|---|---|
|Total item count သိတယ် (loop count သိတယ်)|**Determinate** (% ပြနိုင်တယ်)|student.size() လို total ကို advance သိလို့|
|Total ဘယ်လောက်ရှိမလဲ မသိဘူး (network response time, DB connect time)|**Indeterminate** (loop animation)|% တွက်လို့ မရလို့|

**ဥပမာ**:

```java
// Determinate - QR code 50 ခု generate မယ်ဆိုတာ သိတယ်
progressBar.setMaximum(50);
progressBar.setValue(currentIndex);

// Indeterminate - Server ကို connect လုပ်နေတာ ဘယ်လောက်ကြာမလဲ မသိဘူး
progressBar.setIndeterminate(true);
```

---

## ဘယ်အခါမှာ **သုံးဖို့ မလိုဘူး**

- Task **အလွန်မြန်** (millisecond level) ဖြစ်ရင် — user ခေတ္တခဏ notice တောင် မလုပ်ဘူး
- Simple button click action (Save single record, validation check) — ဒါတွေက instant ဖြစ်တတ်တယ်
- ⚠️ Task ကို background thread (SwingWorker) မသုံးဘဲ run ရင် progress bar ကို ပြသလည်း UI freeze ဖြစ်နေမှာမို့ **အဓိပ္ပာယ်မရှိပါဘူး** — Progress bar ကို SwingWorker (ဒါမှမဟုတ် separate thread) နဲ့ အမြဲတွဲသုံးရမယ်

---

## QRCode Project အတွက် Concrete အကြံပြုချက်

|Feature|Progress bar လိုမလို|
|---|---|
|Single student ID ရိုက်ပြီး QR code 1 ခု generate|❌ မလိုဘူး (instant)|
|Student list အားလုံး (100+) ရဲ့ QR code တစ်ပြိုင်နက် bulk generate|✅ လိုတယ် (Determinate)|
|Excel import/export|✅ လိုတယ် (Determinate)|
|RMI server connect|✅ Indeterminate သင့်တော်တယ်|
|Database backup|✅ လိုတယ် (Determinate or Indeterminate, data size မသိရင်)|

Bulk QR generation feature ကို progress bar နဲ့ implement ချင်ရင် ဒါမှမဟုတ် RMI connection အတွက် indeterminate progress ထည့်ချင်ရင် ပြောပါ၊ code example ပြပေးပါမယ်။

---

RMI server connect အတွက် Progress Bar ကို ဘယ်လို integrate လုပ်ရမလဲ ရှင်းပြပေးပါမယ်။ RMI connection က network operation ဖြစ်လို့ ဘယ်လောက်ကြာမလဲ အတိအကျ မသိတာမို့ **Indeterminate mode** သင့်တော်ပါတယ်။

## အခြေခံ Concept

RMI `Naming.lookup()` ဒါမှမဟုတ် `Registry.lookup()` ခေါ်တဲ့အခါ:

- Network delay ရှိနိုင်တယ် (server slow ဖြစ်နိုင်, connection refuse ဖြစ်နိုင်)
- **Blocking call** ဖြစ်လို့ EDT (main UI thread) ပေါ်မှာ တိုက်ရိုက် ခေါ်ရင် UI freeze ဖြစ်မယ်
- ဒါကြောင့် **SwingWorker** ထဲမှာသာ ခေါ်ရမယ်

---

## Full Example — RMI Client Connect with Progress Bar

```java
import java.rmi.Naming;
import java.rmi.RemoteException;
import javax.swing.*;

public class ConnectPanel extends JPanel {
    private JProgressBar progressBar;
    private JButton connectBtn;
    private JLabel statusLabel;
    private StudentService studentService; // your RMI remote interface

    public ConnectPanel() {
        progressBar = new JProgressBar();
        progressBar.setIndeterminate(false); // စတင်တုန်းက ဖျောက်ထားမယ်
        progressBar.setVisible(false);

        statusLabel = new JLabel("Not connected");
        connectBtn = new JButton("Connect to Server");

        connectBtn.addActionListener(e -> connectToServer());

        add(connectBtn);
        add(progressBar);
        add(statusLabel);
    }

    private void connectToServer() {
        connectBtn.setEnabled(false);
        statusLabel.setText("Connecting...");
        progressBar.setVisible(true);
        progressBar.setIndeterminate(true); // ဘယ်လောက်ကြာမလဲ မသိလို့ indeterminate

        SwingWorker<StudentService, Void> worker = new SwingWorker<>() {
            @Override
            protected StudentService doInBackground() throws Exception {
                // Background thread ပေါ်မှာ run - blocking call ဒီနေရာမှာ ထားရမယ်
                String url = "rmi://192.168.1.10:1099/StudentService";
                return (StudentService) Naming.lookup(url);
            }

            @Override
            protected void done() {
                // EDT ပေါ်မှာ run - UI update ဒီနေရာမှာသာ
                progressBar.setIndeterminate(false);
                progressBar.setVisible(false);
                connectBtn.setEnabled(true);

                try {
                    studentService = get(); // doInBackground ရဲ့ return value/exception ကို ဒီနေရာမှာ ရမယ်
                    statusLabel.setText("Connected!");
                    JOptionPane.showMessageDialog(ConnectPanel.this,
                        "Connected to server successfully!",
                        "Success", JOptionPane.INFORMATION_MESSAGE);
                } catch (Exception ex) {
                    statusLabel.setText("Connection failed");
                    JOptionPane.showMessageDialog(ConnectPanel.this,
                        "Failed to connect: " + ex.getCause().getMessage(),
                        "Connection Error", JOptionPane.ERROR_MESSAGE);
                }
            }
        };

        worker.execute();
    }
}
```

---

## Key Point များ ရှင်းပြချက်

### 1. `get()` method — အရေးကြီးဆုံးအချက်

`doInBackground()` ထဲက return value (ဒါမှမဟုတ် exception) ကို `done()` ထဲမှာ **`get()`** နဲ့ ရယူရတယ်:

```java
try {
    studentService = get(); // success ရရင် object ရမယ်
} catch (Exception ex) {
    // doInBackground() ထဲမှာ throw ဖြစ်ခဲ့တဲ့ exception ဒီနေရာမှာ catch ဖြစ်မယ်
    // ex.getCause() ကနေ actual exception (ConnectException, NotBoundException) ရနိုင်တယ်
}
```

### 2. Timeout Handling (Connection ကြာလွန်းရင်)

RMI connection က default timeout ရှိပေမယ့်၊ ကိုယ်ပိုင် timeout ထည့်ချင်ရင်:

```java
try {
    studentService = get(10, TimeUnit.SECONDS); // 10 seconds ထက် ကြာရင် TimeoutException
} catch (TimeoutException ex) {
    statusLabel.setText("Connection timeout");
    worker.cancel(true);
}
```

### 3. Cancel Button ထည့်ချင်ရင် (User connect ကို ဖျက်ချင်ရင်)

```java
JButton cancelBtn = new JButton("Cancel");
cancelBtn.addActionListener(e -> {
    worker.cancel(true); // background task ကို interrupt
    progressBar.setIndeterminate(false);
    statusLabel.setText("Connection cancelled");
});
```

---

## Reconnect Logic ပါတဲ့ ပိုပြည့်စုံတဲ့ Version

```java
private void connectToServer() {
    connectBtn.setEnabled(false);
    progressBar.setVisible(true);
    progressBar.setIndeterminate(true);
    statusLabel.setText("Connecting to server...");

    SwingWorker<StudentService, String> worker = new SwingWorker<>() {
        @Override
        protected StudentService doInBackground() throws Exception {
            int maxRetries = 3;
            for (int attempt = 1; attempt <= maxRetries; attempt++) {
                try {
                    publish("Attempt " + attempt + " of " + maxRetries);
                    String url = "rmi://192.168.1.10:1099/StudentService";
                    return (StudentService) Naming.lookup(url);
                } catch (Exception ex) {
                    if (attempt == maxRetries) throw ex; // last attempt ကျရင် throw လုပ်
                    Thread.sleep(2000); // 2 seconds စောင့်ပြီး retry
                }
            }
            return null;
        }

        @Override
        protected void process(java.util.List<String> chunks) {
            statusLabel.setText(chunks.get(chunks.size() - 1)); // "Attempt 2 of 3" စသည်ပြမယ်
        }

        @Override
        protected void done() {
            progressBar.setIndeterminate(false);
            progressBar.setVisible(false);
            connectBtn.setEnabled(true);

            try {
                studentService = get();
                statusLabel.setText("Connected!");
            } catch (Exception ex) {
                statusLabel.setText("Connection failed after retries");
                JOptionPane.showMessageDialog(ConnectPanel.this,
                    "Could not connect to server. Please check server status.",
                    "Connection Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };

    worker.execute();
}
```

---

## Application Startup မှာ Auto-Connect ချင်ရင်

Main frame ဖွင့်တဲ့အချိန်မှာ RMI server ကို auto connect ချင်ရင်:

```java
public class MainFrame extends JFrame {
    public MainFrame() {
        initComponents(); // NetBeans GUI builder generated code
        setVisible(true);
        connectToServer(); // window ပေါ်ပြီးတာနဲ့ background connect စတင်
    }
}
```

---

**Summary**: RMI connect က network-dependent operation မို့ **Indeterminate progress bar + SwingWorker** ပေါင်းစပ်သုံးရတယ်။ `doInBackground()` ထဲမှာ `Naming.lookup()` ခေါ်၊ `done()` ထဲမှာ `get()` နဲ့ result/error ကို handle လုပ်ရတယ်။ Retry logic ထည့်ချင်ရင် `publish()`/`process()` နဲ့ status message တွေ update လုပ်နိုင်တယ်။

Server side ကလည်း RMI registry bind လုပ်နေတာ ကြာနိုင်ရင် progress ပြချင်ရင် ပြောပါ၊ ဒါမှမဟုတ် callback-based connection status (auto-reconnect when server drops) implement ချင်ရင်လည်း ပြောပါ။

---
NetBeans GUI Builder (drag-and-drop) မှာ Progress Bar ကို ဘယ်လို ထည့်သုံးရမလဲ ရှင်းပြပေးပါမယ်။

## Step 1: Palette ကနေ Progress Bar Drag ဆွဲထည့်ခြင်း

1. NetBeans **Palette** panel ထဲမှာ **"Swing Controls"** category ကို ဖွင့်ပါ
2. **"Progress Bar"** ကို form design view ပေါ် drag ဆွဲချပါ
3. Auto-generate ဖြစ်လာမယ့် variable name ကို **Properties window** ကနေ ပြောင်းလို့ရတယ် (right-click > Change Variable Name) — ဥပမာ `jProgressBar1` ကနေ `connectProgressBar` လို့ ပြောင်းထားရင် code ဖတ်ရလွယ်တယ်

## Step 2: Properties Window ကနေ Initial Setting လုပ်ခြင်း

Progress bar ကို click ရွေးပြီး **Properties** panel ထဲမှာ:

- `indeterminate` → `false` (initial state, connect စမှ true ပြောင်းမယ်)
- `stringPainted` → `false` (indeterminate mode မှာ % မပြင်ဘူးမို့)
- `visible` → `false` (connect မလုပ်သေးရင် ဖျောက်ထားချင်ရင်)

## Step 3: Button ကို Event Handler တွဲခြင်း

**"Connect to Server"** button ကို double-click နှိပ်လိုက်ရင် NetBeans က `connectBtnActionPerformed()` method ကို **auto-generate** လုပ်ပေးပြီး Source view ဆီ ရောက်သွားမယ်။ ဒီ method ထဲမှာ code ရေးရုံပါပဲ:

```java
private void connectBtnActionPerformed(java.awt.event.ActionEvent evt) {
    connectBtn.setEnabled(false);
    connectProgressBar.setVisible(true);
    connectProgressBar.setIndeterminate(true);
    statusLabel.setText("Connecting to server...");

    SwingWorker<StudentService, Void> worker = new SwingWorker<>() {
        @Override
        protected StudentService doInBackground() throws Exception {
            String url = "rmi://192.168.1.10:1099/StudentService";
            return (StudentService) Naming.lookup(url);
        }

        @Override
        protected void done() {
            connectProgressBar.setIndeterminate(false);
            connectProgressBar.setVisible(false);
            connectBtn.setEnabled(true);

            try {
                studentService = get();
                statusLabel.setText("Connected!");
                JOptionPane.showMessageDialog(MainFrame.this,
                    "Connected to server successfully!",
                    "Success", JOptionPane.INFORMATION_MESSAGE);
            } catch (Exception ex) {
                statusLabel.setText("Connection failed");
                JOptionPane.showMessageDialog(MainFrame.this,
                    "Failed to connect: " + ex.getCause().getMessage(),
                    "Connection Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };

    worker.execute();
}
```

⚠️ **မှတ်ချက်**: `studentService` field ကို class level မှာ declare ထားရမယ် (GUI Builder auto-generate variable တွေရဲ့ အောက်၊ `initComponents()` call ရဲ့ ပြင်ပမှာ):

```java
public class MainFrame extends JFrame {
    private StudentService studentService; // <-- ဒီနေရာမှာ manual ထည့်ရမယ်

    public MainFrame() {
        initComponents(); // GUI Builder auto-generated - ဒီထဲမှာ manual edit မလုပ်ရဘူး
    }
    
    // ... auto-generated event handler methods ...
}
```

## Step 4: NetBeans Editor Layout သတိပြုရမယ့်အချက်

NetBeans GUI Builder က code ကို **2 zone** ခွဲပေးထားတယ်:

```
[Design View]  ←→  [Source View]

initComponents() {
    // 🔒 LOCKED zone - GUI Builder auto manage
    // "Design" tab ကနေ drag/property ပြောင်းမှသာ ဒီနေရာ update ဖြစ်တယ်
    // manual ဒီနေရာမှာ code ရေးရင် GUI Builder ပြန် regenerate တဲ့အခါ ပျက်သွားနိုင်တယ်
}

connectBtnActionPerformed(evt) {
    // ✅ FREE zone - manual code ရေးလို့ရတဲ့ area
    // SwingWorker, business logic တွေ ဒီနေရာမှာ ထည့်ရမယ်
}
```

**Golden Rule**: `initComponents()` method ရဲ့ **အပြင်ဘက်** (event handler method တွေ၊ custom method တွေ၊ field declaration တွေ) မှာသာ manual code ရေးရမယ်။

## Step 5: Import Statement ထည့်ရန်

`Naming`, `SwingWorker` classes သုံးမယ်ဆိုရင် file ရဲ့ အပေါ်ဆုံးမှာ import ထည့်ဖို့လိုမယ် (NetBeans က `Alt+Shift+F` ခေါင်း auto-import fix လုပ်ပေးနိုင်တယ်):

```java
import java.rmi.Naming;
import javax.swing.SwingWorker;
import javax.swing.JOptionPane;
```

---

## Quick Checklist (Drag-and-drop workflow)

|Step|Where|
|---|---|
|1. Progress Bar drag ချ|Design view (Palette → drop)|
|2. `indeterminate`, `stringPainted` initial value|Properties window|
|3. Button double-click → event method auto-gen|Design view → Source view|
|4. SwingWorker code ရေး|Source view, event handler method ထဲ|
|5. Field declare (`studentService`)|Source view, `initComponents()` အပြင်ဘက်|

---

Design view မှာ progress bar layout position (button ဘေးမှာ ထားချင်လား၊ status bar အနေနဲ့ ထားချင်လား) အတွက် GroupLayout constraint issue တွေ့ရင် ပြောပါ၊ ဒါမှမဟုတ် form တစ်ခုလုံးရဲ့ screenshot/structure ပို့ပြီး ဘယ်နေရာမှာ ထည့်ရမလဲ အတိအကျ ညွှန်ပေးပါရစေ။