Swing menu component တွေရဲ့ အသုံးဝင်ပုံကို ရှင်းပြပေးပါမယ်။

## 1. **JMenuBar** (Menu Bar)

- Application window ရဲ့ **အပေါ်ဆုံးတန်း** မှာ ထားတဲ့ container ပါ (File, Edit, View, Help စသည်တို့ ထားတဲ့ တန်း)။
- JMenu (menu) တွေကို host လုပ်ပေးတဲ့ container ဖြစ်ပြီး၊ `frame.setJMenuBar(menuBar)` နဲ့ frame ပေါ် တပ်ဆင်ရတယ်။

## 2. **JMenu** (Menu)

- Menu Bar ပေါ်က click နှိပ်ရင် dropdown list ပေါ်လာတဲ့ menu ခေါင်းစဉ် (heading) ပါ။
- **ဥပမာ**: "File" ဆိုတဲ့ menu ကို click လိုက်ရင် New, Open, Save, Exit စတဲ့ menu item တွေ ပေါ်လာတာမျိုး။
- JMenu ထဲမှာ JMenuItem တွေ ထည့်ရတယ်၊ Sub-menu (nested JMenu) လည်း ထည့်လို့ရတယ်။

## 3. **JMenuItem** (Menu Item)

- Menu ထဲက click နှိပ်လို့ action တစ်ခု run ဖြစ်စေတဲ့ item ပါ (ပုံမှန် Button တစ်ခုနဲ့ တူတယ်၊ menu ထဲမှာ ရှိတာသာ ကွာတယ်)။
- **ဥပမာ**: File > Save, File > Exit။

## 4. **JCheckBoxMenuItem** (Menu Item / Checkbox)

- Menu item ပါပဲ၊ ဒါပေမယ့် on/off state ရှိပြီး tick mark ✓ ပြပေးနိုင်တဲ့ item ပါ (CheckBox logic ကို menu ထဲမှာ ထည့်ထားတာ)။
- **ဥပမာ**: View > "Show Toolbar" ကို tick/untick လုပ်လို့ရတာမျိုး - toggle setting တွေအတွက် သုံးတယ်။

## 5. **JRadioButtonMenuItem** (Menu Item / RadioButton)

- Menu item ပါပဲ၊ ButtonGroup နဲ့ တွဲပြီး group ထဲက **တစ်ခုတည်း** ရွေးနိုင်တဲ့ item ပါ။
- **ဥပမာ**: Format > Alignment > (Left / Center / Right) - တစ်ခုပဲ ရွေးလို့ရအောင် (mutually exclusive)။

## 6. **JPopupMenu** (Popup Menu)

- Menu Bar ကနေ မဟုတ်ဘဲ - **mouse right-click** (context menu) နှိပ်တဲ့အခါ ပေါ်လာတဲ့ menu ပါ။
- **ဥပမာ**: Text field တစ်ခုပေါ် right-click လုပ်လိုက်ရင် Cut/Copy/Paste ပေါ်လာတာမျိုး။ JTable/JList row တစ်ခုပေါ် right-click လုပ်ရင် "Edit", "Delete" options ပေါ်စေချင်ရင်လည်း ဒါကို သုံးရတယ်။

## 7. **Separator** (menu context)

- Menu item တွေကြားမှာ logical group ခွဲပြီး visual line နဲ့ ပိုင်းခြားပြပေးတဲ့ item ပါ (`menu.addSeparator()` ခေါ်ရုံပါပဲ)။
- **ဥပမာ**: File menu ထဲမှာ New/Open/Save တွေကို တစ်အုပ်စု၊ line ခြားပြီး Exit ကို နောက်တစ်အုပ်စုအဖြစ် ခွဲပြတာမျိုး။

---

**Quick code example** (structure ကို မြင်ရအောင်):

```java
JMenuBar menuBar = new JMenuBar();

JMenu fileMenu = new JMenu("File");
JMenuItem newItem = new JMenuItem("New");
JMenuItem openItem = new JMenuItem("Open");
JMenuItem exitItem = new JMenuItem("Exit");

fileMenu.add(newItem);
fileMenu.add(openItem);
fileMenu.addSeparator(); // <-- divider line
fileMenu.add(exitItem);

JMenu viewMenu = new JMenu("View");
JCheckBoxMenuItem showToolbar = new JCheckBoxMenuItem("Show Toolbar", true);
viewMenu.add(showToolbar);

JMenu alignMenu = new JMenu("Alignment");
ButtonGroup alignGroup = new ButtonGroup();
JRadioButtonMenuItem leftAlign = new JRadioButtonMenuItem("Left", true);
JRadioButtonMenuItem centerAlign = new JRadioButtonMenuItem("Center");
alignGroup.add(leftAlign);
alignGroup.add(centerAlign);
alignMenu.add(leftAlign);
alignMenu.add(centerAlign);

menuBar.add(fileMenu);
menuBar.add(viewMenu);
frame.setJMenuBar(menuBar);

// Popup menu (right-click) - table/panel တစ်ခုနဲ့ တွဲသုံးရင်
JPopupMenu popup = new JPopupMenu();
popup.add(new JMenuItem("Edit"));
popup.add(new JMenuItem("Delete"));

studentTable.addMouseListener(new MouseAdapter() {
    public void mousePressed(MouseEvent e) {
        if (e.isPopupTrigger() || SwingUtilities.isRightMouseButton(e)) {
            popup.show(e.getComponent(), e.getX(), e.getY());
        }
    }
});
```

---

**QRCode project အတွက် အကြံပြုချက်**: Student table (JTable) မှာ row တစ်ခုပေါ် right-click နှိပ်ရင် "QR Code ထုတ်မယ်", "Edit", "Delete" options ပြချင်ရင် **JPopupMenu** သုံးရင် သင့်တော်ပါလိမ့်မယ်။ Main window ပေါ်မှာ File/Student/Report စသည့် menu structure ဖန်တီးချင်ရင်တော့ JMenuBar + JMenu + JMenuItem ကို standard အနေနဲ့ သုံးရပါလိမ့်မယ်။