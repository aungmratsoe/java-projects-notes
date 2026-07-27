# Chapter 14 — Custom Scanner UI

## Building a Professional Sci-Fi QR Scanner Interface with `paintComponent()`

Great **aung mrat**.

ဒီ Chapter မှာတော့ `QRScanner.java` ရဲ့ UI ပိုင်းကို နားလည်မယ်။

အထူးသဖြင့် ဒီအပိုင်း:

```java
private javax.swing.JLabel lblCam = new javax.swing.JLabel(){

    @Override
    protected void paintComponent(Graphics g){

    }

};
```

ဒီဟာက ဒီ Project ရဲ့ **most advanced UI part** ဖြစ်ပါတယ်။

---

# 14.1 What is Custom Scanner UI?

ပုံမှန် Swing JLabel ဆိုရင်:

```text
+----------------+
|                |
| Camera Image   |
|                |
+----------------+
```

ပဲရမယ်။

---

ဒါပေမယ့် မင်းရေးထားတာက:

```text
+--------------------------------+
|                                |
|       DARK GLASS OVERLAY       |
|                                |
|          ┌────────┐            |
|          │        │            |
|          │   QR   │            |
|          │        │            |
|          └────────┘            |
|              --------           |
|              LASER              |
|                                |
|       ACTIVE SCANNER            |
+--------------------------------+
```

လိုမျိုး **custom camera HUD interface** ဖြစ်သွားတယ်။

---

# 14.2 Swing Painting System

Swing မှာ Component တစ်ခုကို ဆွဲတဲ့အခါ flow က:

```text
JFrame
 |
 |
 v
JPanel
 |
 |
 v
JLabel
 |
 |
 v
paintComponent()
 |
 |
 v
Your Custom Drawing
```

---

`paintComponent()` က:

> "ဒီ Component ကို ဘယ်လိုပုံစံနဲ့ဆွဲမလဲ"

ဆိုတာကို control လုပ်တဲ့နေရာ။

---

# 14.3 Why Override JLabel?

ပုံမှန်:

```java
JLabel lblCam = new JLabel();
```

ဆိုရင်:

```text
JLabel

 |
 |
 +-- Draw Icon
 +-- Draw Text

```

ပဲလုပ်နိုင်တယ်။

---

ဒါပေမယ့်:

```java
new JLabel(){

@Override
paintComponent(){

}

}
```

ဆိုရင်:

```text
JLabel

 |
 |
 +-- Camera Image
 |
 +-- Custom Frame
 |
 +-- Animation
 |
 +-- Text
 |
 +-- Effects

```

ထပ်ဆွဲနိုင်တယ်။

---

# 14.4 Rendering Order (အရေးကြီး)

ဒီ UI မှာ Drawing Order အရမ်းအရေးကြီးတယ်။

Code execution order:

```
1. super.paintComponent()

        |
        v

2. Camera Image

        |
        v

3. Dark Overlay

        |
        v

4. Scanner Frame

        |
        v

5. Glow Effect

        |
        v

6. Laser

        |
        v

7. Status Text

```

---

ဘာကြောင့် order အရေးကြီးလဲ?

Graphics က layer အလိုက် ဆွဲတာ။

Example:

First:

```
Camera
```

Second:

```
Dark Overlay
```

ဆိုရင် Camera ပေါ်မှာ dark filter တင်သလို ဖြစ်မယ်။

---

# 14.5 Graphics2D

Code:

```java
Graphics2D g2 =
(Graphics2D) g.create();
```

---

`Graphics` ထက် `Graphics2D` က advanced ဖြစ်တယ်။

Support:

- Line thickness
    
- Transparency
    
- Rotation
    
- Gradient
    
- Anti-aliasing
    

---

Example:

Normal Graphics:

```java
g.drawRect();
```

Advanced Graphics2D:

```java
g2.drawRoundRect();

g2.setStroke();

g2.setPaint();

```

---

# 14.6 Anti-Aliasing

Code:

```java
g2.setRenderingHint(
RenderingHints.KEY_ANTIALIASING,
RenderingHints.VALUE_ANTIALIAS_ON
);
```

---

Without:

