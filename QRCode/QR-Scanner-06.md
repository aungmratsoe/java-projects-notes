

အခုရောက်လာတဲ့ **Chapter 6 — ExecutorService** က Java Concurrency ရဲ့ အရေးကြီးဆုံး Topic တွေထဲက တစ်ခုပါ။

ဒီ Chapter ကို နားလည်သွားရင်

- Thread ဆိုတာဘာလဲ
    
- ဘာကြောင့် ExecutorService သုံးတာလဲ
    
- Background Processing
    
- UI Thread
    
- Thread Pool
    
- Runnable
    
- Lambda Thread
    

အားလုံးကို နားလည်သွားမယ်။

---

# Chapter 6 Overview

ဒီ Code ကို ကြည့်ရအောင်။

```java
private final ExecutorService executor =
        Executors.newSingleThreadExecutor();
```

နောက်ပိုင်း

```java
executor.execute(() -> {

    // Webcam Code

});
```

ဒီနှစ်ပိုင်းလုံးက

**Background Thread** ပေါ်မှာ Webcam ကို Run စေတာပါ။

---

# Thread ဆိုတာ ဘာလဲ?

Program တစ်ခု Run ရင်

အနည်းဆုံး

Thread (၁) ခု ရှိတယ်။

Diagram

```text
Java Program

        │

        ▼

Main Thread
```

---

ဥပမာ

```java
public static void main(String[] args){

    System.out.println("Hello");

}
```

ဒီ Program မှာ

Thread (၁) ခုပဲရှိတယ်။

---

ဒါကို

Main Thread

လို့ခေါ်တယ်။

---

# Swing မှာ Thread ဘယ်နှစ်ခုရှိလဲ?

Swing Program မှာ

အဓိက Thread (၂) ခုရှိတယ်။

```text
Application

     │

 ┌───┴─────────┐

 ▼             ▼

Main        EDT

Thread      Thread
```

EDT

=

Event Dispatch Thread

---

ဒီ Thread က

GUI ကို

ဆွဲတယ်။

ဥပမာ

```text
Draw JLabel

Draw Button

Mouse Click

Keyboard

Animation
```

အားလုံးကို

EDT

လုပ်တယ်။

---

# Problem

Webcam က

ဒီလိုလုပ်တယ်။

```java
while(true){

    webcam.getImage();

}
```

ဒီ Code ကို

EDT

ပေါ်မှာ Run ရင်

ဘာဖြစ်မလဲ?

---

Diagram

```text
EDT

↓

Open Camera

↓

Read Image

↓

Read Image

↓

Read Image

↓

Read Image
```

EDT

က

Busy

ဖြစ်သွားတယ်။

---

GUI

ဘာဖြစ်မလဲ?

```text
Window

↓

Freeze

↓

Button

မနှိပ်ရ

↓

Mouse

မရွှေ့ရ
```

Application

Hang

ဖြစ်သွားမယ်။

---

# Solution

ဒါကြောင့်

Background Thread

သုံးရတယ်။

Diagram

```text
EDT

↓

GUI


Background Thread

↓

Webcam

↓

QR Decode
```

အလုပ်ခွဲလိုက်တယ်။

---

# ExecutorService

ဒီလိုမြင်လိုက်ရင်

```java
ExecutorService
```

Thread Manager

ဖြစ်တယ်။

သူက

```text
Create Thread

Reuse Thread

Shutdown Thread
```

အားလုံးလုပ်ပေးတယ်။

---

Diagram

```text
Program

      │

      ▼

ExecutorService

      │

      ▼

Worker Thread
```

---

# Executors

ဒီလိုရေးထားတယ်။

```java
Executors.newSingleThreadExecutor();
```

Executors

ဆိုတာ

Factory Class

ဖြစ်တယ်။

သူက

Executor

Create

လုပ်ပေးတယ်။

---

Diagram

```text
Executors

↓

Create Executor

↓

Return ExecutorService
```

---

# newSingleThreadExecutor()

ဒီ Method က

Thread (၁) ခုပဲ

ဖန်တီးတယ်။

Diagram

```text
Executor

       │

       ▼

Thread #1
```

---

ဒါကြောင့်

Webcam

တစ်ခုတည်း

Run

မယ်။

---

