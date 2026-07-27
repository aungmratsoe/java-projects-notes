

အခုရောက်လာတဲ့ **Chapter 10 — QR Paint Animation (Custom paintComponent + Scanner HUD Design)** က ဒီ Project ရဲ့ **Visual Engineering Layer** ဖြစ်ပါတယ်။

အရင် Chapter တွေမှာ

```
Chapter 4 → Webcam
Chapter 5 → Animation Timer
Chapter 6 → Thread
Chapter 7 → Webcam Loop
Chapter 8 → QR Decode
Chapter 9 → Crypto Verification
```

ပြီးခဲ့ပြီ။

အခု Chapter 10 မှာတော့

**"Scanner UI ကို ဘယ်လို Sci-Fi / Cyberpunk Style ဖြစ်အောင် ဆွဲထားလဲ"**

ကို လေ့လာမယ်။

---

# Chapter 10 Overview

ဒီ Code ရဲ့ အဓိကနေရာက

```java
lblCam = new javax.swing.JLabel(){

    @Override
    protected void paintComponent(Graphics g){

        super.paintComponent(g);

        ...
    }
};
```

ဖြစ်ပါတယ်။

---

Normal JLabel:

```java
JLabel
 |
 |
 Paint Image
```

ဒါပဲလုပ်တယ်။

---

ဒါပေမယ့် ဒီမှာ

JLabel ကို

Custom Component

အဖြစ် ပြောင်းထားတယ်။

---

Architecture:

```
JLabel

    +

paintComponent()

    +

Graphics2D

    ↓

Custom Scanner UI
```

---

# Part 1 — paintComponent() ဆိုတာဘာလဲ?

Java Swing မှာ Component တစ်ခုကို ဆွဲတဲ့အခါ

Swing က ဒီ Method ကို ခေါ်တယ်။

```java
protected void paintComponent(Graphics g)
```

---

Flow:

```
Window Open

      ↓

Swing Paint System

      ↓

paintComponent()

      ↓

Draw UI
```

---

ဥပမာ:

```java
@Override
protected void paintComponent(Graphics g){

    super.paintComponent(g);

    g.drawString(
        "Hello",
        50,
        50
    );

}
```

Result:

```
Hello
```

Screen ပေါ်မှာ ပေါ်လာမယ်။

---

# Part 2 — Why Override JLabel?

ဒီ Project မှာ

Camera Image ကို

JLabel ထဲမှာ ပြထားတယ်။

```java
lblCam.setIcon(
    new ImageIcon(scaledImg)
);
```

ဒါပေမယ့်

Camera Image ပေါ်မှာ

ထပ်ဆွဲချင်တယ်။

ဥပမာ:

```
Camera Image

+ Scanner Frame

+ Laser

+ Text

+ Glow
```

ဒါကြောင့်

JLabel ကို Customize လုပ်ထားတာ။

---

Layer System:

```
paintComponent()

        |
        |
        +----------------+
        |                |
        v                v

Camera Image       Scanner Effect
```

---

# Part 3 — super.paintComponent(g)

Code:

```java
super.paintComponent(g);
```

အရမ်းအရေးကြီးတယ်။

---

ဘာလုပ်တာလဲ?

Parent JLabel ရဲ့ Default Drawing ကို အရင်လုပ်ပေးတယ်။

အဓိကက

```text
Draw Background

↓

Draw Image Icon

↓

Custom Drawing
```

ဖြစ်စေတယ်။

---

မရေးရင်?

```
Old Frame

+

New Frame

=

Ghost Image
```

လိုမျိုး Artifact ဖြစ်နိုင်တယ်။

---

# Part 4 — Graphics2D

Code:

```java
Graphics2D g2 =
(Graphics2D) g.create();
```

---

Graphics ဆိုတာ

Drawing Tool ပါ။

ဥပမာ:

```
Graphics

    |
    +-- drawLine()
    |
    +-- drawRect()
    |
    +-- drawString()
```

---

Graphics2D က ပို Advanced ဖြစ်တယ်။

