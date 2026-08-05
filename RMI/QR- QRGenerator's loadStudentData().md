# `loadStudentData()` — RMI ပြောင်းထားသော Version

Table UI setup logic (column width, row height) အားလုံးကို **အပြောင်းအလဲ မရှိ** ဘဲ ဆက်ထားပြီး၊ `studentDAO.searchStudents()` ကို RMI service ခေါ်ရာမှာ SwingWorker ထဲ ပြောင်းပေးထားပါတယ်:

```java
public void loadStudentData(String keyword) {
    SwingWorker<List<Student>, Void> worker = new SwingWorker<>() {
        @Override
        protected List<Student> doInBackground() throws Exception {
            return studentService.searchStudents(keyword);
        }

        @Override
        protected void done() {
            try {
                List<Student> list = get();

                DefaultTableModel model = new DefaultTableModel(
                        new String[]{"ID", "Std Id", "Name", "Age", "Sex", "Department", "Email", "Dob", "QR Token"}, 0
                ) {
                    @Override
                    public boolean isCellEditable(int row, int col) {
                        return false;
                    }
                };

                SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
                for (Student s : list) {
                    String dob = s.getDob() != null ? sdf.format(s.getDob()) : "";
                    model.addRow(new Object[]{
                        s.getId(),
                        s.getStudentId(),
                        s.getName(),
                        calculateAge(s.getDob()),
                        s.getSex(),
                        s.getDepartment(),
                        s.getEmail(),
                        dob,
                        s.getQrToken()
                    });
                }
                tblStudents.setModel(model);

                // --- ADJUST COLUMN WIDTHS & SPACING HERE --- (ပြောင်းစရာမလို)
                TableColumnModel columnModel = tblStudents.getColumnModel();
                columnModel.getColumn(0).setPreferredWidth(40);  // ID
                columnModel.getColumn(1).setPreferredWidth(40);  // Std Id
                columnModel.getColumn(2).setPreferredWidth(150); // Name
                columnModel.getColumn(3).setPreferredWidth(40);  // Age
                columnModel.getColumn(4).setPreferredWidth(60);  // Sex
                columnModel.getColumn(5).setPreferredWidth(120); // Department
                columnModel.getColumn(6).setPreferredWidth(150); // Email
                columnModel.getColumn(7).setPreferredWidth(130); // Dob
                columnModel.getColumn(8).setPreferredWidth(70);  // QR Token

                tblStudents.setRowHeight(24);
                tblStudents.setIntercellSpacing(new Dimension(8, 4));

            } catch (Exception e) {
                logger.log(Level.SEVERE, "Database fetch error", e);
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Failed to load students: " + e.getCause(),
                    "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

## ပြောင်းလဲထားသော အချက်များ

|မူလ Code|ပြောင်းထားသော Code|အကြောင်းရင်း|
|---|---|---|
|`try { studentDAO.searchStudents(keyword); ... } catch (DataAccessException e)` — synchronous, EDT ပေါ် တိုက်ရိုက်ခေါ်|`doInBackground()` ထဲ RMI call, `done()` ထဲ table populate|Network round-trip ဖြစ်လို့ background thread လိုအပ်|
|Table model creation + `addRow()` loop + column width setting|**အားလုံး `done()` ထဲ** ရွှေ့|`done()` က EDT ပေါ်မှာ run နေလို့ Swing component (JTable) ကို ဒီထဲမှာပဲ ထိသင့်|
|`catch (DataAccessException e)`|`catch (Exception e)` + `e.getCause()`|`get()` က `ExecutionException` ထဲ wrap ပြန်ပေးလို့ `getCause()` ကနေ actual exception ယူရမယ်|
|`e.getMessage()`|`e.getCause()`|RMI/Server-side exception ရဲ့ real message ကို ပြသဖို့|

## ⚠️ သတိထားရမယ့်အချက် — `loadStudentData()` ကို Frequent ခေါ်နေတဲ့ Method

သင့် project ရဲ့ Register/Update/Delete/Generate button တွေအားလုံးက success ဖြစ်ပြီးတိုင်း `loadStudentData("")` ကို ခေါ်နေပါတယ် — SwingWorker version ဖြစ်သွားတဲ့အတွက် **ခေါ်တိုင်း network round-trip တစ်ခါ ထပ်ဖြစ်** ပါလိမ့်မယ်။ ဒါက functionally မှန်ကန်ပေမယ့် (UI freeze မဖြစ်တော့ဘူးဆိုတော့) — Table refresh ကို **သိသိသာသာ delay** ခံစားရရင် (Server latency များနေရင်) “Refreshing…” placeholder row တစ်ခု ထည့်ပြီး UX ပိုကောင်းအောင် လုပ်လို့ရပါတယ်:

```java
// Optional UX enhancement — Table ကို "Loading..." placeholder ပြထားနိုင်
tblStudents.setModel(new DefaultTableModel(new String[]{"Loading..."}, 0));
worker.execute();
```

## `SimpleDateFormat` Thread-Safety မှတ်ချက်

`SimpleDateFormat` instance ကို `done()` ထဲမှာပဲ create ထားတာက **thread-safe** ပါတယ် (SimpleDateFormat က thread-unsafe class ဖြစ်ပေမယ့် — method local variable အနေနဲ့ တစ်ခေါ်တစ်ခါ create ထားလို့ concurrent access ပြဿနာ ဖြစ်နိုင်ချေ မရှိပါ)။ Field level (class-level) `static SimpleDateFormat` ထားမယ်ဆိုရင်တော့ RMI service ဘက်မှာ concurrent client များစွာ access လုပ်ရင် ပြဿနာ ဖြစ်နိုင်ပါတယ် (Part 4 က thread-safety concept အတိုင်း) — ဒီနေရာမှာတော့ local variable ဖြစ်နေလို့ safe ပါတယ်။

---

`calculateAge()` method (client-side helper) ကို ပြပေးလိုပါသလား — RMI ပြောင်းစရာမလိုပါ (local calculation) ဆိုတာ confirm လုပ်ပေးနိုင်ပါတယ်။ ဒါမှမဟုတ် Project ရဲ့ RMI conversion **အားလုံးပြီးပြီ** ဆိုရင် Deployment/Testing checklist ချုပ်ပေးရမလား?

---

# "Refreshing…" Placeholder Row ထည့်နည်း

`loadStudentData()` ကို RMI ခေါ်တာ ကြာနိုင်တဲ့အတွက်, `doInBackground()` **မခေါ်ခင်**မှာ placeholder table model ကို ချက်ချင်း ပြပြီး, result ရလာမှ table ကို actual data နဲ့ overwrite လုပ်ရပါမယ်.

```java
public void loadStudentData(String keyword) {

    // ✅ RMI call မလုပ်ခင် — Placeholder ကို ချက်ချင်းပြ (EDT ပေါ်မှာ instant ဖြစ်)
    DefaultTableModel loadingModel = new DefaultTableModel(
            new String[]{"Status"}, 0
    ) {
        @Override
        public boolean isCellEditable(int row, int col) {
            return false;
        }
    };
    loadingModel.addRow(new Object[]{"Refreshing…"});
    tblStudents.setModel(loadingModel);

    SwingWorker<List<Student>, Void> worker = new SwingWorker<>() {
        @Override
        protected List<Student> doInBackground() throws Exception {
            return studentService.searchStudents(keyword);
        }

        @Override
        protected void done() {
            try {
                List<Student> list = get();

                // ✅ Actual data ရလာမှ real table model နဲ့ ပြန်အစားထိုး
                DefaultTableModel model = new DefaultTableModel(
                        new String[]{"ID", "Std Id", "Name", "Age", "Sex", "Department", "Email", "Dob", "QR Token"}, 0
                ) {
                    @Override
                    public boolean isCellEditable(int row, int col) {
                        return false;
                    }
                };

                SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
                for (Student s : list) {
                    String dob = s.getDob() != null ? sdf.format(s.getDob()) : "";
                    model.addRow(new Object[]{
                        s.getId(), s.getStudentId(), s.getName(),
                        calculateAge(s.getDob()), s.getSex(), s.getDepartment(),
                        s.getEmail(), dob, s.getQrToken()
                    });
                }
                tblStudents.setModel(model);

                TableColumnModel columnModel = tblStudents.getColumnModel();
                columnModel.getColumn(0).setPreferredWidth(40);
                columnModel.getColumn(1).setPreferredWidth(40);
                columnModel.getColumn(2).setPreferredWidth(150);
                columnModel.getColumn(3).setPreferredWidth(40);
                columnModel.getColumn(4).setPreferredWidth(60);
                columnModel.getColumn(5).setPreferredWidth(120);
                columnModel.getColumn(6).setPreferredWidth(150);
                columnModel.getColumn(7).setPreferredWidth(130);
                columnModel.getColumn(8).setPreferredWidth(70);
                tblStudents.setRowHeight(24);
                tblStudents.setIntercellSpacing(new Dimension(8, 4));

            } catch (Exception e) {
                logger.log(Level.SEVERE, "Database fetch error", e);

                // ✅ Error ဖြစ်ရင်လည်း placeholder row ကို error message နဲ့ ပြောင်း
                DefaultTableModel errorModel = new DefaultTableModel(new String[]{"Status"}, 0);
                errorModel.addRow(new Object[]{"Failed to load data"});
                tblStudents.setModel(errorModel);

                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Failed to load students: " + e.getCause(),
                    "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

## အလုပ်လုပ်ပုံ

```
1. loadStudentData() ခေါ်ချင်းမှာ
   → Table ထဲ "Refreshing…" row ချက်ချင်းပေါ် (0.0 second delay)
                    │
2. worker.execute() → doInBackground() 
   background thread ကနေ Server ကို RMI call ပို့
                    │
3. Response ရလာမှ (0.5s, 1s, ဘယ်လောက်ပဲ ကြာကြာ)
   done() ထဲမှာ Actual data ပါတဲ့ table model ကို ပြန်ထည့်
```

## Optional — Loading Spinner/Icon ထပ်ထည့်ချင်ရင်

Text ချည်း "Refreshing…" မကျေနပ်ဘဲ visual indicator ပါချင်ရင် `JProgressBar` (indeterminate mode) ကို table အပေါ် (သို့) status bar မှာ ထည့်နိုင်ပါတယ်:

```java
private JProgressBar loadingBar = new JProgressBar();

// loadStudentData() ရဲ့ အစမှာ
loadingBar.setIndeterminate(true);
loadingBar.setVisible(true);

// done() ရဲ့ အဆုံးမှာ (try/catch ၂ ခုစလုံးအတွက် finally ထဲ ထည့်တာ ပိုကောင်း)
loadingBar.setVisible(false);
loadingBar.setIndeterminate(false);
```

`finally` block ကို `done()` ထဲမှာ ထည့်ရင် success/error ၂ မျိုးစလုံးအတွက် loading indicator ကို consistant ဖျောက်ပေးနိုင်ပါတယ်:

```java
@Override
protected void done() {
    try {
        List<Student> list = get();
        // ... table populate ...
    } catch (Exception e) {
        // ... error handling ...
    } finally {
        loadingBar.setVisible(false); // Success/Error ၂ မျိုးစလုံးမှာ ဖျောက်
    }
}
```

---

Table ကို placeholder ပြနေတုန်း User က Search box ထဲ ထပ်ရိုက်ရင် (ဥပမာ - fast typing) request များစွာ တစ်ပြိုင်နက် fire ဖြစ်တတ်ပါတယ် — Debounce (Timer နဲ့ 300ms delay ထားပြီး search) logic ထည့်ချင်ပါသလား?