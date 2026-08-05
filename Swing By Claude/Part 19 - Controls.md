Swing control (component) တွေရဲ့ အသုံးဝင်ပုံကို ရှင်းပြပေးပါမယ်။

## Selection / Toggle Type

**JToggleButton** (Toggle Button)

- Push လုပ်ရင် "on" state ကနေ "off" state ကို ပြောင်းပြီး၊ visual အနေနဲ့ ဖိထားသလို (pressed-in look) ပြပေးတဲ့ button ပါ။ Bold/Italic ချက်ခြင်း on/off toggle လုပ်တာမျိုး သုံးလို့ရတယ်။

**JCheckBox** (Check Box)

- Option တစ်ခုချင်းစီကို "ရွေးထား/မရွေးထား" (independent, multiple selection ရနိုင်) ပြပေးဖို့ သုံးတယ်။
- **ဥပမာ**: "Remember me", "Subscribe to newsletter" checkbox တွေမျိုး - တစ်ခုချင်းစီ သီးခြားစီ tick နိုင်တယ်။

**JRadioButton** (Radio Button)

- Option အုပ်စုတစ်ခုထဲကနေ **တစ်ခုတည်း** ရွေးချယ်ခိုင်းတဲ့အခါ သုံးတယ် (mutually exclusive)။
- **ဥပမာ**: Gender ရွေးတာ (Male/Female/Other) - တစ်ခုရွေးရင် ကျန်တာတွေ auto uncheck ဖြစ်သွားရမယ်။

**ButtonGroup** (Button Group)

- ⚠️ ဒါက visible component မဟုတ်ပါ (logical grouping object ပါ)။
- JRadioButton များစွာကို "group" တစ်ခုအဖြစ် ချည်နှောင်ပေးပြီး၊ "တစ်ခုတည်းသာ ရွေးနိုင်" ဆိုတဲ့ mutual exclusivity ကို enforce လုပ်ပေးတယ်။ RadioButton နဲ့ အမြဲတွဲသုံးရတယ်။

## List / Data Selection Type

**JComboBox** (Combo Box)

- Dropdown list ဖြစ်ပြီး၊ space သက်သာစေဖို့ option များထဲကနေ တစ်ခု ရွေးနိုင်တဲ့ control ပါ (dropdown ချထားမှသာ list တွေပေါ်တယ်)။
- **ဥပမာ**: Country ရွေးတဲ့ dropdown။

**JList** (List)

- Item list တွေကို တန်းစီပြထားပြီး၊ single ဒါမှမဟုတ် multiple items ကို တစ်ပြိုင်နက် ရွေးနိုင်တဲ့ control ပါ (ComboBox နဲ့ မတူဘဲ list အားလုံး တန်းစီပြထားတယ်)။

## Scrolling / Numeric Type

**JScrollBar** (Scroll Bar)

- Content က visible area ထက် ကြီးတဲ့အခါ၊ up/down ဒါမှမဟုတ် left/right ရွေ့ကြည့်ဖို့ scroll thumb ပါ။ (ပုံမှန်အားဖြင့် JScrollPane ထဲမှာ auto ထည့်ပေးလေ့ရှိတယ်)

**JSlider** (Slider)

- Range value (min-max) ကို drag ဆွဲပြီး ရွေးချယ်ဖို့ control ပါ။
- **ဥပမာ**: Volume control, Brightness adjustment။

**JProgressBar** (Progress Bar)

- Task တစ်ခု ဘယ်လောက် ပြီးစီးပြီလဲ (%) ကို visual အနေနဲ့ ပြပေးတဲ့ control ပါ (user input မလုပ်နိုင်ဘူး - display only)။
- **ဥပမာ**: File download/upload progress။

## Text Input Type (Advanced)

**JFormattedTextField** (Formatted Field)

- Input format ကို restrict/control လုပ်ချင်တဲ့အခါ သုံးတယ် (date format, currency format, phone number format စသည်)။
- **ဥပမာ**: Date field မှာ "dd/MM/yyyy" pattern အတိုင်းပဲ ရိုက်ခွင့်ပြုတာမျိုး။

**JSpinner** (Spinner)

- Up/down arrow နှိပ်ပြီး value (number, date, list item) ကို တစ်ဆင့်ချင်း တိုး/လျှော့ ရွေးနိုင်တဲ့ control ပါ။
- **ဥပမာ**: Quantity ရွေးတဲ့ box (1, 2, 3...)။

## Layout / Display Helper Type

**JSeparator** (Separator)

- Component (menu item, toolbar item) တွေကြားမှာ visual အနေနဲ့ ခွဲခြားပြသဖို့ မျဉ်းကြောင်း (line) ပါ။ Logic မပါဘဲ visual grouping အတွက်သာ။

