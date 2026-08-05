# SignIn UI ကို RMI Client ပြောင်းနည်း

သင့် code ထဲက ပြဿနာက **`UserDAO` ကို client ဘက်ကနေ တိုက်ရိုက် `new` လုပ်နေခြင်း** ပါ — ဒါက DB connection ကို client computer ကနေ တိုက်ရိုက်ဆက်နေတာဖြစ်လို့ RMI architecture နဲ့ **လုံးဝ ဆန့်ကျင်** နေပါတယ်။ `UserDAO` အစား **RMI Service stub** ကို သုံးရပါမယ်၊ ပြီးတော့ **SwingWorker** ထဲမှာ ခေါ်ရပါမယ် (login call က network operation ဖြစ်လို့)။

## Step 1 — RMI Connection ကို Central Place မှာ စီမံမယ် (Singleton Pattern)

App တစ်ခုလုံးအတွက် Registry ကို တစ်ခါပဲ connect လုပ်ဖို့ helper class တစ်ခု ဖန်တီးပါ:

```java
// com.ams.qrcode.client.RmiConnectionManager.java
package com.ams.qrcode.client;

import com.ams.qrcode.service.UserService;
import com.ams.qrcode.service.StudentService;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class RmiConnectionManager {

    private static RmiConnectionManager instance;
    private UserService userService;
    private StudentService studentService;

    private RmiConnectionManager() {}

    public static RmiConnectionManager getInstance() {
        if (instance == null) {
            instance = new RmiConnectionManager();
        }
        return instance;
    }

    public void connect(String serverIp) throws Exception {
        Registry registry = LocateRegistry.getRegistry(serverIp, 1099);
        userService = (UserService) registry.lookup("UserService");
        studentService = (StudentService) registry.lookup("StudentService");
    }

    public UserService getUserService() {
        return userService;
    }

    public StudentService getStudentService() {
        return studentService;
    }
}
```

App ဖွင့်ချင်းမှာ (Splash screen (သို့) SignIn form ပွင့်ခင်း) တစ်ကြိမ် `connect()` ကို ခေါ်ထားရပါမယ် — SignIn form ကိုယ်တိုင်ထဲမှာ ခေါ်ထားလည်း ရပါတယ်။

## Step 2 — SignIn Code ကို ပြောင်း (Before → After)

### ❌ Before (သင့်ရဲ့ လက်ရှိ code)

```java
try {
    UserDAO userDAO = new UserDAO();
    User loggedInUser = userDAO.loginUser(email, password);
    if (loggedInUser != null) {
        JOptionPane.showMessageDialog(this, "Login Successful! Welcome, " + loggedInUser.getUsername(), 
            "Success", JOptionPane.INFORMATION_MESSAGE);
        new Home().setVisible(true);
        this.dispose();
    }
    // ...
}
```

### ✅ After (RMI Service + SwingWorker)

