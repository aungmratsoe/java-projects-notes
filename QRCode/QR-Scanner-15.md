# Chapter 15 — `main()`

## FlatLaf + Look & Feel + Application Startup Lifecycle

Great **aung mrat**.

ဒီ Chapter မှာ `QRScanner.java` ရဲ့ နောက်ဆုံးပိုင်းဖြစ်တဲ့ `main()` method ကို လေ့လာမယ်။

ဒီအပိုင်းက application ရဲ့ **entry point** ဖြစ်ပါတယ်။

Java Application တစ်ခု run လိုက်တဲ့အချိန်မှာ ပထမဆုံးစတင်အလုပ်လုပ်တဲ့နေရာက `main()` ဖြစ်တယ်။

---

# 16.1 Java Application Entry Point

Java Program တိုင်းမှာ:

```java
public static void main(String args[])
```

ရှိရတယ်။

ဒါက JVM ကို ပြောတာ:

> "Program ကို ဒီနေရာကနေ စတင်ပါ"

ဆိုတဲ့ meaning ဖြစ်တယ်။

---

Flow:

```text
User Double Click

        |
        v

JVM Start

        |
        v

main()

        |
        v

Create JFrame

        |
        v

Show UI

```

---

မင်းရဲ့ code:

```java
public static void main(String args[]) {

}
```

---

ဒီ method ထဲမှာ လုပ်နေတဲ့အရာ ၃ ခုရှိတယ်။

```text
1. Default Look & Feel Setup

2. FlatLaf Theme Setup

3. Start Swing UI Thread

```

---

# 16.2 Look & Feel ဆိုတာဘာလဲ?

Swing မှာ Component တွေရဲ့ appearance ကို Look & Feel လို့ခေါ်တယ်။

Example:

Default Java Swing:

```
+----------------+
| JButton        |
+----------------+

```

OS ပေါ်မူတည်ပြီး style ပြောင်းနိုင်တယ်။

---

Windows:

```
[ OK Button ]

```

Linux:

```
( OK Button )

```

macOS:

```
rounded button

```

---

Look & Feel က control လုပ်တာ:

- Button style
    
- TextField style
    
- Menu style
    
- Window decoration
    
- Color
    

---

# 16.3 Default Swing Look & Feel

Java မှာ default Look & Feel တွေရှိတယ်။

ဥပမာ:

## Metal

Old Java style:

```
+-------------+
| Button      |
+-------------+

```

---

## Nimbus

Modern-ish Java style:

```
╭────────╮
│ Button │
╰────────╯

```

---

မင်း code မှာ:

```java
for (
 javax.swing.UIManager.LookAndFeelInfo info :
 javax.swing.UIManager.getInstalledLookAndFeels()
)
```

ရှိတယ်။

---

ဒါက Installed Look & Feel list ကို loop လုပ်တာ။

---

Example:

```text
Installed:

Metal
Nimbus
Windows
GTK

```

---

# 16.4 Nimbus Look & Feel Setup

Code:

```java
if ("Nimbus".equals(info.getName())) {

    UIManager.setLookAndFeel(
        info.getClassName()
    );

    break;
}
```

---

Meaning:

"System ထဲမှာ Nimbus ရှိရင် သုံးမယ်"

---

Flow:

```text
Search Look & Feel

        |
        v

Found Nimbus?

        |
        |
       Yes

        |
        v

Apply Nimbus

```

---

ဒါပေမယ့် ဒီ project မှာ နောက်ထပ် FlatLaf သုံးထားတယ်။

---

# 16.5 FlatLaf Introduction

ဒီ project ရဲ့ main UI style က:

```java
FlatLightLaf
```

ဖြစ်တယ်။

Library:

```
com.formdev.flatlaf
```

---

FlatLaf ဆိုတာ:

> Modern Flat Design Swing Look & Feel Library

ဖြစ်တယ်။

---

Default Swing:

```
Old Java Application

```

FlatLaf:

```
Modern Desktop Application

```

---

Features:

- Modern buttons
    
- Rounded components
    
- Better colors
    
- Dark theme support
    
- Better scaling
    

---

# 16.6 FlatLightLaf Setup

Code:

```java
FlatLightLaf.setup();
```

---

ဒီ line က:

Application တစ်ခုလုံးရဲ့ theme ကို Light mode ပြောင်းတယ်။

---

Before:

```
Default Swing

Button:
[Button]

```

---

After:

```
FlatLaf

╭─────────╮
│ Button  │
╰─────────╯

```

---

# 16.7 FlatDarkLaf

Comment:

```java
//FlatDarkLaf.setup();
```

---

ဒါကို uncomment လုပ်ရင်:

```java
FlatDarkLaf.setup();
```

ဖြစ်မယ်။

---

Result:

Dark UI:

```
████████████

  QR Scanner

████████████

```

---

Security application တွေအတွက် dark theme သုံးတာများတယ်။

ဥပမာ:

- Scanner
    
- Monitoring System
    
- Developer Tools
    

---

# 16.8 UIManager Configuration

Code:

```java
UIManager.put(
"TitlePane.unifiedBackground",
true
);
```

---

UIManager ဆိုတာ:

Swing UI settings ကို control လုပ်တဲ့ class။

---

ဒီ setting က Window Title Bar ကို FlatLaf style နဲ့ ပေါင်းပေးတယ်။

---

Before:

```
+----------------------+
| Java JFrame Title    |
+----------------------+

| Application          |

```

---

After:

