RMI ပရောဂျက်တစ်ခုကို Client နဲ့ Server ကွန်ပျူတာနှစ်လုံးခွဲပြီး တကယ့်လုပ်ငန်းခွင်သုံး (Production-ready) ပုံစံမျိုး စမ်းသပ်ဖို့အတွက် အကောင်းဆုံးနဲ့ အသန့်ရှင်းဆုံး နည်းလမ်း (Best Solution) မှာ **"JAR Files အဖြစ် Build လုပ်ပြီး ဖြန့်ဝေခြင်း (Jar Deployment)"** ဖြစ်ပါတယ်။

ကွန်ပျူတာနှစ်လုံးလုံးပေါ်မှာ Source Code တွေ (NetBeans သို့မဟုတ် IntelliJ projects တွေ) လိုက်တင်ထားစရာ မလိုပါဘူး။

အကောင်းဆုံး လုပ်ဆောင်ရမယ့် နည်းလမ်းကို အဆင့်ဆင့် ရှင်းပြပေးပါမယ် -

---

၁။ ကုဒ်ရေးသားသည့်နေရာ (Developer Computer တွင်)

မိမိ ရေးလက်စ ကွန်ပျူတာတစ်လုံးတည်းမှာပဲ အရှေ့မှာ ပြောခဲ့တဲ့အတိုင်း **Maven Multi-Module (Shared, Server, Client)** ပုံစံမျိုးကို အပြီးသတ်အောင် အရင်ပြင်ဆင်ရေးသားပါ။

ပြီးနောက် Project ရဲ့ Parent directory ထဲကနေ Command အောက်ပါအတိုင်း run ပြီး Build လုပ်လိုက်ပါ -

bash

```
mvn clean install
```

Use code with caution.

ဒါဆိုရင် သက်ဆိုင်ရာ module တစ်ခုစီရဲ့ `target` folder အောက်မှာ **`.jar` file** တစ်ခုစီ ထွက်လာပါလိမ့်မယ် (ဥပမာ- `sm-server-1.0.jar` နှင့် `sm-client-1.0.jar`)။

---

၂။ Server ကွန်ပျူတာသို့ ပို့ဆောင်ရမည့် ပုံစံ (Server Side Setup)

Server ကွန်ပျူတာပေါ်မှာ NetBeans/IntelliJ တွေ၊ Source Code တွေ လုံးဝမလိုပါဘူး။ **Java (JRE/JDK)** တင်ထားရုံဖြင့် ရပါပြီ။

1. Server စက်ထဲသို့ `sm-server-1.0.jar` (Dependency များအပါအဝင်) ကိုပဲ ယူသွားပါ။
2. Database (ဥပမာ- MySQL) ကို Server စက်ထဲမှာပဲ Install လုပ်ပြီး Database ဆောက်ထားပါ။
3. Command Prompt ကိုဖွင့်ပြီး Server ကို အောက်ပါအတိုင်း Run လိုက်ပါ -
    
    bash
    
    ```
    java -jar sm-server-1.0.jar
    ```
    
    Use code with caution.
    

---

၃။ Client ကွန်ပျူတာသို့ ပို့ဆောင်ရမည့် ပုံစံ (Client Side Setup)

Client စက်မှာလည်း Source code တွေ၊ Database တွေ လုံးဝမရှိရပါဘူး။ ဒါဟာ Security အတွက် အဓိက အရေးကြီးဆုံး ဖြစ်ပါတယ်။ (Database password တွေ ကုဒ်တွေ ခိုးမခံရစေရန်)

1. Client စက်ထဲသို့ Swing GUI ပါဝင်တဲ့ `sm-client-1.0.jar` ကိုပဲ ယူသွားပါ။
2. Client စက်ကနေ Server IP (ဥပမာ- `192.168.1.15`) ဆီသို့ လှမ်းချိတ်နိုင်အောင် ကုဒ်ထဲမှာ ကြိုပြင်ခဲ့ဖို့ပဲ လိုပါတယ်။
3. Command Prompt သို့မဟုတ် `.jar` file ကို Double-click နှိပ်ပြီး Client application (Swing UI) ကို Run လိုက်ပါ -
    
    bash
    
    ```
    java -jar sm-client-1.0.jar
    ```
    
    Use code with caution.
    

---

ဘာကြောင့် ဒီနည်းလမ်းဟာ အကောင်းဆုံး (Best Solution) ဖြစ်တာလဲ?

