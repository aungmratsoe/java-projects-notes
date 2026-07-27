အခုရောက်လာတဲ့ **Chapter 2 — Class Variables** က Java OOP မှာ အရမ်းအရေးကြီးတဲ့အပိုင်းပါ။

Professional Developer တစ်ယောက်က Class ကိုဖတ်တဲ့အခါ

1. Import
    
2. **Class Variables**
    
3. Constructor
    
4. Methods
    

ဒီအစဉ်လိုက်ဖတ်ပါတယ်။

ဘာကြောင့်လဲ?

**Class Variables တွေက ဒီ Class ရဲ့ Memory Map ဖြစ်လို့ပါ။**

Method မဖတ်ခင် Variable တွေကိုကြည့်လိုက်တာနဲ့

> ဒီ Class က ဘာ State တွေကို သိမ်းထားလဲ?

ဆိုတာ သိနိုင်ပါတယ်။

---

# QRScanner Memory Structure

ဒီ Class ရဲ့ Variables တွေကို Group ခွဲလိုက်ရင်

```text
                  QRScanner Object
                         │
 ┌─────────────┬──────────────┬─────────────┬─────────────┐
 │             │              │             │
 ▼             ▼              ▼             ▼
 Webcam     Threading      Database      Animation
 │             │              │             │
 ▼             ▼              ▼             ▼
webcam      executor      studentDAO   scanLineY
isRunning   dialogFlag                  Timer
```

ဒီလိုအုပ်စု (၄) ခုရှိပါတယ်။

---

# Variable (1)

```java
private Webcam webcam = null;
```

ဒီ Variable က

**Camera ကို ကိုယ်စားပြုတဲ့ Object** ဖြစ်ပါတယ်။

Diagram

```text
USB Webcam
      │
      ▼
Webcam Library
      │
      ▼
Webcam Object
      │
      ▼
webcam Variable
```

---

## Memory

QRScanner Object ကိုဖန်တီးလိုက်တဲ့အချိန်

```java
QRScanner scanner = new QRScanner();
```

Memory ထဲမှာ

```text
QRScanner

webcam

↓

null
```

ဖြစ်နေတယ်။

---

နောက်ပိုင်း

```java
webcam = Webcam.getDefault();
```

လုပ်လိုက်ရင်

```text
QRScanner

webcam

↓

USB Camera Object
```

ဖြစ်သွားတယ်။

---

## ဘာကြောင့် null ထားတာလဲ?

Camera မဖွင့်သေးဘူး။

ဒါကြောင့်

```java
private Webcam webcam;
```

လို့ရေးလည်းရတယ်။

Java က

Object Variable တွေကို

Default

```text
null
```

ထားပေးတယ်။

ဒါပေမယ့်

```java
= null;
```

ရေးထားတာက

Developer ကို

"ဒီ Variable က နောက်မှ Initialize လုပ်မယ်"

ဆိုတာ

ရှင်းစေတယ်။

---

# Variable (2)

```java
private final ExecutorService executor =
Executors.newSingleThreadExecutor();
```

ဒါက

**Background Thread Manager**

ဖြစ်တယ်။

---

Diagram

```text
QRScanner

      │

Executor

      │

Thread

      │

Webcam

      │

Decode QR
```

---

## ExecutorService ဆိုတာဘာလဲ?

Thread ကို

ကိုယ်တိုင်

```java
new Thread()
```

လုပ်လို့ရတယ်။

ဒါပေမယ့်

Professional Project တွေမှာ

```text
ExecutorService
```

ကို သုံးတယ်။

ဘာကြောင့်?

သူက

- Thread Create
    
- Thread Reuse
    
- Thread Close
    

အားလုံးလုပ်ပေးတယ်။

---

## final

```java
private final ExecutorService
```

ဆိုတာ

Executor Object ကို

နောက်ထပ်

```java
executor =
Executors.newCachedThreadPool();
```

လို

ပြန်မပြောင်းနိုင်တော့ဘူး။

ဒါပေမယ့်

ဒီလို

```java
executor.execute(...)
```

သုံးလို့ရသေးတယ်။

**အရေးကြီးတဲ့အချက်**

`final` က **Reference မပြောင်းနိုင်တာ** ဖြစ်တယ်။