# ဘာကြောင့် Single Thread?

Camera

တစ်လုံးပဲရှိတယ်။

Thread

၁၀ ခု

မလိုဘူး။

---

ဥပမာ

```text
Camera

↓

Thread 1

OK
```

---

ဒီလိုဆို

```text
Camera

↓

Thread1

↓

Thread2

↓

Thread3
```

Camera ကို

Thread

၃ ခုက

တစ်ပြိုင်တည်း

ဖတ်မယ်။

Race Condition

ဖြစ်နိုင်တယ်။

---

ဒါကြောင့်

Single Thread

ရွေးထားတာ။

---

# final

```java
private final ExecutorService executor
```

ဘာကိုဆိုလိုတာလဲ?

Executor Object

ကို

ပြန်မပြောင်းနိုင်ဘူး။

```java
executor =
Executors.newCachedThreadPool();
```

❌

မရ။

---

ဒါပေမယ့်

```java
executor.execute(...)
```

✔

ရတယ်။

---

# execute()

အရေးကြီးဆုံး။

```java
executor.execute(() -> {

});
```

ဒီလိုခေါ်လိုက်တာနဲ့

Background Thread

ပေါ်မှာ

Run

သွားတယ်။

---

Flow

```text
Main Thread

↓

executor.execute()

↓

Worker Thread

↓

Run Code
```

---

# Lambda

ဒီလိုရေးထားတယ်။

```java
() -> {

}
```

တကယ်က

ဒီလိုပဲ။

```java
Runnable task = new Runnable(){

    @Override

    public void run(){

    }

};
```

Java 8

Lambda

သုံးထားတာ။

---

Diagram

```text
Runnable

↓

Lambda

↓

() -> {}
```

---

# Thread Lifecycle

ဒီလိုဖြစ်တယ်။

```text
Executor

↓

Create Thread

↓

Run Task

↓

Wait

↓

Run Again

↓

Shutdown
```

Thread ကို

အမြဲ

အသစ်မဆောက်ဘူး။

Reuse

လုပ်တယ်။

---

# Webcam Thread

သင့် Code ထဲမှာ

ဒီလိုရှိတယ်။

```java
executor.execute(() -> {

    try {

        webcam = Webcam.getDefault();

        webcam.open();

        while(isRunning){

            BufferedImage image =
                    webcam.getImage();

        }

    }

});
```

Flow

```text
Executor

↓

Worker Thread

↓

Open Webcam

↓

Loop

↓

Read Frame

↓

Decode

↓

Repeat
```

---

# Background Loop

```java
while(isRunning)
```

ဒီ Loop က

Background Thread

ပေါ်မှာ

အမြဲ Run

နေတယ်။

Diagram

```text
Thread

↓

Frame1

↓

Frame2

↓

Frame3

↓

Frame4
```

---

# GUI Update

ဒီ Code ကို

သတိထားပါ။

```java
SwingUtilities.invokeLater(() -> {

    lblCam.setIcon(...);

});
```

ဘာကြောင့်

တိုက်ရိုက်

```java
lblCam.setIcon(...)
```

မလုပ်တာလဲ?

---

ဘာကြောင့်ဆိုတော့

Background Thread က

Swing Component

ကို

တိုက်ရိုက်

မပြင်သင့်ဘူး။

Diagram

```text
Background Thread

↓

Image Ready

↓

invokeLater()

↓

EDT

↓

Update JLabel
```

ဒါကို

Thread-safe GUI Update

လို့ခေါ်တယ်။

---

# Executor vs Thread

Beginner

ရေးတတ်တာ

```java
new Thread(() -> {

}).start();
```

ဒါလည်း

ရတယ်။

---

ဒါပေမယ့်

Professional

ရေးတာ

```java
executor.execute(() -> {

});
```

ဘာကြောင့်?

|new Thread()|ExecutorService|
|---|---|
|Thread အသစ်အမြဲဆောက်တယ်|Thread ကို Reuse လုပ်တယ်|
|Performance နည်းနိုင်|Performance ပိုကောင်း|
|Thread Management ကို ကိုယ်တိုင်လုပ်ရ|Executor က စီမံပေးတယ်|
|Project ကြီးလာရင် ခက်|Enterprise Project တွေမှာ အသုံးများ|

