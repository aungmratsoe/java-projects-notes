ဟုတ်ပါတယ်။ **FlatLaf** သုံးနေတယ်ဆိုရင် `MouseListener` ကို Button တိုင်းမှာ ရေးတာထက် **Custom Button Class** တစ်ခုရေးတာက Professional ပိုဖြစ်ပါတယ်။

---

# နည်းလမ်း (၃) မျိုးရှိတယ်

## Method 1 - MouseListener (အစပြုသူ)

ဥပမာ

```java
btnSave.addMouseListener(new MouseAdapter() {

    @Override
    public void mouseEntered(MouseEvent e) {
        btnSave.setBackground(new Color(0,150,255));
    }

    @Override
    public void mouseExited(MouseEvent e) {
        btnSave.setBackground(new Color(0,120,215));
    }

});
```

### အားသာချက်

- လွယ်တယ်
    

### အားနည်းချက်

Button 30 ခုရှိရင်

```text
Button 1

MouseListener

Button 2

MouseListener

Button 3

MouseListener
```

အကုန်ရေးရမယ်။

❌ Maintain လုပ်ရခက်တယ်။

---

# Method 2 - Utility Method

ဥပမာ

```java
HoverEffect.install(btnSave);
HoverEffect.install(btnUpdate);
HoverEffect.install(btnDelete);
```

ဒီလိုလည်းရတယ်။

ဒါပေမယ့်

Button အသစ်ထည့်တိုင်း

```java
HoverEffect.install(...)
```

ရေးနေရတယ်။

---

# Method 3 - Custom Button (Professional)

ဒီနည်းက FlatLaf နဲ့ အလိုက်ဖက်ဆုံးပါ။

ဥပမာ

```text
JButton

↓

HoverButton

↓

Save Button

Update Button

Delete Button
```

HoverButton က

Hover Effect

Pressed Effect

Rounded Corner

Focus Color

Animation

အားလုံးကို တစ်နေရာတည်းမှာ ကိုင်တွယ်နိုင်တယ်။

---

# NetBeans Drag & Drop မှာ သုံးလို့ရလား?

ရပါတယ်။

အရမ်းလည်းကောင်းပါတယ်။

လုပ်ပုံက

## Step 1

```text
src

└── ui

      └── HoverButton.java
```

---

## Step 2

```java
public class HoverButton extends JButton {

}
```

---

## Step 3

Compile

---

## Step 4

Palette

↓

Add From Project...

↓

HoverButton

---

ပြီးရင်

Palette ထဲမှာ

```text
HoverButton
```

ပေါ်လာမယ်။

Drag & Drop လုပ်ရုံပဲ။

---

# FlatLaf နဲ့ ဘာကောင်းလဲ?

FlatLaf က

UI Delegate ကိုသုံးထားတယ်။

ဒါကြောင့်

Custom Button ရေးရင်

Theme မပျက်ဘူး။

ဥပမာ

Dark Theme

↓

Hover

↓

အရောင်ပြောင်း

↓

Theme မပျက်

---

# နောက်ထပ်ထည့်လို့ရတာတွေ

Custom Button မှာ

Hover Color

```java
hoverColor
```

Pressed Color

```java
pressedColor
```

Border Radius

```java
arc
```

Shadow

```java
shadow
```

Animation

```java
fade
```

Ripple

```java
material ripple
```

Cursor

```java
HAND_CURSOR
```

Sound

```java
click sound
```

အားလုံးကို

Class တစ်ခုတည်းမှာ ထည့်လို့ရတယ်။

---

# Student Management System အတွက်

သင့်မှာ

```text
Save

Update

Delete

Register

Login

Generate QR

Export

Import
```

Button တွေ အများကြီးရှိတယ်။

Button တိုင်းမှာ

```java
MouseListener
```

ရေးမယ်ဆိုရင်

Project ကြီးလာရင်

Maintain မလုပ်နိုင်တော့ဘူး။

ဒါကြောင့်

```text
HoverButton

↓

Save

Update

Delete

Login

Export
```

ဒီလိုသုံးတာက Professional ဖြစ်ပါတယ်။

