# `getQrPath()`/`setQrPath()` မလိုတော့ — Convention-based Naming အတွက် ကောင်းသော ဆုံးဖြတ်ချက်ပါ

Database schema ထဲမှာလည်း `qr_path` column **မရှိပါ** — `qr_token` ပဲ ရှိပါတယ်။ ဒါကြောင့် QR image ကို **`studentId` နဲ့ file name convention** (`qrcodes/{studentId}.png`) နဲ့ Server disk ပေါ်မှာသိမ်းရင် Path ကို DB မှာ မှတ်စရာ **လုံးဝ မလိုပါ** — Server ဘက်ကနေ `studentId` ကနေ path ကို **တွက်ချက်ရလို့ရ** (deterministic) ဖြစ်လို့ပါ။

## Service Interface — File Path Field မလိုအောင် ပြင်

```java
// com.ams.qrcode.service.StudentService.java (ထပ်ထည့်ရမယ့် method)
public interface StudentService extends Remote {
    // ... ရှိပြီးသား method များ ...

    // Local File.exists() check ကို ဒီ method က အစားထိုးမယ် (Server disk ကို စစ်ရမှာမို့)
    boolean qrCodeExists(String studentId) throws RemoteException;

    void saveQrImage(String studentId, byte[] imageData) throws RemoteException, DataAccessException;

    byte[] getQrImage(String studentId) throws RemoteException, DataAccessException;
}
```

## Server Implementation

```java
// com.ams.qrcode.service.impl.StudentServiceImpl.java
private static final String QR_FOLDER = "qrcodes/"; // Server ရဲ့ folder

private File getQrFile(String studentId) {
    return new File(QR_FOLDER + studentId + ".png"); // studentId convention — path DB မှာ မမှတ်ရ
}

@Override
public boolean qrCodeExists(String studentId) throws RemoteException {
    return getQrFile(studentId).exists();
}

@Override
public void saveQrImage(String studentId, byte[] imageData) throws RemoteException, DataAccessException {
    try {
        File dir = new File(QR_FOLDER);
        if (!dir.exists()) dir.mkdirs();

        try (FileOutputStream fos = new FileOutputStream(getQrFile(studentId))) {
            fos.write(imageData);
        }
    } catch (IOException e) {
        throw new RemoteException("QR image save failed: " + e.getMessage(), e);
    }
}

@Override
public byte[] getQrImage(String studentId) throws RemoteException, DataAccessException {
    try {
        File file = getQrFile(studentId);
        if (!file.exists()) return null;
        return java.nio.file.Files.readAllBytes(file.toPath());
    } catch (IOException e) {
        throw new RemoteException("QR image read failed: " + e.getMessage(), e);
    }
}
```

## Client — `btnGenerateActionPerformed` ကို RMI ပြောင်း

**Design challenge**: မူလ code မှာ `qrFile.exists()` ကို local disk ကနေ **synchronous** စစ်ပြီးမှ confirm dialog ပြပါတယ်။ RMI ပြောင်းလိုက်ရင် `exists()` check ကလည်း **network call** ဖြစ်သွားလို့ SwingWorker ၂ ဆင့် ခွဲရပါတယ် — ပထမ ဆင့်က "exists check", ဒုတိယ ဆင့်က "generate + upload".

