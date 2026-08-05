NetBeans GUI Builder (drag-and-drop) မှာ JPopupMenu ကို ဘယ်လို create/သုံးရမလဲ ရှင်းပြပေးပါမယ်။

## Step 1: JPopupMenu ကို Palette ကနေ Drag ဆွဲထည့်ခြင်း

JPopupMenu က visible component မဟုတ်လို့ (screen ပေါ်မှာ အမြဲမြင်ရတဲ့ component မဟုတ်ဘူး) — drag ဆွဲချလိုက်တဲ့အခါ Design view ပေါ်မှာ **အမြင်မမြေ icon** (tray) အနေနဲ့ ပေါ်ပါလိမ့်မယ်။

1. Palette ထဲက **"Swing Menus"** category ဖွင့်ပါ
2. **"Popup Menu"** ကို Design view ပေါ် drag ဆွဲချပါ (form ရဲ့ ဘယ်နေရာမှာ drop လုပ်လုပ် ရပါတယ် — visible position မဟုတ်လို့)
3. Design view ရဲ့ **အောက်ခြေ** မှာ (Non-visual components tray) `jPopupMenu1` ဆိုတဲ့ icon ပေါ်လာပါလိမ့်မယ်

## Step 2: PopupMenu ထဲကို Menu Item များ ထည့်ခြင်း

1. `jPopupMenu1` icon ကို **double-click** လုပ်ပါ — ဒါဆိုရင် popup menu ကို "edit mode" ဖွင့်ပေးပြီး Design view ထဲမှာ dropdown structure ပြပါလိမ့်မယ်
2. Palette ကနေ **"Menu Item"** ကို ဒီ dropdown ထဲ drag ဆွဲထည့်ပါ (ဥပမာ: "Edit", "Delete", "Generate QR Code")
3. Item name တစ်ခုချင်းစီကို click ပြီး Properties window ကနေ `text` property ပြောင်းပါ

```
jPopupMenu1
  ├── editMenuItem     ("Edit")
  ├── deleteMenuItem   ("Delete")
  └── generateQRItem   ("Generate QR Code")
```

## Step 3: Menu Item များကို Event Handler တွဲခြင်း

Menu item တစ်ခုချင်းစီကို double-click လုပ်ရင် NetBeans က auto-generate event handler method ပေးပါလိမ့်မယ်:

```java
private void editMenuItemActionPerformed(java.awt.event.ActionEvent evt) {
    int selectedRow = studentTable.getSelectedRow();
    if (selectedRow >= 0) {
        Student s = tableModel.getStudentAt(selectedRow);
        EditStudentDialog dialog = new EditStudentDialog(this, true);
        dialog.setStudentData(s);
        dialog.setVisible(true);
        // ... save logic ...
    }
}

private void deleteMenuItemActionPerformed(java.awt.event.ActionEvent evt) {
    int selectedRow = studentTable.getSelectedRow();
    if (selectedRow >= 0) {
        int confirm = JOptionPane.showConfirmDialog(this,
            "Delete this student?", "Confirm", JOptionPane.YES_NO_OPTION);
        if (confirm == JOptionPane.YES_OPTION) {
            studentDao.delete(tableModel.getStudentAt(selectedRow).getId());
            refreshTable();
        }
    }
}

private void generateQRItemActionPerformed(java.awt.event.ActionEvent evt) {
    int selectedRow = studentTable.getSelectedRow();
    if (selectedRow >= 0) {
        Student s = tableModel.getStudentAt(selectedRow);
        generateQRCode(s.getStudentId());
    }
}
```

## Step 4: JTable ကို Right-Click Listener တွဲခြင်း (⚠️ အရေးကြီးဆုံး Step)

GUI Builder ကနေ auto-generate **မလုပ်ပေးတဲ့** အပိုင်းက ဒါပါ — JTable ပေါ် right-click လုပ်လိုက်ရင် popup menu ပေါ်စေဖို့ **manual code** ထည့်ရပါမယ်။

### Method 1: `setComponentPopupMenu()` (အလွယ်ဆုံးနည်း)

Table ကို click ရွေးပြီး **Properties window** ထဲမှာ `componentPopupMenu` property ကို ရှာပါ — dropdown ကနေ `jPopupMenu1` ကို ရွေးလိုက်ရုံပါပဲ (code manual ရေးစရာ **မလိုပါဘူး**):

```
Properties window (studentTable ရွေးထားစဉ်):
  componentPopupMenu → jPopupMenu1  ✅ ဒါလေးပဲ ရွေးလိုက်ရုံပါ
```

ဒါက **NetBeans GUI Builder နဲ့ အလွယ်ဆုံးနည်း** ဖြစ်ပြီး၊ right-click cursor position ပေါ်မှာ popup auto ပေါ်ပေးပါလိမ့်မယ်။

### Method 2: Manual MouseListener (row ကို auto-select ချင်ရင်)

`componentPopupMenu` property က popup ကို auto ပြပေးပေမယ့် — **right-click လုပ်တဲ့ row ကို auto-select** မလုပ်ပေးပါဘူး (default select ဖြစ်ထားတဲ့ row ပဲ ဆက်ရှိနေမယ်)။ Right-click လုပ်တဲ့ row ကို auto-select ချင်ရင် manual MouseListener ထည့်ရပါမယ် (constructor ထဲ, `initComponents()` အောက်မှာ):

```java
public MainFrame() {
    initComponents();

    studentTable.addMouseListener(new java.awt.event.MouseAdapter() {
        @Override
        public void mousePressed(java.awt.event.MouseEvent evt) {
            if (evt.isPopupTrigger() || SwingUtilities.isRightMouseButton(evt)) {
                int row = studentTable.rowAtPoint(evt.getPoint());
                if (row >= 0) {
                    studentTable.setRowSelectionInterval(row, row); // right-click လုပ်တဲ့ row ကို select
                }
            }
        }
    });
}
```

---

## Separator ထည့်ချင်ရင်

Menu item တွေကြား line ခွဲချင်ရင် — Palette ကနေ **"Separator"** ကို popup menu structure ထဲ drag ဆွဲထည့်ရုံပါပဲ (code ရေးစရာ မလိုပါဘူး):

```
jPopupMenu1
  ├── editMenuItem
  ├── generateQRItem
  ├── ─────────────  (Separator - drag ဆွဲထည့်ရုံ)
  └── deleteMenuItem
```

---

## Quick Checklist

|Step|Action|
|---|---|
|1|Palette → Swing Menus → **Popup Menu** → Design view drag ချ|
|2|`jPopupMenu1` icon double-click → dropdown edit mode ဝင်|
|3|Palette → **Menu Item** တွေ dropdown ထဲ drag ချ (Edit, Delete စသည်)|
|4|Item double-click → event handler code ရေး|
|5|Table ရွေးပြီး Properties → **`componentPopupMenu`** → `jPopupMenu1` ရွေး|
|6 (optional)|Right-click row auto-select ချင်ရင် manual MouseListener ထည့်|

---

QRCode project ရဲ့ student table အတွက် ဒီ popup menu ကို concrete code အပြည့်အစုံ (Edit/Delete/Generate QR item ၃ ခုလုံးနဲ့ တကွ) ရေးပေးရမလား၊ ဒါမှမဟုတ် `componentPopupMenu` property ကို Properties window ထဲမှာ ဘယ်နေရာမှာ ရှာရမလဲ screenshot-level detail လိုချင်ပါသလား?