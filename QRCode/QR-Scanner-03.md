Excellent! အခုရောက်လာတဲ့ **Chapter 3 — Constructor** က OOP ရဲ့ အခြေခံအကျဆုံး Concept တစ်ခုဖြစ်ပြီး ဒီ `QRScanner` Class ရဲ့ **Startup Sequence** ကို ထိန်းချုပ်တဲ့နေရာပါ။

---

# Constructor ဆိုတာ ဘာလဲ?

Constructor ဆိုတာ

> **Object အသစ်တစ်ခု ဖန်တီးတဲ့အချိန်မှာ အလိုအလျောက် အလုပ်လုပ်တဲ့ Method** ဖြစ်ပါတယ်။

ဥပမာ

```java
QRScanner scanner = new QRScanner();
```

ဒီလိုရေးလိုက်တာနဲ့

```java
public QRScanner() {
    ...
}
```

ဒီ Constructor က **အလိုအလျောက် Run** သွားပါတယ်။

---

# Constructor ရဲ့ Syntax

```java
public QRScanner() {
```

ဒီ Line ကိုခွဲကြည့်ရအောင်။

```java
public
```

Class အပြင်ကနေ ခေါ်လို့ရတယ်။

---

```java
QRScanner
```

ဒါက Constructor Name ဖြစ်တယ်။

**Constructor Name က Class Name နဲ့ တူရမယ်။**

Class

```java
public class QRScanner
```

Constructor

```java
public QRScanner()
```

နာမည်တူရမယ်။

---

## ဘာကြောင့် return type မရှိတာလဲ?

Method ဆိုရင်

```java
public void openCamera()
```

ရှိတယ်။

ဒါပေမယ့် Constructor မှာ

```java
public QRScanner()
```

ပဲရှိတယ်။

**void မရှိဘူး။**

ဘာကြောင့်?

Constructor က Value ပြန်ပေးဖို့ မဟုတ်ဘူး။

သူ့အလုပ်က

```text
Object ကို Initialize လုပ်ဖို့
```

ပဲ။

---

# Constructor Flow

ဒီ Constructor ထဲမှာ Code (၃) ခုပဲရှိတယ်။

```java
public QRScanner() {

    initComponents();

    initWebcamScanner();

    addWindowListener(...);

}
```

ဒါပေမယ့် ဒီ (၃) ခုက Project တစ်ခုလုံးကို စတင်ပေးတဲ့ အရေးကြီးဆုံး Code တွေပါ။

---

# Startup Flow Diagram

User က

```java
new QRScanner();
```

ခေါ်လိုက်ရင်

```text
new QRScanner()

        │

        ▼

Constructor

        │

        ▼

initComponents()

        │

        ▼

GUI တည်ဆောက်

        │

        ▼

initWebcamScanner()

        │

        ▼

Camera ဖွင့်

Animation စ

QR Scan စ

        │

        ▼

WindowListener ထည့်

        │

        ▼

Ready
```

ဒါက Program စတဲ့ Flow အစစ်ပါ။

---

# Line 1

```java
initComponents();
```

ဒီ Method ကို

NetBeans GUI Builder က

အလိုအလျောက် Generate လုပ်ထားတာ။

---

## သူဘာလုပ်တာလဲ?

ဒီ Method ထဲမှာ

```java
jPanel1 = new JPanel();

lblCam = new JLabel();

btnHome = new JButton();
```

အစရှိတဲ့

GUI Component အားလုံးကို ဆောက်တယ်။

Diagram

```text
initComponents()

      │

      ▼

Create JFrame

      │

      ▼

Create JPanel

      │

      ▼

Create JLabel

      │

      ▼

Create Buttons

      │

      ▼

Layout

      │

      ▼

Window Ready
```

---

## Memory Diagram

Constructor မခေါ်ခင်

```text
QRScanner

lblCam

↓

null
```

---

`initComponents()`

ပြီးရင်

```text
QRScanner

lblCam

↓

JLabel Object
```

ဖြစ်သွားတယ်။

---

# ဘာကြောင့် initComponents() ကို အရင်ခေါ်တာလဲ?

အရမ်းအရေးကြီးတဲ့ Interview Question ပါ။

ဒီလိုပြောင်းရေးကြည့်။

```java
public QRScanner() {

    initWebcamScanner();

    initComponents();

}
```

---

ဘာဖြစ်မလဲ?

`initWebcamScanner()` ထဲမှာ

```java
lblCam.getHeight()
```

သုံးထားတယ်။

ဒါပေမယ့်

`lblCam`

က

```text
null
```

ဖြစ်နေသေးတယ်။

ဒါဆို

```text
NullPointerException
```

တက်မယ်။

---

ဒါကြောင့်

အစဉ်က

```text
Create GUI

↓

Create Webcam
```

ဖြစ်ရမယ်။

---

# Line 2

```java
initWebcamScanner();
```

GUI ဆောက်ပြီးတာနဲ့

Camera Engine ကို Start လုပ်တယ်။

ဒီ Method ထဲမှာ

- Timer
    
- Thread
    
- Webcam
    
- QR Scanner
    

အားလုံးစတယ်။

---

Flow

```text
initWebcamScanner()

      │

      ▼

Animation Timer

      │

      ▼

Executor Thread

      │

      ▼

Open Webcam

      │

      ▼

Read Frames

      │

      ▼

Scan QR
```

ဒီ Method က Chapter 4 မှာ အသေးစိတ်ရှင်းခဲ့တာပါ။

---

# Line 3

```java
addWindowListener(
```

ဒါက

Window Event ကို နားထောင်တဲ့ Listener။

Java Swing မှာ

Window မှာ Event တွေအများကြီးရှိတယ်။

