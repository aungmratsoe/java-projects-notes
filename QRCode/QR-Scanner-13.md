# Chapter 13 — `closeWebcam()`

## Resource Management + Thread Shutdown + Safe Application Closing

Great **aung mrat**.

ဒီ Chapter မှာ `QRScanner.java` ထဲက ဒီ method ကို နားလည်မယ်။

```java
private void closeWebcam() {
    isRunning = false;

    if (animationTimer != null) {
        animationTimer.stop();
    }

    if (webcam != null && webcam.isOpen()) {
        webcam.close();
    }

    executor.shutdown();
}
```

ဒီ method က code အရ သေးသေးလေးပေမယ့် **Professional Java Application** မှာ အရမ်းအရေးကြီးပါတယ်။

အဓိကတာဝန်က:

> Webcam, Animation Timer, Background Thread တွေကို သေချာပိတ်ပြီး Resource Leak မဖြစ်အောင်လုပ်ခြင်း

---

# 13.1 Why do we need closeWebcam()?

QRScanner Window ကို ဖွင့်လိုက်တဲ့အခါ:

```text
QRScanner Window Open

        |
        |
        v

Webcam Start

        |
        |
        v

Executor Thread Running

        |
        |
        v

Animation Timer Running

```

---

User က Home button နှိပ်တယ်ဆိုပါစို့။

```java
btnHomeActionPerformed()
```

ထဲမှာ:

```java
closeWebcam();

JFrame home = new Home();

home.setVisible(true);

this.dispose();
```

လုပ်ထားတယ်။

---

အကယ်၍ `closeWebcam()` မရှိရင်?

Window ပိတ်သွားပေမယ့်:

```
QRScanner Window

      X

      |
      |
      v

Webcam Thread Still Running
Timer Still Running
Camera Still Open

```

ဖြစ်နိုင်တယ်။

---

Result:

- Camera မီး ဆက်လင်းနေမယ်
    
- CPU အသုံးများမယ်
    
- Memory leak ဖြစ်နိုင်တယ်
    
- Program crash ဖြစ်နိုင်တယ်
    

---

# 13.2 Line 1 — Stop Webcam Loop

```java
isRunning = false;
```

ဒီ variable ကို Chapter 7 မှာ တွေ့ခဲ့တယ်။

Webcam loop:

```java
while(isRunning){

    BufferedImage image = webcam.getImage();

    ...

}
```

---

Normally:

```java
isRunning = true;
```

ဖြစ်နေတယ်။

ဒါကြောင့်:

```java
while(true)
```

လိုပဲ loop ဆက် run နေတယ်။

---

Close လုပ်တဲ့အချိန်:

```java
isRunning = false;
```

ဖြစ်သွားမယ်။

---

Loop:

Before:

```
while(true)

Camera
Camera
Camera
Camera

```

After:

```
while(false)

Stop

```

---

ဒါက Thread ကို graceful shutdown လုပ်တဲ့နည်း။

---

# 13.3 Why not use Thread.stop()?

Java မှာ:

```java
thread.stop();
```

ရှိတယ်။

ဒါပေမယ့် မသုံးသင့်ဘူး။

ဘာကြောင့်?

Thread အလယ်မှာ ရပ်သွားရင်:

```
Database Connection Open

        |
        |
        v

Thread Stop

        |
        |
        v

Connection Never Closed

```

ဖြစ်နိုင်တယ်။

---

Professional way:

```
Request Stop

      |

Thread finishes current work

      |

Exit safely

```

---

ဒီ project မှာ:

```java
isRunning=false
```

သုံးထားတာက correct approach ဖြစ်တယ်။

---

# 13.4 Stop Animation Timer

Code:

```java
if (animationTimer != null) {

    animationTimer.stop();

}
```

---

Chapter 5 မှာ:

```java
animationTimer = new Timer(20, e -> {

    scanLineY +=4;

    lblCam.repaint();

});
```

ရှိခဲ့တယ်။

---

Timer အလုပ်:

```
Every 20ms

    |
    v

Change laser position

    |
    v

Repaint Scanner

```

---

Window ပိတ်ပြီး Timer မရပ်ရင်:

```
Timer

 |
 |
 v

repaint()

 |
 |
 v

Invisible JFrame

```

ကို ဆက်ခေါ်နေမယ်။

---

ဒါက unnecessary CPU usage ဖြစ်တယ်။

---

# 13.5 Null Check

```java
if(animationTimer != null)
```

ဘာကြောင့်?

Java မှာ:

```java
Timer timer;

timer.stop();
```

ဆိုရင်:

```
NullPointerException
```

တက်မယ်။

---

Example:

```java
Timer animationTimer = null;

animationTimer.stop();
```

Result:

```
Exception in thread main

NullPointerException

```

---

ဒါကြောင့်:

```java
if(animationTimer != null)
```

စစ်တယ်။

---

# 13.6 Close Webcam Device

Code:

```java
if (webcam != null && webcam.isOpen()) {

    webcam.close();

}
```

ဒီမှာ condition နှစ်ခုရှိတယ်။

---

## Condition 1

```java
webcam != null
```

Camera object ရှိလား?

