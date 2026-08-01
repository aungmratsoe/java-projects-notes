## Part 6: Menus (JMenuBar) & SwingWorker (Background Threading)

### JMenuBar - Application Menu

Application တွေမှာ File, Edit, Help စတဲ့ standard menu bar တွေကို JMenuBar နဲ့ ဆောက်ပါတယ်။

**Structure:** `JMenuBar` → `JMenu` (File, Edit) → `JMenuItem` (Open, Save, Exit)

```java
import javax.swing.*;
import java.awt.event.*;

public class MenuExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Menu Example");
            
            // MenuBar create
            JMenuBar menuBar = new JMenuBar();
            
            // File Menu
            JMenu fileMenu = new JMenu("File");
            JMenuItem newItem = new JMenuItem("New");
            JMenuItem openItem = new JMenuItem("Open");
            JMenuItem saveItem = new JMenuItem("Save");
            JMenuItem exitItem = new JMenuItem("Exit");
            
            fileMenu.add(newItem);
            fileMenu.add(openItem);
            fileMenu.add(saveItem);
            fileMenu.addSeparator();  // divider line
            fileMenu.add(exitItem);
            
            // Edit Menu
            JMenu editMenu = new JMenu("Edit");
            JMenuItem copyItem = new JMenuItem("Copy");
            JMenuItem pasteItem = new JMenuItem("Paste");
            editMenu.add(copyItem);
            editMenu.add(pasteItem);
            
            // MenuBar ထဲကို Menu တွေထည့်
            menuBar.add(fileMenu);
            menuBar.add(editMenu);
            
            // Frame ကို MenuBar သတ်မှတ်
            frame.setJMenuBar(menuBar);
            
            // Event handling
            exitItem.addActionListener(e -> System.exit(0));
            saveItem.addActionListener(e -> 
                JOptionPane.showMessageDialog(frame, "Saved!")
            );
            
            frame.setSize(400, 300);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

### Keyboard Shortcuts (Mnemonics & Accelerators)

```java
// Mnemonic - Alt+F ကိုနှိပ်ရင် File menu ပွင့်မယ်
fileMenu.setMnemonic(KeyEvent.VK_F);

// Accelerator - Ctrl+S ကိုနှိပ်ရင် direct action ဖြစ်မယ် (menu ဖွင့်စရာမလို)
saveItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));
```

### Checkbox & Radio Menu Items

```java
JCheckBoxMenuItem showToolbar = new JCheckBoxMenuItem("Show Toolbar", true);
showToolbar.addActionListener(e -> {
    boolean isChecked = showToolbar.isSelected();
    System.out.println("Toolbar visible: " + isChecked);
});

JMenu viewMenu = new JMenu("View");
viewMenu.add(showToolbar);
```

### JPopupMenu (Right-click Menu)

```java
JPopupMenu popupMenu = new JPopupMenu();
JMenuItem cutItem = new JMenuItem("Cut");
JMenuItem copyItem = new JMenuItem("Copy");
popupMenu.add(cutItem);
popupMenu.add(copyItem);

panel.addMouseListener(new MouseAdapter() {
    @Override
    public void mousePressed(MouseEvent e) {
        if (e.isPopupTrigger()) {  // right-click detect
            popupMenu.show(panel, e.getX(), e.getY());
        }
    }
    
    @Override
    public void mouseReleased(MouseEvent e) {
        if (e.isPopupTrigger()) {  // OS အလိုက် trigger event မတူလို့ နှစ်ခုလုံး check ရပါတယ်
            popupMenu.show(panel, e.getX(), e.getY());
        }
    }
});
```

---

### SwingWorker - Background Threading (အရေးအကြီးဆုံး Concept)

#### ဘာကြောင့် SwingWorker လိုအပ်လဲ

Swing က **single-threaded** ပါ - UI update အားလုံးက **EDT (Event Dispatch Thread)** ပေါ်မှာပဲ ဖြစ်ရပါတယ်။ File download, database query, heavy calculation လို **time-consuming task** ကို EDT ပေါ်မှာ တိုက်ရိုက် run ရင် **UI ကို freeze (hang)** ဖြစ်စေပါတယ် - button တွေနှိပ်လို့ မရတော့ဘူး၊ window drag လို့ မရတော့ဘူး။

```java
// ❌ Bad - UI freeze ဖြစ်စေမယ်
button.addActionListener(e -> {
    try {
        Thread.sleep(5000);  // 5 seconds heavy task simulate
    } catch (InterruptedException ex) {}
    label.setText("Done!");  // 5 seconds တောင် UI hang ဖြစ်နေမယ်
});
```

**SwingWorker** က background thread မှာ heavy task ကို run ပြီး၊ ပြီးသွားရင် result ကို EDT ပေါ်ကို ပြန်ပို့ပေးပါတယ် (UI freeze မဖြစ်ဘဲ)။

#### SwingWorker Basic Structure

```java
class MyWorker extends SwingWorker<String, Void> {
    // <String, Void> = <result type, intermediate progress type>
    