Object ထဲက State မပြောင်းနိုင်တာ မဟုတ်ဘူး။

ဥပမာ

```java
final List<String> list = new ArrayList<>();

list.add("Aung");   // ✔ ရတယ်

list.add("Mrat");   // ✔ ရတယ်

list = new ArrayList<>();   // ❌ မရဘူး
```

---

# Variable (3)

```java
private volatile boolean isRunning = true;
```

ဒီ Variable က

**Scanner Loop ကို ရပ်ဖို့**

သုံးတာ။

---

Diagram

```text
while(isRunning)

↓

true

↓

Loop

↓

Loop

↓

Loop
```

---

Window ပိတ်လိုက်ရင်

```java
isRunning = false;
```

↓

```text
while(false)

↓

Loop Stop
```

---

## ဘာကြောင့် volatile?

ဒါက

Interview Question တစ်ခုတောင် ဖြစ်တတ်တယ်။

---

Normal boolean

```java
boolean isRunning;
```

ဆိုရင်

Thread တစ်ခုမှာ

```java
isRunning=false;
```

ပြောင်းလိုက်ပေမယ့်

နောက် Thread က

အဟောင်း

```text
true
```

ကို

Cache ထဲက

ဖတ်နေတတ်တယ်။

---

Diagram

```text
CPU Cache

Thread A

↓

true

RAM

↓

false
```

Thread A က

RAM ကို

မဖတ်သေးဘူး။

---

volatile သုံးလိုက်ရင်

```text
Thread

↓

Always Read RAM
```

Memory Synchronization

လုပ်ပေးတယ်။

ဒါကြောင့်

Scanner Thread က

ချက်ချင်း

false

မြင်တယ်။

---

# Variable (4)

```java
private final AtomicBoolean
isDialogShowing =
new AtomicBoolean(false);
```

ဒီ Variable က

**Dialog တစ်ခုပဲ ပြဖို့**

သုံးတာ။

---

Scenario

Camera က

Frame

25 FPS

နဲ့ Run နေတယ်။

QR ကို

Frame ၅၀ ဆက်တိုက်

တွေ့နိုင်တယ်။

---

Diagram

```text
Frame1

QR

↓

Popup

Frame2

QR

↓

Popup

Frame3

QR

↓

Popup
```

Popup

အများကြီး

ပေါ်လာမယ်။

---

ဒါကြောင့်

```java
AtomicBoolean
```

သုံးတယ်။

```text
Dialog Open?

↓

false

↓

Open

↓

true

↓

Next Scan

↓

Ignore
```

---

Dialog ပိတ်ရင်

```java
isDialogShowing.set(false);
```

ပြန်ဖြစ်တယ်။

---

## ဘာကြောင့် boolean မသုံးတာလဲ?

```java
boolean dialog;
```

ဆိုရင်

Thread နှစ်ခု

တစ်ပြိုင်တည်း

ပြောင်းနိုင်တယ်။

AtomicBoolean

က

CPU Level

Atomic Operation

သုံးတယ်။

Thread Safe

ဖြစ်တယ်။

---

# Variable (5)

```java
private final StudentDAOInterface
studentDAO =
new StudentDAO();
```

ဒီ Variable က

Database Access Object

ဖြစ်တယ်။

---

Diagram

```text
QRScanner

↓

studentDAO

↓

MySQL

↓

Student Table
```

---

ဘာကြောင့်

Interface သုံးတာလဲ?

ဒီလိုရေးရင်

```java
StudentDAO studentDAO =
new StudentDAO();
```

MySQL ကို

တင်းတင်းကျပ်ကျပ်

ချိတ်ထားတယ်။

---

Interface သုံးရင်

```text
StudentDAOInterface

↓

StudentDAO

↓

MySQL
```

နောက်ပိုင်း

```text
StudentDAOInterface

↓

MongoStudentDAO
```

ပြောင်းလို့ရတယ်။

ဒါကို

**Programming to an Interface** လို့ခေါ်တယ်။

ဒါက SOLID ရဲ့ **Dependency Inversion Principle (DIP)** နဲ့ ဆက်စပ်တဲ့ Professional Design Pattern တစ်ခုပါ။

---