သူမှာ

- Transparency
    
- Gradient
    
- Anti Aliasing
    
- Stroke Control
    

ရှိတယ်။

---

ဒီ Project မှာ

Cyber UI အတွက် Graphics2D သုံးထားတယ်။

---

# Part 5 — Anti Aliasing

Code:

```java
g2.setRenderingHint(
RenderingHints.KEY_ANTIALIASING,
RenderingHints.VALUE_ANTIALIAS_ON
);
```

---

Anti Aliasing ဆိုတာ

Edge ကို Smooth လုပ်တာ။

---

Without:

```
######
######
######
```

With:

```
  ####
 ######
 ######
  ####
```

လို Smooth ဖြစ်တယ်။

---

# Part 6 — Scanner Box Calculation

Code:

```java
int w = getWidth();
int h = getHeight();
```

Component Size ယူတယ်။

---

ဥပမာ:

```
Width = 242
Height = 264
```

---

ပြီးတော့

```java
int boxSize =
Math.min(w,h)-40;
```

---

ဘာလုပ်လဲ?

Square Scanner Area လုပ်တာ။

---

Example:

```
Width  = 242
Height = 264

min = 242

242 - 40

= 202
```

---

Result:

```
+----------------+
|                |
|   +--------+   |
|   |        |   |
|   | QR BOX |   |
|   |        |   |
|   +--------+   |
|                |
+----------------+
```

---

# Part 7 — Dark Glass Overlay

Code:

```java
g2.setColor(
new Color(10,5,20,150)
);
```

---

Color Constructor:

```
new Color(
 Red,
 Green,
 Blue,
 Alpha
)
```

---

ဒီမှာ

```
Red    = 10
Green  = 5
Blue   = 20
Alpha  = 150
```

---

Alpha ဆိုတာ Transparency။

```
0
|
|
255
```

---

0:

```
Transparent
```

255:

```
Solid
```

---

150 ဆိုတော့

Semi Transparent Dark Layer ဖြစ်တယ်။

---

ပြီးတော့

```java
g2.fillRect()
```

နဲ့

Scanner Box အပြင်ဘက်ကို ဖုံးတယ်။

---

Result:

```
Camera View


+----------------+
| Dark Dark Dark |
|                |
|   QR Area      |
|                |
| Dark Dark Dark |
+----------------+
```

---

ဒါကို

Vignette Effect

လို့ခေါ်တယ်။

---

# Part 8 — Breathing Glow Border

ဒီအပိုင်းက အလန်းဆုံးပါ။

Code:

```java
long time =
System.currentTimeMillis();
```

---

လက်ရှိအချိန်ယူတယ်။

ဥပမာ:

```
100000 ms
```

---

ပြီးတော့

```java
Math.sin()
```

သုံးတယ်။

---

Sin Wave:

```
1

 |
 |      /\
 |     /  \
 |____/    \____

0
```

---

ဒါကြောင့်

Brightness ကို အတက်အကျလုပ်လို့ရတယ်။

---

Code:

```java
float pulse =
(float)(
(Math.sin(time*0.006)+1)/2
);
```

---

Output:

```
0.0

0.5

1.0

0.5

0.0
```

လို ပြောင်းနေမယ်။

---

ဒါကို Alpha ပြောင်းဖို့ သုံးတယ်။

```java
int alpha =
(int)(100+155*pulse);
```

---

Result:

```
Glow

Dim

↓

Bright

↓

Dim

↓

Bright
```

---

ဒါကို

Breathing Animation

လို့ခေါ်တယ်။

---

# Part 9 — Multiple Glow Layers

Code:

```java
for(int i=6;i>=1;i--)
```

---

ဘာလုပ်လဲ?

Border အပြင်မှာ

Layer အများကြီး ဆွဲတယ်။

---

Example:

```
     Glow
   Glow Glow
  +-------+
  | Frame |
  +-------+
```

---

i ကြီးရင်

အပြင်ဆုံး Glow

i သေးရင်

အတွင်း Glow

