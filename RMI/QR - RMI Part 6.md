# QRScanner UI ကို RMI Client ပြောင်းခြင်း

ဒီ code မှာ `studentDAO.getStudentByStudentId(studentId)` **တစ်ခုတည်း** ပဲ DB ကို ဆက်နေတာပါ — ကျန်တဲ့ decrypt/extractValue/calculateAge logic တွေက **local operation** ဖြစ်လို့ ပြောင်းစရာမလိုပါ။ QR scan ဖြစ်တဲ့အခါ method ခေါ်တာဖြစ်လို့ (event-driven, button click မဟုတ်) SwingWorker ချိတ်ပုံ အနည်းငယ် ကွဲပါတယ်.

## ✅ Before → After

```java
private void onQrCodeScanned(String qrData) {
    // === Client-side logic (ပြောင်းစရာမလို) ===
    String decryptedData;
    try {
        decryptedData = CryptoUtils.decrypt(qrData);
    } catch (Exception e) {
        logger.log(Level.SEVERE, "Decrypt error", e);
        decryptedData = null;
    }

    if (decryptedData == null) {
        JOptionPane.showMessageDialog(this,
            "Invalid Code: This is not a student ID QR code.",
            "Invalid QR Code", JOptionPane.WARNING_MESSAGE);
        return;
    }

    String studentId = extractValue(decryptedData, "StudentID:");
    String scannedToken = extractValue(decryptedData, "Token:");

    if (studentId.isEmpty() || scannedToken.isEmpty()) {
        JOptionPane.showMessageDialog(this,
            "Invalid Code: This is not a student ID QR code.",
            "Invalid QR Code", JOptionPane.WARNING_MESSAGE);
        return;
    }

    // === Server call — ဒီနေရာကနေစပြီး SwingWorker ထဲ ပြောင်း ===
    final String finalStudentId = studentId;   // lambda ထဲသုံးရမှာမို့ effectively-final လိုအပ်
    final String finalScannedToken = scannedToken;

    // Scanner UI ကို ထပ်တလဲလဲ scan မထပ်ဖြစ်အောင် ခဏတာ disable
    scannerPanel.setEnabled(false);

    SwingWorker<Student, Void> worker = new SwingWorker<>() {
        @Override
        protected Student doInBackground() throws Exception {
            return studentService.getStudentByStudentId(finalStudentId);
        }

        @Override
        protected void done() {
            scannerPanel.setEnabled(true);

            Student student;
            try {
                student = get();
            } catch (Exception ex) {
                // Server disconnect / DataAccessException — 
                // "Invalid QR" message နဲ့ မရောနှောဘဲ Connection error အနေနဲ့ သီးခြားပြ
                logger.log(Level.SEVERE, "Error during verification", ex);
                JOptionPane.showMessageDialog(QRScanner.this,
                    "Server ဆက်သွယ်မှု မအောင်မြင်ပါ: " + ex.getCause(),
                    "Connection Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            if (student == null) {
                JOptionPane.showMessageDialog(QRScanner.this,
                    "STUDENT NOT FOUND IN DATABASE!\nID: " + finalStudentId,
                    "Access Denied", JOptionPane.ERROR_MESSAGE);
                return;
            }

            if (finalScannedToken.equals(student.getQrToken())) {
                JOptionPane.showMessageDialog(QRScanner.this,
                    "VALID QR CODE!\n\n"
                    + "Student ID: " + student.getStudentId() + "\n"
                    + "Name: " + student.getName() + "\n"
                    + "Age: " + calculateAge(student.getDob()) + "\n"
                    + "Sex: " + student.getSex() + "\n"
                    + "Department: " + student.getDepartment() + "\n"
                    + "Email: " + student.getEmail(),
                    "Access Granted", JOptionPane.INFORMATION_MESSAGE);
            } else {
                JOptionPane.showMessageDialog(QRScanner.this,
                    "EXPIRED OR REGENERATED QR CODE!\n\nPlease use the latest generated QR code.",
                    "Access Denied", JOptionPane.WARNING_MESSAGE);
            }
        }
    };
    worker.execute();
}
```

