JList ကို ဘယ်အခြေအနေမျိုးမှာ သုံးသင့်လဲ ရှင်းပြပေးပါမယ်။

## JList vs JComboBox vs JTable — အရင် ကွာခြားချက် ခွဲသိထားရမယ်

|Component|Display Style|Selection|
|---|---|---|
|**JComboBox**|Dropdown (collapsed), click မှ ပွင့်|Single item ရွေးရုံ|
|**JList**|Item **အားလုံး** တန်းစီပြထား (visible list)|Single ဒါမှမဟုတ် Multiple selection|
|**JTable**|Row + **Column** များစွာ (structured data)|Row/cell selection|

---

## JList သုံးသင့်တဲ့ Concrete Condition များ

### 1. Multiple Item ကို တစ်ပြိုင်နက် ရွေးချင်တဲ့အခါ (Multi-select)

JComboBox က single selection ပဲ ရတယ်၊ JList ကတော့ **Ctrl/Shift + click** နဲ့ item များစွာ ရွေးလို့ရတယ်။

**ဥပမာ**: Bulk QR code generate လုပ်ချင်တဲ့ student list ကနေ student ၅ ယောက် ကို select လုပ်ပြီး "Generate QR for selected" button နှိပ်တာမျိုး

```java
studentJList.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);

// Selected students အားလုံး ရယူတာ
List<Student> selected = studentJList.getSelectedValuesList();
for (Student s : selected) {
    generateQRCode(s.getStudentId());
}
```

### 2. Item List ကို User က **အမြဲမြင်ရအောင်** ပြချင်တဲ့အခါ (Dropdown ချမလိုချင်ရင်)

Space ရှိပြီး၊ user click မလုပ်ဘဲ list အားလုံး တန်းစီ မြင်ချင်ရင် JComboBox (dropdown) အစား JList ကို သုံးတယ်။

**ဥပမာ**: Sidebar navigation panel မှာ Class list တွေ (1A, 1B, 2A...) ကို အမြဲပြထားပြီး click ရွေးလို့ရတာမျိုး

### 3. Selection ပြောင်းတိုင်း Real-time Preview/Filter ပြချင်တဲ့အခါ

**ဥပမာ**: List item click တိုင်း right-side panel မှာ detail ချက်ချင်း ပြောင်းပြချင်ရင်

```java
studentJList.addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {
        Student selected = studentJList.getSelectedValue();
        if (selected != null) {
            showStudentDetail(selected); // right panel update
        }
    }
});
```

### 4. Search/Filter Result List ပြချင်တဲ့အခါ

**ဥပမာ**: Search box မှာ student name ရိုက်ရင်း matching result တွေကို JList ထဲမှာ dynamic filter ပြတာမျိုး (JTable ထက် lightweight, single-column data အတွက် ပိုသင့်တော်)

```java
searchField.addKeyListener(new KeyAdapter() {
    public void keyReleased(KeyEvent e) {
        String query = searchField.getText().toLowerCase();
        DefaultListModel<String> filteredModel = new DefaultListModel<>();
        for (Student s : allStudents) {
            if (s.getName().toLowerCase().contains(query)) {
                filteredModel.addElement(s.getName());
            }
        }
        studentJList.setModel(filteredModel);
    }
});
```

### 5. Single Column, Simple Text Data ဖြစ်တဲ့အခါ (Table လိုမလားချင်ရင်)

Column များစွာ (ID, Name, Age, Class) ပြဖို့ မလိုဘဲ **name/label list** တစ်ခုတည်းသာ ပြချင်ရင် JTable ထက် JList က ပိုရိုးရှင်းပြီး lightweight ပါတယ်။

---

## JList **မသုံးသင့်**တဲ့ အခြေအနေများ

**1. Column များစွာ ပြချင်တဲ့အခါ**

- Student ID, Name, Grade, QR Status ဆိုပြီး column အများကြီး ပြချင်ရင် **JTable** သာ သင့်တော်တယ်

**2. Space ကန့်သတ်ထားတဲ့ Form (Dropdown ချင်ရင်)**

- Screen space နည်းပြီး, item list ကို collapse ထားချင်ရင် **JComboBox** ပိုသင့်တော်တယ်

**3. Hierarchical/Nested Data**