ဖြစ်တယ်။

---

# Part 10 — Main Scanner Frame

Code:

```java
g2.drawRoundRect(
boxX,
boxY,
boxSize,
boxSize,
16,
16
);
```

---

ဒါက

Scanner Border

ဆွဲတာ။

---

Parameters:

```
x
y
width
height
arcWidth
arcHeight
```

---

Result:

```
+----------+
|          |
|          |
|          |
+----------+
```

---

# Part 11 — HUD Corner Brackets

ဒီအပိုင်းက Sci-Fi Look ပေးတာ။

Code:

```java
g2.drawLine()
```

သုံးထားတယ်။

---

Normal Box:

```
+---------+
|         |
|         |
+---------+
```

---

HUD Style:

```
┌────

     QR


────┘
```

---

ဒီ Design ကို

HUD

(Heads-Up Display)

လို့ခေါ်တယ်။

---

သုံးတဲ့နေရာ:

- Military UI
    
- Space UI
    
- Cyber Security UI
    
- Scanner UI
    

---

# Part 12 — Laser Beam

အဓိက Animation Part။

```java
int laserY =
Math.max(
 boxY+5,
 Math.min(
 scanLineY,
 boxY+boxSize-5
 )
);
```

---

ဒါက Laser ကို Box အတွင်းမှာပဲ ထိန်းထားတာ။

---

ဥပမာ:

```
Box Top

 |
 |
 Laser
 |
 |
Box Bottom
```

---

# Outer Glow

```java
fillRect(
boxX,
laserY-18,
boxSize,
36
);
```

---

Laser အနားမှာ Glow Area ဆွဲတယ်။

---

Visual:

```
~~~~~~~~~~~~
-------------
     LASER
-------------
~~~~~~~~~~~~
```

---

# Gradient Trail

Code:

```java
GradientPaint
```

---

Gradient ဆိုတာ

Color တဖြည်းဖြည်း ပြောင်းတာ။

---

Example:

```
Transparent

      ↓

Light

      ↓

White
```

---

ဒါကြောင့် Laser က

ပို Realistic ဖြစ်တယ်။

---

# Bright Core Line

```java
g2.fillRect(
boxX,
laserY,
boxSize,
2
);
```

---

ဒါက

အလယ်က အဖြူလိုင်း။

---

Final:

```
~~~~~~~~~~~
-----------
~~~~~~~~~~~
```

---

# Part 13 — Status Text

Code:

```java
String txt =
"[ ACTIVE SCANNER // SECURE ]";
```

---

Screen အောက်မှာ

System Status ပြတယ်။

---

ဒီလို:

```
[ ACTIVE SCANNER // SECURE ]
```

---

Cyber Security Scanner Feeling ရစေတယ်။

---

# Part 14 — dispose()

Code:

```java
g2.dispose();
```

---

Graphics Resource Release လုပ်တယ်။

---

မလုပ်ရင်:

```
Graphics Object

↓

Memory

↓

Leak
```

ဖြစ်နိုင်တယ်။

---

# Animation Connection

Chapter 5 မှာ

Timer:

```java
scanLineY +=4;
```

လုပ်ခဲ့တယ်။

---

Chapter 10 မှာ

Paint:

```java
laserY = scanLineY;
```

သုံးတယ်။

---

Connection:

```
Timer

↓

Change scanLineY

↓

repaint()

↓

paintComponent()

↓

Draw Laser New Position
```

---

ဒါကြောင့် Animation ဖြစ်လာတာ။

---

# Complete Rendering Pipeline

ဒီ Project ရဲ့ Drawing System:

```
Camera Thread

       |
       v

JLabel Icon Update

       |
       v

paintComponent()

       |
       +----------------+
       |                |
       v                v

Camera Image      Scanner HUD

                       |
                       |
                       v

               Laser Animation
```

---

# Senior Developer Analysis

ဒီ Code မှာ Professional UI Concept တွေ ပါတယ်။

## 1. Custom Painting

Default Component မသုံးဘဲ

