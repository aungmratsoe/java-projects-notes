**Chapter 4 — Webcam Initialization** က သင့် Project တစ်ခုလုံးရဲ့ **Core Engine** ဖြစ်ပါတယ်။

ဒီ Method တစ်ခုတည်းက

- Webcam ဖွင့်တယ်
- Animation စတယ်
- Live Video ပြတယ်
- QR Scan လုပ်တယ်
- QR တွေ့ရင် Verification စတယ်

ဒါကြောင့် Professional Developer တွေက ဒီလို Method ကို **Engine Method** လို့တောင် ခေါ်ကြပါတယ်။

---

# Method တစ်ခုလုံး

```java
private void initWebcamScanner() {

    animationTimer = new Timer(20, e -> {
        ...
    });

    animationTimer.start();

    executor.execute(() -> {

        ...

    });

}
```

ဒီ Method ထဲမှာ

**Engine (၂) ခု** ရှိတယ်။

```
initWebcamScanner()

        │
        │
        ├────────────► Engine #1
        │             Animation Timer
        │
        │
        └────────────► Engine #2
                      Webcam Thread
```

Animation က

Laser Line ကို

အပေါ်အောက် ရွှေ့ပေးတယ်။

Webcam Thread က

Camera ကိုဖွင့်ပြီး

QR ကိုရှာနေတယ်။

Engine နှစ်ခုဟာ

တစ်ချိန်တည်း အလုပ်လုပ်နေတယ်။

---

# ပထမပိုင်း

```
animationTimer = new Timer(20, e -> {
```

ဒီ Timer က

Java Swing Timer ဖြစ်တယ်။

Timer ဆိုတာ

```
Every XX milliseconds

↓

Run code again
```

ဥပမာ

```
Timer(1000)

↓

1 second

↓

Run

↓

1 second

↓

Run

↓

1 second

↓

Run
```

ဒီမှာတော့

```
20 milliseconds
```

တိုင်း Run နေတယ်။

---

## 20 milliseconds ဆိုတာ

1000 ms = 1 second

20 ms

↓

```
1000 / 20 = 50
```

ဆိုတော့

```
50 FPS
```

လောက် Animation Run နေတယ်။

ဒါကြောင့်

Laser က

Smooth ဖြစ်တယ်။

---

# Lambda Expression

```
e -> {
```

ဒီဟာက

Old Java ဆိုရင်

```
new ActionListener(){

    @Override

    public void actionPerformed(ActionEvent e){

    }

}
```

နဲ့တူတယ်။

Java 8 ကစပြီး

ရေးရလွယ်အောင်

Lambda သုံးလာတယ်။

---

# ပထမဆုံး if

```
if (lblCam.getHeight() > 0)
```

ဘာကြောင့် စစ်တာလဲ?

Window မဖွင့်ခင်

JLabel Height က

```
0
```

ဖြစ်နေတတ်တယ်။

ဥပမာ

```
JFrame

Loading...
```

ဒီအချိန်

```
Height = 0
```

ဖြစ်နိုင်တယ်။

ဒါကို

မစစ်ရင်

Laser Position တွေ

မှားသွားမယ်။

---

# scanGoingDown

```
if(scanGoingDown)
```

ဒီ Variable က

```
true

↓

Laser အောက်ဆင်း
```

```
false

↓

Laser အပေါ်တက်
```

---

## Diagram

```
true

↓

████████

────────────

↓

────────────

↓

────────────

↓

Bottom

↓

false

────────────

↑

↑

↑

Top

↓

true
```

ဒီလို Ping Pong Animation ဖြစ်တယ်။

---

# အောက်ဆင်း

```
scanLineY += 4;
```

Laser Position ကို

```
4 pixels
```

တိုးတယ်။

ဥပမာ

```
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

---

## ဘာကြောင့် 4?

1 pixel ဆိုရင်

နှေးလွန်းတယ်။

10 pixel ဆိုရင်

မြန်လွန်းတယ်။

4 က

Smooth ဖြစ်တယ်။

---

# Bottom ရောက်ရင်

```
if(scanLineY >= lblCam.getHeight()-10)
```

ဥပမာ

```
Height

=

260
```

ဆိုရင်

```
250
```

ရောက်ရင်

ပြန်တက်တော့မယ်။

---

```
scanGoingDown = false;
```

Diagram

```
↓

↓

↓

↓

Bottom

↓

false
```

---

# ပြန်တက်

```
scanLineY -= 4;
```

Diagram

```
250

↓

246

↓

242

↓

238
```

---

# Top ရောက်ရင်

```
if(scanLineY<=10)
```

ဆိုရင်

```
scanGoingDown=true;
```

ပြန်အောက်ဆင်းမယ်။

---

ဒါကြောင့်

Laser က

```
↓

↓

↓

↓

↑

↑

↑

↓

↓

↓
```

အမြဲရွေ့နေတယ်။

---

# repaint()

```
lblCam.repaint();
```

ဒီဟာ အရမ်းအရေးကြီးတယ်။

ဘာလုပ်တာလဲ?

```
Old Drawing

↓

Erase

↓

paintComponent()

↓

