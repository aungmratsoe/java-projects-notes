## Part 1: JFrame + Basic Components

### JFrame ဆိုတာဘာလဲ

`JFrame` ဟာ Swing application ရဲ့ **main window** ပါ။ Title bar, minimize/maximize/close button တွေပါတဲ့ standard window တစ်ခုပါ။

```java
import javax.swing.*;

public class MyFirstApp {
    public static void main(String[] args) {
        // EDT (Event Dispatch Thread) ပေါ်မှာ UI ကို run ဖို့
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("My First Swing App");  // window title
            frame.setSize(400, 300);                           // width, height
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // X ကိုနှိပ်ရင် program ပိတ်ဖို့
            frame.setLocationRelativeTo(null);                 // screen center မှာပေါ်ဖို့
            frame.setVisible(true);                            // window ကိုပြသဖို့
        });
    }
}
```

**Line တစ်ခုချင်းစီရဲ့ အဓိပ္ပါယ်:**

- `SwingUtilities.invokeLater()` - Swing components တွေကို EDT (special thread) ပေါ်မှာပဲ create/update လုပ်ရမယ်ဆိုတဲ့ rule ကို follow လုပ်ဖို့ (မလုပ်ရင် unpredictable bugs ဖြစ်နိုင်ပါတယ်)
- `setDefaultCloseOperation()` - window ပိတ်တဲ့အခါ ဘာဖြစ်စေချင်လဲ (`EXIT_ON_CLOSE`, `DISPOSE_ON_CLOSE`, `DO_NOTHING_ON_CLOSE`)
- `setLocationRelativeTo(null)` - window ကို screen center မှာ ပေါ်စေချင်ရင် သုံးပါတယ်
- `setVisible(true)` - **ဒါကို မမေ့ပါနဲ့!** မထည့်ရင် window ဘာမှ မမြင်ရပါဘူး

### JPanel ဆိုတာဘာလဲ

`JPanel` ဟာ **container** တစ်ခုပါ - components (button, label, textfield) တွေကို ထည့်ဖို့ အသုံးပြုပါတယ်။ JFrame ထဲမှာ JPanel တွေ multiple ခု ထည့်နိုင်ပြီး UI ကို organize လုပ်ရလွယ်ကူစေပါတယ်။

```java
JFrame frame = new JFrame("Panel Example");
JPanel panel = new JPanel();  // container ဖန်တီး

panel.add(new JLabel("Hello Swing!"));  // panel ထဲကို label ထည့်

frame.add(panel);  // frame ထဲကို panel ထည့်
frame.setSize(400, 300);
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setVisible(true);
```

### Basic Components များ

#### 1. JLabel (Text/Image ပြသဖို့)

```java
JLabel label = new JLabel("This is a label");
JLabel imageLabel = new JLabel(new ImageIcon("image.png"));  // image ပြသဖို့
```

#### 2. JButton (ခလုတ်)

```java
JButton button = new JButton("Click Me");
```

#### 3. JTextField (တစ်ကြောင်းတည်း text input)

```java
JTextField textField = new JTextField(20);  // 20 = column width
String text = textField.getText();  // user ရေးထားတဲ့ text ကို ရယူဖို့
textField.setText("Default text");  // text ချမှတ်ဖို့
```

#### 4. JTextArea (multi-line text input)

```java
JTextArea textArea = new JTextArea(5, 20);  // 5 rows, 20 columns
textArea.setLineWrap(true);  // စာကြောင်း အလိုအလျောက် wrap လုပ်ဖို့
```

#### 5. JCheckBox

```java
JCheckBox checkBox = new JCheckBox("I agree");
boolean isChecked = checkBox.isSelected();  // check ဖြစ်/မဖြစ် စစ်ဖို့
```

#### 6. JRadioButton (Group ချထားရင် တစ်ခုပဲ ရွေးလို့ရ)

```java
JRadioButton option1 = new JRadioButton("Option 1");
JRadioButton option2 = new JRadioButton("Option 2");

ButtonGroup group = new ButtonGroup();  // group တစ်ခုတည်း ဖြစ်အောင်
group.add(option1);
group.add(option2);
```

#### 7. JComboBox (Dropdown)

```java
String[] items = {"Apple", "Banana", "Orange"};
JComboBox<String> comboBox = new JComboBox<>(items);
String selected = (String) comboBox.getSelectedItem();
```

### Full Example - Components တွေအားလုံး ပေါင်းထည့်ခြင်း

```java
import javax.swing.*;

public class SwingBasicsExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Swing Basics");
            JPanel panel = new JPanel();  // default layout: FlowLayout

            JLabel nameLabel = new JLabel("Name:");
            JTextField nameField = new JTextField(15);
            JButton submitButton = new JButton("Submit");
            JCheckBox subscribeBox = new JCheckBox("Subscribe to newsletter");

            panel.add(nameLabel);
            panel.add(nameField);
            panel.add(subscribeBox);
            panel.add(submitButton);

            frame.add(panel);
            frame.setSize(400, 200);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

ဒါကို run ရင် window တစ်ခုမှာ label, text field, checkbox, button တွေ ပေါ်လာမှာပါ (ဒါပေမယ့် event ဘာမှ မလုပ်သေးလို့ button ကိုနှိပ်ရင် ဘာမှ ဖြစ်မှာမဟုတ်ပါဘူး - ဒါက Part 3 Event Handling မှာ သင်ပါမယ်)။

---

ဒါက **Part 1 (JFrame + Basic Components)** ပါ။ code ကို ကိုယ်တိုင် run ကြည့်ပြီးလား? Question ရှိရင် မေးလို့ရပါတယ်။ နားလည်ပြီဆိုရင် **Part 2 (Layout Managers)** ကို ဆက်သွားမလား?