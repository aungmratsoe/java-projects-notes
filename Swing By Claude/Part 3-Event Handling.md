## Part 3: Event Handling

Event Handling ဟာ Swing ရဲ့ **core concept** ပါ။ User က button နှိပ်တာ၊ text ရိုက်တာ၊ mouse ရွှေ့တာ - ဒါတွေအားလုံးက **"events"** ဖြစ်ပြီး ဒီ events တွေကို ဘယ်လို "listen" လုပ်ပြီး response ပြန်မလဲဆိုတာကို ဒီ part မှာ သင်ပေးပါမယ်။

### Event Handling ရဲ့ Basic Concept

Swing event handling က **3 components** နဲ့ ဆက်နွယ်ပါတယ်:

1. **Event Source** - event ဖြစ်ပေါ်စေတဲ့ component (ဥပမာ - JButton)
2. **Event Listener** - event ဖြစ်တာကို "စောင့်ကြည့်" နေတဲ့ interface
3. **Event Object** - event အကြောင်းအရာ information ပါဝင်တဲ့ object

```java
// Pattern: source.addXxxListener(listener)
button.addActionListener(listener);
```

### ActionListener (အသုံးအများဆုံး)

Button click, menu selection, Enter key press (text field) တွေအတွက် သုံးပါတယ်။

**Method 1: Anonymous Inner Class (Traditional way)**

```java
JButton button = new JButton("Click Me");

button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("Button clicked!");
    }
});
```

**Method 2: Lambda Expression (Modern way - Java 8+, Recommended)**

```java
JButton button = new JButton("Click Me");

button.addActionListener(e -> {
    System.out.println("Button clicked!");
});
```

ActionListener interface မှာ method **တစ်ခုတည်း** (`actionPerformed`) ပဲရှိလို့ lambda expression နဲ့ ရေးလို့ရတာပါ (**functional interface** လို့ခေါ်ပါတယ်)။

### Practical Example - Counter App

```java
import javax.swing.*;
import java.awt.*;

public class CounterApp {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Counter");
            JPanel panel = new JPanel();
            
            JLabel countLabel = new JLabel("Count: 0");
            JButton incrementBtn = new JButton("+1");
            JButton resetBtn = new JButton("Reset");
            
            // counter variable - lambda ထဲက access လုပ်ဖို့ array (effectively final requirement)
            int[] count = {0};
            
            incrementBtn.addActionListener(e -> {
                count[0]++;
                countLabel.setText("Count: " + count[0]);
            });
            
            resetBtn.addActionListener(e -> {
                count[0] = 0;
                countLabel.setText("Count: " + count[0]);
            });
            
            panel.add(countLabel);
            panel.add(incrementBtn);
            panel.add(resetBtn);
            
            frame.add(panel);
            frame.setSize(300, 150);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

**Note:** Lambda expression ထဲမှာ local variable ကို သုံးချင်ရင် "effectively final" ဖြစ်ရပါမယ် (value ပြောင်းလို့မရဘူး)။ ဒါကြောင့် counter လို value ပြောင်းရမယ့် variable တွေအတွက် array (`int[]`) ဒါမှမဟုတ် instance variable ကို သုံးရပါတယ်။

### MouseListener (Mouse Events)

```java
JPanel panel = new JPanel();

panel.addMouseListener(new MouseAdapter() {  // MouseAdapter သုံးတာက method အားလုံး override မလုပ်ချင်လို့
    @Override
    public void mouseClicked(MouseEvent e) {
        System.out.println("Mouse clicked at: " + e.getX() + ", " + e.getY());
    }
    
    @Override
    public void mouseEntered(MouseEvent e) {
        System.out.println("Mouse entered panel");
    }
    
    @Override
    public void mouseExited(MouseEvent e) {
        System.out.println("Mouse exited panel");
    }
});
```

**MouseListener** မှာ method 5 ခုရှိလို့ (`mouseClicked`, `mousePressed`, `mouseReleased`, `mouseEntered`, `mouseExited`) interface ကို တိုက်ရိုက် implement လုပ်ရင် **အားလုံးကို override လုပ်ရပါမယ်**။ ဒါကြောင့် **MouseAdapter** (abstract class) ကို extend လုပ်ပြီး **လိုချင်တဲ့ method ကိုပဲ** override လုပ်တာက ပိုအဆင်ပြေပါတယ်။

### KeyListener (Keyboard Events)

```java
JTextField textField = new JTextField(20);

