NetBeans GUI Builder (drag-and-drop) သုံးမယ်ဆိုရင် FlatLaf setup ပုံစံ အနည်းငယ်ကွာပါတယ် - NetBeans က `initComponents()` auto-generated code ကို separate ထားလို့ ဂရုစိုက်ရမယ့် အချက်တွေ ရှိပါတယ်။

### 1. Library ထည့်နည်း (NetBeans Project)

**Maven project ဆိုရင်:** pom.xml ထဲမှာ FlatLaf dependency ထည့်ပါ (Part 7 မှာပြောခဲ့သလိုပါပဲ)

**Ant-based project (NetBeans default, Maven မဟုတ်ရင်):**

1. Project ကို right-click → **Properties** → **Libraries**
2. **Add JAR/Folder** ကို နှိပ်ပြီး FlatLaf jar file ကို browse လုပ်ပါ (formdev.com ကနေ download လုပ်ထားရပါမယ်)
3. **OK** နှိပ်ပြီး apply လုပ်ပါ

### 2. main() method ထဲမှာ FlatLaf Activate လုပ်ခြင်း

NetBeans GUI Builder နဲ့ `JFrame` create လုပ်ရင် auto-generate ဖြစ်တဲ့ code structure က ဒီလိုပုံစံဖြစ်ပါတယ်:

```java
public class MainForm extends javax.swing.JFrame {

    public MainForm() {
        initComponents();  // NetBeans auto-generated - drag/drop ထားတဲ့ components တွေ
    }

    // ... initComponents() code (DO NOT EDIT manually - NetBeans manages this) ...

    public static void main(String args[]) {
        // FlatLaf ကို ဒီနေရာမှာ setup လုပ်ရပါမယ် - components create ဖြစ်ခင်
        com.formdev.flatlaf.FlatLightLaf.setup();

        java.awt.EventQueue.invokeLater(() -> {
            new MainForm().setVisible(true);
        });
    }
}
```

**အရေးကြီးတဲ့ Rule:** `FlatLightLaf.setup()` ကို **`main()` method ထဲမှာပဲ** ထည့်ပါ၊ **`initComponents()` ထဲမှာ ဘယ်တော့မှ မထည့်ပါနဲ့** - NetBeans က `initComponents()` block ကို GUI Builder နဲ့ auto-manage လုပ်နေလို့ manual code ထည့်ရင် **Design view ကနေ ပြန် regenerate လုပ်တဲ့အခါ code ပျောက်သွားနိုင်ပါတယ်**။

### 3. NetBeans GUI Builder (Design View) ထဲမှာ Preview မမြင်ရတာ

NetBeans Design view (drag-and-drop editor) မှာ FlatLaf theme ကို **preview အနေနဲ့ တန်းမြင်ရမှာ မဟုတ်ပါဘူး** - Design view က NetBeans ရဲ့ default rendering ကိုပဲ ပြသပါတယ်။ FlatLaf theme ကို actual **run လုပ်မှသာ** (F6 / Run Project) မြင်ရမှာပါ။

### 4. Multiple Forms ရှိရင် (Login form, Main form, စသည်)

Project ထဲမှာ JFrame form အများကြီးရှိရင် FlatLaf setup ကို **application ရဲ့ entry point (main() ပါတဲ့ class) တစ်ခုတည်း** မှာပဲ တစ်ကြိမ်ထည့်ရင် လုံလောက်ပါတယ် - form တစ်ခုချင်းစီမှာ ထပ်ထည့်စရာ မလိုပါဘူး (Look and Feel က application-wide setting ဖြစ်လို့ တစ်ခါချိန်ညှိရင် form အားလုံးအတွက် သက်ရောက်ပါတယ်)။

```java
// Application Entry Point class (Main.java ဒါမှမဟုတ် App.java)
public class Main {
    public static void main(String[] args) {
        com.formdev.flatlaf.FlatLightLaf.setup();
        
        java.awt.EventQueue.invokeLater(() -> {
            new LoginForm().setVisible(true);  // login form ကနေ စ
        });
    }
}
```

### 5. NetBeans GUI Builder-Specific Component Properties

NetBeans Design view ရဲ့ **Properties panel** ကနေ component (JButton, JTextField) ကို ရွေးပြီး `putClientProperty` value တွေ (FlatLaf placeholder text, button type စသည်) ကို **Properties → Code → Custom Code** ဒါမှမဟုတ် **client properties** section ကနေ ထည့်လို့ရပါတယ် (NetBeans version အလိုက် interface အနည်းငယ် ကွာနိုင်ပါတယ်)။

လွယ်ကူတဲ့ approach ကတော့ `initComponents()` ပြီးတဲ့နောက် custom constructor code ထဲမှာ ထည့်တာပါ:

```java
public MainForm() {
    initComponents();  // NetBeans auto-generated
    
    // Custom FlatLaf client properties - ဒီနေရာမှာ manual ထည့်လို့ရ (initComponents() ပြီးမှသာ)
    jTextField1.putClientProperty("JTextField.placeholderText", "Enter name");
}
```

**Note:** `initComponents()` **ပြီးမှသာ** custom code ရေးရပါမယ် (constructor ထဲမှာ `initComponents();` line အောက်ကနေ ရေးပါ) - `initComponents()` က component တွေကို create/configure လုပ်နေတာမို့ ၎င်းမပြီးခင် reference လုပ်လို့ မရနိုင်ပါဘူး။

---

**Summary - NetBeans + FlatLaf Workflow:**

1. Library ထည့် (Maven pom.xml သို့မဟုတ် Project Properties → Libraries)
2. `main()` method ထဲမှာ `FlatLightLaf.setup()` ထည့် (entry point class တစ်ခုတည်းမှာပဲ)
3. Design view မှာ FlatLaf preview မမြင်ရဘူး - Run (F6) လုပ်မှ actual theme မြင်ရမယ်
4. Component-specific customization ကို `initComponents()` **ပြီးမှသာ** ထည့်ပါ

Setup လုပ်ကြည့်ပြီးလား? NetBeans version ဘယ်ဟာသုံးနေလဲ (Apache NetBeans ဆိုရင် modern version)၊ error တွေ့ရင် ပြောပါ။