---

# ကျွန်တော့်အကြံပြုချက်

သင့် Project က **Java Swing + FlatLaf + NetBeans Drag & Drop** ဖြစ်တဲ့အတွက် ကျွန်တော်ဆိုရင် **Custom Component** တွေဖန်တီးပြီး သုံးပါမယ်။

ဥပမာ

```text
ui.components

├── HoverButton.java
├── RoundedTextField.java
├── RoundedPasswordField.java
├── RoundedPanel.java
├── SearchTextField.java
├── GradientButton.java
├── ShadowPanel.java
└── AnimatedLabel.java
```

ဒီလို Project ကို စနစ်တကျခွဲထားရင် Form အသစ်ဆောက်တိုင်း Drag & Drop နဲ့ Component တွေကို ပြန်အသုံးပြုလို့ရပြီး၊ Design ပြောင်းချင်တဲ့အခါ Class တစ်ခုကိုပဲ ပြင်ရုံနဲ့ Project တစ်ခုလုံးမှာ သက်ရောက်သွားပါတယ်။

**သင့်လို Student Management System အရွယ် Project အတွက်တော့ Custom Component Approach ကို ရွေးတာက အကောင်းဆုံး Design ဖြစ်ပါတယ်။**

---
ဟုတ်ပါတယ်။ **ဒီနေရာက အရေးကြီးပါတယ်။** FlatLaf သုံးနေတဲ့ Swing Project မှာ Professional approach ဆိုတာ—

> **JButton ကို ကိုယ်တိုင်ပြန်ဆွဲ (`paintComponent`) မလုပ်ဘဲ FlatLaf ကို Button အကုန်ဆွဲခိုင်းပြီး Hover ဖြစ်တဲ့အချိန်မှာ FlatLaf ရဲ့ Style Property ကိုပဲ ပြောင်းပေးတာ** ပါ။

တကယ်တော့ FlatLaf မှာ client properties/style ကနေ component styling လုပ်နိုင်တာကြောင့် **simple hover color တစ်ခုတည်းအတွက် custom painting မလိုပါဘူး**။

ဥပမာ Custom Class တစ်ခုလုပ်မယ်ဆိုရင် ဒီလိုထားတာ ပိုသန့်ပါတယ်။

```java
package ui.components;

import java.awt.Cursor;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import javax.swing.JButton;

public class HoverButton extends JButton {

    private String normalStyle =
            "background: #2563EB;"
          + "foreground: #FFFFFF;"
          + "arc: 12;"
          + "borderWidth: 0;"
          + "focusWidth: 0;";

    private String hoverStyle =
            "background: #1D4ED8;"
          + "foreground: #FFFFFF;"
          + "arc: 12;"
          + "borderWidth: 0;"
          + "focusWidth: 0;";

    private String pressedStyle =
            "background: #1E40AF;"
          + "foreground: #FFFFFF;"
          + "arc: 12;"
          + "borderWidth: 0;"
          + "focusWidth: 0;";

    public HoverButton() {
        init();
    }

    private void init() {

        setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));

        putClientProperty("FlatLaf.style", normalStyle);

        addMouseListener(new MouseAdapter() {

            @Override
            public void mouseEntered(MouseEvent e) {
                if (isEnabled()) {
                    putClientProperty("FlatLaf.style", hoverStyle);
                }
            }

            @Override
            public void mouseExited(MouseEvent e) {
                if (isEnabled()) {
                    putClientProperty("FlatLaf.style", normalStyle);
                }
            }

            @Override
            public void mousePressed(MouseEvent e) {
                if (isEnabled()) {
                    putClientProperty("FlatLaf.style", pressedStyle);
                }
            }

            @Override
            public void mouseReleased(MouseEvent e) {

                if (!isEnabled()) {
                    return;
                }

                if (contains(e.getPoint())) {
                    putClientProperty("FlatLaf.style", hoverStyle);
                } else {
                    putClientProperty("FlatLaf.style", normalStyle);
                }
            }
        });
    }
}
```

### ဒီနည်းမှာ ဘာက Professional ဖြစ်တာလဲ?