**JTextPane** (Text Pane)

- Rich text (font style, color, image, embedded component) ကို ထည့်နိုင်တဲ့ advanced text component ပါ။ Styled document format (StyledDocument) ကို support လုပ်တယ်။

**JEditorPane** (Editor Pane)

- HTML, RTF စတဲ့ format တွေကို render/edit လုပ်နိုင်တဲ့ text component ပါ (JTextPane ရဲ့ superclass)။ Simple HTML viewer/editor လုပ်ချင်ရင် သုံးလို့ရတယ်။

## Hierarchical Data Type

**JTree** (Tree)

- Hierarchical data (parent-child relationship ရှိတဲ့ data) ကို tree structure နဲ့ expand/collapse လုပ်ပြီး ပြသဖို့ control ပါ။
- **ဥပမာ**: File system folder structure, Organization chart, Category-subcategory structure။

---

**Quick Summary**: Selection/Toggle group (CheckBox, RadioButton, ToggleButton) က user ကို choice ရွေးခိုင်းဖို့၊ Text input group (FormattedField, Spinner) က data အမှန်ကန်ဆုံး ရိုက်ခိုင်းဖို့၊ Display group (ProgressBar, Separator) က information ပြသဖို့ဆိုပြီး ခွဲခြားနားလည်ထားရင် မှတ်ရလွယ်ပါလိမ့်မယ်။

QRCode project အတွက် student data ထည့်တဲ့ form တစ်ခု ရေးမယ်ဆိုရင် FormattedField (student ID/date), ComboBox (class/grade ရွေးဖို့) တို့ အသုံးဝင်နိုင်ပါတယ်။ ဆက်လက်ပြီး code example လိုချင်ရင် ပြောပါ။

---
Swing controls တွေအတွက် code example တွေကို component တစ်ခုချင်းစီ ခွဲပြီး ပြပေးပါမယ်။ (Comment တွေကို မြန်မာလို ထည့်ထားပါတယ်)

## 1. JToggleButton

```java
JToggleButton toggleBtn = new JToggleButton("Bold");
toggleBtn.addActionListener(e -> {
    if (toggleBtn.isSelected()) {
        System.out.println("Bold ON");
    } else {
        System.out.println("Bold OFF");
    }
});
panel.add(toggleBtn);
```

## 2. JCheckBox

```java
JCheckBox chkSubscribe = new JCheckBox("Subscribe to newsletter");
chkSubscribe.addItemListener(e -> {
    boolean checked = chkSubscribe.isSelected();
    System.out.println("Subscribed: " + checked);
});
panel.add(chkSubscribe);
```

## 3. JRadioButton + ButtonGroup (တွဲသုံးရတာမို့ တစ်ခုတည်း ပေါင်းရှင်းပါမယ်)

```java
JRadioButton rbMale = new JRadioButton("Male");
JRadioButton rbFemale = new JRadioButton("Female");

ButtonGroup genderGroup = new ButtonGroup(); // logical grouping - screen ပေါ်မှာ မမြင်ရ
genderGroup.add(rbMale);
genderGroup.add(rbFemale);

rbMale.addActionListener(e -> System.out.println("Selected: Male"));
rbFemale.addActionListener(e -> System.out.println("Selected: Female"));

panel.add(rbMale);
panel.add(rbFemale);
```

## 4. JComboBox

```java
String[] classes = {"Grade 1", "Grade 2", "Grade 3"};
JComboBox<String> comboClass = new JComboBox<>(classes);
comboClass.addActionListener(e -> {
    String selected = (String) comboClass.getSelectedItem();
    System.out.println("Selected class: " + selected);
});
panel.add(comboClass);
```

## 5. JList

```java
String[] students = {"Aung Aung", "Su Su", "Ko Ko"};
JList<String> studentList = new JList<>(students);
studentList.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);
JScrollPane listScroll = new JScrollPane(studentList); // list တွေများရင် scroll လိုတတ်တယ်
panel.add(listScroll);
```

## 6. JScrollBar

```java
JScrollBar scrollBar = new JScrollBar(JScrollBar.HORIZONTAL, 0, 10, 0, 100);
scrollBar.addAdjustmentListener(e -> {
    System.out.println("Value: " + e.getValue());
});
panel.add(scrollBar);
```

## 7. JSlider

```java
JSlider volumeSlider = new JSlider(JSlider.HORIZONTAL, 0, 100, 50);
volumeSlider.setMajorTickSpacing(25);
volumeSlider.setPaintTicks(true);
volumeSlider.setPaintLabels(true);
volumeSlider.addChangeListener(e -> {
    System.out.println("Volume: " + volumeSlider.getValue());
});
panel.add(volumeSlider);
```