textField.addKeyListener(new KeyAdapter() {
    @Override
    public void keyPressed(KeyEvent e) {
        if (e.getKeyCode() == KeyEvent.VK_ENTER) {
            System.out.println("Enter key pressed! Text: " + textField.getText());
        }
    }
});
```

**Tip:** Text field အတွက် Enter key ကို detect ဖို့ `KeyListener` ထက် `ActionListener` သုံးတာက ပိုလွယ်ပါတယ် (text field က default ActionListener ကို support လုပ်ပါတယ်):

```java
textField.addActionListener(e -> {
    System.out.println("Enter pressed! Text: " + textField.getText());
});
```

### ItemListener (CheckBox, RadioButton, ComboBox အတွက်)

Selection state ပြောင်းတာကို detect ဖို့ သုံးပါတယ်:

```java
JCheckBox checkBox = new JCheckBox("Subscribe");

checkBox.addItemListener(e -> {
    if (e.getStateChange() == ItemEvent.SELECTED) {
        System.out.println("Checked!");
    } else {
        System.out.println("Unchecked!");
    }
});
```

### WindowListener (Window Events)

Window close, minimize, maximize စတဲ့ events တွေအတွက်:

```java
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowClosing(WindowEvent e) {
        int result = JOptionPane.showConfirmDialog(
            frame, "ထွက်မှာ သေချာပါသလား?", "Confirm", JOptionPane.YES_NO_OPTION
        );
        if (result == JOptionPane.YES_OPTION) {
            System.exit(0);
        }
    }
});
```

**Note:** ဒီလိုသုံးမယ်ဆိုရင် `setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE)` ထားပေးရပါမယ် (မဟုတ်ရင် confirm dialog မပေါ်ခင် window ပိတ်သွားနိုင်ပါတယ်)။

### Complete Example - Login Form with Event Handling

```java
import javax.swing.*;
import java.awt.*;

public class LoginForm {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Login");
            frame.setLayout(new GridLayout(3, 2, 5, 5));
            
            JLabel userLabel = new JLabel("Username:");
            JTextField userField = new JTextField();
            JLabel passLabel = new JLabel("Password:");
            JPasswordField passField = new JPasswordField();  // password hidden
            JButton loginButton = new JButton("Login");
            JLabel statusLabel = new JLabel("");
            
            loginButton.addActionListener(e -> {
                String username = userField.getText();
                String password = new String(passField.getPassword());
                
                if (username.equals("admin") && password.equals("1234")) {
                    statusLabel.setText("Login Success!");
                } else {
                    statusLabel.setText("Invalid credentials!");
                }
            });
            
            frame.add(userLabel);
            frame.add(userField);
            frame.add(passLabel);
            frame.add(passField);
            frame.add(loginButton);
            frame.add(statusLabel);
            
            frame.setSize(300, 150);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

### Quick Summary

|Listener|ဘယ်အခါသုံးလဲ|
|---|---|
|`ActionListener`|Button click, menu, Enter key (text field)|
|`MouseListener/Adapter`|Mouse click, enter, exit|
|`KeyListener/Adapter`|Keyboard press/release|
|`ItemListener`|CheckBox, RadioButton, ComboBox selection|
|`WindowListener/Adapter`|Window open/close/minimize|

---

ဒါက **Part 3 (Event Handling)** ပါ။ Code တွေ run ကြည့်ပြီးလား? နားလည်ပြီဆိုရင် **Part 4 (Dialogs - JOptionPane, JFileChooser)** ကို ဆက်သွားမလား?