ကိုယ်ပိုင် UI Engine ရေးထားတယ်။

---

## 2. Layer Rendering

```
Layer 1:
Camera

Layer 2:
Dark Overlay

Layer 3:
Scanner Frame

Layer 4:
Glow

Layer 5:
Laser

Layer 6:
Text
```

---

## 3. Time Based Animation

```java
System.currentTimeMillis()
```

သုံးပြီး

Frame Independent Animation လုပ်ထားတယ်။

---

## 4. Graphics Optimization

- create()
    
- dispose()
    
- RenderingHint
    

တွေသုံးထားတာ Professional Style ဖြစ်တယ်။

---

# Chapter 10 Summary

ဒီ Chapter မှာ သင်လေ့လာခဲ့တာတွေက

✅ `paintComponent()` က Custom UI Drawing ရဲ့ အခြေခံဖြစ်တယ်။

✅ JLabel ကို Override လုပ်ပြီး Scanner HUD တစ်ခုဖန်တီးထားတယ်။

✅ `Graphics2D` က Advanced Drawing API ဖြစ်တယ်။

✅ Anti-aliasing က Edge တွေကို Smooth ဖြစ်စေတယ်။

✅ Alpha Transparency နဲ့ Dark Glass Effect ဖန်တီးတယ်။

✅ `Math.sin()` နဲ့ Breathing Glow Animation လုပ်တယ်။

✅ Multiple Border Layer တွေနဲ့ Neon Glow Effect ရတယ်။

✅ Scanner Corner Brackets က HUD Design ဖြစ်တယ်။

✅ Laser Beam က Timer က update လုပ်တဲ့ `scanLineY` ကို အသုံးပြုပြီး Animation ဖြစ်လာတယ်။

---

Great **aung mrat**. ဒီ Chapter 10 ရဲ့ `paintComponent()` ကို အခု **Senior Java Swing Graphics level** နဲ့ line-by-line breakdown လုပ်မယ်။

ဒီ code က ရိုးရိုး drawing မဟုတ်ပါဘူး။ ဒါက **Custom Rendering Pipeline** တစ်ခုပါ။

အဓိက idea:

```
JLabel
 |
 |-- Camera Image (JLabel Icon)
 |
 |-- paintComponent()
       |
       |-- Dark Overlay
       |-- Scanner Frame
       |-- Glow Animation
       |-- HUD Corners
       |-- Laser Animation
       |-- Status Text
```

---

# Chapter 10.1 — Anonymous JLabel Creation

Code:

```java
new javax.swing.JLabel() {

    @Override
    protected void paintComponent(java.awt.Graphics g) {

    }

}
```

ဒီမှာ normal JLabel မဟုတ်တော့ဘူး။

Normally:

```java
JLabel lblCam = new JLabel();
```

ဆိုရင် JLabel က သူ့ default paint ကိုပဲ သုံးတယ်။

ဒါပေမယ့် ဒီမှာ:

```java
new JLabel(){

}
```

နဲ့ JLabel ကို inherit လုပ်ပြီး anonymous subclass တစ်ခု ဖန်တီးထားတယ်။

Equivalent:

```java
class ScannerLabel extends JLabel {

    @Override
    protected void paintComponent(Graphics g){

    }

}
```

လိုမျိုးပဲ။

---

အဓိကရည်ရွယ်ချက်:

Camera image ပေါ်မှာ ကိုယ်ပိုင် effect တွေ ထပ်ဆွဲချင်လို့ပါ။

ဥပမာ:

```
Before:

+----------------+
| Camera Image   |
|                |
+----------------+


After:

+----------------+
| Camera Image   |
|    ┌────┐      |
|    │ QR │      |
|    └────┘      |
|      -----     |
|      Laser     |
+----------------+
```

---

# Chapter 10.2 — paintComponent()

```java
protected void paintComponent(Graphics g)
```

ဒီ method ကို Swing Painting System က အလိုအလျောက် ခေါ်ပါတယ်။

Flow:

```
Window repaint()
        |
        v
Swing Paint Manager
        |
        v
paintComponent()
        |
        v
Your Drawing Code
```

---

ဘယ်အချိန်မှာ ပြန်ခေါ်လဲ?

ဥပမာ:

```java
lblCam.repaint();
```

ခေါ်တဲ့အခါ။

Chapter 5 Timer မှာ:

```java
animationTimer = new Timer(20, e -> {

    scanLineY +=4;

    lblCam.repaint();

});
```

ရှိတယ်။

ဒါကြောင့် laser လှုပ်တာ။

---

# Chapter 10.3 — super.paintComponent(g)

```java
super.paintComponent(g);
```

ဒါက အရေးကြီးပါတယ်။

ဒီ line က parent JLabel ရဲ့ painting ကို အရင်လုပ်ပေးတယ်။

ဆိုလိုတာ:

```
Step 1

Draw JLabel

(Camera Image)


Step 2

Draw Custom Effect

(Your Code)
```

ဖြစ်ပါတယ်။

---

မရေးရင်?

```
Camera Image

+

Your Drawing
```

ပျောက်နိုင်တယ်။

---

# Chapter 10.4 — Webcam Check

```java
if (webcam == null || !webcam.isOpen()) {
    return;
}
```

Meaning:

Webcam မရှိရင် drawing မလုပ်နဲ့။

---

Condition 1:

```java
webcam == null
```

Camera object မဖန်တီးရသေး။

---

Condition 2:

```java
!webcam.isOpen()
```

Camera ပိတ်ထားတယ်။

---

ဘာကြောင့်?

မလိုအပ်တဲ့ repaint တွေကို တားတာ။

---

# Chapter 10.5 — Create Graphics2D

```java
Graphics2D g2 =
(Graphics2D) g.create();
```

`Graphics` ကို copy လုပ်တာ။

ဘာကြောင့် copy လုပ်လဲ?

Original Graphics ကို မထိခိုက်အောင်။

---

Example:

Bad:

```java
g.setColor(Color.RED);
```

ဒါဆို parent component တွေပါ သက်ရောက်နိုင်တယ်။

---

Good:

```java
Graphics2D g2 = (Graphics2D)g.create();
```

ကိုယ်ပိုင် drawing environment ရတယ်။

---

# Chapter 10.6 — Anti Aliasing

```java
g2.setRenderingHint(
    RenderingHints.KEY_ANTIALIASING,
    RenderingHints.VALUE_ANTIALIAS_ON
);
```

Anti-aliasing = Edge smoothing

Without:

```
+------+
|      |
+------+
```

Pixel edge ကြမ်းတယ်။

With:

```
╭──────╮
│      │
╰──────╯
```

Smooth ဖြစ်တယ်။

---

ဒီ project မှာ

- Round Rectangle
    
- Glow
    
- Text
    

တွေရှိလို့ လိုအပ်တယ်။

---

# Chapter 10.7 — Component Size

```java
int w = getWidth();
int h = getHeight();
```

ဒီ JLabel ရဲ့ size ယူတာ။

ဥပမာ:

```
lblCam

width  = 242
height = 264
```

---

# Chapter 10.8 — Scanner Box Calculation

```java
int boxSize = Math.min(w,h)-40;
```

အဓိပ္ပါယ်:

Width နဲ့ Height ထဲက အသေးဆုံးကိုယူပြီး 40 pixel လျှော့တယ်။

---

Example:

```
w = 242
h = 264


min = 242


242 - 40

= 202
```

Scanner square:

```
+----------------+
|                |
|    +------+    |
|    | QR   |    |
|    +------+    |
|                |
+----------------+
```

---

Position:

```java
int boxX =
(w-boxSize)/2;
```

Horizontal center.

Example:

```
242 - 202

40 / 2

20
```

x = 20

---

```java
int boxY =
(h-boxSize)/2;
```

Vertical center.

---

# Chapter 10.9 — Dark Glass Overlay

Code:

```java
g2.setColor(
new Color(10,5,20,150)
);
```