```java
private void loginButtonActionPerformed(java.awt.event.ActionEvent evt) {
    String email = emailField.getText().trim();
    String password = new String(passwordField.getPassword());

    // Input validation — network call မလုပ်ခင် client ဘက်မှာပဲ စစ်လိုက်
    if (email.isEmpty() || password.isEmpty()) {
        JOptionPane.showMessageDialog(this, "Email/Password ဖြည့်ပါ", "Warning", JOptionPane.WARNING_MESSAGE);
        return;
    }

    loginButton.setEnabled(false); // Double-click ကာကွယ်
    loginButton.setText("Connecting...");

    SwingWorker<User, Void> worker = new SwingWorker<>() {
        @Override
        protected User doInBackground() throws Exception {
            // ⚠️ UserDAO အစား RMI Service stub ကို ခေါ်
            UserService userService = RmiConnectionManager.getInstance().getUserService();
            return userService.loginUser(email, password);
        }

        @Override
        protected void done() {
            loginButton.setEnabled(true);
            loginButton.setText("Login");

            try {
                User loggedInUser = get(); // doInBackground() ရဲ့ return value

                if (loggedInUser != null) {
                    JOptionPane.showMessageDialog(SignIn.this,
                        "Login Successful! Welcome, " + loggedInUser.getUsername(),
                        "Success", JOptionPane.INFORMATION_MESSAGE);

                    new Home(loggedInUser).setVisible(true); // Home ကို user info ပါ pass
                    SignIn.this.dispose();
                } else {
                    JOptionPane.showMessageDialog(SignIn.this,
                        "Email (သို့) Password မှားနေပါသည်", "Login Failed", JOptionPane.ERROR_MESSAGE);
                }

            } catch (Exception ex) {
                // Server disconnect / DataAccessException / network error အားလုံး ဒီမှာ catch
                Throwable cause = ex.getCause();
                String msg = (cause != null) ? cause.getMessage() : ex.getMessage();
                JOptionPane.showMessageDialog(SignIn.this,
                    "Server ဆက်သွယ်မှု မအောင်မြင်ပါ: " + msg,
                    "Connection Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

## Step 3 — App Start မှာ RMI Connect လုပ်ရမယ့်နေရာ

`main()` (သို့) SignIn form ရဲ့ constructor ထဲမှာ connect လုပ်နိုင်ပါတယ်:

```java
public class SignIn extends javax.swing.JFrame {

    public SignIn() {
        initComponents();
        connectToServer();
    }

    private void connectToServer() {
        SwingWorker<Void, Void> worker = new SwingWorker<>() {
            @Override
            protected Void doInBackground() throws Exception {
                RmiConnectionManager.getInstance().connect("192.168.1.10");
                return null;
            }

            @Override
            protected void done() {
                try {
                    get();
                    // Connect success — ဘာမှ ထပ်လုပ်စရာမလို
                } catch (Exception ex) {
                    JOptionPane.showMessageDialog(SignIn.this,
                        "Server နှင့် ချိတ်ဆက်၍ မရပါ။ Server run ထားခြင်း/Network ကို စစ်ဆေးပါ",
                        "Connection Failed", JOptionPane.ERROR_MESSAGE);
                    loginButton.setEnabled(false); // Server မရှိရင် login ခလုတ် disable
                }
            }
        };
        worker.execute();
    }
    // ...
}
```

## အဓိက ပြောင်းလဲမှု အနှစ်ချုပ်

|Before|After|
|---|---|
|`new UserDAO()`|`RmiConnectionManager.getInstance().getUserService()`|
|`userDAO.loginUser(...)` — Event Dispatch Thread ပေါ်တိုက်ရိုက်ခေါ်|`SwingWorker.doInBackground()` ထဲမှာ ခေါ် (UI freeze မဖြစ်အောင်)|
|Exception handling မရှိ (visible code မှာ)|`RemoteException` (server disconnect) ကို catch လုပ်ပြီး user-friendly message ပြ|
|`import com.ams.qrcode.dao.UserDAO;`|`import com.ams.qrcode.service.UserService;` (DAO import client ဘက်က လုံးဝ ပြယ်)|

## `Home` Constructor ကိုပါ ပြောင်းသင့်ချက်

`new Home()` ထဲမှာ `loggedInUser` ကို pass ထားတာက **login ဝင်ထားတဲ့ user context ကို app တစ်ခုလုံးမှာ သိလိုအပ်** လို့ပါ (ဥပမာ - "Welcome, [name]" header, permission checks)။ `Home` class ရဲ့ constructor ကို `User` parameter လက်ခံအောင် ပြင်ဖို့ လိုပါလိမ့်မယ်:

```java
public class Home extends javax.swing.JFrame {
    private final User currentUser;

    public Home(User currentUser) {
        this.currentUser = currentUser;
        initComponents();
        welcomeLabel.setText("Welcome, " + currentUser.getUsername());
    }
}
```

---

`Home` form ထဲက Student list/table ကို RMI `StudentService` နဲ့ ချိတ်ပုံ (JTable populate + SwingWorker) ကူညီရမလား? ဒါမှမဟုတ် Register/SignUp screen ကို ပြောင်းရင် ကူညီရမလား?