---

## Condition 2

```java
webcam.isOpen()
```

Camera currently open ဖြစ်လား?

---

Both true ဖြစ်မှ:

```java
webcam.close();
```

လုပ်မယ်။

---

Example:

Before:

```
Webcam

Status:
OPEN

Camera LED:
ON

```

After:

```
Webcam

Status:
CLOSED

Camera LED:
OFF

```

---

# 13.7 Shutdown ExecutorService

Code:

```java
executor.shutdown();
```

ဒီဟာက အရေးကြီးဆုံးပါ။

---

Chapter 6 မှာ:

```java
private final ExecutorService executor =
Executors.newSingleThreadExecutor();
```

ရှိခဲ့တယ်။

---

Webcam code:

```java
executor.execute(() -> {

    while(isRunning){

       ...

    }

});
```

ဆိုတာ:

Background Thread တစ်ခု create လုပ်တယ်။

---

Architecture:

```
Swing EDT Thread

        |
        |
        +------ UI


Executor Thread

        |
        |
        +------ Webcam Reading

```

---

Window ပိတ်ပြီးရင်:

Executor ကိုလည်း ပိတ်ရမယ်။

---

`shutdown()` ဆိုတာ:

> New tasks မလက်ခံတော့ဘူး၊ လက်ရှိ task တွေပြီးရင် stop

---

Flow:

```
shutdown()

    |
    |
    v

No new task

    |
    |
    v

Existing task finish

    |
    |
    v

Thread terminate

```

---

# 13.8 Complete Closing Sequence

User clicks Home:

```
btnHomeActionPerformed()

        |
        v

closeWebcam()

        |
        |
        +----------------+
        |                |
        v                v

isRunning=false    stop Timer


        |
        v

webcam.close()


        |
        v

executor.shutdown()


        |
        v

Open Home JFrame


        |
        v

dispose QRScanner

```

---

# 13.9 Relationship With Constructor

Constructor မှာ:

```java
public QRScanner(){

    initComponents();

    initWebcamScanner();

    addWindowListener(...)

}
```

ရှိတယ်။

---

ဒီအပိုင်း:

```java
addWindowListener(
new WindowAdapter(){

@Override
public void windowClosing(WindowEvent e){

    closeWebcam();

}

});
```

---

ဒါက user က X button နှိပ်တဲ့အခါ:

```
X Button

    |
    v

windowClosing()

    |
    v

closeWebcam()

```

ခေါ်ပေးတယ်။

---

# 13.10 Resource Lifecycle

ဒီ Project မှာ Resource Lifecycle:

## Start

```
QRScanner Open

      |
      v

Webcam.getDefault()

      |
      v

webcam.open()

      |
      v

executor.execute()

      |
      v

Timer.start()

```

---

## End

```
QRScanner Close

      |
      v

Timer.stop()

      |
      v

webcam.close()

      |
      v

executor.shutdown()

```

---

ဒါက Professional lifecycle management ဖြစ်တယ်။

---

# 13.11 What if we forget closeWebcam()?

## Case 1 — Webcam

```
Camera remains open

↓

Other app cannot access camera

↓

Error
```

---

## Case 2 — Executor

```
Background thread alive

↓

Application may not exit

```

---

## Case 3 — Timer

```
Continuous repaint

↓

CPU usage increase

```

---

# 13.12 Better Professional Version

ဒီ code က ကောင်းပါတယ်။

ဒါပေမယ့် production level မှာ:

```java
private void closeWebcam(){

    isRunning=false;


    if(animationTimer != null){

        animationTimer.stop();

    }


    if(webcam != null){

        if(webcam.isOpen()){

            webcam.close();

        }

        webcam = null;

    }


    if(!executor.isShutdown()){

        executor.shutdownNow();

    }

}
```

လိုမျိုးရေးနိုင်တယ်။

---

ထပ်ထည့်ထားတာ:

## webcam = null

Memory reference ဖြုတ်တယ်။

---

## isShutdown()

Already shutdown ဖြစ်နေရင် ထပ်မခေါ်ဘူး။

---

## shutdownNow()

Emergency stop.

---

# Chapter 13 Summary

`closeWebcam()` က:

|Code|Purpose|
|---|---|
|`isRunning=false`|Webcam loop stop|
|`animationTimer.stop()`|Laser animation stop|
|`webcam.close()`|Release camera hardware|
|`executor.shutdown()`|Stop background thread|

---

# Senior Developer View

ဒီ method က Line 10 လောက်ပဲရှိပေမယ့် အောက်က Concept တွေပါဝင်တယ်:

```
Thread Management
+
Resource Cleanup
+
Lifecycle Management
+
Memory Leak Prevention
+
Graceful Shutdown
```

---

အခု QRScanner Class ရဲ့ Lifecycle ပြီးပြီ။

Next Chapter:

# Chapter 14 — `btnHomeActionPerformed()` + JFrame Navigation Architecture

မှာ

```
QRScanner
   |
close resources
   |
dispose()
   |
Home JFrame
```

Swing Application မှာ Screen Navigation ကို Professional ဘယ်လိုလုပ်သင့်လဲ လေ့လာမယ်။