## 8. JProgressBar

```java
JProgressBar progressBar = new JProgressBar(0, 100);
progressBar.setValue(0);
progressBar.setStringPainted(true); // % text ပြချင်ရင်

// Simulate progress (SwingWorker သုံးရင် ပိုမှန်ပါတယ်)
Timer timer = new Timer(100, e -> {
    int value = progressBar.getValue();
    if (value < 100) {
        progressBar.setValue(value + 5);
    }
});
timer.start();
panel.add(progressBar);
```

## 9. JFormattedTextField

```java
try {
    MaskFormatter dateFormatter = new MaskFormatter("##/##/####"); // dd/MM/yyyy
    dateFormatter.setPlaceholderCharacter('_');
    JFormattedTextField dateField = new JFormattedTextField(dateFormatter);
    panel.add(dateField);
} catch (java.text.ParseException ex) {
    ex.printStackTrace();
}
```

## 10. JSpinner

```java
SpinnerNumberModel spinnerModel = new SpinnerNumberModel(1, 1, 100, 1); // initial, min, max, step
JSpinner quantitySpinner = new JSpinner(spinnerModel);
quantitySpinner.addChangeListener(e -> {
    System.out.println("Quantity: " + quantitySpinner.getValue());
});
panel.add(quantitySpinner);
```

## 11. JSeparator

```java
JSeparator separator = new JSeparator(JSeparator.HORIZONTAL);
panel.add(separator); // menu/toolbar item တွေကြားမှာ visual divider အနေနဲ့ ခွဲချင်ရင် သုံးတယ်
```

## 12. JTextPane (rich/styled text)

```java
JTextPane textPane = new JTextPane();
StyledDocument doc = textPane.getStyledDocument();

Style redStyle = textPane.addStyle("RedStyle", null);
StyleConstants.setForeground(redStyle, Color.RED);
StyleConstants.setBold(redStyle, true);

try {
    doc.insertString(doc.getLength(), "Important Notice!\n", redStyle);
    doc.insertString(doc.getLength(), "Normal text here.", null);
} catch (BadLocationException ex) {
    ex.printStackTrace();
}
panel.add(new JScrollPane(textPane));
```

## 13. JEditorPane (HTML content)

```java
JEditorPane editorPane = new JEditorPane();
editorPane.setContentType("text/html");
editorPane.setText("<html><body><h2>Welcome</h2><p style='color:blue'>This is HTML content.</p></body></html>");
editorPane.setEditable(false);
panel.add(new JScrollPane(editorPane));
```

## 14. JTree

```java
DefaultMutableTreeNode root = new DefaultMutableTreeNode("Students");
DefaultMutableTreeNode grade1 = new DefaultMutableTreeNode("Grade 1");
grade1.add(new DefaultMutableTreeNode("Aung Aung"));
grade1.add(new DefaultMutableTreeNode("Su Su"));

DefaultMutableTreeNode grade2 = new DefaultMutableTreeNode("Grade 2");
grade2.add(new DefaultMutableTreeNode("Ko Ko"));

root.add(grade1);
root.add(grade2);

JTree studentTree = new JTree(root);
studentTree.addTreeSelectionListener(e -> {
    DefaultMutableTreeNode node = (DefaultMutableTreeNode) studentTree.getLastSelectedPathComponent();
    if (node != null) {
        System.out.println("Selected: " + node.getUserObject());
    }
});
panel.add(new JScrollPane(studentTree));
```

---

**မှတ်ချက်**: NetBeans GUI Builder (drag-and-drop mode) သုံးနေတဲ့အခန်းမို့ - control တွေကို Palette ကနေ drag ဆွဲထည့်ပြီးရင် code တွေက auto generate ဖြစ်သွားမှာပါ (`initComponents()` method ထဲမှာ)။ အပေါ်က code တွေက manual/pure code ရေးရင် ဘယ်လို API သုံးရမလဲဆိုတာ reference အနေနဲ့ ပေးထားတာပါ - listener logic (`addActionListener`, `addChangeListener` စတာတွေ) ကတော့ GUI Builder သုံးရင်လည်း အလားတူ ရေးရပါလိမ့်မယ်။

QRCode project ထဲမှာ ဘယ် control ကို ဘယ်လို data (Student model) နဲ့ တွဲသုံးချင်တာလဲဆိုတာ ပြောရင် ပိုပြီး specific ဖြစ်တဲ့ example ပြပေးပါမယ်။