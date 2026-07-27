
အခုရောက်လာတဲ့ **Chapter 5 — Timer Animation** က ဒီ Project ရဲ့ **UI Animation Engine** ဖြစ်ပါတယ်။

ဒီ Chapter ကို နားလည်သွားရင်

- Java Swing Animation
    
- Timer
    
- Event Loop
    
- repaint()
    
- paintComponent()
    

အားလုံးကို နားလည်သွားမယ်။

---

# Chapter 5 Overview

ဒီ Animation Code က

```java
animationTimer = new Timer(20, e -> {

    if (lblCam.getHeight() > 0) {

        if (scanGoingDown) {

            scanLineY += 4;

            if (scanLineY >= lblCam.getHeight() - 10) {
                scanGoingDown = false;
            }

        } else {

            scanLineY -= 4;

            if (scanLineY <= 10) {
                scanGoingDown = true;
            }

        }

        lblCam.repaint();
    }

});

animationTimer.start();
```

ဒီ Code က

Laser Scanner ကို

```
↓

↓

↓

↓

↑

↑

↑
```

ဒီလို

အပေါ်အောက် ရွေ့စေတဲ့ Animation Engine ပါ။

---

# Animation Flow

```
Timer

↓

20ms

↓

Update scanLineY

↓

repaint()

↓

paintComponent()

↓

Draw New Laser

↓

20ms

↓

Repeat
```

ဒီ Process က

အမြဲတမ်း Loop လုပ်နေတယ်။

---

# Timer ဆိုတာ ဘာလဲ?

ဒီလိုမြင်လိုက်ရင်

```java
Timer
```

အချိန်တိုင်း Event တစ်ခု Run ပေးတဲ့ Object ဖြစ်တယ်။

ဥပမာ

```
Every 1 Second

↓

Print Hello
```

```java
Timer timer =
new Timer(1000, e -> {

    System.out.println("Hello");

});
```

Output

```
Hello

Hello

Hello

Hello
```

1 Second တိုင်း Run မယ်။

---

ဒီ Project မှာတော့

```java
new Timer(20,...)
```

20 milliseconds တိုင်း Run တယ်။

---

# 20 ဆိုတာဘာလဲ?

```java
Timer(20,...)
```

20 ms

=

0.02 Second

---

1 Second မှာ

```
1000 / 20

=

50
```

Timer Event

**50 ကြိမ် Run တယ်။**

---

ဒါကြောင့်

Animation က

Smooth ဖြစ်တယ်။

---

# Constructor

```java
animationTimer =
new Timer(
```

ဒီ Line က

Timer Object

အသစ်ဆောက်တာ။

Memory

```
animationTimer

↓

Timer Object
```

---

# Parameter (1)

```java
20
```

Delay

Milliseconds

---

ဥပမာ

```
1000

↓

1 FPS
```

```
500

↓

2 FPS
```

```
100

↓

10 FPS
```

```
20

↓

50 FPS
```

FPS

=

Frames Per Second

---

# Parameter (2)

```java
e -> {
```

Lambda Expression

တကယ်က

ဒီလိုပဲ။

```java
new ActionListener(){

    @Override

    public void actionPerformed(
        ActionEvent e){

    }

}
```

Java 8 ကနေ

Lambda ရေးလို့ရတယ်။

```
ActionListener

↓

Lambda

↓

e -> {}
```

---

# Timer Tick

20ms တိုင်း

ဒီ Block ကို Run မယ်။

```
Timer Tick

↓

Code Run

↓

20ms

↓

Code Run

↓

20ms

↓

Code Run
```

---

# First Condition

```java
if(lblCam.getHeight()>0)
```

ဘာကြောင့် စစ်တာလဲ?

---

Program စစချင်း

Label က

မပေါ်သေးဘူး။

```
lblCam

Height

↓

0
```

ဖြစ်နေတတ်တယ်။

---

ဒီအချိန်

Animation မလုပ်သေးဘူး။

---

နောက်ပိုင်း

Window ပေါ်လာရင်

```
lblCam

Height

↓

264
```

ဖြစ်သွားတယ်။

---

# Main Logic

ဒီ Code

```java
if(scanGoingDown)
```

ဆိုတာ

Direction ကို စစ်တာ။

---

Memory

```
scanGoingDown

↓

true
```

ဆိုရင်

```
↓

↓

↓

```

အောက်ဆင်း။

---

```
false
```

ဆိုရင်

```
↑

↑

↑
```

အပေါ်တက်။

---

# Going Down

```java
scanLineY += 4;
```

ဒီ Line က

Laser Position ကို

4 Pixel

အောက်ရွှေ့တာ။

---

Memory

```
scanLineY

0

↓

4

↓

8

↓

12

↓

16

↓

20
```

---

ဘာကြောင့် 4?

1 Pixel ဆိုရင်

```
0

1

2

3
```

နှေးလွန်းတယ်။

---

10 Pixel

ဆိုရင်

```
0

10

20

30
```

မြန်လွန်းတယ်။

---

4 Pixel

က

Smooth

ဖြစ်တယ်။

---

# Bottom Detection

```java
if(scanLineY>=
lblCam.getHeight()-10)
```

ဒါက

Bottom ရောက်ပြီလား?

စစ်တာ။

---

ဥပမာ

```
Label Height

=

264
```

---

```
264-10

=

254
```

Laser

254

ရောက်ရင်

Direction

ပြောင်းမယ်။

---

# Direction Change

```java
scanGoingDown=false;
```

Memory

```
scanGoingDown

↓

false
```

ဖြစ်သွားတယ်။

---

ဒါကြောင့်

နောက် Tick က

