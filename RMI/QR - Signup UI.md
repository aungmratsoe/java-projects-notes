# Signup Button — RMI ပြောင်းထားသော Version

```java
private void btnSignupActionPerformed(java.awt.event.ActionEvent evt) {
    // 5. User Object တည်ဆောက်ခြင်း — client-side, ပြောင်းစရာမလို
    User user = new User();
    user.setUsername(username);
    user.setEmail(email);
    user.setPhone(fullPhone);
    user.setPassword(password);

    btnSignup.setEnabled(false); // Double-click ကာကွယ်

    SwingWorker<Boolean, Void> worker = new SwingWorker<>() {
        @Override
        protected Boolean doInBackground() throws Exception {
            // 6. UserDAO အစား UserService (RMI) ကို ခေါ်
            UserService userService = RmiConnectionManager.getInstance().getUserService();
            return userService.registerUser(user);
        }

        @Override
        protected void done() {
            btnSignup.setEnabled(true);

            try {
                boolean success = get();

                if (success) {
                    JOptionPane.showMessageDialog(SignUp.this,
                        "Account created successfully! Please sign in.",
                        "Success", JOptionPane.INFORMATION_MESSAGE);

                    new SignIn().setVisible(true);
                    SignUp.this.dispose();
                } else {
                    JOptionPane.showMessageDialog(SignUp.this,
                        "Registration failed. Username or Email may already exist.",
                        "Error", JOptionPane.ERROR_MESSAGE);
                }

            } catch (Exception ex) {
                ex.printStackTrace();
                JOptionPane.showMessageDialog(SignUp.this,
                    "Error: " + ex.getCause(),
                    "Exception Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

## ပြောင်းလဲထားသော အချက်များ

|မူလ Code|ပြောင်းထားသော Code|
|---|---|
|`new UserDAO()` — local instance|`RmiConnectionManager.getInstance().getUserService()` — Server ရဲ့ RMI stub|
|`userDAO.registerUser(user)` — EDT ပေါ် တိုက်ရိုက်ခေါ်|`doInBackground()` ထဲ RMI call|
|`new SignIn().setVisible(true); this.dispose();`|`done()` ထဲမှာ **အတူတူပဲ** ထား — success ဖြစ်ပြီးမှသာ navigate|
|`ex.getMessage()`|`ex.getCause()` — RMI exception ရဲ့ real message|

## ⚠️ Password ပို့ပုံ သတိပေးချက် (ယခင်က ဆွေးနွေးထားသလို)

`user.setPassword(password)` ကို **plain text အတိုင်း** RMI ကနေတဆင့် Server ဆီ ပို့နေပါတယ် — Encryption (SSL) မထားသေးရင် WiFi ပေါ်က traffic sniff ဖြစ်နိုင်ပါတယ်။ Production အတွက်ဆိုရင် Server ဘက်မှာ password ကို **hash** (BCrypt) လုပ်ပြီးမှ DB ထဲ save သင့်ပါတယ်:

```java
// UserServiceImpl.registerUser() ထဲမှာ
@Override
public boolean registerUser(User user) throws RemoteException {
    // Server ဘက်မှာပဲ hash လုပ်ပြီးမှ DAO ဆီ ပို့
    String hashedPassword = BCrypt.hashpw(user.getPassword(), BCrypt.gensalt());
    user.setPassword(hashedPassword);
    return dao.registerUser(user);
}
```

`BCrypt` library (jBCrypt, ဒါမှမဟုတ် Spring Security Crypto) ကို Server module ရဲ့ pom.xml မှာ dependency ထည့်ရပါလိမ့်မယ် — Client module ကတော့ dependency ထပ်ထည့်စရာမလို (Server ဘက်ကပဲ hash လုပ်တာမို့)။

---

`BCrypt` dependency setup (pom.xml) နဲ့ `UserServiceImpl.loginUser()` ကို hash-compare logic ပါ ပြင်ပေးရမလား — password verify တဲ့အခါ plain text နဲ့ တိုက်ရိုက် `.equals()` မလုပ်တော့ဘဲ `BCrypt.checkpw()` သုံးရပါလိမ့်မယ်။