ဥပမာ

```text
Window Open

Window Close

Window Minimize

Window Restore

Window Activated
```

---

ဒီ Code က

Close Event ကိုပဲ

နားထောင်ထားတယ်။

---

```java
new WindowAdapter()
```

ဘာကြောင့်

Adapter

သုံးတာလဲ?

---

Java မှာ

WindowListener Interface မှာ

Method (၇) ခုလောက်ရှိတယ်။

```java
windowOpened()

windowClosing()

windowClosed()

windowActivated()

windowIconified()

...
```

တကယ်လိုတာက

```java
windowClosing()
```

တစ်ခုပဲ။

ဒါကြောင့်

```java
WindowAdapter
```

သုံးလိုက်တာ။

---

Diagram

```text
WindowAdapter

        │

        ├──────── windowOpened()

        ├──────── windowClosed()

        ├──────── windowClosing()

        ├──────── windowActivated()

        └──────── ...
```

လိုတဲ့ Method ကိုပဲ Override လုပ်တယ်။

---

# Override

```java
@Override
```

ဘာကိုဆိုလိုတာလဲ?

WindowAdapter ထဲက

```java
windowClosing()
```

ကို

ပြန်ရေးတာ။

---

# WindowEvent

```java
windowClosing(WindowEvent e)
```

User က

ဒီ Button ကိုနှိပ်တယ်။

```text
┌───────────────┐

Scanner

           ❌

└───────────────┘
```

အဲ့ဒီ Event ကို

Java က

```java
windowClosing(...)
```

ခေါ်ပေးတယ်။

---

# closeWebcam()

ဒီမှာ

အရေးကြီးဆုံး။

```java
closeWebcam();
```

ဒီ Method က

```text
Stop Timer

↓

Stop Thread

↓

Close Camera

↓

Shutdown Executor
```

အားလုံးလုပ်တယ်။

---

## ဘာကြောင့် Webcam ကိုပိတ်ရတာလဲ?

မပိတ်ရင်

Camera က

OS မှာ

Lock ဖြစ်နေတတ်တယ်။

ဥပမာ

```text
Scanner ပိတ်

↓

Camera LED

↓

On
```

ဖြစ်နိုင်တယ်။

---

နောက်တစ်ခါ

Camera ဖွင့်ရင်

```text
Camera Busy
```

Error တက်နိုင်တယ်။

---

ဒါကြောင့်

Professional Project မှာ

Resource ကို

သေချာ Release လုပ်ရတယ်။

---

# Constructor Timeline

ဒီ Timeline ကိုကြည့်ရင် ပိုရှင်းမယ်။

```text
Program Start

      │

      ▼

new QRScanner()

      │

      ▼

Constructor

      │

      ▼

initComponents()

GUI တည်ဆောက်

      │

      ▼

initWebcamScanner()

Timer Start

Camera Open

QR Scanner Start

      │

      ▼

addWindowListener()

Close Event Register

      │

      ▼

Window Ready

      │

      ▼

User Scan QR

      │

      ▼

User Click ❌

      │

      ▼

windowClosing()

      │

      ▼

closeWebcam()

      │

      ▼

Release Resources
```

---

# Constructor မှာ Method (၃) ခုကို ဘာကြောင့် ခွဲရေးတာလဲ?

တချို့ Beginner တွေက ဒီလိုရေးတတ်တယ်။

```java
public QRScanner() {

    // GUI Code (500 lines)

    // Webcam Code (300 lines)

    // Window Event (100 lines)

}
```

ဒီလိုဆိုရင် Constructor က

**900+ lines** ဖြစ်သွားမယ်။

ဖတ်ရခက်တယ်။

Maintain လုပ်ရခက်တယ်။

ဒါကြောင့် Professional Developer တွေက

```java
public QRScanner() {

    initComponents();

    initWebcamScanner();

    addWindowListener(...);

}
```

လို **Small Methods** ခွဲရေးတယ်။

ဒါကို

> **High Cohesion + Clean Code**

လို့ခေါ်တယ်။

---

# Constructor Summary

|Code|Purpose|
|---|---|
|`initComponents()`|GUI Components အားလုံးကို ဖန်တီးပြီး Layout ချတယ်|
|`initWebcamScanner()`|Timer, Webcam, QR Scanner Engine ကို စတင်တယ်|
|`addWindowListener()`|Window Close Event ကို နားထောင်တယ်|
|`windowClosing()`|User က ❌ နှိပ်တဲ့အချိန် ခေါ်ခံရတယ်|
|`closeWebcam()`|Webcam, Thread, Timer, Executor ကို သေချာပိတ်ပြီး Resource တွေ Release လုပ်တယ်|

---

## 🎯 Senior Developer Review

ဒီ Constructor က **အလွန်သန့်ရှင်းတဲ့ Constructor** ဖြစ်ပါတယ်။

ကောင်းတဲ့အချက်တွေက

- ✅ Constructor ထဲမှာ Business Logic မထည့်ထားဘူး။
    
- ✅ Initialization ကို Method သေးသေးလေးတွေ ခွဲထားတယ်။
    
- ✅ GUI ကို အရင်ဆောက်ပြီးမှ Webcam ကို စတာ မှန်ကန်တဲ့ Initialization Order ဖြစ်တယ်။
    
- ✅ Window ပိတ်တဲ့အချိန် Resource Cleanup ကို မမေ့ထားဘူး။ ဒါက Camera, Thread, Memory Leak တွေကို ကာကွယ်ပေးတယ်။
    

ဒါက Clean Code နဲ့ Object Initialization အတွက် ကောင်းမွန်တဲ့ Pattern တစ်ခုလို့ ပြောနိုင်ပါတယ်။