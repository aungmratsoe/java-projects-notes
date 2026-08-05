# Model Class ၂ ခုကို RMI-Ready ဖြစ်အောင် စစ်ကြည့်ရအောင်

## ✅ Good News — Date/Timestamp အတွက် ပြင်စရာမလို

`java.util.Date` နဲ့ `java.sql.Timestamp` နှစ်ခုစလုံး **JDK ကိုယ်တိုင် `Serializable` ဖြစ်အောင် ရေးထားပြီးသား** ဖြစ်လို့ ဒီ field တွေအတွက် ပူစရာမလိုပါ။

## ⚠️ ပြင်ရမယ့်အချက် — `Serializable` Interface မထည့်ရသေးပါ

RMI method (`getAllStudents()`, `loginUser()` စသည်) က object ကို **network ကနေတဆင့်** ပို့ရမှာမို့ **class ကိုယ်တိုင် `Serializable` implement မလုပ်ရသေးရင် `NotSerializableException` throw ဖြစ်ပါလိမ့်မယ်**။

### Student.java — ပြင်ရမယ့်ပုံ

```java
package com.ams.qrcode.model;

import java.io.Serializable;
import java.sql.Timestamp;
import java.util.Date;

public class Student implements Serializable {          // ← ဒါပဲ ထပ်ထည့်ရမယ်
    private static final long serialVersionUID = 1L;    // ← ဒါလည်း ထည့်သင့်

    private Integer id;
    private String studentId;
    // ... field, constructor, getter/setter အားလုံး မပြောင်းရပါ
}
```

### User.java — ပြင်ရမယ့်ပုံ

```java
package com.ams.qrcode.model;

import java.io.Serializable;

public class User implements Serializable {             // ← ဒါပဲ ထပ်ထည့်ရမယ်
    private static final long serialVersionUID = 1L;    // ← ဒါလည်း ထည့်သင့်

    private int id;
    private String username;
    // ... field, constructor, getter/setter အားလုံး မပြောင်းရပါ
}
```

**ဒါပဲ ပြောင်းရမှာပါ** — Field/Constructor/Getter/Setter logic ကို လုံးဝ ပြင်စရာမလိုပါ။

## `serialVersionUID` ဘာလုပ်ဖို့လဲ

Server နဲ့ Client က `Student.class` version မတူညီရင် (ဥပမာ - Server ဘက်မှာ field အသစ်ထည့်ပြီး Client ဘက် compile ပြန်မလုပ်ရသေးရင်) `serialVersionUID` မတူညီလို့ `InvalidClassException` ဖြစ်နိုင်ပါတယ်။ Explicit ထည့်ထားရင် ဒီ error message ပိုနားလည်လွယ်ပါတယ် (field ထပ်ထည့်တိုင်း UID မပြောင်းချင်ရင် manual ထည့်ထားတာက အကောင်းဆုံးပါ)။

## ⚠️ Security ပြဿနာတစ်ခု — `User` Model ထဲက `password` Field

```java
public class User {
    private String password; // ← Login response ထဲမှာ ဒါပါလာလိမ့်မယ်
}
```

`UserService.loginUser()` က `User` object ကို client ဆီ ပြန်ပို့တဲ့အခါ **password field ပါ တွဲပို့** သွားနိုင်ပါတယ် (hash ဖြစ်ပါစေ, plain ဖြစ်ပါစေ) — Client ဘက်မှာ password ကို memory ထဲ မထားသင့်ပါ။ `UserServiceImpl.loginUser()` ထဲမှာ password ကို clear လုပ်ပြီးမှ ပြန်ပို့တာ ပိုကောင်းပါတယ်:

```java
@Override
public User loginUser(String email, String plainPassword) throws RemoteException, DataAccessException {
    User user = dao.loginUser(email, plainPassword);
    if (user != null) {
        user.setPassword(null); // ← Client ဆီ password ကို ပြန်မပို့ဘူး
    }
    return user;
}
```

## Field Type အားလုံး Double-check

|Field Type|Serializable လား|
|---|---|
|`Integer`, `String`, `int`|✅ (Wrapper/primitive types)|
|`java.util.Date`|✅|
|`java.sql.Timestamp`|✅|

Model class ၂ ခုစလုံးက field type အားလုံး Serializable-safe ဖြစ်နေပြီး ပြဿနာ မရှိပါ — `implements Serializable` ကို class level မှာ ထည့်ရုံလောက်ပါပဲ လိုအပ်ပါတယ်။

---

`StudentDAOImpl`/`UserDAOImpl` (actual DB logic) ကို ပြပေးရင် SQLException handling ကို RMI-safe ဖြစ်အောင် ပြန်စစ်ပေးနိုင်ပါတယ်၊ ဒါမှမဟုတ် Client ဘက်က Login screen (JFrame) UI ကို RMI service ချိတ်ပုံ full example ကူညီရမလား?