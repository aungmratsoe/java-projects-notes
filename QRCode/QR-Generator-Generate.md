# Chapter 18 — QRGenerator `btnGenerateActionPerformed()`

## Secure QR Code Generation Flow (UUID + AES Encryption + ZXing + Database Update)

Great **aung mrat**.

ဒီ method က `QRGenerator` Form ရဲ့ **Generate QR Code Button** ဖြစ်ပါတယ်။

ဒီ method က Register method ထက် ပို advanced ဖြစ်ပါတယ်။

ဘာလုပ်သလဲဆိုရင်:

> JTable မှာရွေးထားတဲ့ Student ကိုယူ → Token စစ် → Token အသစ်ဖန်တီး → Student Data ကို Encrypt → QR Code Generate → PNG Save → UI Preview ပြ

ဖြစ်ပါတယ်။

---

# 18.1 Complete Flow Overview

ဒီ method တစ်ခုလုံးရဲ့ architecture:

```
User Click Generate Button

          |
          v

Get Selected JTable Row

          |
          v

Extract Student Data

          |
          v

Check Existing QR File

          |
          v

Create / Regenerate UUID Token

          |
          v

Update Token Database

          |
          v

Create QR Payload

          |
          v

AES Encrypt Data

          |
          v

Generate QR Image

          |
          v

Save PNG File

          |
          v

Show QR Preview

          |
          v

Refresh JTable

```

---

# Chapter 18.2 — Get Selected Student Row

Code:

```java
int selectedRow = tblStudents.getSelectedRow();
```

---

`tblStudents` က JTable ဖြစ်ပါတယ်။

Example:

```
------------------------------------------------
No | ID      | Name        | Department
------------------------------------------------
1  | ST001   | Aung        | IT
2  | ST002   | Mg Mg       | CS
3  | ST003   | Su Su       | SE
------------------------------------------------

```

User က row 2 ကို click လုပ်ရင်:

```java
selectedRow = 1;
```

ဖြစ်မယ်။

---

Java JTable index က:

```
First row  = 0
Second row = 1
Third row  = 2

```

ဖြစ်ပါတယ်။

---

# Chapter 18.3 — Check Student Selected?

Code:

```java
if(selectedRow == -1)
```

---

JTable မှာ ဘာမှမရွေးရင်:

```java
getSelectedRow()
```

က:

```
-1

```

return လုပ်တယ်။

---

Example:

User:

```
Click Generate
```

မရွေးထားဘူး။

Result:

```
selectedRow = -1

```

---

Then:

```java
JOptionPane.showMessageDialog(
...
"Please select a student"
);

return;
```

---

Flow:

```
No Student Selected

        |
        v

Show Warning

        |
        v

Stop Method

```

---

# Chapter 18.4 — Extract Data From JTable

Code:

```java
String studentId = getCellValue(selectedRow, 1);
String name = getCellValue(selectedRow, 2);
String age = getCellValue(selectedRow, 3);
String sex = getCellValue(selectedRow, 4);
String dept = getCellValue(selectedRow, 5);
String email = getCellValue(selectedRow, 6);
String existingToken = getCellValue(selectedRow, 7);

```

---

JTable data:

```
Column Index

0 = No
1 = Student ID
2 = Name
3 = Age
4 = Sex
5 = Department
6 = Email
7 = QR Token

```

---

Example row:

```
0    1       2        3
--------------------------------
1 | ST001 | Aung | 32 |

4       5        6
-------------------------
Male | IT | email@gmail.com


7
------------
abc-123-token

```

---

Result:

```java
studentId = "ST001";

name = "Aung";

existingToken = "abc-123-token";

```

---

# Chapter 18.5 — Check QR File Exists

Code:

```java
File qrFile =
new File("qrcodes/" + studentId + ".png");

boolean qrFileExists =
qrFile.exists();

```

---

Example:

Student:

```
ST001

```

File:

```
project

 |
 +-- qrcodes

        |
        +-- ST001.png

```

---

ရှိရင်:

```java
qrFileExists = true;

```

မရှိရင်:

```java
qrFileExists = false;

```

---

Important Design:

ဒီ comment:

```java
// A QR code exists ONLY if the file actually exists on disk.

```