```
+-------+
|       |
+-------+
```

Edge တွေ pixel ကြမ်းမယ်။

---

With:

```
╭───────╮
│       │
╰───────╯
```

Smooth ဖြစ်မယ်။

---

ဒီ project မှာ:

- Round rectangle
    
- Glow
    
- Text
    

ရှိတဲ့အတွက် မဖြစ်မနေလိုတယ်။

---

# 14.7 Scanner Box Design

Code:

```java
int boxSize = Math.min(w,h)-40;
```

---

Goal:

Camera area ထဲမှာ square scanning zone တစ်ခုဖန်တီးတာ။

Example:

Component:

```
Width = 242
Height = 264
```

---

Calculate:

```
min(242,264)

=242

242-40

=202
```

---

Result:

```
+----------------+
|                |
|    +------+    |
|    |      |    |
|    | QR   |    |
|    |      |    |
|    +------+    |
|                |
+----------------+

```

---

# 14.8 Dark Glass Vignette

Code:

```java
g2.setColor(
new Color(10,5,20,150)
);
```

---

Color format:

```java
new Color(
    red,
    green,
    blue,
    alpha
)
```

---

Example:

```java
new Color(10,5,20,150)
```

means:

```
Red   = 10
Green = 5
Blue  = 20
Alpha = 150
```

---

Alpha:

```
0     transparent

255   solid
```

---

150 ဆိုတော့ semi-transparent dark layer ဖြစ်တယ်။

---

Effect:

Before:

```
Camera full brightness
```

After:

```
Dark cinematic scanner
```

---

# 14.9 Pulse Glow Border

ဒီအပိုင်းက animation effect ဖြစ်တယ်။

Code:

```java
long time =
System.currentTimeMillis();
```

---

Current time ကိုယူတယ်။

ဥပမာ:

```
100000 ms
```

---

Then:

```java
Math.sin(time * 0.006)
```

အသုံးပြုတယ်။

Sin wave:

```
      1
      |
0 ----|---- 0
      |
     -1

```

---

ဒါကို convert:

```java
(float)
((sin + 1)/2)
```

လုပ်တယ်။

Result:

```
0.0
0.5
1.0
```

ပဲရမယ်။

---

ဒါကို pulse လို့သုံးတယ်။

Effect:

```
Brightness:

Low
 |
 |
High
 |
 |
Low

```

---

ဒီလို:

```
Breathing Glow
```

ဖြစ်တယ်။

---

# 14.10 Multiple Glow Layers

Code:

```java
for(int i=6;i>=1;i--)
```

---

Glow တစ်ကြောင်းတည်းဆိုရင်:

```
+---------+
```

ပဲဖြစ်မယ်။

---

Layer များစွာဆို:

```
    Glow
  Glow Glow
+-----------+
|  Scanner  |
+-----------+
  Glow Glow
    Glow

```

---

i ကြီး:

```
Outer Glow
```

i သေး:

```
Inner Glow
```

---

ဒါကြောင့် premium effect ရတယ်။

---

# 14.11 Scanner Corner HUD

ဒီအပိုင်းက Sci-Fi feeling ပေးတာ။

Normal frame:

```
+----------+
|          |
|    QR    |
|          |
+----------+

```

---

HUD style:

```
┌

       QR


              ┐


└

              ┘

```

---

Code:

```java
g2.drawLine()
```

နဲ့ corner တစ်ခုချင်းဆွဲတယ်။

---

Top Left:

```java
g2.drawLine(
boxX,
boxY,
boxX+corner,
boxY
);
```

Horizontal line.

---

```java
g2.drawLine(
boxX,
boxY,
boxX,
boxY+corner
);
```

Vertical line.

---

Result:

```
┌
│
```

---

# 14.12 Laser Scanner Animation

ဒီ project ရဲ့ main attraction ဖြစ်တယ်။

Variable:

```java
private int scanLineY;
```

---

Timer က update:

```
scanLineY++

```

ပြီးရင်:

```java
lblCam.repaint();
```

---

Paint မှာ:

```java
int laserY = scanLineY;
```

---

Result:

```
+----------+

    |
    |
    |
----------  Laser

    |
    |

+----------+

```

