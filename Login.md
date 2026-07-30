ဒီ `Login` class ဟာ အလွန်လှပတဲ့ Animation တွေနဲ့ အဆင့်မြင့် Biometric Login UI တစ်ခုကို ဖန်တီးထားတာပါ။ သူ့ရဲ့ အဓိက အစိတ်အပိုင်းတွေကို အောက်ပါအတိုင်း အပိုင်းလိုက် ရှင်းပြပေးပါမယ်။

### ၁။ Animation စနစ် (Animation System)

ဒီ UI ရဲ့ အဓိက ပုံစံက Fingerprint ပုံလေးမှာ တောက်ပတဲ့ အလင်း (Glow) နဲ့ စကင်ဖတ်နေတဲ့ လေဆာ (Laser sweep) ပါဝင်ပါတယ်။

- **`Timer`**: Swing ရဲ့ `Timer` ကိုသုံးပြီး Animation တွေကို စဉ်ဆက်မပြတ် လည်ပတ်စေပါတယ်။
    
    - `pulseTimer`: Fingerprint ပုံလေးရဲ့ ပတ်လည်မှာ အလင်းတန်းလေးတွေ တောက်ပနေအောင် (`pulseAlpha`) လုပ်ပေးပါတယ်။
        
    - `laserTimer`: အပေါ်ကနေ အောက်ကို ရွေ့လျားနေတဲ့ လေဆာရောင်ခြည် (Laser line) အတွက် `laserYFraction` ကို အသုံးပြုထားပါတယ်။
        
- **`paintComponent`**: ဒီနေရာမှာ Java2D (`Graphics2D`) ကို အသုံးပြုပြီး ပုံတွေကို ဆွဲပါတယ်။ `isError` ဆိုတဲ့ Boolean variable ကိုသုံးပြီး၊ မှန်ကန်ရင် အစိမ်းရောင်၊ မှားယွင်းရင် အနီရောင် (Red) အရောင်ကို ပြောင်းလဲပေးပါတယ်။
    

### ၂။ Passcode ထည့်သွင်းခြင်း (Passcode Dialog)

- **`promptPasscode()`**: ဒီ Method က `JOptionPane` ကိုသုံးပြီး Password ရိုက်ထည့်ဖို့ Box တစ်ခုကို ပေါ်လာစေပါတယ်။
    
- **Auto-focus**: `AncestorListener` ကို သုံးထားတာက Dialog ပေါ်လာတာနဲ့ Password ရိုက်ရမယ့်နေရာ (`pf`) ကို အလိုအလျောက် Focus ရောက်သွားအောင် လုပ်ပေးတာပါ။ ဒါကြောင့် User အနေနဲ့ Dialog ပေါ်လာတာနဲ့ Password စရိုက်လို့ ရပါတယ်။
    

### ၃။ အဆင့်ဆင့် လုပ်ဆောင်ချက် (Logic Flow)

1. **UI စတင်ခြင်း (`initCustomDesign`)**: Window ပွင့်တာနဲ့ `startCoolScanAnimation()` ကို ခေါ်ပြီး Animation တွေကို စတင်ပါတယ်။
    
2. **Password စစ်ဆေးခြင်း**: User ရိုက်လိုက်တဲ့ Password က `1234` ဒါမှမဟုတ် `admin` ဖြစ်ရင် -
    
    - `isError = false` ဆိုပြီး သတ်မှတ်တယ်။
        
    - `updateStatus` ကိုသုံးပြီး အောင်မြင်ကြောင်း ပြသတယ်။
        
    - `Timer` တစ်ခုကို သုံးပြီး ခဏစောင့်ကာ `Home` screen ကို ပြောင်းပေးတယ်။
        
3. **မှားယွင်းပါက**: `isError = true` လို့ သတ်မှတ်ပြီး, Animation ကို အနီရောင်နဲ့ ပြန်စပါတယ်။
    

### ၄။ UI ဒီဇိုင်း (Custom Panel)

- **`CustomPurplePanel`**: ဒါက `JPanel` ကို Extension လုပ်ထားတာပါ။ `GradientPaint` ကို သုံးပြီး အောက်ခြေမှာ ခရမ်းရောင် အနုအရင့် (Gradient background) ကို ဖန်တီးထားတာပါ။
    
- **FlatLaf**: `com.formdev.flatlaf` library ကိုသုံးထားလို့ UI ပုံစံက ခေတ်မီပြီး သပ်ရပ်နေတာပါ။
    