---

# shutdown()

Project ထဲမှာ

```java
executor.shutdown();
```

ရှိတယ်။

ဘာကြောင့်?

Thread

ကို

မပိတ်ရင်

```text
Window Close

↓

Thread Running

↓

CPU Usage

↓

Memory Leak
```

ဖြစ်နိုင်တယ်။

---

ဒါကြောင့်

Program

ပိတ်ရင်

Executor

ကို

Shutdown

လုပ်ရတယ်။

---

# Executor Timeline

```text
Program Start

      │

      ▼

Create Executor

      │

      ▼

Create Worker Thread

      │

      ▼

execute()

      │

      ▼

Open Webcam

      │

      ▼

while(isRunning)

      │

      ▼

Capture Frame

      │

      ▼

Decode QR

      │

      ▼

Update GUI

      │

      ▼

Repeat

      │

      ▼

Window Close

      │

      ▼

shutdown()

      │

      ▼

Thread End
```

---

# ဘာကြောင့် ExecutorService က ဒီ Project အတွက် သင့်တော်တာလဲ?

ဒီ Project မှာ

- Webcam က အဆက်မပြတ် Frame ဖမ်းနေတယ်။
    
- QR Decode က CPU အလုပ်ရှိတယ်။
    
- GUI က အမြဲ Responsive ဖြစ်နေရမယ်။
    

ဒါကြောင့်

```text
EDT (GUI)
     │
     ├── Button Click
     ├── Paint Screen
     └── Animation

Background Thread
     │
     ├── Open Webcam
     ├── Capture Frame
     ├── Decode QR
     └── Sleep 40ms
```

ဆိုပြီး အလုပ်ခွဲထားတာက Professional Design ဖြစ်ပါတယ်။

---

# Senior Developer Notes

ဒီ Code မှာ ကောင်းတဲ့အချက်တွေက

```java
private final ExecutorService executor =
        Executors.newSingleThreadExecutor();
```

- ✅ Webcam အတွက် Thread တစ်ခုတည်း သုံးထားတယ်။
    
- ✅ GUI Thread နဲ့ Webcam Thread ကို ခွဲထားတယ်။
    
- ✅ `SwingUtilities.invokeLater()` နဲ့ GUI Update ကို EDT ပေါ်မှာပဲ လုပ်ထားတယ်။
    
- ✅ `shutdown()` နဲ့ Resource Cleanup လုပ်ထားတယ်။
    
- ✅ `isRunning` Flag နဲ့ Thread ကို Graceful Stop လုပ်ထားတယ်။
    

---

# Chapter Summary

ဒီ Chapter မှာ သင်လေ့လာခဲ့တာတွေက

- ✅ **Thread** ဆိုတာ Program အတွင်းက အလုပ်လုပ်တဲ့ လမ်းကြောင်း (execution path) ဖြစ်တယ်။
    
- ✅ **Swing EDT** က GUI ကို ကိုင်တွယ်ပြီး Heavy Task မလုပ်သင့်ဘူး။
    
- ✅ **ExecutorService** က Thread Management ကို Professional ပုံစံနဲ့ လုပ်ပေးတယ်။
    
- ✅ **`newSingleThreadExecutor()`** က Worker Thread တစ်ခုပဲ ဖန်တီးပြီး Webcam Task အတွက် သင့်တော်တယ်။
    
- ✅ **`execute()`** က Task ကို Background Thread ပေါ်မှာ Run စေတယ်။
    
- ✅ **`SwingUtilities.invokeLater()`** နဲ့ Background Thread ကနေ GUI ကို လုံခြုံစွာ Update လုပ်တယ်။
    
- ✅ **`shutdown()`** က Executor ကို သေချာပိတ်ပြီး Resource Leak မဖြစ်အောင် ကာကွယ်ပေးတယ်။
    

ဒီ Concept ကို နားလည်ထားရင် Java Swing တင်မကဘဲ JavaFX, Android (`Executor`, `Handler`, `Looper`), Spring Boot (`@Async`) စတဲ့ Java Ecosystem တစ်လျှောက်မှာ Thread နဲ့ Background Task တွေကို ပိုလွယ်ကူစွာ နားလည်နိုင်ပါလိမ့်မယ်။