Meaning:

Database token ရှိတာနဲ့ QR ရှိတယ်လို့ မယူဆဘူး။

---

Why?

Possible situation:

Database:

```
qrToken = abc-123

```

But:

```
qrcodes/ST001.png

(delete)

```

ဖြစ်နိုင်တယ်။

---

ဒါကြောင့် File ကို check လုပ်တယ်။

---

# Chapter 18.6 — Existing QR Regeneration

Code:

```java
if(qrFileExists)
```

---

Meaning:

"ဒီ Student အတွက် QR PNG ရှိပြီးသားလား?"

---

ရှိရင်:

Dialog ပြမယ်။

```java
JOptionPane.showConfirmDialog()

```

---

Message:

```
A QR code already exists.

Regenerating will invalidate
previous QR code.

Continue?

```

---

ဘာကြောင့်လဲ?

Token ပြောင်းသွားရင်:

Old QR:

```
Token = AAA

```

New QR:

```
Token = BBB

```

ဖြစ်သွားမယ်။

---

Scanner မှာ:

Old QR:

```
AAA != Database BBB

```

ဆိုတော့:

```
Access Denied

```

ဖြစ်မယ်။

---

ဒါက Security Feature ဖြစ်တယ်။

---

# Chapter 18.7 — Generate New Token

Code:

```java
currentToken =
UUID.randomUUID().toString();

```

---

UUID ဆိုတာ:

Universally Unique Identifier

---

Example:

```
550e8400-e29b-41d4-a716-446655440000

```

---

အသုံး:

- Unique ID
    
- Security Token
    
- Session ID
    

---

ဒီ project မှာ:

```
Student
    |
    |
    QR Token

```

အဖြစ်သုံးတယ်။

---

# Chapter 18.8 — Update Database Token

Code:

```java
studentDAO.updateQrToken(
studentId,
currentToken
);

```

---

Database:

Before:

|StudentID|qrToken|
|---|---|
|ST001|OLD123|

---

After:

|StudentID|qrToken|
|---|---|
|ST001|NEW456|

---

DAO Flow:

```
QRGenerator

     |
     v

StudentDAO

     |
     v

UPDATE students
SET qr_token=?

```

---

# Chapter 18.9 — No Existing Token

Code:

```java
else if(
currentToken == null ||
currentToken.trim().isEmpty()
)

```

---

Meaning:

Database မှာ token မရှိဘူး။

Example:

```
qrToken = null

```

---

ဒါဆို:

```java
UUID.randomUUID()

```

နဲ့ token အသစ်ဖန်တီးတယ်။

---

# Chapter 18.10 — Create QR Payload

Code:

```java
String rawQrData = String.format(
"StudentID: %s\nName: %s\nAge: %s\nSex: %s\nDept: %s\nEmail: %s\nToken: %s",
...
);

```

---

ဒါက QR ထဲထည့်မယ့် original data ဖြစ်တယ်။

Example:

```
StudentID: ST001
Name: Aung Mrat
Age: 32
Sex: Male
Dept: Computer
Email: aung@gmail.com
Token: abc-123

```

---

`\n`

ဆိုတာ:

New line

---

QR data structure:

```
Line 1
Line 2
Line 3

```

ဖြစ်အောင်လုပ်တာ။

---

# Chapter 18.11 — AES Encryption

Code:

```java
String qrData =
CryptoUtils.encrypt(rawQrData);

```

---

အရေးကြီးဆုံး Security Part ပါ။

---

Without encryption:

QR Scanner နဲ့ scan လုပ်ရင်:

```
StudentID: ST001
Name: Aung
Email: xxx@gmail.com

```

မြင်ရမယ်။

---

With AES:

```
8Fj39dkd82jdks92kd92jd...

```

ပဲမြင်ရမယ်။

---

Flow:

```
Student Data

      |
      v

AES Encryption

      |
      v

Cipher Text

      |
      v

QR Code

```

---

# Chapter 18.12 — Encryption Failed Check

Code:

```java
if(qrData == null)

```

---

Encryption error ဖြစ်ရင်:

CryptoUtils:

```java
return null;

```

လုပ်တယ်။

---

ဒါဆို:

```
Failed to encrypt QR code data

```

