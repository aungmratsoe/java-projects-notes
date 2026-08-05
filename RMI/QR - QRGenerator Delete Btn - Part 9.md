# `deleteStudentData()` — Server-side Convention Naming ဖြင့် ပြင်ဆင်ထားသော Version

QR image ကို **Server disk ပေါ်မှာ `studentId` convention** နဲ့ သိမ်းထားပြီးမို့ (Path field DB ထဲ မမှတ်တော့ဘူးလို့ ဆုံးဖြတ်ထားတဲ့အတိုင်း) — Local file delete logic ကို **Server ဆီ RMI call** အဖြစ် လုံးဝ ပြောင်းရပါမယ်:

```java
private void deleteStudentData(String studentId) {
    btnDelete.setEnabled(false);

    SwingWorker<Boolean, Void> worker = new SwingWorker<>() {
        @Override
        protected Boolean doInBackground() throws Exception {
            // ၁. Server ရဲ့ disk ပေါ်က QR Code image ဖိုင်ကို ဖျက်ခြင်း
            // (studentId convention နဲ့ Server ဘက်က file path ကို ကိုယ်တိုင် တွက်ချက်ဖျက်မယ်)
            studentService.deleteQrImage(studentId);

            // ၂. Database မှ ကျောင်းသား အချက်အလက်ကို ဖျက်ခြင်း
            return studentService.deleteStudent(studentId);
        }

        @Override
        protected void done() {
            btnDelete.setEnabled(true);
            try {
                boolean isDeletedFromDb = get();

                if (isDeletedFromDb) {
                    JOptionPane.showMessageDialog(QRGenerator.this,
                        "The student has been successfully deleted.",
                        "Success", JOptionPane.INFORMATION_MESSAGE);

                    // ၃. Table Data များကို Refresh လုပ်ပေးခြင်း
                    loadStudentData("");
                } else {
                    JOptionPane.showMessageDialog(QRGenerator.this,
                        "There was an error while deleting from the database.",
                        "Error", JOptionPane.ERROR_MESSAGE);
                }

            } catch (Exception e) {
                logger.log(Level.SEVERE, "Error deleting student", e);
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "An error occurred while deleting: " + e.getCause(),
                    "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

## Server Implementation — `deleteQrImage()` (Convention-based)

```java
// com.ams.qrcode.service.impl.StudentServiceImpl.java
@Override
public void deleteQrImage(String studentId) throws RemoteException, DataAccessException {
    File qrFile = getQrFile(studentId); // studentId + ".png" convention — path field မလိုပါ

    if (qrFile.exists()) {
        boolean deleted = qrFile.delete();
        if (deleted) {
            logger.info("QR code file deleted successfully: " + qrFile.getPath());
        } else {
            logger.warning("Failed to delete QR code file: " + qrFile.getPath());
        }
    }
    // File မရှိရင်တောင် error မဟုတ်ပါ — "delete" ရဲ့ ရလဒ်က "မရှိတော့ဘူး" ဖြစ်လိုက်ရုံပါ
}
```

## ပြောင်းလဲထားသော အချက်များ — Summary

|မူလ Code (Local file)|ပြောင်းထားသော Code (RMI + Server-side)|
|---|---|
|`new File("qrcodes/" + studentId + ".png")` — Client disk ကို တိုက်ရိုက်ကိုင်|`studentService.deleteQrImage(studentId)` — Server ဆီ delete instruction ပို့|
|`qrFile.exists()`, `qrFile.delete()` — Client ဘက်က local file operation|Server ဘက် `StudentServiceImpl` ထဲမှာသာ run (client ဘက် File I/O လုံးဝ မလိုတော့)|
|Local delete + DB delete — Sequential, ၂ ခု သီးခြား try-catch block|`doInBackground()` **တစ်ခုတည်း** ထဲမှာ RMI call ၂ ခု အစဉ်လိုက် ခေါ် (Round-trip အနည်းဆုံး)|
|`logger.info(...)` — Client ဘက်က log|Server ဘက် logger ကနေ log (Server console/log file ထဲ ပေါ်မယ်)|

## Client ဘက်က `File`/`import java.io.File` Import ဖယ်ရှားနိုင်ပါပြီ

QR file operation အားလုံးက Server ဘက်ကိုပဲ ရွှေ့သွားပြီမို့ `QRGenerator.java` (Client-side class) ထဲက `import java.io.File;` ကို **လုံးဝ ဖယ်ရှားလို့ရ**ပါပြီ — Client code ဟာ Business/file logic လုံးဝ မပါဘဲ **UI + Service call** ချည်းသာ ကျန်တော့မှာပါ (Clean separation).

---

`btnGenerateActionPerformed()` (Previous turn က ပြောင်းထားတဲ့ version) နဲ့ ဒီ delete logic တွေ ပေါင်းစပ်ကြည့်ရင် project ရဲ့ RMI conversion **အတော်များများ ပြီးသွားပါပြီ** — Server-side folder permission (Server run ထားတဲ့ user account မှာ `qrcodes/` folder ကို write ခွင့်ရှိရမယ်) ကိုပါ သတိပေးလိုပါတယ်။ Attendance/Reports screen ရှိသေးရင် ဆက်ကူညီရမလား၊ ဒါမှမဟုတ် Full project ကို **Testing checklist** တစ်ခု ချုပ်ပေးရမလား?