```java
private void btnGenerateActionPerformed(java.awt.event.ActionEvent evt) {
    int selectedRow = tblStudents.getSelectedRow();
    if (selectedRow == -1) {
        JOptionPane.showMessageDialog(this, "Please select a student from the table first!", 
            "Select Student", JOptionPane.WARNING_MESSAGE);
        return;
    }

    // Table ကနေ data ယူတာ — client-side, ပြောင်းစရာမလို
    final String studentId = getCellValue(selectedRow, 1);
    final String name = getCellValue(selectedRow, 2);
    final String age = getCellValue(selectedRow, 3);
    final String sex = getCellValue(selectedRow, 4);
    final String dept = getCellValue(selectedRow, 5);
    final String email = getCellValue(selectedRow, 6);
    final String existingToken = getCellValue(selectedRow, 8);

    btnGenerate.setEnabled(false);

    // ===== Stage 1 — QR file Server ပေါ်မှာ ရှိမရှိ စစ် =====
    SwingWorker<Boolean, Void> checkWorker = new SwingWorker<>() {
        @Override
        protected Boolean doInBackground() throws Exception {
            return studentService.qrCodeExists(studentId); // Local File.exists() အစား RMI call
        }

        @Override
        protected void done() {
            try {
                boolean qrFileExists = get();

                if (qrFileExists) {
                    // File ရှိရင် warning ပြ (client-side dialog, ပြောင်းစရာမလို)
                    int option = JOptionPane.showConfirmDialog(
                            QRGenerator.this,
                            "A QR code already exists for " + name + " - StdID (" + studentId + ").\n"
                            + "Regenerating it will invalidate any previously printed/saved QR code.\n"
                            + "Do you want to continue?",
                            "Confirm Regeneration",
                            JOptionPane.OK_CANCEL_OPTION,
                            JOptionPane.WARNING_MESSAGE
                    );
                    if (option != JOptionPane.OK_OPTION) {
                        btnGenerate.setEnabled(true);
                        return;
                    }
                }

                // Stage 2 ကို စ (either regenerate confirmed, or file didn't exist yet)
                generateAndUploadQr(studentId, name, age, sex, dept, email, existingToken, qrFileExists);

            } catch (Exception ex) {
                btnGenerate.setEnabled(true);
                logger.log(Level.SEVERE, "QR existence check error", ex);
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Failed to check QR status: " + ex.getCause(),
                    "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    checkWorker.execute();
}

// ===== Stage 2 — Token update (လိုအပ်ရင်) + QR generate + Server ဆီ upload =====
private void generateAndUploadQr(String studentId, String name, String age, String sex,
                                   String dept, String email, String existingToken, 
                                   boolean isRegenerate) {

    SwingWorker<BufferedImage, Void> genWorker = new SwingWorker<>() {
        @Override
        protected BufferedImage doInBackground() throws Exception {
            String currentToken = existingToken;

            // Token အသစ် လိုအပ်ရင် (regenerate, or empty token) — server ကို update
            if (isRegenerate || currentToken == null || currentToken.trim().isEmpty()) {
                currentToken = java.util.UUID.randomUUID().toString();
                studentService.updateQrToken(studentId, currentToken);
            }

            // Payload ဖန်တီးခြင်း — local logic, ပြောင်းစရာမလို
            String rawQrData = String.format(
                    "StudentID: %s\nName: %s\nAge: %s\nSex: %s\nDept: %s\nEmail: %s\nToken: %s",
                    studentId, name, age, sex, dept, email, currentToken
            );

            // Encrypt — local logic
            String qrData = CryptoUtils.encrypt(rawQrData);
            if (qrData == null) {
                throw new IllegalStateException("Failed to encrypt QR code data!");
            }

            // QR image ကို local library (ZXing) နဲ့ ဆွဲ — high-res (print quality)
            BufferedImage printImage = qrUtil.generateQRCode(qrData, PRINT_QR_SIZE, PRINT_QR_SIZE);

            // byte[] ပြောင်းပြီး Server ဆီ upload
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ImageIO.write(printImage, "png", baos);
            studentService.saveQrImage(studentId, baos.toByteArray());

            // Preview အတွက် display-size image ကို ပြန်ပေး (done() ထဲမှာ JLabel ထဲထည့်ဖို့)
            return qrUtil.generateQRCode(qrData, DISPLAY_QR_SIZE, DISPLAY_QR_SIZE);
        }

        @Override
        protected void done() {
            btnGenerate.setEnabled(true);
            try {
                BufferedImage previewImage = get();

                lblQRcode.setText("");
                lblQRcode.setIcon(new ImageIcon(previewImage));

                loadStudentData(txtSearch.getText().trim());
                selectStudentRowById(studentId);

                JOptionPane.showMessageDialog(QRGenerator.this,
                    "QR Code generated and saved successfully!", 
                    "Success", JOptionPane.INFORMATION_MESSAGE);

            } catch (Exception ex) {
                logger.log(Level.SEVERE, "QR Generation error", ex);
                JOptionPane.showMessageDialog(QRGenerator.this,
                    "Failed to generate QR Code: " + ex.getCause(),
                    "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    };
    genWorker.execute();
}
```

## ဒီ Design ရဲ့ အဓိက အချက်များ

|အချက်|ရှင်းလင်းချက်|
|---|---|
|**Print-quality file ကို local disk မှာ မသိမ်းတော့ဘူး**|`qrUtil.saveQRCodeToFile(...)` ကို ဖျက်လိုက်ပြီး, `printImage` ကို byte[] ပြောင်းပြီး Server ဆီ `saveQrImage()` ကနေ တိုက်ရိုက် upload|
|**Preview image ကို Server ကနေ ပြန်မဒေါင်းလုတ်ဘူး**|`doInBackground()` ထဲမှာ `DISPLAY_QR_SIZE` version ကို local generate ပြီး တန်းပြန်ပေး (Server round-trip တစ်ခါထပ်မလို — performance ပိုကောင်း)|
|**`qrCodeExists()` service method အသစ်**|Local `File.exists()` check ကို RMI call တစ်ခုနဲ့ အစားထိုး|
|**Two-stage SwingWorker**|Confirm dialog ကို EDT ပေါ်မှာ ပြရမှာမို့ Stage 1 (check) ပြီးမှ Stage 2 (generate+upload) စ|
|**`qr_path` DB column မလို**|`studentId` ကနေ file name (`studentId + ".png"`) ကို Server ဘက်က တွက်ချက်ရလို့ Path ကို သီးခြား မှတ်စရာမလို|

## Model ထဲမှာ ဘာမှ ထပ်ထည့်စရာ မလိုတော့ပါ

```java
// Student.java ကို ဒီအတိုင်း ချန်ထားလို့ရပါတယ် (qrPath field မထည့်ရ)
public class Student implements Serializable {
    private Integer id;
    private String studentId;
    private String name;
    private Integer age;
    private String sex;
    private Date dob;
    private String department;
    private String email;
    private String qrToken;  // ← ဒါပဲ လိုအပ်ပါတယ်
    private Timestamp createdAt;
    private Timestamp updatedAt;
    // ...
}
```

---

`qrUtil.generateQRCode()` / `saveQRCodeToFile()` method (ZXing wrapper) ကို ပြပေးရင် `saveQRCodeToFile()` ကို ဖျက်ပြီး **local file save လုံးဝ မလိုအောင်** clean up လုပ်ပေးနိုင်ပါတယ် — ဒါမှမဟုတ် QR Scanner UI (Access verification screen) ကို ဒီ pattern အတိုင်း connect ပြန်ပြီး review လုပ်ချင်ပါသလား?