Color format:

```java
new Color(
Red,
Green,
Blue,
Alpha
)
```

---

Values:

```
Red   = 10
Green = 5
Blue  = 20
Alpha = 150
```

Alpha = transparency

```
0    invisible

255  solid
```

150 ဆိုတော့ semi-transparent.

---

ပြီးတော့:

```java
fillRect()
```

နဲ့ box အပြင်ဘက်ကို dark လုပ်တယ်။

---

ဒီ 4 ခု:

```java
fillRect(0,0,w,boxY);

fillRect(0,boxY,boxX,boxSize);

fillRect(...right...)

fillRect(...bottom...)
```

ဘာဖြစ်လဲ?

Before:

```
Camera

++++++++++++
+          +
+    QR    +
+          +
++++++++++++
```

After:

```
████████████
██        ██
██   QR   ██
██        ██
████████████
```

---

QR area ပဲ bright ဖြစ်မယ်။

---

# Chapter 10.10 — Breathing Glow Animation

ဒီအပိုင်းက advanced ပါ။

```java
long time =
System.currentTimeMillis();
```

Current time milliseconds ရယူတယ်။

---

Example:

```
100000 ms
```

---

Next:

```java
Math.sin(time * 0.006)
```

Sin wave သုံးတယ်။

Sin result:

```
-1
 |
 |
 0
 |
 |
 1
```

---

ဒါကို:

```java
(Math.sin(...) + 1) / 2
```

လုပ်တယ်။

ဘာရလဲ?

```
0.0

0.5

1.0
```

အတွင်းမှာပဲ ရတယ်။

---

ဒါကို pulse လို့ခေါ်တယ်။

---

Brightness:

```java
int alpha =
(int)(100 + 155 * pulse);
```

---

pulse = 0

```
alpha = 100
```

pulse = 1

```
alpha =255
```

---

Result:

```
Glow dim

     ↓

Glow bright

     ↓

Glow dim
```

Breathing effect.

---

# Chapter 10.11 — Multiple Glow Layers

```java
for(int i=6;i>=1;i--)
```

ဘာကြောင့် loop သုံးလဲ?

Glow တစ်ကြောင်းမဟုတ်ဘဲ layer များစွာလိုလို့။

---

Example:

```
      Glow
   Glow Glow
 +----------+
 |  Frame   |
 +----------+
```

---

Code:

```java
g2.drawRoundRect(
boxX-i,
boxY-i,
boxSize+i*2,
boxSize+i*2,
16,
16
);
```

---

i ကြီး:

```
ပိုကျယ်
ပိုဝေး
```

i သေး:

```
အတွင်းနီး
```

---

ဒါကြောင့် glow ဖြစ်တယ်။

---

# Chapter 10.12 — Main Scanner Border

```java
g2.setColor(Color.WHITE);
```

White frame.

---

Stroke:

```java
g2.setStroke(
new BasicStroke(2f)
);
```

Line thickness = 2 pixels

---

Draw:

```java
drawRoundRect()
```

Result:

```
╭────────╮
│        │
│        │
╰────────╯
```

---

# Chapter 10.13 — HUD Corner Brackets

ဒါက Sci-Fi scanner feeling ပေးတဲ့အပိုင်း။

ဥပမာ:

```
┌────────┐
│        │
│   QR   │
│        │
└────────┘
```

မဟုတ်ဘူး။

ဒီလို:

```
┌──


        QR


        ──┐
```

---

Top Left:

```java
g2.drawLine(
boxX-2,
boxY,
boxX+corner,
boxY
);
```

Horizontal line.

ပြီးတော့:

```java
g2.drawLine(
boxX,
boxY-2,
boxX,
boxY+corner
);
```

Vertical line.

Result:

```
┌
│
```

---

# Chapter 10.14 — Laser Position

```java
int laserY =
Math.max(
 boxY+5,
 Math.min(
 scanLineY,
 boxY+boxSize-5
 )
);
```

ဒီ code က laser ကို box ထဲမှာ lock လုပ်ထားတာ။

