NetBeans GUI Builder (drag-and-drop) မှာ JDialog ကို ဘယ်လို create/သုံးရမလဲ အသေးစိတ် ရှင်းပြပေးပါမယ်။

## Step 1: JDialog Form အသစ် Create လုပ်ခြင်း

JDialog ကို main JFrame form ထဲမှာ drag ဆွဲထည့်လို့ **မရပါဘူး** — JDialog က **သီးခြား Form file** တစ်ခု အနေနဲ့ create ရပါတယ်။

1. Project ထဲက package (ဥပမာ `com.ams.qrcode.ui`) ပေါ် **right-click**
2. **New → Other...** ကို ရွေးပါ
3. Categories list ထဲက **"Swing GUI Forms"** ကို ရွေးပါ
4. File Types ထဲက **"JDialog Form"** ကို ရွေးပါ
5. **Next** နှိပ်ပြီး Class Name ထည့်ပါ (ဥပမာ: `EditStudentDialog`)
6. **Finish** နှိပ်ပါ

ဒါဆိုရင် `EditStudentDialog.java` ဆိုတဲ့ file အသစ်တစ်ခု auto-generate ဖြစ်ပြီး Design view ဖွင့်ပေးပါလိမ့်မယ်။

## Step 2: Dialog ထဲကို Component များ Drag ဆွဲထည့်ခြင်း

Normal JFrame form ထဲ component ထည့်သလိုပဲ — Palette ကနေ JLabel, JTextField, JButton (Save, Cancel) တွေကို Design view ပေါ် drag ဆွဲချပါ။

```java
// NetBeans auto-generate လုပ်ပေးမယ့် code structure
public class EditStudentDialog extends javax.swing.JDialog {
    public EditStudentDialog(java.awt.Frame parent, boolean modal) {
        super(parent, modal);
        initComponents(); // GUI Builder auto-generated
    }
    // ... auto-generated component code ...
}
```

⚠️ **သတိပြုရန်**: Auto-generate ဖြစ်တဲ့ constructor `(java.awt.Frame parent, boolean modal)` ကို NetBeans က default ပေးထားတာပါ — modal dialog ဖြစ်ချင်ရင် `modal = true` ဖြစ်ဖို့ လိုပါတယ်။

## Step 3: Save/Cancel Button ကို Event Handler တွဲခြင်း

Design view ထဲမှာ Save button ကို double-click လုပ်ပါ — NetBeans က event handler method auto-generate လုပ်ပေးမယ်:

```java
private void saveBtnActionPerformed(java.awt.event.ActionEvent evt) {
    // Data ကို field object ထဲ သိမ်းမယ်
    this.studentName = nameField.getText();
    this.saved = true; // parent ကို "save ဖြစ်ပြီ" ဆိုတာ သိအောင်
    dispose(); // dialog ပိတ်မယ်
}

private void cancelBtnActionPerformed(java.awt.event.ActionEvent evt) {
    this.saved = false;
    dispose(); // dialog ပိတ်ရုံပဲ၊ data မသိမ်း
}
```

## Step 4: Data Field များနဲ့ Getter Method ထည့်ခြင်း

Dialog ကနေ parent frame ဆီ data ပြန်ပို့ဖို့ field + getter method ထည့်ရပါတယ် (`initComponents()` **အပြင်ဘက်**မှာ manual ထည့်ရမယ်):

```java
public class EditStudentDialog extends javax.swing.JDialog {
    private boolean saved = false;
    private String studentName;

    public EditStudentDialog(java.awt.Frame parent, boolean modal) {
        super(parent, modal);
        initComponents();
    }

    // Existing student data ကို dialog ဖွင့်ချိန်မှာ pre-fill လုပ်ချင်ရင်
    public void setStudentData(Student student) {
        nameField.setText(student.getName());
    }

    public boolean isSaved() {
        return saved;
    }

    public String getStudentName() {
        return studentName;
    }

    private void saveBtnActionPerformed(java.awt.event.ActionEvent evt) {
        this.studentName = nameField.getText();
        this.saved = true;
        dispose();
    }

    private void cancelBtnActionPerformed(java.awt.event.ActionEvent evt) {
        this.saved = false;
        dispose();
    }
}
```

## Step 5: Main Frame ကနေ Dialog ကို ဖွင့်ခြင်း

Main Frame (student list window) ရဲ့ "Edit" button event handler ထဲမှာ:

```java
private void editBtnActionPerformed(java.awt.event.ActionEvent evt) {
    EditStudentDialog dialog = new EditStudentDialog(this, true); // this = parent frame, true = modal
    dialog.setStudentData(selectedStudent); // optional: existing data pre-fill

    dialog.setLocationRelativeTo(this); // parent frame အလယ်မှာ ပေါ်ဖို့
    dialog.setVisible(true); // ⚠️ dialog ပိတ်တဲ့အထိ code ဒီနေရာမှာ block ဖြစ်နေမယ် (modal ဖြစ်လို့)

    // dialog.dispose() ခေါ်ပြီးမှ ဒီအောက်က code တွေ run ဖြစ်မယ်
    if (dialog.isSaved()) {
        String newName = dialog.getStudentName();
        // Database update, table refresh စတာတွေ ဒီနေရာမှာ လုပ်ရမယ်
        studentDao.updateName(selectedStudent.getId(), newName);
        refreshTable();
    }
}
```

---

## Modal ဆိုတာ ဘာလဲ (အရေးကြီးလို့ ထပ်ရှင်းပြပါမယ်)

- `modal = true` → Dialog ပွင့်နေချိန် parent frame ကို click **လုံးဝ မရ** (dialog ပိတ်မှသာ parent ကို ပြန် interact လုပ်လို့ရမယ်)
- `modal = false` → Parent frame ကို dialog ပွင့်နေချိန်တောင် interact လုပ်လို့ရသေးတယ်

Edit/Add form အများစုအတွက် **modal = true** ပဲ သုံးလေ့ရှိပါတယ် (data consistency ကို ထိန်းဖို့)။

---

## QRCode Project အတွက် Concrete Flow

```
MainFrame (student table)
   ↓ [Edit button click]
   → new EditStudentDialog(this, true)
   → dialog.setStudentData(selectedStudent)
   → dialog.setVisible(true)   ← code ဒီနေရာမှာ block
   ↓ [dialog.dispose() ခေါ်ပြီးမှ]
   → dialog.isSaved() check
   → true ဆိုရင် DAO update + refreshTable()
```

---

**Quick Checklist**:

|Step|Action|
|---|---|
|1|Package → New → Other → Swing GUI Forms → **JDialog Form**|
|2|Palette ကနေ component drag ဆွဲထည့်|
|3|Save/Cancel button double-click → event handler auto-gen|
|4|`saved` flag + getter method manual ထည့် (initComponents() အပြင်)|
|5|Parent frame ကနေ `new Dialog(this, true)` ခေါ်ပြီး `setVisible(true)`|

Dialog ထဲမှာ FormattedField (student ID) ဒါမှမဟုတ် ComboBox (class ရွေး) ထည့်ချင်တဲ့ concrete form layout ရှိရင် ပြောပါ၊ code ပိုပြည့်စုံအောင် ရေးပေးပါမယ်။