    @Override
    protected String doInBackground() throws Exception {
        // Background thread မှာ run မယ့် heavy task
        Thread.sleep(3000);  // simulate heavy work
        return "Task completed!";
    }
    
    @Override
    protected void done() {
        // doInBackground() ပြီးသွားရင် EDT ပေါ်မှာ run မယ်
        try {
            String result = get();  // doInBackground() ရဲ့ return value ရယူ
            System.out.println(result);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

// Usage
MyWorker worker = new MyWorker();
worker.execute();  // start running
```

#### Practical Example - File Download Simulation with Progress Bar

```java
import javax.swing.*;
import java.awt.*;

public class SwingWorkerExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Download Simulation");
            JPanel panel = new JPanel();
            
            JProgressBar progressBar = new JProgressBar(0, 100);
            progressBar.setStringPainted(true);  // percentage text ပြသဖို့
            JButton startButton = new JButton("Start Download");
            JLabel statusLabel = new JLabel("Ready");
            
            startButton.addActionListener(e -> {
                startButton.setEnabled(false);  // task run နေစဉ် button disable
                
                SwingWorker<Void, Integer> worker = new SwingWorker<>() {
                    @Override
                    protected Void doInBackground() throws Exception {
                        for (int i = 0; i <= 100; i += 10) {
                            Thread.sleep(300);  // simulate download chunk
                            publish(i);  // progress update ကို process() ဆီ ပို့ဖို့
                        }
                        return null;
                    }
                    
                    @Override
                    protected void process(java.util.List<Integer> chunks) {
                        // publish() ကနေ ပို့လိုက်တဲ့ values ကို EDT ပေါ်မှာ handle
                        int latest = chunks.get(chunks.size() - 1);
                        progressBar.setValue(latest);
                    }
                    
                    @Override
                    protected void done() {
                        statusLabel.setText("Download Complete!");
                        startButton.setEnabled(true);
                    }
                };
                
                worker.execute();
            });
            
            panel.add(progressBar);
            panel.add(startButton);
            panel.add(statusLabel);
            
            frame.add(panel);
            frame.setSize(350, 150);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

**Key methods:**

- `doInBackground()` - background thread ပေါ်မှာ run (heavy task ရေးရာ)
- `publish(values)` - progress update ကို `process()` ဆီ ပို့ဖို့ (background thread ကနေ ခေါ်ရပါတယ်)
- `process(chunks)` - `publish()` ကနေ ပို့လိုက်တဲ့ data ကို **EDT ပေါ်မှာ** handle (UI update လုပ်ရာမှာ သုံးပါ)
- `done()` - task ပြီးသွားရင် **EDT ပေါ်မှာ** run (UI final update, cleanup)
- `get()` - `doInBackground()` ရဲ့ return value ရယူဖို့ (`done()` ထဲမှာ ခေါ်ပါ)

### Quick Summary

|Concept|Purpose|
|---|---|
|`JMenuBar/JMenu/JMenuItem`|Application menu ဆောက်ဖို့|
|`JPopupMenu`|Right-click context menu|
|`SwingWorker`|Long-running task ကို UI freeze မဖြစ်အောင် background run ဖို့|
|`doInBackground()`|Heavy task ရေးရာ (background thread)|
|`process()` / `done()`|UI update ရေးရာ (EDT thread)|

---

ဒါက **Part 6 (Menus & SwingWorker)** ပါ။ SwingWorker က intermediate-level concept ဖြစ်လို့ နားလည်ဖို့ အချိန်ယူနိုင်ပါတယ် - question ရှိရင် မေးလို့ရပါတယ်။

**Part 7** မှာ Custom Painting (Graphics2D) ကို ဆက်သင်ပေးရမလား၊ ဒါမှမဟုတ် အခု FlatLaf setup ကို လုပ်ကြည့်ရအောင်လား?