# Animation Variables

```java
private int scanLineY = 0;
```

Laser Line ရဲ့

Y Position

---

ဥပမာ

```text
0

↓

4

↓

8

↓

12

↓

16
```

ဒီလို

အောက်ဆင်းနေတယ်။

---

```java
private boolean scanGoingDown = true;
```

Direction

---

```text
true

↓

Down
```

```text
false

↓

Up
```

---

Diagram

```text
↓

↓

↓

↓

Bottom

↑

↑

↑

Top
```

---

```java
private Timer animationTimer;
```

Laser Animation ကို

Run မယ့်

Swing Timer

---

Memory

```text
animationTimer

↓

Timer Object

↓

20ms

↓

Tick

↓

Tick

↓

Tick
```

---

# Static Variable

```java
private static final Map<DecodeHintType,Object>
DECODE_HINTS
```

ဒီ Variable က

Instance Variable မဟုတ်ဘူး။

Class Variable

ဖြစ်တယ်။

---

ဘာကြောင့် static?

QRScanner

၁၀ ခု

ဖွင့်ရင်

Hints

၁၀ ခု

မလိုဘူး။

တစ်ခုတည်း

Share လုပ်တယ်။

---

Diagram

```text
QRScanner1

        │

QRScanner2

        │

QRScanner3

        │

──────────────

        │

DECODE_HINTS
```

---

## final

ဒီ Map Reference ကို

ပြန်မပြောင်းနိုင်ဘူး။

```java
DECODE_HINTS = new EnumMap<>();
```

❌ မရဘူး။

ဒါပေမယ့်

```java
DECODE_HINTS.put(...)
```

✔ ရတယ်။

---

# Logger

```java
private static final Logger logger
```

Professional Logging

အတွက်။

---

ဘာကြောင့် static?

Logger က

Class တစ်ခုလုံး

Share လုပ်တယ်။

Instance တိုင်း

အသစ်မဆောက်ဘူး။

---

# Variable Summary

|Variable|Type|Purpose|
|---|---|---|
|`webcam`|Webcam|Camera Object ကို ကိုယ်စားပြုတယ်|
|`executor`|ExecutorService|Background Thread ကို စီမံတယ်|
|`isRunning`|`volatile boolean`|Scanner Loop ကို စ/ရပ် ထိန်းချုပ်တယ်|
|`isDialogShowing`|AtomicBoolean|Dialog တစ်ခုပဲ ပြစေဖို့ Thread-safe Flag|
|`studentDAO`|StudentDAOInterface|Database ထဲက Student Data ကို ရယူတယ်|
|`scanLineY`|int|Laser Line ရဲ့ လက်ရှိ Y Position|
|`scanGoingDown`|boolean|Laser ရဲ့ Direction (အောက်/အပေါ်)|
|`animationTimer`|Timer|Laser Animation ကို အချိန်မှန် Update လုပ်တယ်|
|`DECODE_HINTS`|`static final Map`|ZXing QR Decode Settings ကို Class တစ်ခုလုံး Share လုပ်တယ်|
|`logger`|`static final Logger`|Error နဲ့ Event တွေကို Log လုပ်တယ်|

---

## 🎯 Senior Developer အမြင်

ဒီ Variables တွေကို ကြည့်လိုက်တာနဲ့ ဒီ Class က **State Management** ကို သေသေချာချာ စဉ်းစားထားတာ မြင်ရပါတယ်။

- **Resource State** → `webcam`
    
- **Concurrency State** → `executor`, `volatile`, `AtomicBoolean`
    
- **Business Logic State** → `studentDAO`
    
- **UI State** → `scanLineY`, `scanGoingDown`, `animationTimer`
    
- **Shared Configuration** → `DECODE_HINTS`
    
- **Application Logging** → `logger`
    

ဒါဟာ Professional Java Swing Application တွေမှာ တွေ့ရလေ့ရှိတဲ့ Structure ဖြစ်ပါတယ်။ Variable တစ်ခုချင်းစီမှာ တာဝန်တစ်ခုစီ ရှင်းရှင်းလင်းလင်းရှိနေပြီး Responsibility မရောထွေးတာက Code Maintainability ကို ပိုကောင်းစေပါတယ်။