- Grade → Class → Student ဆိုတဲ့ parent-child ဖြစ်နေရင် **JTree** ကသာ သင့်တော်တယ် (ရှေ့ session မှာ ရှင်းခဲ့ပြီး)

---

## Decision Table

|Condition|Recommended|
|---|---|
|Single item ရွေး, space သက်သာချင်|JComboBox|
|Multiple item တစ်ပြိုင်နက် ရွေးချင်|**JList** (multi-select)|
|Column များစွာ ပါတဲ့ structured data|JTable|
|Parent-child hierarchy|JTree|
|Real-time filter/search result list|**JList**|
|List အမြဲမြင်ရအောင် ပြချင် (dropdown မလို)|**JList**|

---

## QRCode Project အတွက် Concrete Use Case

**Bulk QR Generation Feature** အတွက် JList က တော်တော်လေး relevant ဖြစ်နိုင်ပါတယ်:

```java
// Main table (JTable) ဘေးမှာ, "Bulk Actions" panel တစ်ခု ထည့်ချင်ရင်
DefaultListModel<Student> listModel = new DefaultListModel<>();
for (Student s : studentDao.getAll()) {
    listModel.addElement(s);
}
studentJList.setModel(listModel);
studentJList.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);

generateSelectedBtn.addActionListener(e -> {
    List<Student> selected = studentJList.getSelectedValuesList();
    if (selected.isEmpty()) {
        JOptionPane.showMessageDialog(this, "Please select at least one student.");
        return;
    }
    // SwingWorker + ProgressBar နဲ့ ရှေ့က session မှာ ရှင်းခဲ့သလို bulk generate
});
```

---

**Summary**: JList ကို **multiple selection လိုအပ်တဲ့အခါ** ဒါမှမဟုတ် **single-column data ကို dropdown မဟုတ်ဘဲ list အနေနဲ့ အမြဲပြချင်တဲ့အခါ** သုံးပါ။ QRCode project ရဲ့ main student data ကို ပြရင် JTable ကိုပဲ ဆက်သုံးပြီး၊ Bulk QR generation feature အတွက် student multiple-select UI လိုအပ်မှသာ JList ကို ထည့်သုံးဖို့ အကြံပြုပါတယ်။

---
---
NetBeans GUI Builder (drag-and-drop) မှာ JList ကို ဘယ်လို create/setup လုပ်ရမလဲ ရှင်းပြပေးပါမယ်။

## Step 1: Palette ကနေ JList Drag ဆွဲထည့်ခြင်း

1. Palette ထဲက **"Swing Controls"** category ဖွင့်ပါ
2. **"List"** ကို Design view ပေါ် drag ဆွဲချပါ
3. NetBeans က default sample data (Item 1, Item 2, Item 3, Item 4, Item 5) နဲ့ auto-populate လုပ်ပေးပါလိမ့်မယ် — placeholder ပါ၊ manual ပြောင်းရပါမယ်

⚠️ **JTree လိုပဲ — JList ကိုလည်း JScrollPane ထဲမှာ wrap ထားရမယ်** (item များများ ဖြစ်လာရင် scroll လိုအပ်လို့):

```
jScrollPane1
  └── jList1
```

Palette ကနေ **"Scroll Pane"** ကို အရင် drag ချ၊ ပြီးမှ ၎င်းအထဲကို **"List"** ကို drag ဆွဲထည့်ပါ။

## Step 2: Default Sample Data ကို ဖျက်ခြင်း

`jList1` ကို ရွေးပြီး Properties window ထဲမှာ **`model`** property ကို ရှာပါ — ellipsis (`...`) button နှိပ်ရင် **"Configure Elements"** dialog ပွင့်ပါလိမ့်မယ်။ ဒီနေရာမှာ static data ကို GUI ကနေ manual ဖြည့်လို့ရပေမယ့် — **Database ကနေ student list dynamic ဆွဲမယ်ဆိုရင် code ကနေ ဆောက်တာက ပိုသင့်တော်ပါတယ်**။

## Step 3: List Data ကို Code ကနေ Manual ဆောက်ခြင်း (Recommended)

`initComponents()` **အပြင်ဘက်**မှာ, constructor ထဲ ဒါမှမဟုတ် custom method ထဲမှာ:

```java
public MainFrame() {
    initComponents(); // GUI Builder auto-generated
    
    setupStudentList(); // <-- manual method call
}

private DefaultListModel<Student> listModel; // class field, initComponents() အပြင်

private void setupStudentList() {
    listModel = new DefaultListModel<>();
    
    List<Student> students = studentDao.getAll(); // your DAO method
    for (Student s : students) {
        listModel.addElement(s);
    }
    
    studentJList.setModel(listModel); // GUI Builder auto-created variable name
    studentJList.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);
}
```

## Step 4: Custom Display Format ထည့်ခြင်း (Student Object ကို Name အနေနဲ့ ပြချင်ရင်)

Default အားဖြင့် `Student.toString()` ကို JList က ခေါ်ပြီး ပြပါလိမ့်မယ် — Object ရဲ့ memory address (`Student@1a2b3c`) မျိုး ပြသွားနိုင်လို့ **CellRenderer** ထည့်ရပါမယ် (ဒါမှမဟုတ် `Student` class ထဲမှာ `toString()` override):

**Option A: `Student.toString()` override (ရိုးရိုးဆုံးနည်း)**

```java
// Student.java model class ထဲမှာ
@Override
public String toString() {
    return name + " (" + studentId + ")";
}
```

**Option B: Custom CellRenderer (ပိုပြီး flexible, icon ထည့်ချင်ရင်)**

```java
studentJList.setCellRenderer(new DefaultListCellRenderer() {
    @Override
    public Component getListCellRendererComponent(JList<?> list, Object value,
            int index, boolean isSelected, boolean cellHasFocus) {
        super.getListCellRendererComponent(list, value, index, isSelected, cellHasFocus);
        if (value instanceof Student) {
            Student s = (Student) value;
            setText(s.getName() + " - " + s.getStudentId());
        }
        return this;
    }
});
```

## Step 5: Selection Event ထည့်ခြင်း (Events Tab ကနေ)

Design view ထဲမှာ `jList1` ကို ရွေးပြီး Properties window ရဲ့ **Events tab (⚡)** ကို ဖွင့်ပါ — **`valueChanged`** event ကို ရှာပြီး method name ရိုက်ထည့်ပါ:

```java
private void studentJListValueChanged(javax.swing.event.ListSelectionEvent evt) {
    if (evt.getValueIsAdjusting()) return; // drag ဆွဲစဉ်ကာလ event ထပ်ခါထပ်ခါ fire မဖြစ်အောင်

    List<Student> selected = studentJList.getSelectedValuesList();
    selectedCountLabel.setText(selected.size() + " student(s) selected");
}
```

## Step 6: Multi-select Mode ကို Properties ကနေ Set လုပ်ခြင်း

`jList1` ရွေးပြီး Properties window ထဲမှာ **`selectionMode`** property ကို ရှာပါ — dropdown ကနေ ရွေးလို့ရတယ် (code manual ရေးစရာ မလို):

|Option|ရလဒ်|
|---|---|
|`SINGLE_SELECTION`|Item တစ်ခုတည်း ရွေးလို့ရ|
|`SINGLE_INTERVAL_SELECTION`|Item ဆက်တိုက် range ရွေးလို့ရ (Shift+click)|
|`MULTIPLE_INTERVAL_SELECTION`|Item မည်သည့်နေရာမဆို ရွေးလို့ရ (Ctrl+click)|

Bulk QR generation feature အတွက် **`MULTIPLE_INTERVAL_SELECTION`** ကို ရွေးထားရမယ်။

---

## Full QRCode Project Example (Bulk QR Generate Panel)

```java
public class BulkQRPanel extends JPanel {
    private DefaultListModel<Student> listModel;

    public BulkQRPanel() {
        initComponents(); // GUI Builder auto-generated
        setupStudentList();

        generateBtn.addActionListener(e -> generateSelectedQRCodes());
    }

    private void setupStudentList() {
        listModel = new DefaultListModel<>();
        for (Student s : studentDao.getAll()) {
            listModel.addElement(s);
        }
        studentJList.setModel(listModel);
        studentJList.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);
    }

    private void generateSelectedQRCodes() {
        List<Student> selected = studentJList.getSelectedValuesList();
        if (selected.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Please select at least one student.");
            return;
        }

        progressBar.setMaximum(selected.size());
        SwingWorker<Void, Integer> worker = new SwingWorker<>() {
            @Override
            protected Void doInBackground() {
                for (int i = 0; i < selected.size(); i++) {
                    generateQRCode(selected.get(i).getStudentId());
                    publish(i + 1);
                }
                return null;
            }
            @Override
            protected void process(List<Integer> chunks) {
                progressBar.setValue(chunks.get(chunks.size() - 1));
            }
            @Override
            protected void done() {
                JOptionPane.showMessageDialog(BulkQRPanel.this, "QR codes generated!");
            }
        };
        worker.execute();
    }
}
```