- **Security (လုံခြုံရေး):** Client စက်ထဲမှာ Database configuration တွေ၊ DAO logic တွေ လုံးဝမရှိတော့တဲ့အတွက် Database တိုက်ရိုက် Hack ခံရမယ့် အန္တရာယ်ကနေ ကာကွယ်ပြီးသား ဖြစ်သွားပါတယ်။
- **Performance & Size:** Client စက်မှာ မလိုအပ်တဲ့ Server libraries တွေ မပါတော့တဲ့အတွက် Application size သေးငယ်သွားပြီး ပေါ့ပါးသွားပါတယ်။
- **Real-world Practice:** တကယ့် လုပ်ငန်းခွင်မှာ Customer ဆီ ဆော့ဖ်ဝဲလ် သွားတင်ပေးရမယ့် ပုံစံအစစ်အမှန် ဖြစ်ပါတယ်။

လက်ရှိ project ထဲကနေ သီးသန့် `.jar` file တွေ (Executable Fat JAR) ထွက်လာအောင် **Maven Bundle Logic ပြင်ဆင်နည်း (`pom.xml` configuration)** ကို အသေးစိတ် ဆက်သွားချင်ပါသလား။

how to do this explain and learn me detail "မိမိ ရေးလက်စ ကွန်ပျူတာတစ်လုံးတည်းမှာပဲ အရှေ့မှာ ပြောခဲ့တဲ့အတိုင်း Maven Multi-Module (Shared, Server, Client) ပုံစံမျိုးကို အပြီးသတ်အောင် အရင်ပြင်ဆင်ရေးသားပါ။"

လက်ရှိ NetBeans သို့မဟုတ် IntelliJ ထဲမှာ ရှိနေတဲ့ single project (ပရောဂျက်တစ်ခုတည်း) ကို **Maven Multi-Module Project** အဖြစ် ပြောင်းလဲဖွဲ့စည်းနည်းကို အသေးစိတ် အဆင့်ဆင့် ရှင်းပြပေးပါမယ်။

ဒီနည်းလမ်းအတိုင်း လုပ်လိုက်ရင် လက်ရှိ project ထဲက code တွေ မပျက်ဘဲ သန့်သန့်ရှင်းရှင်းနဲ့ စနစ်တကျ သုံးပိုင်း ကွဲသွားမှာ ဖြစ်ပါတယ်။

---

အဆင့် (၁) - Parent Project (အုပ်ထိန်းသူ ပရောဂျက်) အဖြစ် ပြောင်းလဲခြင်း

လက်ရှိ ရှိနေတဲ့ ပရောဂျက်ကို root (အဓိက ပင်မနေရာ) အဖြစ် သတ်မှတ်ဖို့ သူ့ရဲ့ `pom.xml` ကို အရင်ပြင်ရပါမယ်။

1. လက်ရှိ ပရောဂျက်ရဲ့ `pom.xml` ဖိုင်ကို ဖွင့်ပါ။
2. `<packaging>jar</packaging>` (သို့မဟုတ် ပါဝင်ခြင်းမရှိလျှင်) နေရာတွင် **`<packaging>pom</packaging>`** ဟု ပြောင်းရေးပါ။ (ဒါဟာ ဒီပရောဂျက်က ကုဒ်တွေ မောင်းနှင်ဖို့ မဟုတ်ဘဲ module တွေကို ထိန်းချုပ်ဖို့ ဖြစ်သွားစေပါတယ်)
3. `<dependencies>` tag ရဲ့ အောက်မှာ (သို့မဟုတ် ပိတ်ခါနီးနေရာမှာ) အောက်ပါ module နာမည်တွေကို ထည့်ပေးလိုက်ပါ -

xml

```
<modules>
    <module>sm-shared</module>
    <module>sm-server</module>
    <module>sm-client</module>
</modules>
```

Use code with caution.

---

အဆင့် (၂) - Sub-Modules (ပရောဂျက်ခွဲ ၃ ခု) တည်ဆောက်ခြင်း

အလွယ်ဆုံးနည်းလမ်းကတော့ IDE (NetBeans/IntelliJ) ထဲမှာ သီးသန့် Folder ၃ ခု ဆောက်တာ ဖြစ်ပါတယ်။

