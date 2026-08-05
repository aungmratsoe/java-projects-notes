# Client Computer vs Server Computer — Package အလိုက် အပြည့်အစုံ ခွဲပေးထားပါမယ်

## 🖥️ Server Computer ပေါ်မှာ ရှိရမယ့် Packages

```
com.ams.qrcode.model            ← Student, User (Serializable) — Server ဘက်လည်း လိုအပ်
com.ams.qrcode.exceptions       ← DataAccessException — Server ဘက်လည်း လိုအပ်
com.ams.qrcode.service          ← Remote interfaces (StudentService, UserService)
com.ams.qrcode.service.impl     ← StudentServiceImpl, UserServiceImpl (UnicastRemoteObject)
com.ams.qrcode.dao              ← StudentDAOInterface, StudentDAOImpl, UserDAOInterface, UserDAOImpl
com.ams.qrcode.db               ← DBConnection (JDBC — MySQL connect)
com.ams.qrcode.server           ← ServerMain.java (Registry create + bind)
com.ams.qrcode.utils            ← Server-side utility ရှိရင် (ဥပမာ - QR file naming helper, BCrypt wrapper)
```

## 💻 Client Computer ပေါ်မှာ ရှိရမယ့် Packages

```
com.ams.qrcode.model            ← Student, User (Serializable) — Client ဘက်လည်း လိုအပ် (RMI response လက်ခံဖို့)
com.ams.qrcode.exceptions       ← DataAccessException — Client ဘက်လည်း လိုအပ် (catch လုပ်ဖို့)
com.ams.qrcode.service          ← Remote interfaces — Client ဘက်လည်း လိုအပ် (lookup + cast ဖို့)
com.ams.qrcode.ui               ← SignIn, SignUp, Home, QRGenerator, QRScanner (JFrame/JPanel)
com.ams.qrcode.components       ← Custom Swing components
com.ams.qrcode.client           ← ClientMain.java, RmiConnectionManager.java
com.ams.qrcode.utils            ← Client-side utility ရှိရင် (ဥပမာ - CryptoUtils, QR generate wrapper (ZXing), date format helper)
```

## 🔁 Package Overlap — "Shared" ဆိုတာ ဘယ်လို အလုပ်လုပ်လဲ

|Package|Server|Client|အကြောင်းရင်း|
|---|:-:|:-:|---|
|`model`|✅|✅|Object ကို Client ↔ Server ကြား serialize ပို့ရမှာမို့ **နှစ်ဖက်စလုံးမှာ (source code) တူတူ ရှိရမယ်**|
|`exceptions`|✅|✅|Server က throw, Client က catch — နှစ်ဖက်စလုံး `.class` file တူညီရမယ်|
|`service` (Remote interfaces)|✅|✅|"Contract" — Server က implement, Client က lookup+cast လုပ်ဖို့|
|`service.impl`|✅|❌|Implementation logic — Client ဘက်ကို ထုတ်ပေးစရာမလို (Client က interface stub ကိုပဲ သိရင် လုံလောက်)|
|`dao`|✅|❌|DB query logic — Client ဆီ ရောက်ရင် DB credential/structure ပေါက်ကြားနိုင်|
|`db`|✅|❌|JDBC connection — Client ကို **လုံးဝ မထုတ်ရ**|
|`ui`, `components`|❌|✅|Swing UI — Server မှာ run စရာမလို (Server က console app ဖြစ်နိုင်)|
|`server` (ServerMain)|✅|❌|Server startup logic|
|`client` (ClientMain, RmiConnectionManager)|❌|✅|Client startup logic|

## Maven Module အနေနဲ့ ခွဲမယ်ဆိုရင် (အကြံပြု)

```
qrcode-parent/
├── qrcode-common/          ← model, exceptions, service (interfaces) — Server+Client dependency
│   └── com.ams.qrcode.{model, exceptions, service}
│
├── qrcode-server/          ← common ကို dependency ထား + server-only packages
│   └── com.ams.qrcode.{dao, db, service.impl, server}
│
└── qrcode-client/          ← common ကို dependency ထား + client-only packages
    └── com.ams.qrcode.{ui, components, client, utils}
```