---

## Quick Checklist

|Step|Action|
|---|---|
|1|Palette → Swing Controls → **Scroll Pane** drag ချ → ထဲကို **List** drag ထည့်|
|2|Default sample data ဖျက်|
|3|`setupStudentList()` method ရေးပြီး constructor ကနေ ခေါ်|
|4|`Student.toString()` override (ဒါမှမဟုတ် custom CellRenderer)|
|5|Properties → **`selectionMode`** → `MULTIPLE_INTERVAL_SELECTION` ရွေး|
|6|Properties → Events tab (⚡) → `valueChanged` → event handler|

---

Student object အတွက် icon (profile picture) ပါတဲ့ custom renderer ထည့်ချင်ရင်၊ ဒါမှမဟုတ် JList selection ကို JTable filter နဲ့ ချိတ်ဆက်တဲ့ concrete code လိုချင်ရင် ပြောပါ။

---
# Part 3

Student object အတွက် icon (profile picture) ပါတဲ့ custom ListCellRenderer ကို ရှင်းပြပေးပါမယ်။

## Concept

Default `DefaultListCellRenderer` က text ပဲ ပြပေးနိုင်တယ် — icon ပါစေချင်ရင် `JLabel`-based custom renderer class တစ်ခု ရေးပြီး `setIcon()` + `setText()` ကို override method ထဲမှာ ခေါ်ရပါတယ်။

---

## Step 1: Student Model မှာ Profile Picture Path Field ရှိရမယ်

```java
public class Student implements Serializable {
    private String studentId;
    private String name;
    private String photoPath; // profile picture file path (e.g. "photos/S001.jpg")

    // getters/setters...
    public String getPhotoPath() { return photoPath; }
}
```

## Step 2: Custom ListCellRenderer Class ရေးခြင်း

Separate class file အနေနဲ့ ရေးတာက reusable ဖြစ်ပြီး code ရှင်းပါတယ် (`com.ams.qrcode.ui` package ထဲမှာ ထားလို့ရတယ်):

```java
package com.ams.qrcode.ui;

import com.ams.qrcode.model.Student;
import javax.swing.*;
import java.awt.*;
import java.io.File;

public class StudentListCellRenderer extends JLabel implements ListCellRenderer<Student> {

    private static final int ICON_SIZE = 40; // profile picture size (px)
    private final ImageIcon defaultIcon; // photo မရှိရင် ပြမယ့် default icon

    public StudentListCellRenderer() {
        setOpaque(true); // background color ပြသင့်ဖို့ လိုအပ်တယ်
        setIconTextGap(10); // icon နဲ့ text ကြား space
        setHorizontalAlignment(LEFT);

        // Default placeholder icon (photo မရှိတဲ့ student အတွက်)
        ImageIcon raw = new ImageIcon(getClass().getResource("/icons/default_avatar.png"));
        Image scaled = raw.getImage().getScaledInstance(ICON_SIZE, ICON_SIZE, Image.SCALE_SMOOTH);
        defaultIcon = new ImageIcon(scaled);
    }

    @Override
    public Component getListCellRendererComponent(JList<? extends Student> list, Student student,
            int index, boolean isSelected, boolean cellHasFocus) {

        // Text setup
        setText(student.getName() + " (" + student.getStudentId() + ")");

        // Icon setup
        setIcon(loadStudentIcon(student));

        // Selection color
        if (isSelected) {
            setBackground(list.getSelectionBackground());
            setForeground(list.getSelectionForeground());
        } else {
            setBackground(list.getBackground());
            setForeground(list.getForeground());
        }

        return this;
    }

    private ImageIcon loadStudentIcon(Student student) {
        String path = student.getPhotoPath();

        if (path == null || path.isEmpty() || !new File(path).exists()) {
            return defaultIcon; // photo မရှိရင် default icon ပြ
        }

        try {
            ImageIcon icon = new ImageIcon(path);
            Image scaled = icon.getImage().getScaledInstance(ICON_SIZE, ICON_SIZE, Image.SCALE_SMOOTH);
            return new ImageIcon(scaled);
        } catch (Exception ex) {
            return defaultIcon; // load fail ရင်လည်း default icon ပြ
        }
    }
}
```