---

ဥပမာ:

```
Scanner Box


Top
 |
 |
Laser
 |
 |
Bottom
```

ဘယ်တော့မှ box အပြင်မထွက်ဘူး။

---

# Chapter 10.15 — Laser Glow

```java
g2.fillRect(
boxX,
laserY-18,
boxSize,
36
);
```

Laser အနားမှာ glow area ဆွဲတယ်။

---

Visual:

```
~~~~~~~~~~~~
------------
    LASER
------------
~~~~~~~~~~~~
```

---

# Chapter 10.16 — Gradient Trail

```java
GradientPaint laserGlow =
new GradientPaint(...)
```

Gradient ဆိုတာ:

```
Transparent

     ↓

White
```

---

ဒီမှာ:

```java
new Color(255,255,255,0)
```

transparent

နဲ့

```java
new Color(255,255,255,220)
```

white

ကြား transition လုပ်တယ်။

---

# Chapter 10.17 — Core Laser Line

```java
g2.fillRect(
boxX,
laserY,
boxSize,
2
);
```

ဒါက အလယ်က အဖြူလိုင်း။

```
----------------
```

---

# Chapter 10.18 — Cyber Status Text

Font:

```java
new Font(
"JetBrains Mono",
BOLD,
12
)
```

---

Text:

```java
[ ACTIVE SCANNER // SECURE ]
```

---

Center:

```java
fm.stringWidth(txt)
```

နဲ့ text width ယူတယ်။

---

Formula:

```
textX =
(screen width - text width) / 2
```

---

Shadow:

```java
drawString(txt,textX,textY+1)
```

အရင်ဆွဲ။

---

Main:

```java
drawString(txt,textX,textY)
```

နောက်ဆွဲ။

---

Result:

```
shadow
ACTIVE SCANNER
```

ပို depth ရတယ်။

---

# Chapter 10.19 — Dispose

```java
g2.dispose();
```

Graphics resource ပြန်လွှတ်တာ။

အရေးကြီးပါတယ်။

---

မလုပ်ရင်:

```
Many repaint

↓

Many Graphics Object

↓

Memory leak
```

ဖြစ်နိုင်တယ်။

---

# Complete Rendering Order

ဒီ paintComponent() ရဲ့ order:

```
1. Draw Camera Image
        |
        v
2. Dark Overlay
        |
        v
3. Glow Border
        |
        v
4. White Frame
        |
        v
5. HUD Corners
        |
        v
6. Laser Glow
        |
        v
7. Laser Line
        |
        v
8. Status Text
```

---

# Senior Developer View

ဒီ code ထဲမှာ ပါတဲ့ Concept:

|Concept|Usage|
|---|---|
|Custom Painting|Override paintComponent|
|Graphics2D|Advanced drawing|
|Alpha Composite|Glass overlay|
|Animation Loop|Timer + repaint|
|Sin Wave|Breathing glow|
|Layer Rendering|UI effects|
|HUD Design|Scanner frame|
|Gradient|Laser effect|
|Resource Management|dispose()|

---

ဒီ `paintComponent()` က Java Swing နဲ့ရေးထားတဲ့ **mini game engine / HUD renderer** လိုမျိုးပါ။

နောက် Chapter 11 မှာတော့ ဒီ Scanner ရဲ့ counterpart ဖြစ်တဲ့ **QR Generator System** ကို သွားမယ်:

```
Student Data

↓

Create Token

↓

AES Encrypt

↓

Generate QR

↓

Save PNG

↓

Database Store Token
```

အဲ့ဒီ flow ကို လေ့လာရင် ဒီ Project တစ်ခုလုံးရဲ့ Architecture ကို complete နားလည်သွားမယ်။

---

**Chapter 11 — QR Generator Flow (Encrypt → Generate QR → Save PNG → Database Token)** မှာတော့ Scanner ရဲ့ တစ်ဖက်ဖြစ်တဲ့ **QR Code Creation System** ကို လေ့လာမယ်။