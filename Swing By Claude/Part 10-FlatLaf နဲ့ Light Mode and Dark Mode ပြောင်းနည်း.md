# FlatLaf နဲ့ Light Mode / Dark Mode ပြောင်းနည်း — လက်တွေ့ ဥပမာ

## 1. Dependency ထည့်ရန် (Maven)

```xml
<dependency>
    <groupId>com.formdev</groupId>
    <artifactId>flatlaf</artifactId>
    <version>3.4</version>
</dependency>
```

## 2. Basic Toggle Button ဥပမာ

Button တစ်ခုနှိပ်ရင် Light နဲ့ Dark theme ချက်ချင်းပြောင်းသွားမယ့် full example ပါ:

```java
import com.formdev.flatlaf.FlatLightLaf;
import com.formdev.flatlaf.FlatDarkLaf;
import javax.swing.*;
import java.awt.*;

public class ThemeSwitchDemo {
    private static boolean isDark = false;

    public static void main(String[] args) {
        // App စတင်တဲ့အခါ Light mode default ထားမယ်
        FlatLightLaf.setup();

        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Theme Switch Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 300);
            frame.setLayout(new FlowLayout());

            JLabel label = new JLabel("Hello FlatLaf!");
            JButton toggleButton = new JButton("Switch to Dark Mode");

            // Button နှိပ်တဲ့အခါ theme ပြောင်းမယ့် logic
            toggleButton.addActionListener(e -> {
                isDark = !isDark;
                try {
                    if (isDark) {
                        UIManager.setLookAndFeel(new FlatDarkLaf());
                        toggleButton.setText("Switch to Light Mode");
                    } else {
                        UIManager.setLookAndFeel(new FlatLightLaf());
                        toggleButton.setText("Switch to Dark Mode");
                    }
                    // အရေးကြီးတဲ့ အဆင့် - UI ကို refresh လုပ်ရန်
                    SwingUtilities.updateComponentTreeUI(frame);
                } catch (UnsupportedLookAndFeelException ex) {
                    ex.printStackTrace();
                }
            });

            frame.add(label);
            frame.add(toggleButton);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

## 3. အရေးကြီးဆုံး အချက် — `updateComponentTreeUI()`

Theme ပြောင်းတဲ့အခါ `UIManager.setLookAndFeel()` ကို ခေါ်ရုံနဲ့ **UI ချက်ချင်းမပြောင်းပါ**။ Frame ပေါ်က component အားလုံးကို UI အသစ်ဖြင့် refresh လုပ်ဖို့ **`SwingUtilities.updateComponentTreeUI(frame)`** ကို မဖြစ်မနေ ခေါ်ရပါမယ်။

- Frame တစ်ခုတည်း ဆိုရင် `updateComponentTreeUI(frame)`
- Frame ပေါင်းများစွာ ရှိရင် ဖွင့်ထားသမျှ frame တိုင်းအတွက် loop ပတ်ပြီး ခေါ်ပေးရပါမယ်:

```java
for (Window window : Window.getWindows()) {
    SwingUtilities.updateComponentTreeUI(window);
}
```

## 4. System-wide Dark Mode ကို Auto Detect လုပ်ချင်ရင်

FlatLaf မှာ OS ရဲ့ current theme (Windows/Mac dark mode setting) ကို detect လုပ်ပေးတဲ့ helper method လည်း ပါပါတယ်:

```java
// Windows/Mac ရဲ့ system theme အလိုက် auto ရွေးမယ်
boolean systemIsDark = FlatLaf.isLafDark();
```

## 5. Smooth Animation ချင်ရင် (Optional)

Theme switch effect ကို ပိုချောမွေ့စေချင်ရင် FlatLaf Extras package ထဲက animation utilities တွေ ရှိပါတယ်၊ ဒါပေမယ့် basic learning stage မှာတော့ အပေါ်က code ပဲ လုံလောက်ပါတယ်။

---

# ဟုတ်ကဲ့၊ မှန်ပါတယ် — Custom Color သတ်မှတ်ပုံ လက်တွေ့ ဥပမာ

Component တစ်ခုချင်းစီရဲ့ color ကို hardcode လုပ်ထားရင် theme ပြောင်းတဲ့အခါ **ပြောင်းသွားမှာ မဟုတ်ပါဘူး** (FlatLaf ရဲ့ default colors ကို override လုပ်ထားလို့ပါ)။ ဒါကြောင့် "theme-aware" ဖြစ်အောင် ဘယ်လိုလုပ်ရမလဲ ကြည့်ကြရအောင်။

## 1. ❌ မကောင်းတဲ့ နည်း (Hardcoded Color)

```java
JButton button = new JButton("Click Me");
button.setBackground(Color.WHITE);   // Dark mode မှာ ဆက်ဖြူနေမယ်
button.setForeground(Color.BLACK);   // ပြောင်းမှာ မဟုတ်ဘူး
```

ဒီလိုမျိုး fix လုပ်ထားရင် Dark mode ပြောင်းလိုက်ရင်တောင် button ကတော့ white background ပဲ ဆက်ရှိနေမှာပါ။

## 2. ✅ ကောင်းတဲ့ နည်း (1) — UIManager Default Colors သုံးနည်း

FlatLaf ရဲ့ theme-defined colors (`UIManager.getColor()`) ကို reference လုပ်ပါ။ Theme ပြောင်းတဲ့အခါ auto ပြောင်းသွားပါလိမ့်မယ်:

```java
JPanel panel = new JPanel();
panel.setBackground(UIManager.getColor("Panel.background"));