## Step 3: JList မှာ Custom Renderer ကို Set လုပ်ခြင်း

Main frame constructor ထဲ (`initComponents()` အပြင်ဘက်):

```java
public MainFrame() {
    initComponents();

    setupStudentList();
}

private void setupStudentList() {
    DefaultListModel<Student> listModel = new DefaultListModel<>();
    for (Student s : studentDao.getAll()) {
        listModel.addElement(s);
    }

    studentJList.setModel(listModel);
    studentJList.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);
    studentJList.setCellRenderer(new StudentListCellRenderer()); // <-- custom renderer ချိတ်
    studentJList.setFixedCellHeight(50); // icon size (40) + padding အတွက် row height ချဲ့
}
```

---

## Result (Visual)

```
┌─────────────────────────────────┐
│ [👤]  Aung Aung (S001)           │
│ [👤]  Su Su (S002)               │  ← photo ရှိရင် actual photo, မရှိရင် default avatar
│ [🖼️]  Ko Ko (S003)               │
└─────────────────────────────────┘
```

---

## ⚠️ Performance Tip (Important)

Photo file တွေကို JList render တိုင်း disk ကနေ ပြန်ဖတ်ရင် (`new ImageIcon(path)` ခေါ်တိုင်း) — item အများကြီး scroll လုပ်ရင် **lag** ဖြစ်နိုင်ပါတယ်။ Photo count များရင် **icon cache** သုံးရင် ပိုမြန်ပါတယ်:

```java
public class StudentListCellRenderer extends JLabel implements ListCellRenderer<Student> {

    private final Map<String, ImageIcon> iconCache = new HashMap<>(); // path → icon cache
    private final ImageIcon defaultIcon;

    // ... constructor same as above ...

    private ImageIcon loadStudentIcon(Student student) {
        String path = student.getPhotoPath();
        if (path == null || path.isEmpty()) return defaultIcon;

        // Cache ထဲမှာ ရှိပြီးသားဆိုရင် ပြန်သုံး (disk ပြန်မဖတ်တော့ဘူး)
        if (iconCache.containsKey(path)) {
            return iconCache.get(path);
        }

        try {
            File file = new File(path);
            if (!file.exists()) {
                iconCache.put(path, defaultIcon);
                return defaultIcon;
            }
            ImageIcon icon = new ImageIcon(path);
            Image scaled = icon.getImage().getScaledInstance(ICON_SIZE, ICON_SIZE, Image.SCALE_SMOOTH);
            ImageIcon result = new ImageIcon(scaled);
            iconCache.put(path, result); // cache ထဲ သိမ်း
            return result;
        } catch (Exception ex) {
            iconCache.put(path, defaultIcon);
            return defaultIcon;
        }
    }
}
```

---

## QR Code ကိုယ်တိုင်ကို Icon အနေနဲ့ ပြချင်ရင် (Bonus)

Profile picture အစား QR code image ကိုယ်တိုင် preview ပြချင်ရင်လည်း logic တူတူပါပဲ — `photoPath` field ကို `qrCodePath` (ဒါမှမဟုတ် naming convention အတိုင်း `qrcodes/{studentId}.png`) လို့ ပြောင်းရုံပါ:

```java
private ImageIcon loadQRIcon(Student student) {
    String qrPath = "qrcodes/" + student.getStudentId() + ".png"; // convention-based path
    // ... same loading logic
}
```

---

**Summary**: `ListCellRenderer<Student>` interface ကို implement လုပ်ထားတဲ့ custom `JLabel` subclass ရေးပြီး `getListCellRendererComponent()` ထဲမှာ `setIcon()` + `setText()` ကို configure လုပ်ရပါတယ်။ Performance အတွက် icon cache (`HashMap`) သုံးဖို့ အကြံပြုပါတယ် — Student list ကြီးလာရင် (100+ students) especially အရေးကြီးပါတယ်။

`default_avatar.png` icon file ကို project resource folder (`src/main/resources/icons/`) ထဲ ထည့်ဖို့ လိုအပ်ပါလိမ့်မယ် — resource path setup အကြောင်း ပိုသိချင်ရင် ပြောပါ။