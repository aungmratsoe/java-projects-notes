# pom.xml — Java 25 + RMI + Swing (FlatLaf) Project

Good news — **RMI ဟာ JDK ရဲ့ built-in feature ဖြစ်လို့ (`java.rmi.*` package) Maven dependency ဘာမှ ထပ်ထည့်စရာ မလိုပါ**။ Compiler ကို Java 25 အတွက် configure လုပ်ရုံနဲ့ FlatLaf dependency ပဲ ထည့်ပေးရပါမယ်။

## Full pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                              http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>rmi-swing-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.release>25</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <flatlaf.version>3.6</flatlaf.version>
    </properties>

    <dependencies>
        <!-- FlatLaf — Swing UI ကို modern look ပေးဖို့ -->
        <dependency>
            <groupId>com.formdev</groupId>
            <artifactId>flatlaf</artifactId>
            <version>${flatlaf.version}</version>
        </dependency>

        <!-- RMI က java.rmi.* package ဖြစ်လို့ dependency မလိုပါ (JDK built-in) -->
    </dependencies>

    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <!-- Compiler plugin — Java 25 support ဖို့ latest version လိုအပ် -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
                <configuration>
                    <release>25</release>
                </configuration>
            </plugin>

            <!-- Shade plugin — Dependency (FlatLaf) ပါတဲ့ "fat jar" 
                 ဖန်တီးဖို့ (Client/Server computer မှာ FlatLaf jar 
                 သီးခြား install စရာမလိုအောင်) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.6.0</version>
                <executions>
                    <!-- Server jar (Main-Class: Server) -->
                    <execution>
                        <id>server-jar</id>
                        <phase>package</phase>
                        <goals><goal>shade</goal></goals>
                        <configuration>
                            <finalName>rmi-server</finalName>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.example.rmi.Server</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                    <!-- Client jar (Main-Class: RmiClientGUI) -->
                    <execution>
                        <id>client-jar</id>
                        <phase>package</phase>
                        <goals><goal>shade</goal></goals>
                        <configuration>
                            <finalName>rmi-client</finalName>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.example.rmi.RmiClientGUI</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## အရေးကြီးတဲ့ အချက်များ

### ၁. `maven.compiler.release` — `<source>`/`<target>` အစား သုံးရမယ့် အကြောင်း

Java 25 လို modern version တွေအတွက် `<release>25</release>` ကိုပဲ သုံးပါ — `<source>`/`<target>` သီးခြားစီ ထားတာထက် ပိုသေချာပါတယ် (bootclasspath issue မဖြစ်အောင် Maven ကိုယ်တိုင် စီမံပေးလို့ပါ)။

### ၂. `maven-compiler-plugin` Version

Java 25 ဟာ 2025 နှောင်းပိုင်းထွက်တဲ့ version ဖြစ်လို့ **compiler plugin အဟောင်း (3.10.x အောက်) တွေက support မလုပ်နိုင်** ပါဘူး။ `3.13.0` (သို့) ပိုအသစ်သုံးပါ။ Maven ကိုယ်တိုင်လည်း recent version (3.9+) လိုအပ်ပါတယ်။

```bash
mvn -version   # Maven version စစ်ရန်
```

### ၃. Two Jar ထုတ်တဲ့ Setup ဘာကြောင့်

- Server computer မှာ `Server.java` ကိုပဲ run ရမှာလို့ **rmi-server.jar**
- Client computer မှာ `RmiClientGUI.java` (Swing) ကိုပဲ run ရမှာလို့ **rmi-client.jar**
- `mvn package` တစ်ခါ run ရင် jar **၂ ခုစလုံး** `target/` folder ထဲ ထွက်လာမယ် (server ကို client jar လိုမှာ, client ကို server jar လိုမှာ မဟုတ်လို့ ဒီလို ခွဲထုတ်တာ)

### ၄. Build & Run

```bash
mvn clean package

# Server computer ပေါ်မှာ
java -jar target/rmi-server.jar

# Client computer ပေါ်မှာ
java -jar target/rmi-client.jar
```

Fat jar ဖြစ်တဲ့အတွက် FlatLaf ကို client computer မှာ သီးခြား install/copy လုပ်စရာ **မလိုပါ** — jar file တစ်ခုတည်း copy လိုက်ရင် client computer ပေါ်မှာ run လို့ရပါပြီ (JDK 25 ပဲ install ထားရင် လုံလောက်ပါတယ်)။

### ၅. NetBeans Maven Project မှာ

သင့် project ကို NetBeans GUI Builder နဲ့ ရေးနေတယ်ဆိုရင် NetBeans က pom.xml ထဲက properties တွေကို auto-detect လုပ်ပါလိမ့်မယ်၊ ဒါပေမယ့် Java 25 support အတွက် **NetBeans version အသစ်ဆုံး** (23+ or later) သုံးထားကြောင်း သေချာအောင် စစ်ပါ — NetBeans version အဟောင်းတွေက Java 25 syntax/features ကို IDE မှာ properly highlight/compile မလုပ်နိုင်တာ ဖြစ်နိုင်ပါတယ်။

---

FlatLaf ကို `RmiClientGUI` main() ထဲမှာ `setLookAndFeel()` ချိတ်ပုံ ပြပေးရမလား၊ ဒါမှမဟုတ် shade plugin setup မှာ error တက်ရင် debug လုပ်ချင်ပါသလား?