```
↓

↓

↓

Bottom

↓

Direction Change

↑

↑

↑
```

---

# Going Up

```java
scanLineY-=4;
```

အပေါ်ပြန်တက်တယ်။

---

Memory

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

# Top Detection

```java
if(scanLineY<=10)
```

ဘာကြောင့်

0

မဟုတ်တာလဲ?

---

Laser Thickness

Glow

Border

ရှိတယ်။

0

ရောက်ရင်

Border နဲ့

တိုက်နိုင်တယ်။

---

ဒါကြောင့်

```
10 Pixel

Padding
```

ထားတယ်။

---

# Direction Back

```java
scanGoingDown=true;
```

Memory

```
false

↓

true
```

---

Loop

```
↓

↓

↓

Bottom

↑

↑

↑

Top

↓

↓

↓

Bottom
```

အဆုံးမရှိ လည်နေတယ်။

---

# repaint()

နောက်ဆုံး

```java
lblCam.repaint();
```

ဒီ Line က

အရေးကြီးဆုံး။

---

ဘာလုပ်တာလဲ?

```
Update Variable

↓

Redraw Screen
```

---

ဒါမရေးရင်

Memory ထဲမှာ

```
scanLineY

0

4

8

12
```

ပြောင်းနေမယ်။

Screen မှာတော့

မပြောင်းဘူး။

---

Diagram

```
scanLineY

↓

Changed

↓

repaint()

↓

paintComponent()

↓

Draw Laser

↓

User See Animation
```

---

# repaint() အလုပ်လုပ်ပုံ

```
Timer

↓

repaint()

↓

Swing Event Queue

↓

paintComponent()

↓

Graphics Draw
```

**`repaint()` က တိုက်ရိုက် မဆွဲဘူး။**

သူက Swing ကို

> "ဒီ Component ကို ပြန်ဆွဲပေးပါ"

လို့ Request ပို့တာ။

ပြီးမှ Swing က သင့်ရဲ့

```java
paintComponent(Graphics g)
```

ကို ခေါ်ပေးတယ်။

---

# repaint() ဘာကြောင့် သုံးတာလဲ?

သင် Project ထဲမှာ

```java
protected void paintComponent(Graphics g)
```

ရေးထားတယ်။

အဲ့ဒီ Method ထဲမှာ

```java
int laserY =
Math.max(...scanLineY...)
```

သုံးထားတယ်။

ဆိုတော့

```
scanLineY

↓

New Value

↓

paintComponent()

↓

New Laser Position
```

ဖြစ်သွားတယ်။

---

# Animation Timeline

```
Program Start

↓

Timer Start

↓

20ms

↓

scanLineY = 4

↓

repaint()

↓

paintComponent()

↓

Laser Draw

↓

20ms

↓

scanLineY = 8

↓

repaint()

↓

paintComponent()

↓

Laser Draw

↓

20ms

↓

scanLineY = 12

↓

Repeat Forever
```

---

# Timer.stop()

Project ထဲမှာ

```java
animationTimer.stop();
```

လည်းရှိတယ်။

ဘာကြောင့်?

Window ပိတ်ပြီး

Timer မရပ်ရင်

```
Program Closed

↓

Timer Still Running

↓

CPU Usage

↓

Memory Leak
```

ဖြစ်နိုင်တယ်။

ဒါကြောင့်

Resource Cleanup လုပ်ရတယ်။

---

# Why Swing Timer?

Java မှာ Timer အမျိုးအစား ၂ မျိုး အများဆုံးတွေ့ရတယ်။

|Swing Timer (`javax.swing.Timer`)|`java.util.Timer`|
|---|---|
|GUI Animation အတွက် သင့်တော်|Background Task အတွက် သင့်တော်|
|Event Dispatch Thread (EDT) ပေါ်မှာ Run တယ်|သီးခြား Thread ပေါ်မှာ Run တယ်|
|`repaint()`၊ `JLabel` Update လုပ်တာ လုံခြုံ|Swing Component ကို တိုက်ရိုက် Update မလုပ်သင့်|

ဒီ Project မှာ GUI Animation ဖြစ်လို့ **Swing Timer** ကို ရွေးထားတာ မှန်ပါတယ်။

---

# Chapter Summary

ဒီ Chapter မှာ သင်လေ့လာခဲ့တာတွေက

- ✅ `Timer` က သတ်မှတ်ထားတဲ့ အချိန်အတိုင်း Event ကို ထပ်ခါတလဲလဲ Run ပေးတယ်။
    
- ✅ `20ms` Delay ဆိုတာ တစ်စက္ကန့်ကို အကြမ်းဖျင်း **50 FPS** Animation ရစေတယ်။
    
- ✅ `scanLineY` က Laser ရဲ့ လက်ရှိ Position ကို သိမ်းထားတယ်။
    
- ✅ `scanGoingDown` က Direction (အပေါ်/အောက်) ကို ထိန်းတယ်။
    
- ✅ `repaint()` က Screen ကို ပြန်ဆွဲဖို့ Swing ကို Request ပို့တယ်။
    
- ✅ အမှန်တကယ် Drawing လုပ်တာက `paintComponent()` ထဲမှာ ဖြစ်တယ်။
    
- ✅ Window ပိတ်တဲ့အချိန် `animationTimer.stop()` လုပ်တာက Resource Leak မဖြစ်အောင် ကာကွယ်ပေးတယ်။
    

ဒီ Chapter ကို နားလည်ထားရင် Java Swing မှာ Loading Animation, Progress Animation, Game Loop, Scanner Effect, Dashboard Animation စတာတွေကို ကိုယ်တိုင် ရေးနိုင်မယ့် အခြေခံကို ရရှိသွားပါပြီ။