## ⚠️ အရေးကြီးတဲ့ ပြောင်းလဲချက် — Error Handling ခွဲထားခြင်း

မူလ code မှာ **decrypt error** နဲ့ **DB error** နှစ်ခုစလုံးကို `catch (Exception e)` တစ်ခုတည်းနဲ့ "Invalid QR Code" message တစ်ခုတည်း ပြခဲ့ပါတယ်။ RMI ပြောင်းလိုက်ရင် **Server disconnect ဖြစ်တာ** ကိုလည်း "Invalid QR Code" လို့ ပြလိုက်ရင် user က "QR code ပျက်နေတယ်" လို့ အထင်မှားနိုင်ပါတယ် — တကယ်က server ချို့ယွင်းနေတာဖြစ်လို့။ ဒါကြောင့် အထက်က code မှာ:

- **Decrypt fail** → "Invalid QR Code" (client-side error, ပြောင်းစရာမလို)
- **Server call fail** → "Connection Error" (ခွဲထားတယ်, user က server အသစ်ချိတ်ရမှန်း သိအောင်)
- **Student not found** → "Access Denied" (server ကနေ `null` ပြန်လာတာ, valid response)

## Continuous Scanning ရှိရင် သတိထားရမယ့်အချက်

QR Scanner ဟာ **camera feed ကနေ frame တိုင်း scan** နေတတ်ပါတယ် (real-time scanning library သုံးထားရင်) — ဒီလိုဆိုရင် `onQrCodeScanned()` ကို **second တစ်ခုအတွင်း ထပ်ခါထပ်ခါ** ခေါ်နိုင်ပါတယ်။ RMI call တစ်ခုစီက network round-trip ဖြစ်လို့:

```java
// Class field အနေနဲ့ flag တစ်ခု ထား
private volatile boolean isVerifying = false;

private void onQrCodeScanned(String qrData) {
    if (isVerifying) return; // Verify လုပ်နေဆဲဆို scan အသစ်ကို ignore

    // ... decrypt/extract logic ...

    isVerifying = true;
    SwingWorker<Student, Void> worker = new SwingWorker<>() {
        @Override
        protected Student doInBackground() throws Exception {
            return studentService.getStudentByStudentId(finalStudentId);
        }

        @Override
        protected void done() {
            isVerifying = false;
            // ... dialog logic ...
        }
    };
    worker.execute();
}
```

ဒါမှမဟုတ်ရင် Student ID တူတူကို RMI request **ငါးခြောက်ခု တစ်ပြိုင်နက် server ဆီ ပို့နေမယ်** — Server ဘက် load မလိုအပ်ဘဲ တက်စေနိုင်ပါတယ်။

## `CryptoUtils.decrypt()` — Client ဘက်မှာပဲ ဆက်ထားရမယ်

QR code ရဲ့ AES decrypt လုပ်တာက **local operation** ဖြစ်ပြီး DB access မလိုအပ်ပါ — ဒီ class ကို Server ဘက် ပို့စရာမလိုပါ၊ Client ဘက်မှာပဲ ဆက်ထားလို့ရပါတယ်။ ဒါပေမယ့် **AES key** ကို client-side code ထဲမှာ hardcode ထားထားရင် (`CryptoUtils` ထဲက key), reverse-engineer ဖြစ်လွယ်ပါတယ် — QR token verify ကို server ဘက်ကနေ ပြန်စစ်နေတာမို့ security risk အနည်းငယ်ပဲ ရှိပါတယ်၊ ဒါပေမယ့် **production အတွက်ဆိုရင် key ကို server ဘက်ကနေပဲ decrypt လုပ်ပေးဖို့** (client က encrypted string ကိုပဲ server ဆီ ပို့ပြီး server က decrypt+verify+response ပြန်ပေးတဲ့ pattern) စဉ်းစားသင့်ပါတယ်။

---

Decrypt logic ကို server-side ပြောင်းချင်ရင် (`verifyQrToken(encryptedData)` ဆိုတဲ့ single service method) ကူညီရမလား၊ ဒါမှမဟုတ် project ရဲ့ ကျန်တဲ့ screen (Attendance history, Reports) တွေ ရှိသေးရင် ဆက်ကူညီရမလား?