Draw Again
```

ဆိုတော့

Laser Line က

လှုပ်နေသလို မြင်ရတယ်။

---

# animationTimer.start()

```
animationTimer.start();
```

ဒီ Line မရှိရင်

Timer မစဘူး။

ဥပမာ

```
ကားဝယ်တယ်

↓

Engine Start မလုပ်ဘူး

↓

ကားမသွားဘူး
```

ဒါနဲ့တူတယ်။

---

# ဒုတိယပိုင်း

```
executor.execute(() -> {
```

ဒီဟာက

Project ရဲ့

Main Engine

ဖြစ်တယ်။

---

## ဘာကြောင့် Thread သုံးတာလဲ?

ဒီလိုသာရေးရင်

```
while(true){

    webcam.getImage();

}
```

GUI Thread ထဲမှာ Run မယ်။

```
GUI

↓

Freeze

↓

Window Not Responding
```

ဖြစ်သွားမယ်။

---

ဒါကြောင့်

Background Thread ဖွင့်တယ်။

```
GUI Thread

↓

Window

↓

Responsive
```

တစ်ဖက်မှာ

```
Background Thread

↓

Webcam

↓

QR Scan
```

နှစ်ခုခွဲလုပ်တယ်။

---

# ExecutorService

```
Executors.newSingleThreadExecutor();
```

ဘာဆိုလိုလဲ?

```
Only One Thread
```

ပဲ Run မယ်။

Diagram

```
Executor

↓

Thread #1

↓

Webcam

↓

Decode

↓

Repeat
```

Thread အသစ်တွေ

ထပ်မဖန်တီးတော့ဘူး။

Performance ကောင်းတယ်။

---

# ဘာကြောင့် Thread အသစ်ဖွင့်တာလဲ?

ဒီလို Timeline နဲ့ စဉ်းစားကြည့်ပါ။

```
Main(UI) Thread
───────────────────────────────────────

Window ဖွင့်
↓

Button ဆွဲ
↓

Mouse Click

↓

User Interaction
```

Webcam ကတော့

```
Background Thread
───────────────────────────────────────

Camera ဖွင့်
↓

Frame ဖမ်း
↓

QR Decode
↓

Frame ဖမ်း
↓

QR Decode
↓

Frame ဖမ်း
↓

QR Decode
```

နှစ်ခုကို သီးခြားခွဲထားတဲ့အတွက်

- GUI က မတက်တက်မကျပ်
- Camera က အမြဲ Run နေ
- User က Window ကို ရွှေ့၊ ပိတ်၊ Button နှိပ်လို့ရ

---

# ဒီအပိုင်းရဲ့ Flow

```
initWebcamScanner()

        │
        │
        ├──────────────► Timer (20 ms)
        │                    │
        │                    ▼
        │            scanLineY ပြောင်း
        │                    │
        │                    ▼
        │             repaint()
        │
        │
        └──────────────► Background Thread
                             │
                             ▼
                     Webcam ဖွင့်ဖို့ ပြင်ဆင်
```

---

## ဒီ Lesson မှာ သင်ယူခဲ့တဲ့ Java Concepts

|Code|Concept|ရည်ရွယ်ချက်|
|---|---|---|
|`Timer(20, ...)`|Swing Timer|Animation ကို 20 ms တိုင်း Update လုပ်ရန်|
|`e -> {}`|Lambda Expression|ActionListener ကို တိုတိုရေးရန်|
|`scanLineY += 4`|Animation Logic|Laser Line ကို ရွှေ့ရန်|
|`scanGoingDown`|State Variable|အပေါ်/အောက် Direction ကို ထိန်းရန်|
|`repaint()`|Swing Painting|`paintComponent()` ကို ပြန်ဆွဲခိုင်းရန်|
|`ExecutorService`|Thread Pool|Webcam ကို Background Thread မှာ Run ရန်|
|`newSingleThreadExecutor()`|Single Worker Thread|Webcam Task တစ်ခုတည်းကို လုံခြုံစွာ စီမံရန်|

## Senior Developer Note

ဒီ Method ကိုကြည့်ပြီး Senior Developer တစ်ယောက်က ချီးကျူးမယ့်အချက်က **Animation (`Timer`) နဲ့ Webcam Processing (`ExecutorService`) ကို သီးခြားခွဲထားတာ** ဖြစ်ပါတယ်။ Animation က Swing Event Dispatch Thread (EDT) ပေါ်မှာ အလုပ်လုပ်ပြီး Webcam Capture/QR Decode က Background Thread ပေါ်မှာ အလုပ်လုပ်တာကြောင့် UI မကျပ်ဘဲ Smooth ဖြစ်နေပါတယ်။

**Chapter 5** မှာတော့ `executor.execute()` အတွင်းက

```
webcam = Webcam.getDefault();
...
while (isRunning) {
    ...
}
```

ကို **Line-by-Line** ခွဲပြီး Webcam က Camera ကို ဘယ်လိုရှာတယ်၊ Frame ကို ဘယ်လိုဖမ်းတယ်၊ Live Video ကို JLabel ပေါ် ဘယ်လိုတင်တယ်ဆိုတာကို Memory Diagram နဲ့ အသေးစိတ် ဆက်ရှင်းပြပါမယ်။