`paintComponent()` မရှိတာကို သတိထားပါ။

Button ကို ဘယ်လိုဆွဲရမယ်ဆိုတာ **FlatLaf က ဆက်ပြီး တာဝန်ယူထားတယ်**။

ကျွန်တော်တို့က State ပဲပြောတယ်—

```text
Normal
   ↓
Mouse Enter
   ↓
Hover
   ↓
Mouse Press
   ↓
Pressed
   ↓
Mouse Exit
   ↓
Normal
```

ဒါကြောင့် custom painting နဲ့ FlatLaf ကို တိုက်ခိုက်နေတာမျိုး မဖြစ်တော့ပါဘူး။

### ပိုကောင်းတဲ့အဆင့်

တကယ်လို့ သင့်ရည်ရွယ်ချက်က **Button အားလုံးမှာ hover style တူတူဖြစ်စေချင်တာ** ဆိုရင် `HoverButton` class တောင် မလိုအပ်အောင် FlatLaf ရဲ့ global UI defaults / style configuration ကို သုံးနိုင်ပါတယ်။

Custom class ကတော့ **Save / Delete / Update စတဲ့ button type တစ်ခုချင်းစီမှာ normal/hover/pressed behavior ကို reusable component အဖြစ် encapsulate လုပ်ချင်တဲ့အခါ** ပိုအသုံးဝင်ပါတယ်။

ဥပမာ နောက်ဆုံးမှာ—

```text
HoverButton
│
├── PrimaryButton
│      Save
│      Register
│      Login
│
├── DangerButton
│      Delete
│      Remove
│
└── SuccessButton
       Approve
       Confirm
```

လိုမျိုး Design System အသေးစားတစ်ခုတည်ဆောက်နိုင်ပါတယ်။

**အရေးကြီးဆုံးက `setBackground()` ကို mouse event တိုင်းမှာ ပြောင်းတာထက် FlatLaf style ကို ပြောင်းတာက FlatLaf ကိုသုံးနေတဲ့ project အတွက် ပိုသင့်တော်ပါတယ်။**

---
သင့် Project က **NetBeans Drag & Drop + FlatLaf** ဖြစ်တဲ့အတွက် အသုံးပြုနည်းကို Step by Step ပြပါမယ်။

---

# Step 1. HoverButton.java ဖန်တီးပါ

ဥပမာ

```text
src
│
├── ui
│     └── components
│            └── HoverButton.java
```

Code ကို ထည့်ပါ။

ပြီးရင်

**Clean and Build Project**

လုပ်ပါ။

---

# Step 2. Palette ထဲထည့်ပါ

NetBeans Menu

```text
Tools

↓

Palette

↓

Swing/AWT Components

↓

Add From Project...
```

ပြီးရင်

```text
HoverButton
```

ကိုရွေးပါ။

Finish

---

# Step 3. Drag & Drop

Palette ထဲမှာ

```text
HoverButton
```

ပေါ်လာမယ်။

အဲဒါကို

JPanel ပေါ်

Drag & Drop လုပ်ပါ။

ဥပမာ

```text
+-----------------------+

Student Registration

[ HoverButton ]

+-----------------------+
```

---

# Step 4. Properties

Properties မှာ

```text
Text

↓

Save
```

Background

```text
#2563EB
```

Foreground

```text
White
```

---

Run

Result

```text
██████████████

Save

██████████████
```

Hover

```text
██████████████

Save

██████████████
```

ပိုမှောင်လာမယ်။

---

# Step 5. Event

Drag & Drop နဲ့ JButton သုံးသလိုပဲ

Double Click

လုပ်ပါ။

NetBeans က

```java
private void hoverButton1ActionPerformed(
        java.awt.event.ActionEvent evt) {

}
```

Generate ပေးမယ်။

အဲဒီထဲမှာ

```java
JOptionPane.showMessageDialog(
        this,
        "Saved Successfully"
);
```

ထည့်ရုံပါ။

---

# Step 6. Code

လိုချင်ရင်

```java
hoverButton1.setText("Update");
```

ဒါမှမဟုတ်