`qrcode-common` ကို `qrcode-server` နဲ့ `qrcode-client` **နှစ်ခုစလုံးက** Maven dependency အနေနဲ့ ထည့်ရမယ် — ဒါဆို `model`/`exceptions`/`service` ကို **တစ်နေရာတည်း တည်ဆောက်ပြီး, jar ၂ ခုစလုံးမှာ compile-time မှာ auto-sync** ဖြစ်နေမှာပါ (Class version မတူတာစတဲ့ error ရှောင်နိုင်).

## ⚠️ Compile-time Sync အရေးကြီးပုံ

`model`/`exceptions`/`service` package ကို Server ဘက်မှာ ပြင်လိုက်ရင် (field အသစ်ထည့်တာ, method signature ပြောင်းတာ) — **Client ဘက်ကိုလည်း ထပ်တူ update လုပ်ပြီး ပြန် compile ချရပါမယ်**။ Maven multi-module structure သုံးထားရင် `mvn clean install` တစ်ခါတည်း run ရင် module အားလုံး sync ဖြစ်သွားလို့ ဒီပြဿနာ ရှောင်နိုင်ပါတယ်။

---

`qrcode-common`/`qrcode-server`/`qrcode-client` module ၃ ခုအတွက် pom.xml အပြည့်အစုံ (parent + child ၃ ခု) ရေးပေးရမလား၊ ဒါမှမဟုတ် NetBeans ထဲမှာ existing single project ကနေ module ၃ ခု ပြန်ခွဲထုတ်နည်း (step-by-step) ကူညီရမလား?

---

ပြထားတဲ့ Project Structure (Multi-module Maven project ပုံစံ) အရ **`qrcode-common`** ကို `qrcode-server` နဲ့ `qrcode-client` တို့ရဲ့ **dependency** တစ်ခုအနေနဲ့ ချိတ်ဆက်အသုံးပြုဖို့အတွက် Maven ရဲ့ Module နှင့် Project Dependencies စနစ်ကို အသုံးပြုရပါမယ်။

အောက်ပါ အဆင့်အတိုင်း အလွယ်တကူ ပြုလုပ်နိုင်ပါတယ်-

---

### အဆင့် ၁ - Parent `pom.xml` တွင် Modules များကို ကြေညာခြင်း

အဓိက Parent Project (`qrcode-parent`) ရဲ့ `pom.xml` ထဲတွင် အောက်ပါအတိုင်း `modules` များကို ထည့်သွင်းပေးရပါမယ်။ ဒါမှသာ Maven က Sub-modules တွေကို သိရှိမှာ ဖြစ်ပါတယ်။

```xml
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.ams</groupId>
    <artifactId>qrcode-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>qrcode-common</module>
        <module>qrcode-server</module>
        <module>qrcode-client</module>
    </modules>
</project>

```

---

### အဆင့် ၂ - `qrcode-common` ၏ `pom.xml` ကို ဖန်တီးခြင်း

`qrcode-common` တွင် သီးသန့် Models, Exceptions နှင့် Service Interfaces များကို ထည့်သွင်းမည်ဖြစ်ပြီး၊ ၎င်း၏ `pom.xml` ကို အောက်ပါအတိုင်း ရေးသားပါ-

```xml
<project ...>
    <parent>
        <groupId>com.ams</groupId>
        <artifactId>qrcode-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>qrcode-common</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!-- packaging ကို jar ဟု သတ်မှတ်ပေးပါ -->
    <packaging>jar</packaging>
</project>

```

---

### အဆင့် ၃ - `qrcode-server` တွင် `qrcode-common` ကို Dependency ထည့်ခြင်း

`qrcode-server` ရဲ့ `pom.xml` ထဲတွင် `qrcode-common` ကို အောက်ပါအတိုင်း Dependency အဖြစ် ထည့်သွင်းပေးပါ-

```xml
<project ...>
    <parent>
        <groupId>com.ams</groupId>
        <artifactId>qrcode-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>qrcode-server</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- qrcode-common ကို ဆာဗာ၏ dependency အဖြစ် ချိတ်ဆက်ခြင်း -->
        <dependency>
            <groupId>com.ams</groupId>
            <artifactId>qrcode-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>

```

---