### အနှစ်ချုပ် အချက်များ

- **`paintComponent` ကို override လုပ်ထားခြင်း**: ဒါက ဒီ UI ရဲ့ အသက်ပါ။ အပေါ်ယံ ပုံသေ Icon တစ်ခုတည်းကို မသုံးဘဲ ကိုယ်ပိုင် Graphic တွေ ဆွဲထားလို့ ပိုလှတာပါ။
    
- **SwingUtilities.invokeLater**: ဒါကို `main` method မှာ သုံးထားတာက သင်အရင်က မေးထားတဲ့အတိုင်း GUI ကို Thread-safe ဖြစ်အောင် (EDT ထဲမှာ) အလုပ်လုပ်စေဖို့ ဖြစ်ပါတယ်။
    

**သင်ကိုယ်တိုင် ပြင်ဆင်ကြည့်လိုပါက:** ဒီ code ထဲမှာ `1234` ဆိုတဲ့ Password နေရာကို သင်ပြောင်းချင်တဲ့ Password နဲ့ အစားထိုးကြည့်နိုင်ပါတယ်။ နောက်ပြီး Animation ရဲ့ အမြန်နှုန်းကို ပြောင်းချင်ရင် `Timer(50, ...)` ထဲက `50` ဆိုတဲ့ ဂဏန်းကို လျှော့တာ၊ တိုးတာမျိုး စမ်းသပ်ကြည့်နိုင်ပါတယ်။

---

ဒီ `Login.java` code က **Java Swing** နဲ့ရေးထားတဲ့ **Modern Biometric Login UI** ဖြစ်ပါတယ်။ Fingerprint Animation, Laser Scan Animation, Passcode Verification, Home Screen Navigation စတဲ့ features တွေပါဝင်ပါတယ်။

အောက်မှာ **line by line** နဲ့ **code breakdown** လုပ်ပြီး မြန်မာလိုအသေးစိတ်ရှင်းပြပါမယ်။

---

# 1. Package

```java
package com.ams.qrcode.ui;
```

ဒီ class က

```
com
 └── ams
      └── qrcode
             └── ui
                    Login.java
```

package ထဲမှာရှိတယ်ဆိုတာကို ပြောတာပါ။

---

# 2. Import

```java
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.Ellipse2D;
import java.awt.geom.Rectangle2D;
import javax.swing.*;
```

ဒီ imports တွေက UI ရေးဖို့လိုတဲ့ Library တွေဖြစ်ပါတယ်။

ဥပမာ

```
java.awt.*
```

ဆိုတာ

- Color
    
- Font
    
- Cursor
    
- Graphics
    

တို့ပါလာတယ်။

---

```
javax.swing.*
```

ကတော့

- JFrame
    
- JPanel
    
- JLabel
    
- JButton
    
- Timer
    
- JOptionPane
    

တို့ပါလာတယ်။

---

# 3. Class

```java
public class Login extends JFrame
```

အဓိပ္ပါယ်

```
Login
    ↓
JFrame
```

Login class ဟာ JFrame ကို inherit လုပ်ထားတာဖြစ်ပါတယ်။

ဒါကြောင့်

```
Login
```

ဟာ Window တစ်ခုဖြစ်သွားတယ်။

---

# 4. Logger

```java
private static final Logger logger =
        Logger.getLogger(Login.class.getName());
```

ဒီ Logger က Error တွေကို Console ထဲမှာ Log ထုတ်ဖို့ပါ။

ဥပမာ

```
Error launching Home screen
```

လိုမျိုး။

---

# 5. Variables

```java
private boolean isScanning = false;
```

Scanner အလုပ်လုပ်နေသလား။

```
true
```

↓

Scanning

```
false
```

↓

မလည်သေး

---

```java
private boolean isError = false;
```

Login Fail ဖြစ်သလား။

```
true
```

↓

Fingerprint အနီရောင်

```
false
```

↓

အစိမ်းရောင်

---

```java
private Timer pulseTimer;
```

Glow Animation Timer

---

```java
private Timer laserTimer;
```

Laser Animation Timer

---

```java
private float pulseAlpha = 0.3f;
```

Glow Transparency

```
0
```

↓

ပျောက်

```
1
```

↓

အပြည့်

---

```java
private float laserYFraction = 0;
```

Laser Line ရဲ့ Position

```
0
```

↓

Top

```
1
```

↓

Bottom

---

# 6. Constructor

```java
public Login() {
    initComponents();
    initCustomDesign();
}
```