```java
hoverButton1.setEnabled(false);
```

အားလုံး JButton နဲ့တူတူပဲ။

---

# Button အသစ်ထည့်ချင်ရင်

```text
HoverButton

↓

Copy

↓

Paste
```

ပြီး

Text ပဲပြောင်း။

```text
Save

↓

Update

↓

Delete

↓

Register
```

Hover Effect က

အလိုလိုရတယ်။

---

# Professional Version

တကယ်တော့ Professional Project တွေမှာ

ဒီလို Class တစ်ခုတည်းမလုပ်ဘူး။

ဒီလိုခွဲထားတယ်။

```text
ui

└── components

      ├── PrimaryButton.java

      ├── SuccessButton.java

      ├── DangerButton.java

      ├── WarningButton.java

      └── SecondaryButton.java
```

ဥပမာ

Primary

```text
Blue
```

Success

```text
Green
```

Danger

```text
Red
```

Warning

```text
Orange
```

---

ပြီးရင် Drag & Drop လုပ်ရုံပဲ။

```text
Palette

↓

PrimaryButton

↓

Save
```

```text
Palette

↓

DangerButton

↓

Delete
```

```text
Palette

↓

SuccessButton

↓

Register
```

တစ်ခါရေးပြီး

Project တစ်ခုလုံးမှာ ပြန်သုံးလို့ရတယ်။

---

# ဒါပေမယ့်...

ကျွန်တော် အပေါ်ကပေးထားတဲ့

```java
putClientProperty(
    "FlatLaf.style",
    "background:#2563EB"
);
```

နည်းက **FlatLaf Version ပေါ်မူတည်ပြီး** `revalidate()` သို့မဟုတ် `repaint()` ခေါ်ဖို့ လိုနိုင်ပါတယ်။ အချို့ Version တွေမှာ Hover လုပ်တဲ့အချိန် Style မပြောင်းသလို ဖြစ်နိုင်ပါတယ်။

## သင့်အတွက် အကြံပြုချက်

သင့် Project က **FlatLaf + Student Management System** ဖြစ်ပြီး UI ကို Professional ဖြစ်စေချင်တာဆိုရင် **MouseListener မဟုတ်ဘဲ FlatLaf Style Class** ကို သုံးတာက အကောင်းဆုံး မဟုတ်သေးပါဘူး။

**FlatLaf က အကြံပြုထားတဲ့ Styling API (`FlatClientProperties`) နဲ့ Theme-friendly Button Components** ကို အသုံးပြုတာက ပိုမှန်ပါတယ်။

အဲဒီနည်းနဲ့ဆို

- ✨ Smooth Hover
    
- 🌙 Dark/Light Theme အလိုက် အလိုအလျောက်လိုက်ဖက်
    
- 🎨 Theme ပြောင်းရင် Code မပြင်ရ
    
- ⚡ Performance ပိုကောင်း
    
- 🧩 FlatLaf Design ကို မပျက်စေ
    

ပြီး Professional Application တွေလို Component Library တည်ဆောက်နိုင်ပါတယ်။

**ဒီနည်းက အပေါ်က Custom HoverButton ထက် ပိုအဆင့်မြင့်ပြီး Enterprise Swing Project တွေမှာ အသုံးများတဲ့ နည်းလမ်းဖြစ်ပါတယ်။**

---

အရမ်းကောင်းတဲ့ ရွေးချယ်မှုပါ။

**ဒီနည်းက FlatLaf ကို သုံးရတဲ့ အဓိကအကြောင်းရင်းပါ။**

ကျွန်တော်လည်း **ဒီနည်းကိုပဲ သုံးပါတယ်**။ `paintComponent()` ကို Override လုပ်တာထက် FlatLaf Styling API ကို သုံးတာက ပိုသန့်၊ ပိုမြန်ပြီး Maintain လုပ်ရလည်း လွယ်ပါတယ်။

---

# FlatLaf မှာ Component ကို ဘယ်လို Style လုပ်လဲ?

FlatLaf မှာ Component တိုင်းဟာ

```text
JButton

↓

UI Delegate

↓

FlatLaf
```

