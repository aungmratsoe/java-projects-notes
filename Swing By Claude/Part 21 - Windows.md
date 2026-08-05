Swing window-level component (top-level container) တွေရဲ့ အသုံးပြုပုံကို ရှင်းပြပေးပါမယ်။

## 1. **JFrame** (Frame)

- Application ရဲ့ **main window** ပါ - title bar, minimize/maximize/close button ပါတဲ့ standalone window။ App တစ်ခုမှာ ပုံမှန် JFrame တစ်ခုပဲ ရှိလေ့ရှိတယ် (main window)။

```java
public class MainFrame extends JFrame {
    public MainFrame() {
        setTitle("Student Management");
        setSize(800, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // X နှိပ်ရင် app ပိတ်ဖို့
        setLocationRelativeTo(null); // screen အလယ်မှာ ပေါ်ဖို့

        JPanel panel = new JPanel();
        panel.add(new JLabel("Welcome"));
        add(panel);

        setVisible(true);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new MainFrame());
    }
}
```

**Close operation options များ**:

- `EXIT_ON_CLOSE` — app တစ်ခုလုံး ပိတ်တယ် (main window အတွက်)
- `DISPOSE_ON_CLOSE` — window ကိုပဲ ပိတ်တယ်၊ app ဆက်run နေတယ် (secondary window အတွက်)

## 2. **JDialog** (Dialog)

- Frame ပေါ်က **popup window** ဖြစ်ပြီး၊ user input တောင်းခံတာ၊ confirmation၊ error message ပြတာစတဲ့ အလုပ်တွေအတွက် သုံးတယ်။
- Frame ကို "parent" အဖြစ် ချိတ်ပြီး ဖွင့်ရတယ်။

```java
// Custom JDialog (student edit form example)
JDialog editDialog = new JDialog(mainFrame, "Edit Student", true); 
// true = modal (dialog ပိတ်မချင်း parent frame ကို back မထွားနိုင်)
editDialog.setSize(400, 300);
editDialog.setLocationRelativeTo(mainFrame);

JPanel dialogPanel = new JPanel();
JTextField nameField = new JTextField(20);
JButton saveBtn = new JButton("Save");

saveBtn.addActionListener(e -> {
    System.out.println("Saved: " + nameField.getText());
    editDialog.dispose(); // dialog ပိတ်ဖို့
});

dialogPanel.add(new JLabel("Name:"));
dialogPanel.add(nameField);
dialogPanel.add(saveBtn);
editDialog.add(dialogPanel);
editDialog.setVisible(true);
```

**Built-in JOptionPane dialogs** (ရိုးရိုး message/confirm dialog များအတွက် NetBeans/Swing မှာ အသင့်ပါလာတဲ့ shortcut):

```java
// Information message
JOptionPane.showMessageDialog(mainFrame, "Student saved successfully!", "Success", JOptionPane.INFORMATION_MESSAGE);

// Confirmation (Yes/No)
int result = JOptionPane.showConfirmDialog(mainFrame, "Delete this student?", "Confirm Delete", JOptionPane.YES_NO_OPTION);
if (result == JOptionPane.YES_OPTION) {
    // delete logic
}

// Input dialog
String input = JOptionPane.showInputDialog(mainFrame, "Enter student ID:");
```

**JDialog vs JFrame ကွာခြားချက်**: JFrame က independent main window၊ JDialog က parent frame ရှိမှ သုံးလို့ရတဲ့ dependent (child) window ပါ - parent ပိတ်ရင် dialog လည်း auto ပိတ်တတ်တယ်။

## 3. **JColorChooser** (Color Chooser)

- User ကို color palette ကနေ color တစ်ခု ရွေးခိုင်းတဲ့ built-in dialog ပါ (RGB, HSB, Swatches tab တွေ ပါပြီးသား)။

```java
JButton colorBtn = new JButton("Choose Color");
colorBtn.addActionListener(e -> {
    Color selectedColor = JColorChooser.showDialog(
        mainFrame,           // parent
        "Pick a Color",      // dialog title
        Color.BLUE           // initial color
    );
    if (selectedColor != null) { // user Cancel နှိပ်ရင် null ပြန်တတ်တယ်
        colorBtn.setBackground(selectedColor);
        System.out.println("Selected: " + selectedColor);
    }
});
```

**Embedded version** (dialog မဟုတ်ဘဲ panel ထဲမှာ တိုက်ရိုက်ထည့်ချင်ရင်):

```java
JColorChooser colorChooser = new JColorChooser();
colorChooser.getSelectionModel().addChangeListener(e -> {
    Color c = colorChooser.getColor();
    System.out.println("Live color: " + c);
});
panel.add(colorChooser);
```

---

**QRCode project အတွက် အကြံပြုချက်**:

- **JFrame** — main student list window
- **JDialog** — "Add/Edit Student" form (modal dialog အနေနဲ့ ဖွင့်ရင် ပိုသင့်တော်တယ်, `JOptionPane.showConfirmDialog` ကို Delete confirmation အတွက် သုံးလို့ရတယ်)
- **JColorChooser** — QR code ရဲ့ foreground/background color ကို user ကိုယ်တိုင် customize ချင်ရင် သုံးလို့ရတယ် (optional feature)

NetBeans GUI Builder သုံးရင် JDialog ကို separate form (New > JDialog Form) အနေနဲ့ ဖန်တီးနိုင်ပြီး၊ main frame ကနေ `new EditStudentDialog(this, true).setVisible(true)` ခေါ်ရုံပါပဲ။