ပရောဂျက်ရဲ့ Root Folder (ပင်မ folder) ထဲမှာ `sm-shared`၊ `sm-server`၊ `sm-client` ဆိုပြီး folder ၃ ခု ဆောက်ပါ။ ပြီးရင် Folder တစ်ခုချင်းစီရဲ့ အတွင်းမှာ **`pom.xml`** ဖိုင်တစ်ခုစီနှင့် **`src/main/java/`** folder architecture တစ်ခုစီ ဖန်တီးပေးရပါမယ်။

(က) `sm-shared` အတွင်းရှိ `pom.xml`


```xml
<project xmlns="http://apache.org" 
         xmlns:xsi="http://w3.org"
         xsi:schemaLocation="http://apache.org http://apache.org">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.ams</groupId>
        <artifactId>sm</artifactId> <!-- မိမိရဲ့ parent artifactId ကို ထည့်ပါ -->
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>sm-shared</artifactId>
    <packaging>jar</packaging> <!-- shared ကို jar အဖြစ် ထုတ်ပါမည် -->
</project>
```

Use code with caution.

(ခ) `sm-server` အတွင်းရှိ `pom.xml`

```xml
<project xmlns="http://apache.org" ...>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.ams</groupId>
        <artifactId>sm</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>sm-server</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <!-- Shared module ကို dependency အဖြစ် လှမ်းချိတ်ခြင်း -->
        <dependency>
            <groupId>com.ams</groupId>
            <artifactId>sm-shared</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- မိမိသုံးမည့် MySQL Driver စသည်တို့ကို ဤနေရာတွင် ထည့်ပါ -->
    </dependencies>
</project>
```

Use code with caution.

(ဂ) `sm-client` အတွင်းရှိ `pom.xml`