နဲ့ Render လုပ်ပါတယ်။

ကျွန်တော်တို့က

```java
button.setBackground(Color.RED);
```

လို ပြောင်းလို့ရတယ်။

ဒါပေမယ့် FlatLaf က

ပိုကောင်းတဲ့နည်းတစ်ခု ပေးထားတယ်။

```java
button.putClientProperty(
        "FlatLaf.style",
        "..."
);
```

ဒါကို **Styling API** လို့ခေါ်တယ်။

---

# Example 1

သာမန် JButton

```java
JButton btn = new JButton("Save");
```

Style

```java
btn.putClientProperty(
        "FlatLaf.style",
        "background:#2563EB;" +
        "foreground:#FFFFFF;"
);
```

Result

```text
██████████████

Save

██████████████
```

---

# Example 2

Rounded Corner

```java
btn.putClientProperty(
    "FlatLaf.style",
    "arc:20"
);
```

Result

```text
╭────────────╮

 Save

╰────────────╯
```

---

# Example 3

Border

```java
btn.putClientProperty(
    "FlatLaf.style",
    "borderWidth:2;" +
    "borderColor:#00AEEF;"
);
```

---

# Example 4

Font

```java
btn.putClientProperty(
    "FlatLaf.style",
    "font:bold +3"
);
```

---

# Example 5

Focus

```java
btn.putClientProperty(
    "FlatLaf.style",
    "focusWidth:0"
);
```

ဒါဆို

Blue Focus Ring မပေါ်တော့ဘူး။

---

# Multiple Style

```java
btn.putClientProperty(
    "FlatLaf.style",
    "arc:15;" +
    "background:#2563EB;" +
    "foreground:#FFFFFF;" +
    "borderWidth:0;" +
    "focusWidth:0;" +
    "font:bold +1"
);
```

ဒီတစ်ကြောင်းတည်းနဲ့

Button တစ်ခုလုံးကို Style လုပ်နိုင်တယ်။

---

# Professional Way

ဒီလိုမရေးဘူး။

```java
btnSave.putClientProperty(...);

btnUpdate.putClientProperty(...);

btnDelete.putClientProperty(...);
```

Button 100 ရှိရင်

100 ကြောင်းရေးရမယ်။

---

Professional တွေက

ဒီလိုရေးတယ်။

```java
public final class ButtonStyles {

    public static final String PRIMARY =
            "arc:15;"
          + "background:#2563EB;"
          + "foreground:#FFFFFF;"
          + "focusWidth:0;"
          + "borderWidth:0;";

    public static final String DANGER =
            "arc:15;"
          + "background:#DC2626;"
          + "foreground:#FFFFFF;"
          + "focusWidth:0;"
          + "borderWidth:0;";

}
```

အသုံးပြုပုံ

```java
btnSave.putClientProperty(
    "FlatLaf.style",
    ButtonStyles.PRIMARY
);

btnDelete.putClientProperty(
    "FlatLaf.style",
    ButtonStyles.DANGER
);
```

ဒီလိုဆို

Project ကြီးလာလည်း

Style ကို

တစ်နေရာတည်းမှာ ပြင်ရုံပဲ။

---

# FlatClientProperties

FlatLaf က

String တစ်ခုတည်းမဟုတ်ဘူး။

Constant တွေလည်းပေးထားတယ်။

ဥပမာ

```java
import com.formdev.flatlaf.FlatClientProperties;
```

ပြီးရင်

```java
button.putClientProperty(
        FlatClientProperties.STYLE,
        "arc:15"
);
```

ဒီလိုရေးလို့ရတယ်။

ဒီဟာက

```java
"FlatLaf.style"
```

နဲ့ အတူတူပါပဲ။

ဒါပေမယ့်

Typo မဖြစ်တော့ဘူး။

---

# Custom Component

Professional Project မှာ

ဒီလိုရေးတယ်။

```java
public class PrimaryButton extends JButton {

    public PrimaryButton() {

        putClientProperty(

                FlatClientProperties.STYLE,

                ButtonStyles.PRIMARY

        );

    }

}
```

အသုံးပြုပုံ

