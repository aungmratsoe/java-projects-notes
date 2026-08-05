# QRGenerator UI — DAO ခေါ်နေတဲ့ Method အားလုံးကို RMI Service ပြောင်းခြင်း

Method တွေအားလုံးမှာ ပုံစံ တူတူပါပဲ — `studentDAO.xxx()` ကို `studentService.xxx()` လို့ ပြောင်းပြီး **SwingWorker ထဲ ထည့်ရုံပါပဲ**။ Register button ကို အသေးစိတ်ဆုံး ပြပြီး၊ ကျန်တာတွေကို ဒီ pattern အတိုင်း တိုတိုပြပါမယ်။

## Field ကို Class အစမှာ ပြောင်းထားရမယ်

```java
// ❌ Before
private StudentDAO studentDAO = new StudentDAO();

// ✅ After
private StudentService studentService = RmiConnectionManager.getInstance().getStudentService();
```

## 1️⃣ Register Button

```java
private void registerButtonActionPerformed(java.awt.event.ActionEvent evt) {
    Student student = buildStudentFromForm(); // Form ကနေ Student object တည်ဆောက်တဲ့ method ရှိပြီးသားလို့ ယူဆ

    registerButton.setEnabled(false);

    SwingWorker<Void, Void> worker = new SwingWorker<>() {
        private boolean duplicate = false;
        private String duplicateOwnerName = "";

        @Override
        protected Void doInBackground() throws Exception {
            // Network round-trip ၂ ခါ လိုအပ်တာကို background thread ထဲမှာ အားလုံး ပြီးအောင်လုပ်
            Student existingStudent = studentService.getStudentByStudentId(student.getStudentId());
            if (existingStudent != null) {
                duplicate = true;
                duplicateOwnerName = existingStudent.getName();
                return null;
            }
            studentService.saveStudent(student);
            return null;
        }

        @Override
        protected void done() {
            registerButton.setEnabled(true);
            try {
                get(); // Exception ရှိမရှိ စစ်

                if (duplicate) {
                    JOptionPane.showMessageDialog(QRGenerator.this,
                        "Student ID '" + student.getStudentId() + "' is already taken by " + duplicateOwnerName + ".\nPlease use a unique Student ID.",
                        "Duplicate Student ID", JOptionPane.WARNING_MESSAGE);
                    return;
                }

                loadStudentData(""); // ဒါလည်း DAO ခေါ်နေရင် အောက်မှာ ပြင်ပေးပါမယ်
                handleClear();
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Student registered successfully!", "Success", JOptionPane.INFORMATION_MESSAGE);

            } catch (Exception ex) {
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Server error: " + ex.getCause(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

**Key point**: `getStudentByStudentId()` + `saveStudent()` — Network call **၂ ခု** ကို `doInBackground()` **တစ်ခုတည်း** ထဲမှာ အားလုံး ပြီးအောင် လုပ်ပါ။ Method တစ်ခုချင်းစီအတွက် SwingWorker သီးခြား ဆောက်ရင် UI ပိုနှေးသွားပါလိမ့်မယ် (round-trip ၂ ခါ ခွဲပြီး wait လုပ်ရလို့)။

## 2️⃣ Update Button

```java
private void btnUpdateActionPerformed(java.awt.event.ActionEvent evt) {
    Student formStudent = extractAndValidateStudentData();
    if (formStudent == null) {
        return; // Validation fail ရင် ရပ် — client-side ဖြစ်လို့ ပြောင်းစရာမလို
    }

    btnUpdate.setEnabled(false); // Double-click ကာကွယ်

    SwingWorker<Boolean, Void> worker = new SwingWorker<>() {
        // true = existing student ကို update, false = new student insert
        @Override
        protected Boolean doInBackground() throws Exception {
            String stdId = formStudent.getStudentId();
            Student existingStudent = studentService.getStudentByStudentId(stdId);

            if (existingStudent != null) {
                existingStudent.setName(formStudent.getName());
                existingStudent.setAge(formStudent.getAge());
                existingStudent.setSex(formStudent.getSex());
                existingStudent.setEmail(formStudent.getEmail());
                existingStudent.setDepartment(formStudent.getDepartment());
                existingStudent.setDob(formStudent.getDob());
                studentService.updateStudent(existingStudent);
                return true;
            } else {
                studentService.saveStudent(formStudent);
                return false;
            }
        }

        @Override
        protected void done() {
            btnUpdate.setEnabled(true);
            String stdId = formStudent.getStudentId();

            try {
                boolean wasUpdate = get();

                loadStudentData("");
                handleClear();

                if (wasUpdate) {
                    JOptionPane.showMessageDialog(QRGenerator.this,
                        "Student ID '" + stdId + "' updated successfully!",
                        "Success", JOptionPane.INFORMATION_MESSAGE);
                } else {
                    JOptionPane.showMessageDialog(QRGenerator.this,
                        "Student ID '" + stdId + "' was not found, so it was inserted as a new student!",
                        "Success", JOptionPane.INFORMATION_MESSAGE);
                }

            } catch (Exception e) {
                logger.log(Level.SEVERE, "Failed to update or save student", e);
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Operation failed: " + e.getCause(),
                    "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```


**Note**: `existingStudent.setXxx(...)` logic (server ကနေ ရလာတဲ့ object ကို client ဘက်မှာ field ပြင်ခြင်း) ကို **client memory ထဲမှာပဲ** လုပ်ပြီးမှ `updateStudent()` ကို ခေါ်တာမို့ RMI နဲ့ ကိုက်ညီပါတယ် — ပြောင်းစရာ logic မလိုပါ။

## 3️⃣ Delete Button

```java
if (confirm == JOptionPane.YES_OPTION) {
    deleteStudentData(studentId); // ဒီ method ကိုပဲ ပြင်ရမှာပါ
}
```

`deleteStudentData()` method ထဲကို ပြင်ရမယ်:

```java
private void deleteStudentData(String studentId) {
    SwingWorker<Boolean, Void> worker = new SwingWorker<>() {
        @Override
        protected Boolean doInBackground() throws Exception {
            return studentService.deleteStudent(studentId);
        }

        @Override
        protected void done() {
            try {
                boolean deleted = get();
                if (deleted) {
                    loadStudentData("");
                    JOptionPane.showMessageDialog(QRGenerator.this,
                        "Student deleted successfully!", "Success", JOptionPane.INFORMATION_MESSAGE);
                } else {
                    JOptionPane.showMessageDialog(QRGenerator.this,
                        "Student not found or already deleted.", "Notice", JOptionPane.WARNING_MESSAGE);
                }
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Server error: " + ex.getCause(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

## 4️⃣ Generate QR Button

```java
private void generateQrButtonActionPerformed(java.awt.event.ActionEvent evt) {
    // ... option dialog / currentToken check logic (client-side, ပြောင်းစရာမလို) ...

    boolean needsNewToken = (option == JOptionPane.OK_OPTION)
        || (currentToken == null || currentToken.trim().isEmpty());

    generateQrButton.setEnabled(false);

    SwingWorker<String, Void> worker = new SwingWorker<>() {
        @Override
        protected String doInBackground() throws Exception {
            String token = currentToken;
            if (needsNewToken) {
                token = java.util.UUID.randomUUID().toString();
                studentService.updateQrToken(studentId, token);
            }
            return token;
        }

        @Override
        protected void done() {
            generateQrButton.setEnabled(true);
            try {
                currentToken = get();

                // QR payload ဖန်တီးတာနဲ့ QR image ဆွဲတာ (network မလိုတဲ့ local logic) 
                // ကတော့ done() ထဲမှာပဲ ဆက်လုပ်လို့ရပါတယ် (EDT ပေါ်မှာ)
                String rawQrData = String.format(
                    "StudentID: %s\nName: %s\nAge: %s\nSex: %s\nDept: %s\nEmail: %s\nToken: %s",
                    studentId, /* ... other fields ... */ currentToken
                );
                // generateAndDisplayQrImage(rawQrData); ← QR generation logic ရှိပြီးသားအတိုင်း ဆက်ခေါ်

            } catch (Exception ex) {
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Server error: " + ex.getCause(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

**Important**: QR image ဖန်တီးတဲ့ logic (`ZXing` စတဲ့ library သုံးပြီး barcode/QR ဆွဲတာ) ဟာ **local operation** ဖြစ်လို့ (network မလိုပါ) — `doInBackground()` ထဲ ထည့်စရာမလိုဘဲ `done()` ထဲမှာ (EDT ပေါ်) ဆက်လုပ်လို့ရပါတယ်။

## 5️⃣ Search Helper Method

```java
private void loadStudentData(String keyword) {
    SwingWorker<List<Student>, Void> worker = new SwingWorker<>() {
        @Override
        protected List<Student> doInBackground() throws Exception {
            return studentService.searchStudents(keyword);
        }

        @Override
        protected void done() {
            try {
                List<Student> list = get();
                // JTable model ထဲ populate လုပ်တဲ့ logic ရှိပြီးသားအတိုင်း ဆက်သုံး
                populateTable(list);
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Server error: " + ex.getCause(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

**Note**: `loadStudentData("")` ကို Register/Update button တွေထဲက `done()` ကနေ ခေါ်နေတာက **SwingWorker ထဲက SwingWorker ခေါ်တာ** ဖြစ်ပေမယ့် — `done()` က EDT ပေါ်မှာ run နေလို့ ဒီထဲက `loadStudentData()` ခေါ်တာ **လုံးဝ ပြဿနာမရှိပါ** (Button click ကနေ ခေါ်သလိုပဲ safe ပါတယ်)။

## အနှစ်ချုပ် — Conversion Pattern

|DAO Pattern|RMI Pattern|
|---|---|
|`studentDAO.xxx()` — တိုက်ရိုက်ခေါ်|`studentService.xxx()` — SwingWorker ထဲကနေ ခေါ်|
|Network call ၂ ခု+ ရှိရင် method တစ်ခုချင်းစီ SwingWorker ခွဲ|Network call **အားလုံး** ကို `doInBackground()` **တစ်ခုထဲ** ပေါင်းစည်း|
|Local logic (QR image generate, validation)|`done()` ထဲမှာပဲ ဆက်ထား — SwingWorker မလိုပါ|
|Exception (SQLException)|`ex.getCause()` ကနေ actual error message ထုတ်ကြည့်ရမယ်|

---

`RmiConnectionManager` ကို app စတင်တဲ့အခါ (SignIn form) connect လုပ်ပြီးသားလို့ ယူဆထားပါတယ် — `QRGenerator` form က `SignIn` ကို bypass လုပ်ပြီး တိုက်ရိုက်ဖွင့်နိုင်တဲ့ scenario ရှိရင် (testing အတွက်) connect check logic ထပ်ထည့်ပေးရမလား၊ ဒါမှမဟုတ် Delete/Update button တွေမှာ "Server connection lost" ဖြစ်ရင် login screen ပြန်ပို့ဖို့ (Session-aware error handling) ကူညီရမလား?