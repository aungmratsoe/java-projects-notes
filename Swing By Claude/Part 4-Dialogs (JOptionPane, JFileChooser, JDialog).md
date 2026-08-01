## Part 4: Dialogs (JOptionPane, JFileChooser, JDialog)

Dialog ဆိုတာ user ကို quick message ပြပြီး response ရယူဖို့ popup window လေးတွေပါ။ Swing မှာ dialog အမျိုးမျိုး built-in ပါဝင်ပါတယ်။

### JOptionPane (အသုံးအများဆုံး Dialog)

`JOptionPane` က static method တွေနဲ့ dialog အမျိုးမျိုး quickly ပြသနိုင်ပါတယ် - custom class ဆောက်စရာ မလိုပါဘူး။

#### 1. Message Dialog (Info ပြသဖို့)

```java
JOptionPane.showMessageDialog(frame, "Operation completed successfully!");

// Type သတ်မှတ်ချင်ရင်
JOptionPane.showMessageDialog(
    frame, 
    "File saved successfully!", 
    "Success",                              // dialog title
    JOptionPane.INFORMATION_MESSAGE         // icon type
);

JOptionPane.showMessageDialog(
    frame,
    "An error occurred!",
    "Error",
    JOptionPane.ERROR_MESSAGE               // error icon (red X)
);
```

**Message types:** `INFORMATION_MESSAGE`, `WARNING_MESSAGE`, `ERROR_MESSAGE`, `QUESTION_MESSAGE`, `PLAIN_MESSAGE`

#### 2. Confirm Dialog (Yes/No ဆုံးဖြတ်ချက်ယူဖို့)

```java
int result = JOptionPane.showConfirmDialog(
    frame,
    "Are you sure you want to delete this file?",
    "Confirm Delete",
    JOptionPane.YES_NO_OPTION
);

if (result == JOptionPane.YES_OPTION) {
    System.out.println("Deleting file...");
} else {
    System.out.println("Cancelled");
}
```

**Option types:** `YES_NO_OPTION`, `YES_NO_CANCEL_OPTION`, `OK_CANCEL_OPTION`

**Return values:** `YES_OPTION`, `NO_OPTION`, `CANCEL_OPTION`, `OK_OPTION`, `CLOSED_OPTION`

#### 3. Input Dialog (User ဆီက text ရယူဖို့)

```java
String name = JOptionPane.showInputDialog(frame, "Enter your name:");

if (name != null && !name.trim().isEmpty()) {
    JOptionPane.showMessageDialog(frame, "Hello, " + name + "!");
}
```

**Note:** User က Cancel နှိပ်ရင် `null` ပြန်ပေးပါတယ်။ ဒါကြောင့် null check လုပ်ရပါတယ်။

Dropdown options ပါတဲ့ input dialog:

```java
String[] options = {"Small", "Medium", "Large"};
String size = (String) JOptionPane.showInputDialog(
    frame,
    "Choose size:",
    "Size Selection",
    JOptionPane.QUESTION_MESSAGE,
    null,
    options,
    options[0]  // default selected value
);
```

### JFileChooser (File Open/Save Dialog)

#### File Open Dialog

```java
JFileChooser fileChooser = new JFileChooser();
int result = fileChooser.showOpenDialog(frame);

if (result == JFileChooser.APPROVE_OPTION) {
    java.io.File selectedFile = fileChooser.getSelectedFile();
    System.out.println("Selected file: " + selectedFile.getAbsolutePath());
}
```

#### File Save Dialog

```java
JFileChooser fileChooser = new JFileChooser();
int result = fileChooser.showSaveDialog(frame);

if (result == JFileChooser.APPROVE_OPTION) {
    java.io.File fileToSave = fileChooser.getSelectedFile();
    System.out.println("Saving to: " + fileToSave.getAbsolutePath());
}
```

#### File Filter (specific file types ပဲ ပြချင်ရင်)

```java
JFileChooser fileChooser = new JFileChooser();
fileChooser.setFileFilter(
    new FileNameExtensionFilter("Text Files (*.txt)", "txt")
);
fileChooser.setFileFilter(
    new FileNameExtensionFilter("Image Files", "jpg", "png", "gif")
);

int result = fileChooser.showOpenDialog(frame);
```

#### Folder Selection