### အဆင့် ၄ - `qrcode-client` တွင် `qrcode-common` ကို Dependency ထည့်ခြင်း

နည်းတူစွာပင် `qrcode-client` ရဲ့ `pom.xml` ထဲတွင်လည်း `qrcode-common` ကို Dependency အဖြစ် ထည့်ပေးရပါမယ်-

```xml
<project ...>
    <parent>
        <groupId>com.ams</groupId>
        <artifactId>qrcode-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>qrcode-client</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- qrcode-common ကို ကလိုင်းယန်တ်၏ dependency အဖြစ် ချိတ်ဆက်ခြင်း -->
        <dependency>
            <groupId>com.ams</groupId>
            <artifactId>qrcode-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>

```

---

### 💡 အနှစ်ချုပ် လုပ်ဆောင်ရမည့်ပုံစံ

၁။ အထက်ပါအတိုင်း `pom.xml` ဖိုင်များကို ပြင်ဆင်ပြီးပါက Root Parent Folder (`qrcode-parent`) နေရာတွင် Terminal/Command Prompt ဖွင့်ပါ။
၂။ Maven ဖြင့် အောက်ပါ Command ကို ရိုက်၍ Build လုပ်ပါ-

```bash
mvn clean install

```

၃။ ဤသို့ လုပ်ဆောင်လိုက်ခြင်းအားဖြင့် `qrcode-common` သည် Jar ဖိုင်အဖြစ် Build ဖြစ်သွားမည်ဖြစ်ပြီး၊ `qrcode-server` နှင့် `qrcode-client` တို့က ၎င်း Common ထဲရှိ Interfaces များနှင့် Models များကို ချောမွေ့စွာ `import` လုပ်၍ အသုံးပြုနိုင်သွားမည် ဖြစ်ပါသည်။

---

Maven POM ဖိုင်များတွင် `<modelVersion>4.0.0</modelVersion>` ကို ထည့်သွင်းရခြင်း၏ အဓိက အကြောင်းရင်းမှာ **Maven Project Model ၏ ဗားရှင်း (Model Version)** ကို သတ်မှတ်ပေးရန် ဖြစ်ပါသည်။

အသေးစိတ် အကြောင်းရင်းများမှာ အောက်ပါအတိုင်း ဖြစ်ပါသည်-

- **Maven ၏ စံနှုန်းသတ်မှတ်ချက် (Standard Version):** `4.0.0` ဆိုသည်မှာ Maven 2 နှင့် Maven 3/4 တို့တွင် အသုံးပြုရန် တရားဝင် သတ်မှတ်ထားသော Pom Model ၏ တစ်ခုတည်းသော လက်ခံနိုင်သည့် ဗားရှင်းစံနှုန်း ဖြစ်သည်။
    
- **POM Structure ကို နားလည်စေရန်:** Maven က ဤဖိုင်ကို ဖတ်ရှုသည့်အခါ မည်သည့် Structure (ဥပမာ- `<groupId>`, `<artifactId>`, `<modules>`, `<dependencies>` စသည်တို့) ဖြင့် ရေးသားထားသည်ကို သိရှိစေရန်အတွက် ဤ Model Version ကို မဖြစ်မနေ စစ်ဆေးပါသည်။
    
- **Backward Compatibility / အနာဂတ်အတွက် ပံ့ပိုးမှု:** Maven 2 အစောပိုင်း ကာလများက မော်ဒယ်ဗားရှင်း အဟောင်းများ ရှိခဲ့သော်လည်း ယခုလက်ရှိ အသုံးပြုနေကြသော Maven ဗားရှင်းအားလုံးသည် Model Version `4.0.0` ကိုသာ အဓိက အပြည့်အစုံ ပံ့ပိုးပေးထားပါသည်။
    

**အနှစ်ချုပ်:** ၎င်းလိုင်းသည် Maven အနေဖြင့် သင်၏ `pom.xml` ဖိုင်ကို မည်သည့် စည်းမျဉ်းစည်းကမ်း (Schema) အတိုင်း ဘာသာပြန်ဆို (Parse) ရမည်ကို ပြောပြပေးသော အဓိက သော့ချက် တစ်ခု ဖြစ်ပါသည်။