Program စဖွင့်တာနဲ့

```
Login()
```

ခေါ်တယ်။

ပြီးရင်

```
initComponents();
```

↓

UI တည်ဆောက်တယ်။

ပြီးတော့

```
initCustomDesign();
```

↓

Animation

Mouse Event

Window Event

တို့ကို Configure လုပ်တယ်။

---

# 7. initCustomDesign()

ဒီ Method ထဲမှာ UI Logic အကုန်ရှိပါတယ်။

---

## Window Title

```java
setTitle("Student Verification - Biometric Login");
```

Window Title

↓

```
Student Verification - Biometric Login
```

---

## Resize ပိတ်

```java
setResizable(false);
```

Window Size မပြောင်းနိုင်တော့ဘူး။

---

## Mouse Listener

```java
lblFingerprint.addMouseListener(...)
```

Fingerprint ကို Click လုပ်ရင်

```
promptPasscode();
```

ခေါ်မယ်။

Flow:

```
User Click

↓

Mouse Listener

↓

promptPasscode()
```

---

# 8. Pulse Timer

```java
pulseTimer = new Timer(50, e -> {
```

50ms တိုင်း Run မယ်။

---

ဒီ Logic

```java
pulseAlpha += 0.02;
```

Glow ကို တဖြည်းဖြည်းကြီးလာစေတယ်။

ပြီးရင်

```java
pulseAlpha -= 0.02;
```

ပြန်လျော့သွားတယ်။

ဒီလိုဖြစ်နေတယ်။

```
0.2

↓

0.4

↓

0.6

↓

0.8

↓

0.6

↓

0.4

↓

0.2
```

ဒါကြောင့်

Glow က

```
Pulse

Pulse

Pulse
```

လုပ်နေတယ်။

---

# 9. Laser Timer

```java
laserYFraction += 0.012;
```

Laser Position တိုးသွားတယ်။

```
0

↓

0.1

↓

0.2

↓

0.3

↓

...

↓

1
```

1 ကျော်သွားရင်

```java
laserYFraction = 0;
```

ပြန်စတယ်။

ဒါကြောင့်

Laser က

```
Top

↓

↓

↓

Bottom

↓

Top
```

အမြဲလည်နေတယ်။

---

# 10. Window Listener

```java
windowOpened()
```

Window ဖွင့်တာနဲ့

```
startCoolScanAnimation();
```

Run မယ်။

---

Window ပိတ်ရင်

```java
stopAnimations();
```

Timer တွေအကုန် Stop လုပ်မယ်။

---

# 11. startCoolScanAnimation()

```java
isScanning = true;
```

Scanning စပြီ။

---

```java
pulseTimer.start();
```

Glow Animation စ။

---

```java
laserTimer.start();
```

Laser Animation စ။

---

Status ကို

```
SCANNING BIOMETRICS
```

ပြောင်းတယ်။

---

# 12. stopAnimations()

ဒီ Method က

```
pulseTimer.stop();

laserTimer.stop();
```

လုပ်ပြီး

Variables တွေ Reset ပြန်လုပ်တယ်။

---

# 13. updateStatus()

```java
lblStatus.setText(...)
```

ဒီ Method က

Status Label ကို Update လုပ်တာပါ။

ဥပမာ

```
ACCESS GRANTED
```

သို့မဟုတ်

```
VERIFICATION FAILED
```

HTML ကို အသုံးပြုပြီး Label ထဲမှာ စာနှစ်ကြောင်း၊ အရောင်နဲ့ Format လုပ်ထားပါတယ်။

---

# 14. promptPasscode()

ဒီ Method က Login ရဲ့ အဓိက Logic ပါ။

```
JPasswordField pf = new JPasswordField();
```

Password Box တစ်ခုဖန်တီးတယ်။

ပြီးရင်

```
JOptionPane.showConfirmDialog(...)
```

Popup ပြတယ်။

---

Password စစ်တဲ့အပိုင်း

```java
if ("1234".equals(pin) ||
    "admin".equalsIgnoreCase(pin))
```

ဆိုလိုတာ

```
1234
```

ဒါမှမဟုတ်

```
admin
```

ရိုက်ရင် Login အောင်မယ်။

---

အောင်ရင်

```
stopAnimations();

ACCESS GRANTED

↓

Home()
```

သွားမယ်။

---

မအောင်ရင်

```
isError = true;
```

ဖြစ်သွားတယ်။

ပြီးတော့

```
VERIFICATION FAILED
```

ပြပြီး