```java
JFileChooser folderChooser = new JFileChooser();
folderChooser.setFileSelectionMode(JFileChooser.DIRECTORIES_ONLY);

int result = folderChooser.showOpenDialog(frame);
if (result == JFileChooser.APPROVE_OPTION) {
    System.out.println("Selected folder: " + folderChooser.getSelectedFile());
}
```

### JColorChooser (Color Picker)

```java
Color selectedColor = JColorChooser.showDialog(
    frame,
    "Choose a color",
    Color.BLUE  // default color
);

if (selectedColor != null) {
    panel.setBackground(selectedColor);
}
```

### Custom JDialog (Custom-built Dialog)

Built-in dialog တွေနဲ့ မလုံလောက်ရင် custom `JDialog` ကို ကိုယ်တိုင် ဆောက်နိုင်ပါတယ်:

```java
import javax.swing.*;
import java.awt.*;

public class CustomDialogExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Main Window");
            JButton openDialogBtn = new JButton("Open Custom Dialog");
            
            openDialogBtn.addActionListener(e -> {
                JDialog dialog = new JDialog(frame, "Custom Dialog", true);  // true = modal
                dialog.setLayout(new FlowLayout());
                
                JLabel label = new JLabel("This is a custom dialog");
                JButton closeBtn = new JButton("Close");
                closeBtn.addActionListener(ev -> dialog.dispose());  // dialog ပိတ်ဖို့
                
                dialog.add(label);
                dialog.add(closeBtn);
                dialog.setSize(250, 100);
                dialog.setLocationRelativeTo(frame);
                dialog.setVisible(true);
            });
            
            frame.add(openDialogBtn);
            frame.setSize(300, 150);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

**Important parameters:**

- `new JDialog(parent, title, modal)` - `modal = true` ဆိုရင် dialog ပိတ်မချင်း parent window ကို interact မလုပ်နိုင်ပါ (blocking)
- `dialog.dispose()` - dialog ကို memory ကနေ ဖျက်ပစ်ဖို့ (close button behavior)

### Practical Example - Delete Confirmation with File Save

```java
import javax.swing.*;
import java.awt.*;

public class DialogExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Dialog Demo");
            JPanel panel = new JPanel();
            
            JButton saveBtn = new JButton("Save File");
            JButton deleteBtn = new JButton("Delete Item");
            
            saveBtn.addActionListener(e -> {
                JFileChooser fileChooser = new JFileChooser();
                int result = fileChooser.showSaveDialog(frame);
                if (result == JFileChooser.APPROVE_OPTION) {
                    JOptionPane.showMessageDialog(
                        frame,
                        "Saved to: " + fileChooser.getSelectedFile().getName(),
                        "Success",
                        JOptionPane.INFORMATION_MESSAGE
                    );
                }
            });
            
            deleteBtn.addActionListener(e -> {
                int confirm = JOptionPane.showConfirmDialog(
                    frame,
                    "Delete this item permanently?",
                    "Confirm Delete",
                    JOptionPane.YES_NO_OPTION,
                    JOptionPane.WARNING_MESSAGE
                );
                if (confirm == JOptionPane.YES_OPTION) {
                    JOptionPane.showMessageDialog(frame, "Item deleted!");
                }
            });
            
            panel.add(saveBtn);
            panel.add(deleteBtn);
            
            frame.add(panel);
            frame.setSize(300, 150);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

### Quick Summary

|Dialog|ဘယ်အခါသုံးလဲ|
|---|---|
|`showMessageDialog`|Info/error message ပြသဖို့|
|`showConfirmDialog`|Yes/No decision ယူဖို့|
|`showInputDialog`|User ဆီက text/selection ရယူဖို့|
|`JFileChooser`|File open/save|
|`JColorChooser`|Color ရွေးဖို့|
|Custom `JDialog`|Built-in dialog မလုံလောက်ရင် ကိုယ်တိုင်ဆောက်ဖို့|

---

ဒါက **Part 4 (Dialogs)** ပါ။ **Part 5** မှာ JTable, JList (Model-based components) တွေကို ဆက်သင်ပေးပါမယ် - ဒါတွေက business application တွေအတွက် အရေးအကြီးဆုံး component တွေထဲက ပါဝင်ပါတယ်။ ဆက်သွားမလား?