```java
PrimaryButton btn =
        new PrimaryButton();
```

ဘာမှမလုပ်ရတော့ဘူး။

အလိုလို

Blue Theme ဖြစ်နေပြီ။

---

# FlatLaf Design System

Enterprise Project တွေမှာ

```text
ui

└── theme

      ├── Colors.java

      ├── ButtonStyles.java

      ├── TextFieldStyles.java

      ├── TableStyles.java

      └── Theme.java
```

ပြီးရင်

Component

```text
ui

└── components

      ├── PrimaryButton

      ├── DangerButton

      ├── SuccessButton

      ├── RoundedPanel

      └── SearchTextField
```

ဒီ Structure က အရမ်း Maintain လုပ်ရလွယ်ပါတယ်။

---

# သင့်အတွက် Learning Roadmap

သင်က **Student Management System** ကို Professional အဆင့်တည်ဆောက်ချင်တာဆိုတော့ ကျွန်တော် ဒီအစီအစဉ်နဲ့ သင်ပေးချင်ပါတယ်။

1. **Lesson 1** – FlatClientProperties အခြေခံ (`STYLE`, `STYLE_CLASS`)
    
2. **Lesson 2** – FlatLaf Styling Language (`arc`, `background`, `foreground`, `borderWidth`, `focusWidth`, `font`...)
    
3. **Lesson 3** – Theme System (Light/Dark Mode ကို Code မပြင်ဘဲ အလုပ်လုပ်အောင်)
    
4. **Lesson 4** – Reusable Components (`PrimaryButton`, `DangerButton`, `RoundedTextField`, `SearchField`)
    
5. **Lesson 5** – Design System တည်ဆောက်ခြင်း (Colors, Typography, Spacing, Icons)
    

ဒီလမ်းကြောင်းနဲ့သွားရင် Drag & Drop ကို ဆက်သုံးနိုင်သလို UI ကလည်း Professional ဖြစ်လာမယ်။

**ဒီလိုသင်သွားရင် သင့် Student Management System က Java Swing Project တစ်ခုမဟုတ်တော့ဘဲ၊ ကိုယ်ပိုင် UI Component Library ပါတဲ့ Professional Desktop Application တစ်ခု ဖြစ်လာပါလိမ့်မယ်။**



**အဖြေတိုကတော့ — ✅ ရပါတယ်။**

ဒါပေမယ့် **ဘယ်လိုသုံးမလဲ** ဆိုတာကို နားလည်ဖို့ လိုပါတယ်။

---

# နည်းလမ်း (၁) - Drag & Drop + Code (⭐⭐⭐⭐⭐ Recommended)

ဒီနည်းက Professional Project တွေမှာ အသုံးအများဆုံးပါ။

NetBeans မှာ

Drag & Drop နဲ့

```text
JButton
```

ချလိုက်ပါ။

NetBeans က

```java
private javax.swing.JButton btnSave;
```

ကို Generate လုပ်ပေးမယ်။

ပြီးရင် Constructor ထဲမှာ

```java
public StudentForm() {
    initComponents();

    btnSave.putClientProperty(
        FlatClientProperties.STYLE,
        ButtonStyles.PRIMARY
    );
}
```

ဒါပဲ။

UI ကိုတော့ Drag & Drop နဲ့ ဆောက်တယ်။

Style ကိုတော့ Code နဲ့ ထည့်တယ်။

---

# နည်းလမ်း (၂) - Custom Component + Drag & Drop (⭐⭐⭐⭐)

ဥပမာ

```java
public class PrimaryButton extends JButton {

    public PrimaryButton() {

        putClientProperty(
            FlatClientProperties.STYLE,
            ButtonStyles.PRIMARY
        );

    }

}
```

ပြီးရင်

Palette

↓

Add From Project

↓

PrimaryButton

Drag & Drop

ဒါလည်းရတယ်။

---

# နည်းလမ်း (၃) - Palette ထဲက JButton ကို Override (❌ မလုပ်သင့်)

ဒီနည်းကို မလုပ်ပါနဲ့။

ဘာလို့လဲဆိုတော့

NetBeans Update

