# Update Button — RMI ပြောင်းထားသော Version

သင့် code ရဲ့ structure (`extractAndValidateStudentData()` validation, `logger` error logging) ကို အတိအကျ ထိန်းထားပြီး `studentDAO` ကို `studentService` (RMI) အဖြစ်၊ network call တွေကို SwingWorker ထဲ ပြောင်းပေးထားပါတယ်:

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

## ပြောင်းလဲထားသော အချက်များ

|မူလ Code|ပြောင်းထားသော Code|အကြောင်းရင်း|
|---|---|---|
|`studentDAO.getStudentByStudentId()` / `updateStudent()` / `saveStudent()` — EDT ပေါ်တိုက်ရိုက်ခေါ်|`doInBackground()` ထဲသို့ ရွှေ့|Network call ဖြစ်လို့ EDT မှာ တိုက်ရိုက်ခေါ်ရင် UI freeze ဖြစ်နိုင်|
|`try { ... } catch (Exception e)` block တစ်ခုတည်း|`doInBackground()` (data logic) + `done()` ရဲ့ `catch` (result/error handling) ၂ ပိုင်း ခွဲ|SwingWorker ရဲ့ structure အလိုက် `get()` ကနေ Exception ကို ပြန်ဖမ်းရမယ်|
|`e.getMessage()`|`e.getCause()`|RMI exception ဟာ `ExecutionException` ထဲ wrap ဖြစ်နေလို့ **actual error** ကို `getCause()` ကနေ ယူရမယ်|
|Logic order (update/insert/loadStudentData/handleClear/dialog)|**အတူတူပဲ** ထားထားတယ်|Business logic ကို ဘာမှ ပြောင်းစရာမလို — Data access layer ကိုပဲ ပြောင်းတာပါ|

## သတိထားရမယ့် Edge Case — `logger.log()` ကို `done()` ထဲမှာ ခေါ်ခြင်း

`logger.log(Level.SEVERE, ...)` ကို `done()` ထဲမှာ ခေါ်ထားတာက **EDT ပေါ်** မှာ run နေတာမို့ safe ပါတယ် (Loggers က thread-safe ဖြစ်ပြီးသားလို့ EDT ပေါ်ကနေ ခေါ်လို့ လုံးဝ ပြဿနာမရှိပါ)။

---

`extractAndValidateStudentData()` method ကိုလည်း ပြပေးနိုင်ရင် (Date/DOB conversion ပါလာနိုင်ခြေရှိလို့) စစ်ပေးနိုင်ပါတယ် — ဒါမှမဟုတ် ကျန်တဲ့ screen (Attendance/Reports) code ကို ဆက်ပြောင်းရမလား?