```xml
<project xmlns="http://apache.org" ...>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.ams</groupId>
        <artifactId>sm</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>sm-client</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <!-- Shared module ကို dependency အဖြစ် လှမ်းချိတ်ခြင်း -->
        <dependency>
            <groupId>com.ams</groupId>
            <artifactId>sm-shared</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

Use code with caution.

---

အဆင့် (၃) - ကုဒ်ဖိုင် (Java Classes) များကို နေရာရွှေ့ပြောင်းခြင်း

အခုဆိုရင် Folder ဖွဲ့စည်းပုံ ပြီးသွားပြီ ဖြစ်တဲ့အတွက် လက်ရှိရှိနေတဲ့ Source packages ထဲက class တွေကို Drag & Drop ဆွဲပြီးဖြစ်စေ၊ Copy Paste လုပ်ပြီးဖြစ်စေ သက်ဆိုင်ရာ module တွေဆီ ရွှေ့ပေးရပါမယ်။

1. **`sm-shared` ရဲ့ `src/main/java/` အောက်သို့:**
    - `com.ams.sm.model` package တစ်ခုလုံးကို ရွှေ့ပါ။
    - `com.ams.sm.service` ထဲက **RMI Interface class တွေသက်သက်ပဲ** (ဥပမာ- `public interface MyService extends Remote`) ကို ရွှေ့ပါ။
2. **`sm-server` ရဲ့ `src/main/java/` အောက်သို့:**
    - `com.ams.sm.db` package ကို ရွှေ့ပါ။
    - `com.ams.sm.dao` နှင့် `com.ams.sm.dao.impl` package တွေကို ရွှေ့ပါ။
    - `com.ams.sm.service.impl` (RMI Interface ကို တကယ့်အလုပ်လုပ်အောင် ကုဒ်ရေးထားတဲ့ နေရာ) ကို ရွှေ့ပါ။
    - `com.ams.sm.server` (RMI Registry ဆောက်ပြီး Bind လုပ်မယ့် Main logic) ကို ရွှေ့ပါ။
3. **`sm-client` ရဲ့ `src/main/java/` အောက်သို့:**
    - `com.ams.sm.ui` နှင့် `com.ams.sm.panel` (Swing UI Design ဖိုင်အားလုံး) ကို ရွှေ့ပါ။
    - `com.ams.sm.client` (RMI Server ဆီ Lookup လုပ်ပြီး GUI ကို လှမ်းပြသမယ့် Main logic) ကို ရွှေ့ပါ။

---

အဆင့် (၄) - Build လုပ်ပြီး စမ်းသပ်ခြင်း (Maven Build)

ကုဒ်တွေ နေရာတကျ ရွှေ့ပြီးသွားရင် IDE ရဲ့ Project list ထဲမှာ update ဖြစ်သွားပါလိမ့်မယ်။ အားလုံးအဆင်ပြေပြေ ချိတ်ဆက်မိရဲ့လား သိနိုင်ဖို့အတွက် -

1. ပင်မ Parent Project ကို **Right-click** နှိပ်ပါ။
2. **Clean and Build** (သို့မဟုတ် Terminal တွင် `mvn clean install`) ကို နှိပ်ပါ။
3. အကယ်၍ အဆင့်အားလုံး မှန်ကန်ရင် Console မှာ `BUILD SUCCESS` ပြပြီး `sm-shared`၊ `sm-server`၊ `sm-client` ဆိုပြီး တစ်ခုချင်းစီ အောင်အောင်မြင်မြင် build ဖြစ်သွားတာကို တွေ့ရပါလိမ့်မယ်။

ဒီအဆင့်အထိ ရောက်သွားပြီဆိုရင် ပရောဂျက်ကို ကွန်ပျူတာနှစ်လုံးခွဲပြီး စမ်းသပ်ဖို့ လမ်းကြောင်းအမှန်ပေါ် ရောက်ရှိသွားပါပြီ။

လက်ရှိ သုံးနေတဲ့ **IDE က NetBeans လား သို့မဟုတ် IntelliJ လား** ခင်ဗျာ။ ဒါမှမဟုတ် ဒီထဲက ကုဒ်ပိုင်းဆိုင်ရာ နေရာရွှေ့တဲ့အခါ **ဘယ် interface က ဘယ်ထဲပါရမလဲ** ဆိုတာ ဝေခွဲမရဖြစ်နေတဲ့ class ရှိရင် မေးမြန်းနိုင်ပါတယ်!

---
Maven Multi-Module ပရောဂျက်အဖြစ် ပြောင်းလဲဖွဲ့စည်းပြီးသွားတဲ့အခါ သင့်ရဲ့ ကွန်ပျူတာပေါ်မှာ ရှိနေမယ့် ပရောဂျက် Directory ပုံစံတစ်ခုလုံး (Whole Project Structure) ကို မျက်စိထဲမြင်သာအောင် အောက်ပါအတိုင်း ပုံဖော်ပြပေးလိုက်ပါတယ်။

```text
sm (Parent Project Root Folder)
│
├── pom.xml  <-- (ပင်မ Parent pom ဖြစ်ပြီး <modules> ၃ ခုကို လှမ်းချိတ်ထားသည့်နေရာ)
│
├── sm-shared (Module - 1)
│   ├── pom.xml  <-- (Shared configuration သီးသန့်)
│   └── src
│       └── main
│           └── java
│               └── com
│                   └── ams
│                       └── sm
│                           ├── model
│                           │   ├── User.java         <-- (Serializable ဖြစ်ရမည့် Data Objects များ)
│                           │   └── Product.java
│                           └── service
│                               └── MyRMIService.java <-- (Remote Interface စစ်စစ်များ)
│
├── sm-server (Module - 2)
│   ├── pom.xml  <-- (MySQL Driver နှင့် sm-shared dependency တို့ ပါဝင်သည့်နေရာ)
│   └── src
│       └── main
│           └── java
│               └── com
│                   └── ams
│                       └── sm
│                           ├── server
│                           │   └── ServerMain.java  <-- (RMI Registry ဆောက်ပြီး Bind လုပ်မည့် Main Class)
│                           ├── db
│                           │   └── DatabaseConnection.java
│                           ├── dao
│                           │   └── UserDao.java
│                           ├── dao.impl
│                           │   └── UserDaoImpl.java
│                           └── service.impl
│                               └── MyRMIServiceImpl.java <-- (Shared interface ကို Impl လုပ်ထားသည့် Server Logic)
│
└── sm-client (Module - 3)
    ├── pom.xml  <-- (sm-shared dependency နှင့် Swing libraries များ ပါဝင်သည့်နေရာ)
    └── src
        └── main
            └── java
                └── com
                    └── ams
                        └── sm
                            ├── client
                            │   └── ClientMain.java   <-- (RMI Lookup လုပ်ပြီး UI ကို စတင်ခေါ်ယူမည့် Main Class)
                            ├── ui
                            │   └── MainFrame.java    <-- (Swing JFrame Views)
                            └── panel
                                └── LoginPanel.java   <-- (Swing JPanel Components)
