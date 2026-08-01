## FlatLaf Setup - Modern Look and Feel

FlatLaf ဟာ Java Swing application တွေကို modern, flat design (IntelliJ IDEA/VS Code style) အနေနဲ့ ပြောင်းပေးတဲ့ **Look and Feel library** ပါ။ Default Swing UI (Metal, Windows classic look) ထက် professional ပြီး modern ဖြစ်ပါတယ်။

### Step 1: FlatLaf Library ထည့်ခြင်း

**Maven (pom.xml)**

```xml
<dependency>
    <groupId>com.formdev</groupId>
    <artifactId>flatlaf</artifactId>
    <version>3.5.4</version>
</dependency>
```

**Gradle (build.gradle)**

```groovy
implementation 'com.formdev:flatlaf:3.5.4'
```

**Manual JAR (Maven/Gradle မသုံးရင်)**

FlatLaf ရဲ့ official site (https://www.formdev.com/flatlaf/) ကနေ jar file download လုပ်ပြီး project ရဲ့ classpath ထဲ ထည့်ရပါမယ် (IDE မှာ Project Structure → Libraries → Add)။

**Note:** version number တွေက အချိန်နဲ့အမျှ update ဖြစ်နေတတ်လို့ setup လုပ်တဲ့အချိန်မှာ [Maven Central](https://mvnrepository.com/artifact/com.formdev/flatlaf) ကနေ latest version ကို double-check လုပ်သင့်ပါတယ်။

### Step 2: FlatLaf ကို Application ထဲ Activate လုပ်ခြင်း

`main()` method ရဲ့ **အစောဆုံး** မှာ Look and Feel ကို set လုပ်ရပါတယ် (UI components create လုပ်ခင်)။

```java
import com.formdev.flatlaf.FlatLightLaf;
import javax.swing.*;

public class MyApp {
    public static void main(String[] args) {
        // FlatLaf ကို UI components create လုပ်ခင် အရင်ဆုံး set လုပ်ရမယ်
        FlatLightLaf.setup();
        
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("My FlatLaf App");
            frame.add(new JButton("Click Me"));
            frame.setSize(400, 300);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

### FlatLaf ရဲ့ Built-in Themes (Light / Dark)

```java
import com.formdev.flatlaf.FlatLightLaf;
import com.formdev.flatlaf.FlatDarkLaf;
import com.formdev.flatlaf.FlatIntelliJLaf;
import com.formdev.flatlaf.FlatDarculaLaf;

// Light theme
FlatLightLaf.setup();

// Dark theme
FlatDarkLaf.setup();

// IntelliJ-style light theme
FlatIntelliJLaf.setup();

// Darcula-style dark theme (IntelliJ dark)
FlatDarculaLaf.setup();
```

**Alternative syntax (UIManager သုံးပြီး):**

```java
try {
    UIManager.setLookAndFeel(new FlatLightLaf());
} catch (UnsupportedLookAndFeelException e) {
    System.err.println("Failed to initialize FlatLaf");
}
```

### Theme ကို Runtime မှာ Switch လုပ်ခြင်း (Light ↔ Dark)

```java
JButton themeToggleBtn = new JButton("Toggle Dark Mode");
boolean[] isDark = {false};

themeToggleBtn.addActionListener(e -> {
    try {
        if (isDark[0]) {
            UIManager.setLookAndFeel(new FlatLightLaf());
        } else {
            UIManager.setLookAndFeel(new FlatDarkLaf());
        }
        isDark[0] = !isDark[0];
        
        // Existing components တွေကို theme update ဖြစ်အောင် refresh လုပ်ရမယ်
        SwingUtilities.updateComponentTreeUI(frame);
    } catch (UnsupportedLookAndFeelException ex) {
        ex.printStackTrace();
    }
});
```

`SwingUtilities.updateComponentTreeUI(frame)` ကို **မမေ့ပါနဲ့** - ဒါမှ already-created components တွေရဲ့ appearance ကို theme အသစ်နဲ့ update လုပ်ပေးမှာပါ (window ကို ပြန် restart လုပ်စရာ မလိုတော့ပါ)။

### FlatLaf UI Customization (Custom Colors/Properties)

FlatLaf မှာ UI properties တွေကို customize လုပ်နိုင်ပါတယ်:

```java
UIManager.put("Button.arc", 15);  // button corner ကို round လုပ်ဖို့
UIManager.put("Component.focusColor", Color.BLUE);
UIManager.put("TextComponent.arc", 10);
```

### Component-level Styling (Client Properties)

FlatLaf က component တစ်ခုချင်းစီအတွက်လည်း specific styling ပေးနိုင်ပါတယ်:

```java
JButton primaryBtn = new JButton("Primary Action");
primaryBtn.putClientProperty("JButton.buttonType", "roundRect");

JTextField searchField = new JTextField();
searchField.putClientProperty("JTextField.placeholderText", "Search...");

JTextField iconField = new JTextField();
iconField.putClientProperty("JTextField.showClearButton", true);
```

### Complete Example

```java
import com.formdev.flatlaf.FlatDarkLaf;
import javax.swing.*;
import java.awt.*;

public class FlatLafExample {
    public static void main(String[] args) {
        FlatDarkLaf.setup();  // dark theme ကို activate
        
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("FlatLaf Demo");
            JPanel panel = new JPanel(new FlowLayout());
            
            JTextField textField = new JTextField(15);
            textField.putClientProperty("JTextField.placeholderText", "Enter your name");
            
            JButton button = new JButton("Submit");
            
            panel.add(new JLabel("Name:"));
            panel.add(textField);
            panel.add(button);
            
            frame.add(panel);
            frame.setSize(400, 150);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

### FlatLaf Extras (Optional Add-ons)

FlatLaf မှာ additional feature တွေအတွက် extra library တွေရှိပါတယ်:

```xml
<!-- Extra icons/components -->
<dependency>
    <groupId>com.formdev</groupId>
    <artifactId>flatlaf-extras</artifactId>
    <version>3.5.4</version>
</dependency>

<!-- SVG icon support -->
<dependency>
    <groupId>com.formdev</groupId>
    <artifactId>flatlaf-swingx</artifactId>
    <version>3.5.4</version>
</dependency>
```

### Important Notes

1. **Order matters** - `FlatXxxLaf.setup()` ကို UI create လုပ်ခင် **အရင်ဆုံး** ခေါ်ရပါမယ်
2. **JDK compatibility** - FlatLaf က Java 8+ support လုပ်ပါတယ်
3. **Existing custom colors/fonts** - Look and Feel ပြောင်းသွားတဲ့အခါ hardcode လုပ်ထားတဲ့ colors တွေက override ဖြစ်နိုင်လို့ FlatLaf-compatible UIManager color keys တွေ သုံးတာက ပိုကောင်းပါတယ်

---

Setup ကို run ကြည့်ပြီးလား? Error တွေ့ရင် (ဥပမာ - dependency resolve မဖြစ်တာ) ပြောနိုင်ပါတယ်။