```
╭──────────────────────╮
│ QR Scanner           │
│                      │
│ Application          │
╰──────────────────────╯

```

---

# 16.9 Exception Handling

Code:

```java
try {

    FlatLightLaf.setup();

}
catch(Exception ex){

    System.err.println(
    "Failed to initialize FlatLaf"
    );

}
```

---

ဘာကြောင့် try-catch လိုလဲ?

Library မရှိရင်:

```
ClassNotFoundException

```

ဖြစ်နိုင်တယ်။

---

Application ကို crash မဖြစ်စေချင်လို့ catch လုပ်ထားတယ်။

---

# 16.10 Swing Event Dispatch Thread (EDT)

အရေးကြီးဆုံးအပိုင်း:

```java
java.awt.EventQueue.invokeLater(
() -> new QRScanner().setVisible(true)
);
```

---

ဒီ line ကို beginner တွေ အများကြီးမသိကြဘူး။

Swing မှာ Thread ၃ မျိုးလို စဉ်းစားလို့ရတယ်။

---

## Main Thread

```text
main()

```

---

## Event Dispatch Thread (EDT)

```text
Button Click
Mouse Event
Painting

```

---

## Background Thread

မင်း project မှာ:

```text
Webcam Thread

ExecutorService

```

---

Architecture:

```
             JVM

              |
              |

          main thread

              |
              v

        EventQueue

              |
              v

             EDT

              |
       +------+------+

       UI       Events


Background:

Executor

Webcam

```

---

# 16.11 Why invokeLater?

Wrong:

```java
public static void main(String[] args){

    new QRScanner()
    .setVisible(true);

}
```

---

ဒါက အလုပ်လုပ်နိုင်တယ်။

ဒါပေမယ့် professional မဟုတ်ဘူး။

---

Correct:

```java
EventQueue.invokeLater(
() -> {

    new QRScanner()
    .setVisible(true);

}
);

```

---

Meaning:

> UI creation ကို Swing UI Thread မှာ run လုပ်ပါ

---

Benefits:

- Thread safety
    
- Smooth rendering
    
- Prevent UI freeze
    

---

# 16.12 Application Startup Complete Flow

အခု run လိုက်တဲ့အခါ:

```
User Runs Program

        |
        v

JVM

        |
        v

main()

        |
        v

Load Look & Feel

        |
        v

FlatLightLaf.setup()

        |
        v

Configure UIManager

        |
        v

invokeLater()

        |
        v

EDT Starts

        |
        v

new QRScanner()

        |
        v

Constructor

        |
        +----------------+
        |                |
        v                v

 initComponents()   initWebcamScanner()

        |
        v

Window Visible

```

---

# 16.13 Relationship With Constructor

ဒီ line:

```java
new QRScanner()
```

ခေါ်လိုက်ရင်:

```java
public QRScanner(){

    initComponents();

    initWebcamScanner();

}
```

run မယ်။

---

ဒါကြောင့်:

```
main()

 |
 v

Constructor

 |
 +----------------+
 |
 init UI
 |
 start Camera

```

---

# 16.14 Why FlatLaf Before JFrame Creation?

အရေးကြီးတယ်။

Correct:

```java
FlatLightLaf.setup();

new QRScanner();

```

---

Wrong:

```java
new QRScanner();

FlatLightLaf.setup();

```

---

ဘာဖြစ်မလဲ?

Frame create ပြီးမှ theme ပြောင်းရင်:

- Component တချို့ update မဖြစ်နိုင်
    
- Style မညီနိုင်
    

---

Rule:

```
Set Theme

     ↓

Create UI

```

---

# 16.15 Production Startup Pattern

Professional Swing Application:

```java
public static void main(String[] args){

    setupLookAndFeel();

    EventQueue.invokeLater(() -> {

        createMainWindow();

    });

}

```

---

ဒီလိုခွဲရေးလို့ရတယ်။

---

Example:

```java
private static void setupLookAndFeel(){

    FlatLightLaf.setup();

}


private static void createMainWindow(){

    JFrame frame =
    new QRScanner();

    frame.setVisible(true);

}

```

---

ပို maintainable ဖြစ်တယ်။

---

# 16.16 Chapter 16 Summary

`main()` ရဲ့ responsibilities:

|Code|Purpose|
|---|---|
|`UIManager`|Swing appearance control|
|Nimbus setup|Default theme|
|`FlatLightLaf.setup()`|Modern UI theme|
|`TitlePane.unifiedBackground`|Window styling|
|`invokeLater()`|Start UI safely|
|`new QRScanner()`|Create application window|

---

# Complete Application Startup Architecture

```
                 Application Start


                       |
                       v

                    main()


                       |
                       v

              Configure Look & Feel


                       |
                       v

                  FlatLaf


                       |
                       v

              Swing Event Thread


                       |
                       v

                 QRScanner JFrame


                       |
          +------------+-------------+

          |                          |

     Initialize UI              Start Webcam


          |                          |

          +------------+-------------+

                       |

                  User Interaction

```

---

## Senior Developer View

ဒီ `main()` method က သေးသေးလေးပေမယ့် ဒီ Concepts တွေပါဝင်တယ်:

```
✔ Application Bootstrap
✔ UI Thread Management
✔ Theme Initialization
✔ Swing Lifecycle
✔ Dependency Initialization
✔ Desktop Application Startup Pattern
```

---

ဒီအထိဆိုရင် `QRScanner.java` တစ်ခုလုံးကို complete breakdown ပြီးပါပြီ။

