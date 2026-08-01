## Part 5: JTable, JList (Model-based Components)

Swing ရဲ့ **most powerful** components တွေထဲက JTable နဲ့ JList ပါ။ ဒါတွေက **Model-View architecture** သုံးထားလို့ data ကို component ကနေ **separate** ထားနိုင်ပါတယ် (data ပြောင်းရင် UI auto-update ဖြစ်ပါတယ်)။

### JList (Simple List Display)

#### Basic JList (Static data)

```java
String[] items = {"Apple", "Banana", "Orange", "Mango"};
JList<String> list = new JList<>(items);

JScrollPane scrollPane = new JScrollPane(list);  // scroll လိုအပ်ရင် JScrollPane နဲ့ wrap
frame.add(scrollPane);
```

#### DefaultListModel (Dynamic data - add/remove လုပ်ချင်ရင်)

```java
DefaultListModel<String> listModel = new DefaultListModel<>();
listModel.addElement("Apple");
listModel.addElement("Banana");
listModel.addElement("Orange");

JList<String> list = new JList<>(listModel);

// Item အသစ်ထည့်ခြင်း
listModel.addElement("Mango");

// Item ဖျက်ခြင်း
listModel.remove(0);  // index 0 ကိုဖျက်

// Selected item ရယူခြင်း
String selected = list.getSelectedValue();

// Selection listener
list.addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {  // event fire ထပ်ထပ်ဖြစ်တာကို prevent
        System.out.println("Selected: " + list.getSelectedValue());
    }
});
```

**Selection Mode:**

```java
list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);        // တစ်ခုပဲရွေးလို့ရ
list.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION); // Ctrl/Shift ကိုင်ပြီး multiple ရွေးလို့ရ
```

### JTable (Table Display - အသုံးအများဆုံး)

#### Basic JTable (Static data)

```java
String[] columnNames = {"Name", "Age", "City"};
Object[][] data = {
    {"John", 25, "Yangon"},
    {"Mary", 30, "Mandalay"},
    {"Tom", 22, "Naypyidaw"}
};

JTable table = new JTable(data, columnNames);
JScrollPane scrollPane = new JScrollPane(table);  // JTable ကိုတော့ scrollpane ထဲမှာ ထားရမယ်
frame.add(scrollPane);
```

**Important:** JTable ကို JScrollPane ထဲမှာ **အမြဲထည့်ပါ** - ဒါမှ table header (column names) ကို correctly ပြသနိုင်ပြီး scroll လုပ်လို့ရမှာပါ။

#### DefaultTableModel (Dynamic data - Recommended way)

```java
String[] columnNames = {"Name", "Age", "City"};
DefaultTableModel model = new DefaultTableModel(columnNames, 0);  // 0 = initial row count

JTable table = new JTable(model);

// Row အသစ်ထည့်ခြင်း
model.addRow(new Object[]{"John", 25, "Yangon"});
model.addRow(new Object[]{"Mary", 30, "Mandalay"});

// Row ဖျက်ခြင်း
model.removeRow(0);  // index 0 ကိုဖျက်

// Cell value ပြောင်းခြင်း
model.setValueAt("Bangkok", 0, 2);  // row 0, column 2 ကို "Bangkok" ပြောင်း

// Cell value ရယူခြင်း
Object value = model.getValueAt(0, 0);  // row 0, column 0

// Selected row ရယူခြင်း
int selectedRow = table.getSelectedRow();
if (selectedRow != -1) {
    String name = (String) table.getValueAt(selectedRow, 0);
}
```

#### Practical Example - CRUD Table (Add/Delete rows)