FlatLaf Update

ဖြစ်တဲ့အချိန်

ပြဿနာတက်နိုင်တယ်။

---

# ကျွန်တော် ဘယ်နည်းသုံးလဲ?

ဥပမာ

Student Management System

```text
Login Form

Register Form

Student Form

Teacher Form

Attendance Form
```

ရှိတယ်ဆိုပါစို့။

Drag & Drop နဲ့

Form အားလုံးဆောက်မယ်။

ပြီးရင်

```java
private void applyStyles() {

    btnSave.putClientProperty(
        FlatClientProperties.STYLE,
        ButtonStyles.PRIMARY
    );

    btnDelete.putClientProperty(
        FlatClientProperties.STYLE,
        ButtonStyles.DANGER
    );

    btnUpdate.putClientProperty(
        FlatClientProperties.STYLE,
        ButtonStyles.SUCCESS
    );

}
```

Constructor

```java
public StudentForm() {

    initComponents();

    applyStyles();

}
```

ဒါက အရမ်းသန့်တယ်။

---

# ပိုပြီး Professional ဖြစ်တဲ့ နည်း

Form တိုင်းမှာ

```java
btnSave.putClientProperty(...)
```

ရေးနေရတာတောင် မကြိုက်ဘူးဆိုရင်

ဒီလို Utility Class လုပ်တယ်။

```java
public final class ThemeManager {

    public static void primary(JButton button) {

        button.putClientProperty(
                FlatClientProperties.STYLE,
                ButtonStyles.PRIMARY);

    }

    public static void danger(JButton button) {

        button.putClientProperty(
                FlatClientProperties.STYLE,
                ButtonStyles.DANGER);

    }

}
```

အသုံးပြုပုံ

```java
ThemeManager.primary(btnSave);

ThemeManager.primary(btnLogin);

ThemeManager.danger(btnDelete);
```

Code က ပိုသန့်သွားတယ်။

---

# Hover Effect ကရော?

ဒါက FlatLaf ရဲ့ လက်ရှိအခြေအနေကို နားလည်ထားဖို့လိုပါတယ်။

`FlatClientProperties.STYLE` က **static style** (background, arc, font, border...) တွေကို သတ်မှတ်ဖို့ အဓိကသုံးတာပါ။

**Mouse hover state ကို STYLE တစ်ခုတည်းနဲ့ မပြောင်းနိုင်ပါဘူး။**

Hover ဖြစ်တဲ့အချိန်မှာ

```java
mouseEntered
```

မှာ

```java
button.putClientProperty(
    FlatClientProperties.STYLE,
    ButtonStyles.PRIMARY_HOVER
);
button.repaint();
```

လိုမျိုး Style ပြောင်းပေးရပါတယ်။

ဒါမှမဟုတ် FlatLaf theme (.properties/.json) ထဲမှာ hover colors ကို သတ်မှတ်နိုင်ပါတယ်။

---

## ကျွန်တော့်ရဲ့ အကြံပြုချက် (သင့် Project အတွက်)

သင့် **Student Management System** က ကြီးလာမယ့် Project ဖြစ်တဲ့အတွက် ဒီ Structure ကို သုံးပါ။

```text
src
│
├── theme
│   ├── AppColors.java
│   ├── ButtonStyles.java
│   └── ThemeManager.java
│
├── ui
│   ├── LoginForm.java
│   ├── StudentForm.java
│   └── Dashboard.java
```

- ✅ UI ကို NetBeans **Drag & Drop** နဲ့ ဆောက်ပါ။
    
- ✅ Style ကို `ThemeManager` နဲ့ Apply လုပ်ပါ။
    
- ✅ Reusable Button Styles (`PRIMARY`, `SUCCESS`, `DANGER`) ထားပါ။
    
- ✅ FlatLaf ကို Rendering တာဝန်ပေးထားပြီး Custom Painting ကို မလိုအပ်မှသာ ရေးပါ။
    

ဒီနည်းက Maintain လုပ်ရလွယ်ပြီး Project ကြီးလာတဲ့အခါ အကျိုးအရှိဆုံးပါ။