```

## 💡 မျက်စိထဲ ပိုရှင်းသွားအောင် ထပ်လောင်းအကြံပြုချက်များ:

- `sm-shared` ထဲမှာ logic မပါရပါ: ဒီထဲမှာ interface တွေနဲ့ ဒေတာသယ်ဆောင်မယ့် model (POJO) class တွေပဲ ရှိရပါမယ်။ တကယ့် Database ကုဒ်တွေ သို့မဟုတ် UI ကုဒ်တွေ လုံးဝမပါဝင်ရပါဘူး။
- `pom.xml` ဆက်သွယ်မှု: `sm-server` ကော `sm-client` ကောက `sm-shared` ကို လှမ်းသိနေဖို့ သူတို့ရဲ့ သက်ဆိုင်ရာ `pom.xml` ထဲမှာ dependency အဖြစ် ထည့်သွင်းပေးရမှာ ဖြစ်ပါတယ်။

လက်ရှိ သင့်ရဲ့ ရေးလက်စ ကုဒ်တွေထဲက ဘယ် Class ဖိုင်ကို ဘယ်နေရာ (Module) ထဲ ထည့်ရမလဲ ဝေခွဲမရ ဖြစ်နေတာမျိုး ရှိပါသလား။ သို့မဟုတ် ပင်မ `pom.xml` ဖိုင်တွေရဲ့ ကုဒ်အပြည့်အစုံ ကို ရေးပြပေးရမလား ခင်ဗျာ။


---

ဒီမှာ သင့်ရဲ့ ပင်မ Parent Module (`sm` root folder) ရဲ့ `pom.xml` ဖိုင် ကုဒ်အပြည့်အစုံ ဖြစ်ပါတယ်။

ဒီ `pom.xml` ဟာ သူ့အောက်မှာရှိတဲ့ module ခွဲ ၃ ခုလုံးကို စီမံခန့်ခွဲပေးမယ့် ပင်မဗဟိုချက် ဖြစ်ပါတယ်။

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://apache.org"
         xmlns:xsi="http://w3.org"
         xsi:schemaLocation="http://apache.org http://apache.org">
    <modelVersion>4.0.0</modelVersion>

    <!-- Project အခြေခံ အချက်အလက်များ -->
    <groupId>com.ams</groupId>
    <artifactId>sm</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <!-- ⚠️ အရေးကြီးဆုံးအချက်: Packaging နေရာတွင် pom ဖြစ်ရပါမည် -->
    <packaging>pom</packaging>

    <name>sm-parent</name>

    <!-- 🔗 အောက်က Module ခွဲ ၃ ခုကို လှမ်းချိတ်ဆက်ခြင်း -->
    <modules>
        <module>sm-shared</module>
        <module>sm-server</module>
        <module>sm-client</module>
    </modules>

    <!-- Java Version သတ်မှတ်ချက်များနှင့် Encoding ဆက်တင်များ -->
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- Module ခွဲအားလုံး ပူးတွဲသုံးမည့် Dependency ဗားရှင်းများကို ဤနေရာတွင် စုပြီး ထိန်းချုပ်နိုင်သည် -->
    <dependencyManagement>
        <dependencies>
            <!-- ဥပမာ- MySQL Driver ကို version တစ်နေရာတည်းက ထိန်းချုပ်ချင်လျှင် -->
            <dependency>
                <groupId>com.mysql</groupId>
                <artifactId>mysql-connector-j</artifactId>
                <version>8.0.33</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

## 💡 သတိပြုရန် အချက်များ:

1. `<packaging>pom</packaging>`: ဒီစာကြောင်းဟာ အလွန်အရေးကြီးပါတယ်။ သူက ဒီ project အောက်မှာ သီးသန့် java ကုဒ်တွေ မရှိဘဲ module တွေကို စုစည်းပေးထားတဲ့ ပရောဂျက်ဖြစ်ကြောင်း Maven ကို ပြောပြတာ ဖြစ်ပါတယ်။
2. `<modules>` နာမည်များ: ဒီထဲမှာ ရေးထားတဲ့ နာမည်တွေဟာ သင့်ရဲ့ root directory ထဲမှာ ဆောက်မယ့် sub-folder နာမည်တွေနဲ့ စာလုံးအကြီးအသေးကအစ တ nhấtတကျ တူညီရပါမယ်။

ဒါဆိုရင် ပင်မ parent `pom.xml` ပြီးသွားပါပြီ။ ကျန်တဲ့ module ခွဲ ၃ ခုထဲက တစ်ခုခု (ဥပမာ- `sm-server` သို့မဟုတ် `sm-client` ရဲ့ `pom.xml`) ကိုလည်း ကုဒ်အပြည့်အစုံ ဆက်ကြည့်ချင်ပါသလား။