ပြမယ်။

---

# Chapter 18.13 — Create QR Directory

Code:

```java
File dir = new File("qrcodes");

if(!dir.exists()){
    dir.mkdirs();
}

```

---

Folder မရှိရင် create လုပ်တယ်။

Before:

```
Project

(no qrcodes)

```

After:

```
Project

 |
 +--qrcodes

```

---

# Chapter 18.14 — Save QR PNG

Code:

```java
qrUtil.saveQRCodeToFile(
qrData,
"qrcodes/"+studentId+".png",
PRINT_QR_SIZE,
PRINT_QR_SIZE
);

```

---

Example:

```
qrcodes/ST001.png

```

ဖြစ်သွားမယ်။

---

ဒီ function က:

```
String Data

      |
      v

ZXing

      |
      v

BitMatrix

      |
      v

PNG File

```

လုပ်ပေးတယ်။

---

# Chapter 18.15 — Generate Preview Image

Code:

```java
BufferedImage qrImage =
qrUtil.generateQRCode(
qrData,
DISPLAY_QR_SIZE,
DISPLAY_QR_SIZE
);

```

---

Difference:

## Save File

```
High Resolution

For Printing

```

---

## Preview

```
Small Size

For JLabel Display

```

---

Architecture:

```
QR Data

   |
   +------------+

   |            |

PNG File     JLabel Preview

```

---

# Chapter 18.16 — Display QR on JLabel

Code:

```java
lblQRcode.setText("");

lblQRcode.setIcon(
new ImageIcon(qrImage)
);

```

---

Before:

```
+-------------+
| Generate QR |
+-------------+

```

---

After:

```
+-------------+
|             |
|  █▀█ █▀█    |
|  █ █ █ █    |
|  QR IMAGE   |
|             |
+-------------+

```

---

# Chapter 18.17 — Refresh JTable

Code:

```java
loadStudentData(
txtSearch.getText().trim()
);

```

---

Why?

Because:

Database token changed.

Old table:

```
qrToken OLD

```

New:

```
qrToken NEW

```

---

Refresh:

```
Database
   |
   v
 JTable

```

---

# Chapter 18.18 — Restore Selection

Code:

```java
selectStudentRowById(studentId);

```

---

Problem:

Refresh လုပ်ရင် JTable selection ပျောက်သွားမယ်။

Before:

```
[ST001] selected

```

After reload:

```
nothing selected

```

---

ဒီ method က:

```
Find ST001

Select again

```

လုပ်ပေးတယ်။

---

# Chapter 18.19 — Success Message

Code:

```java
JOptionPane.showMessageDialog(
"QR Code generated and saved successfully!"
);

```

---

Result:

```
Success

QR Code generated and saved successfully!

```

---

# Chapter 18.20 — Exception Handling

Code:

```java
catch(Exception e)

```

---

Possible errors:

- Encryption error
    
- File permission
    
- Database error
    
- ZXing error
    

---

Log:

```java
logger.log(
Level.SEVERE,
"QR Generation error",
e
);

```

---

User:

```
Failed to generate QR Code

```

---

# Complete Security Architecture

ဒီ button က:

```
Student Information

        |
        v

UUID Token Creation

        |
        v

AES Encryption

        |
        v

ZXing QR Generation

        |
        +----------------+

        |                |

    Save PNG        Display Preview


        |

        v

Update Database Token


```

---

# Senior Developer Analysis

ဒီ method မှာ Professional Concepts:

|Concept|Purpose|
|---|---|
|JTable Selection|User choice|
|UUID Token|Security identity|
|AES Encryption|Data protection|
|DAO Pattern|Database separation|
|File System Check|Prevent overwrite issue|
|QR Versioning|Token invalidation|
|Exception Handling|Stability|
|UI Synchronization|Better UX|

---

အရေးကြီးဆုံး Security Idea က:

```
QR Code = Encrypted Student Data + Unique Token

Scanner
   |
Decrypt
   |
Check Database Token
   |
Allow / Reject

```

ဖြစ်ပါတယ်။

ဒါကြောင့် ဒီ QR System က Simple QR Generator မဟုတ်ဘဲ **Secure Student Identity Verification System Architecture** ဖြစ်နေပြီ။