JLabel label = new JLabel("Hello");
label.setForeground(UIManager.getColor("Label.foreground"));

JButton button = new JButton("Submit");
button.setBackground(UIManager.getColor("Button.background"));
button.setForeground(UIManager.getColor("Button.foreground"));
```

`updateComponentTreeUI()` ခေါ်တဲ့အခါ UIManager values တွေ auto refresh ဖြစ်သွားလို့ color တွေလည်း update ဖြစ်သွားပါလိမ့်မယ်။

## 3. ✅ ကောင်းတဲ့ နည်း (2) — Custom Color ကို သီးသန့် သတ်မှတ်ချင်ရင် (`.properties` file)

သင့် app အတွက် custom brand color (ဥပမာ - special button color) ကို light/dark နှစ်မျိုးလုံးအတွက် သီးခြား သတ်မှတ်ချင်ရင် FlatLaf properties file နှစ်ခု ဖန်တီးနိုင်ပါတယ်:

**`FlatLightLafCustom.properties`**

```properties
MyButton.background = #4CAF50
MyButton.foreground = #FFFFFF
```

**`FlatDarkLafCustom.properties`**

```properties
MyButton.background = #2E7D32
MyButton.foreground = #E0E0E0
```

ပြီးရင် code ထဲမှာ:

```java
button.putClientProperty("FlatLaf.styleClass", "MyButton");
```

## 4. ✅ ကောင်းတဲ့ နည်း (3) — Runtime မှာ တိုက်ရိုက် Style (Inline)

Properties file မလိုချင်ရင် `putClientProperty("FlatLaf.style", ...)` နဲ့လည်း တိုက်ရိုက် ရေးလို့ရပါတယ်:

```java
button.putClientProperty("FlatLaf.style",
    "background: #4CAF50; foreground: #FFFFFF; borderWidth: 0; arc: 8");
```

ဒီနည်းက component တစ်ခုချင်းစီအတွက် quick styling လုပ်ချင်ရင် အသုံးဝင်ပေမယ့် — Light/Dark အလိုက် **auto မပြောင်းပါ**၊ manually logic ရေးပေးရပါမယ်။

## 5. Theme ပြောင်းတဲ့အခါ Custom Color ကိုပါ ပြောင်းချင်ရင် (Practical Full Example)

```java
private static boolean isDark = false;

toggleButton.addActionListener(e -> {
    isDark = !isDark;
    try {
        UIManager.setLookAndFeel(isDark ? new FlatDarkLaf() : new FlatLightLaf());

        // Custom color ကို manual update
        myButton.setBackground(isDark ? new Color(46, 125, 50) : new Color(76, 175, 80));
        myLabel.setForeground(isDark ? Color.LIGHT_GRAY : Color.DARK_GRAY);

        SwingUtilities.updateComponentTreeUI(frame);
    } catch (Exception ex) {
        ex.printStackTrace();
    }
});
```

---

## အကျဉ်းချုပ် (Summary)

|နည်းလမ်း|Auto Theme-aware?|Use Case|
|---|---|---|
|`UIManager.getColor()`|✅ ဟုတ်ပါတယ်|Default look ကို ဒီအတိုင်းသုံးချင်ရင်|
|`.properties` file|✅ ဟုတ်ပါတယ်|Custom brand color ကို theme နှစ်ခုလုံးအတွက် သတ်မှတ်ချင်ရင်|
|Inline style|❌ Manual ရေးရမယ်|Quick, one-off styling|
|Hardcoded `setBackground()`|❌|မသုံးသင့်ပါ|

Practically ဆိုရင် **UIManager.getColor() ကို primary color source** အနေနဲ့သုံးပြီး၊ special/branded color တွေအတွက်ပဲ `.properties` file approach ကို ထပ်ဖြည့်သုံးတာက best practice ဖြစ်ပါတယ်။

Panel, Button, Label အားလုံးပါတဲ့ complete demo application တစ်ခု အပြည့်အစုံ code ရေးပြပေးရမလား?