Fingerprint ကို အနီရောင် Animation နဲ့ ဆက်ပြသွားတယ်။

---

# 15. navigateToHome()

```java
Home homeFrame = new Home();
```

Home Window ဖွင့်တယ်။

```
homeFrame.setVisible(true);
```

Home ကိုပြ။

```
this.dispose();
```

Login Window ကိုပိတ်တယ်။

Error ဖြစ်ရင် Logger မှာ Log ထုတ်ပြီး Status ကို `"VERIFICATION ERROR"` လို့ပြသပါတယ်။

---

# 16. Custom Fingerprint Painting

ဒီအပိုင်းက Swing ရဲ့ အရေးကြီးဆုံးနည်းလမ်းတစ်ခုဖြစ်တဲ့

```java
protected void paintComponent(Graphics g)
```

ကို Override လုပ်ထားတာပါ။

လုပ်ဆောင်ပုံက အဆင့်လိုက်ဆိုရင်

```
paintComponent()

↓

Draw Glow

↓

Draw Fingerprint Icon

↓

Draw Laser

↓

Finish
```

---

## Theme Color

```java
Color themeColor =
    isError ?
    RED :
    GREEN;
```

```
Login OK

↓

Green

Login Fail

↓

Red
```

---

## Glow

```java
g2.fill(new Ellipse2D.Float(...));
```

Fingerprint ပတ်လည်မှာ

```
○
```

လို Glow Circle ဆွဲတယ်။

---

## Laser

```java
g2.fillRect(...)
```

Laser Line

```
------------
```

ဆွဲတယ်။

ပြီးတော့

```
GradientPaint
```

နဲ့ Glow Effect ထပ်ထည့်ထားလို့ အလင်းတန်းက ပိုသဘာဝကျပြီး လှပပါတယ်။

---

# 17. initComponents()

ဒီ Method က NetBeans GUI Builder က Generate လုပ်ပေးထားတဲ့ UI Layout ဖြစ်ပါတယ်။

အဓိက Components တွေက

- `lblFingerprint` → Fingerprint Icon
    
- `lblStatus` → Status စာသား
    
- `btnPinFallback` → "Enter Passcode / PIN" Button
    
- `pnlBackground` → Background Panel
    

တို့ကို ဖန်တီးပြီး `GroupLayout` နဲ့ နေရာချထားပါတယ်။

---

# 18. CustomPurplePanel

```java
class CustomPurplePanel extends JPanel
```

ဒီ Panel က

```java
GradientPaint
```

အသုံးပြုပြီး

```
Purple

↓

Dark Purple
```

Gradient Background ကိုဆွဲပေးပါတယ်။

---

# 19. main()

```java
public static void main(...)
```

Program စတဲ့နေရာပါ။

```
FlatLightLaf.setup();
```

Look & Feel ကို FlatLaf အဖြစ်သတ်မှတ်တယ်။

ပြီးတော့

```java
new Login().setVisible(true);
```

နဲ့ Login Window ကိုပြသပါတယ်။

---

# Overall Flow Diagram

```text
Program Start
      │
      ▼
 Login()
      │
      ▼
initComponents()
      │
      ▼
initCustomDesign()
      │
      ▼
Window Opened
      │
      ▼
Start Pulse + Laser Animation
      │
      ▼
User Click Fingerprint/Button
      │
      ▼
Passcode Dialog
      │
      ▼
Correct? ───────► YES
   │                 │
   │                 ▼
   │        Stop Animation
   │                 │
   │                 ▼
   │       ACCESS GRANTED
   │                 │
   │                 ▼
   │          Home Screen
   │
   ▼
NO
   │
   ▼
isError = true
   │
   ▼
Red Fingerprint Animation
   │
   ▼
VERIFICATION FAILED
```

ဒီ code မှာ အသုံးပြုထားတဲ့ Java Concepts တွေကတော့ **Inheritance (`extends JFrame`)**, **Event Handling (MouseListener, WindowListener, ActionListener)**, **Swing `Timer`**, **Anonymous Inner Class**, **Lambda Expressions (`e -> {}`)**, **Custom Painting (`paintComponent`)**, **Graphics2D**, **GradientPaint**, **JOptionPane**, နဲ့ **Swing Thread (`SwingUtilities.invokeLater`)** တို့ဖြစ်ပါတယ်။ ၎င်းတို့ပေါင်းစပ်ပြီး Modern Animated Login UI တစ်ခုကို တည်ဆောက်ထားတာဖြစ်ပါတယ်။