```java
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;

public class TableCRUDExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Employee Table");
            frame.setLayout(new BorderLayout());
            
            // Table setup
            String[] columns = {"Name", "Position", "Salary"};
            DefaultTableModel model = new DefaultTableModel(columns, 0);
            JTable table = new JTable(model);
            JScrollPane scrollPane = new JScrollPane(table);
            
            // Sample data
            model.addRow(new Object[]{"Aung Aung", "Developer", 500000});
            model.addRow(new Object[]{"Su Su", "Designer", 450000});
            
            // Input fields
            JPanel inputPanel = new JPanel(new FlowLayout());
            JTextField nameField = new JTextField(10);
            JTextField positionField = new JTextField(10);
            JTextField salaryField = new JTextField(10);
            JButton addButton = new JButton("Add");
            JButton deleteButton = new JButton("Delete Selected");
            
            addButton.addActionListener(e -> {
                String name = nameField.getText();
                String position = positionField.getText();
                String salary = salaryField.getText();
                
                if (!name.isEmpty() && !position.isEmpty() && !salary.isEmpty()) {
                    model.addRow(new Object[]{name, position, salary});
                    nameField.setText("");
                    positionField.setText("");
                    salaryField.setText("");
                }
            });
            
            deleteButton.addActionListener(e -> {
                int selectedRow = table.getSelectedRow();
                if (selectedRow != -1) {
                    model.removeRow(selectedRow);
                } else {
                    JOptionPane.showMessageDialog(frame, "Please select a row to delete");
                }
            });
            
            inputPanel.add(new JLabel("Name:"));
            inputPanel.add(nameField);
            inputPanel.add(new JLabel("Position:"));
            inputPanel.add(positionField);
            inputPanel.add(new JLabel("Salary:"));
            inputPanel.add(salaryField);
            inputPanel.add(addButton);
            inputPanel.add(deleteButton);
            
            frame.add(inputPanel, BorderLayout.NORTH);
            frame.add(scrollPane, BorderLayout.CENTER);
            
            frame.setSize(600, 300);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

### Table Cell Editability ထိန်းချုပ်ခြင်း

Default အနေနဲ့ JTable cell တွေကို double-click ပြီး edit လုပ်လို့ရပါတယ်။ Read-only ဖြစ်စေချင်ရင် `DefaultTableModel` ကို override လုပ်ရပါတယ်:

```java
DefaultTableModel model = new DefaultTableModel(columns, 0) {
    @Override
    public boolean isCellEditable(int row, int column) {
        return false;  // cell အားလုံး read-only
    }
};
```

Column တစ်ခုချင်းစီအလိုက် ထိန်းချုပ်ချင်ရင်:

```java
DefaultTableModel model = new DefaultTableModel(columns, 0) {
    @Override
    public boolean isCellEditable(int row, int column) {
        return column != 0;  // column 0 (Name) ကိုပဲ edit မလို့ရအောင်
    }
};
```

### Table Selection Listener

```java
table.getSelectionModel().addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {
        int row = table.getSelectedRow();
        if (row != -1) {
            String name = (String) table.getValueAt(row, 0);
            System.out.println("Selected: " + name);
        }
    }
});
```

### Column Width ချိန်ညှိခြင်း

```java
table.getColumnModel().getColumn(0).setPreferredWidth(150);  // column 0 ရဲ့ width
table.getColumnModel().getColumn(1).setPreferredWidth(100);
```

### Quick Summary

|Component|Model|Method (add/remove)|
|---|---|---|
|`JList`|`DefaultListModel`|`addElement()`, `remove(index)`|
|`JTable`|`DefaultTableModel`|`addRow()`, `removeRow(index)`|

**Key Points:**

- Dynamic data (add/delete လုပ်ရင်) - **Model class** (`DefaultListModel`, `DefaultTableModel`) ကို သုံးပါ (raw array မသုံးပါနဲ့)
- JTable ကို JScrollPane နဲ့ **အမြဲ wrap** လုပ်ပါ
- `e.getValueIsAdjusting()` ကို selection listener တွေမှာ check လုပ်ပါ (duplicate event fire ကို prevent ဖို့)

---

ဒါက **Part 5 (JTable, JList)** ပါ။ Business applications တွေမှာ တော်တော်များများသုံးရမယ့် component တွေဖြစ်လို့ code ကို run ကြည့်ဖို့ recommend ပါတယ်။

**Part 6** မှာ Menus (JMenuBar) နဲ့ SwingWorker (background threading) တို့ကို ဆက်သင်ပေးရမလား၊ ဒါမှမဟုတ် FlatLaf setup ကို အရင်လုပ်ချင်ပါသလား?