---

# 14.13 Laser Glow

Outer glow:

```java
fillRect(
boxX,
laserY-18,
boxSize,
36
);
```

---

Effect:

```
~~~~~~~~~~~~
------------
   LASER
------------
~~~~~~~~~~~~

```

---

# 14.14 Gradient Laser Tail

Code:

```java
GradientPaint
```

---

Gradient:

```
Transparent

      ↓

Bright White

```

---

Visual:

```
      |
      |
------
LASER
------
      |
```

---

ဒါက movie scanner effect လို ဖြစ်စေတယ်။

---

# 14.15 Status Text HUD

Code:

```java
String txt =
"[ ACTIVE SCANNER // SECURE ]";
```

---

Font:

```java
JetBrains Mono
```

---

ဘာကြောင့် Mono Font?

Cyber / Terminal feeling ရတယ်။

Example:

```
[ SYSTEM ONLINE ]
[ SCANNING... ]
[ ACCESS GRANTED ]

```

လို UI တွေအတွက် သင့်တော်တယ်။

---

# 14.16 Text Centering

Code:

```java
FontMetrics fm =
g2.getFontMetrics();
```

---

Text width သိဖို့။

Formula:

```
Text Position =

(Screen Width - Text Width) / 2

```

---

Example:

```
Screen = 300

Text = 200


(300-200)/2

=50

```

---

Text center ရောက်မယ်။

---

# 14.17 Shadow Text Effect

First:

```java
g2.setColor(
new Color(255,255,255,50)
);

drawString(
txt,
x,
y+1
);
```

---

ဒါက shadow layer။

---

Second:

```java
g2.setColor(Color.WHITE);

drawString(
txt,
x,
y
);
```

---

Result:

```
ACTIVE SCANNER

(shadow depth)

```

---

# 14.18 Graphics Cleanup

Code:

```java
g2.dispose();
```

---

Graphics resource ပြန်လွှတ်တယ်။

မလုပ်ရင်:

```
Repaint
Repaint
Repaint

↓

Memory usage increase

```

ဖြစ်နိုင်တယ်။

---

# 14.19 Complete UI Rendering Flow

Final:

```
Camera Frame

      |
      v

super.paintComponent()

      |
      v

Dark Glass Filter

      |
      v

Scanner Box

      |
      v

Pulse Glow

      |
      v

HUD Corners

      |
      v

Laser Animation

      |
      v

Status Text

      |
      v

dispose()

```

---

# 14.20 Senior Developer Analysis

ဒီ Custom Scanner UI မှာ ပါတဲ့ Concepts:

|Concept|Usage|
|---|---|
|Custom Painting|Override paintComponent|
|Graphics2D|Advanced rendering|
|Alpha Transparency|Glass effect|
|Animation|Timer + repaint|
|Mathematical Animation|Sin wave|
|Layer Rendering|HUD design|
|Gradient|Laser tail|
|FontMetrics|Text alignment|
|Resource Cleanup|dispose|

---

# Final Architecture View

ဒီ QR Scanner UI က:

```
                 QRScanner JFrame

                       |
                       v

                 lblCam JLabel

                       |
                       v

              Custom paintComponent()

                       |

        +--------------+--------------+

        |              |              |

    Camera       Scanner HUD      Animation

    Image        Graphics        Timer

```

---

**Senior Developer Note**

ဒီ UI approach ကို Java Swing မှာသုံးတာက:

- Security Scanner
    
- Attendance System
    
- Face Recognition
    
- Industrial Camera Software
    
- POS Verification System
    

တွေမှာ အသုံးများတဲ့ pattern ဖြစ်ပါတယ်။

နောက် Chapter 15 မှာ:

# Chapter 15 — QRScanner Event Flow (Button, Window, Mouse, Navigation)

ကို ဆက်ပြီးလေ့လာမယ်။

အဲဒီမှာ:

```
Home Button
     |
closeWebcam()
     |
dispose()
     |
Open Home Frame


Generate QR Click
     |
QRGenerator Frame

```

ဆိုတဲ့ JFrame Navigation Architecture ကို breakdown လုပ်မယ်။