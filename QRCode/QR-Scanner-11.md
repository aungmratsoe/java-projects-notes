# Chapter 11 — QR Generator Flow

## Encrypt → Generate QR → Save PNG → Database Token


အခု Chapter 11 က Scanner ရဲ့ opposite side ဖြစ်ပါတယ်။

Chapter 9 မှာ ကျွန်တော်တို့က

```text
QR Code
   |
   v
Scan
   |
   v
Decrypt
   |
   v
Verify Student
```

လုပ်ခဲ့တယ်။

အခု Chapter 11 မှာတော့

**QR Code ကို ဘယ်လို create လုပ်သလဲ?**

ကို လေ့လာမယ်။

---

# Chapter 11 Overview

Student QR Creation Pipeline:

```text
Student Information

        |
        v

Create QR Payload

        |
        v

Generate Random Token

        |
        v

AES Encryption

        |
        v

ZXing QR Generation

        |
        v

Save PNG File

        |
        v

Store Token in Database
```

---

## Full Architecture

ဒီ System မှာ

### QR Generator

```text
Student
   |
   |
QRGenerator.java
   |
   |
CryptoUtils.java
   |
   |
QRUtils.java
   |
   |
StudentDAO.java
   |
   |
MySQL Database
```

---

# Part 1 — Student Data Collection

ပထမဆုံး Student Information လိုတယ်။

ဥပမာ:

Database ထဲမှာ

|Column|Value|
|---|---|
|student_id|ST001|
|name|Aung Mrat|
|department|Computer Science|
|email|[aung@gmail.com](mailto:aung@gmail.com)|

ရှိတယ်။

---

QR ထဲထည့်မယ့် Data:

```text
StudentID:ST001
Token:ABCD123456
```

---

ဘာကြောင့် Name, Email မထည့်တာလဲ?

Security အတွက်။

QR ထဲမှာ

```text
StudentID
+
Secret Token
```

လောက်ပဲ လိုတယ်။

---

# Part 2 — Generate Unique Token

အရေးကြီးဆုံးအပိုင်း။

Token ဆိုတာ:

```
Random Secret Identifier
```

ဖြစ်တယ်။

ဥပမာ:

```
ST001
+
8F92A7XK
```

---

Java မှာ:

```java
String token =
UUID.randomUUID().toString();
```

လိုမျိုး generate လုပ်နိုင်တယ်။

Example:

Before:

```
ST001
```

After:

```
ST001
Token:f7a82c91-4b3d-42aa
```

---

ဘာကြောင့် Token လိုလဲ?

အကယ်၍ QR ထဲမှာ

```text
StudentID:ST001
```

ပဲ ထည့်ရင်

Attacker က QR အသစ်လုပ်နိုင်တယ်။

---

Fake:

```
StudentID:ST001
```

ဖြစ်နိုင်တယ်။

---

Token ပါရင်:

```
StudentID:ST001
Token:f7a82c91
```

Database နဲ့ compare လုပ်ရမယ်။

---

# Part 3 — Create QR Payload

Code concept:

```java
String qrPayload =
        "StudentID:" + studentId
        + "\n"
        + "Token:" + token;
```

---

Result:

```
StudentID:ST001
Token:f7a82c91
```

---

ဒီ data ကို encrypt မလုပ်ခင် payload လို့ခေါ်တယ်။

---

# Part 4 — AES Encryption

အခု Plain Text ကို

CryptoUtils သုံးပြီး encrypt လုပ်မယ်။

```java
String encrypted =
        CryptoUtils.encrypt(qrPayload);
```

---

Before:

```
StudentID:ST001
Token:f7a82c91
```

---

After:

```
Q9x8K2LmA91Zx==
```

---

QR ထဲမှာ ဘာထည့်မလဲ?

မူရင်း Data မဟုတ်ဘူး။

Encrypted Data ဖြစ်တယ်။

---

Flow:

```
Student Data

     |
     v

AES Encryption

     |
     v

Random String

     |
     v

QR Code
```

---

# Part 5 — Why Encrypt Before QR?

မ encrypt လုပ်ဘူးဆိုရင်:

Mobile Scanner နဲ့ scan လုပ်ကြည့်ရင်

```
StudentID:ST001
Token:ABC123
```

မြင်ရမယ်။

---

Encrypt လုပ်ထားရင်:

Third-party Scanner:

```
8sH29Ks92Lx==
```

ပဲ မြင်မယ်။

---

အဲ့ဒါကြောင့် Comment မှာ:

```java
/**
 * Third-party QR scanners will only see encrypted Base64 string data.
 */
```

ရေးထားတာ။

---

# Part 6 — Generate QR with ZXing

ဒီနေရာမှာ ZXing Library သုံးတယ်။

Library:

```
Google ZXing
```

---

အဓိက Class:

```java
QRCodeWriter
```

or

```java
MultiFormatWriter
```

---

Example:

```java
BitMatrix bitMatrix =
new MultiFormatWriter().encode(
    encryptedData,
    BarcodeFormat.QR_CODE,
    width,
    height
);
```

---

Input:

```
Q9x8K2LmA91Zx==
```

---

Output:

```
██████████
██  ██  ██
██      ██
██████████
```

---

# Part 7 — BitMatrix Concept

QR Code ကို တိုက်ရိုက် Image မလုပ်ဘူး။

အရင်ဆုံး

```java
BitMatrix
```

လုပ်တယ်။

---

BitMatrix ဆိုတာ:

2D Boolean Array လိုမျိုး။

Example:

```
true  false true

false true  false

true  true  false
```

---

QR:

```
Black = true

White = false
```

---

Flow:

```
Text

 |

BitMatrix

 |

BufferedImage

 |

PNG
```

---

# Part 8 — Error Correction Level

Chapter 11 မှာ အရေးကြီးတဲ့အချက်။

```java
hints.put(
EncodeHintType.ERROR_CORRECTION,
ErrorCorrectionLevel.H
);
```

---

QR Code မှာ Error Correction ရှိတယ်။

Level:

```
L  ~7%

M  ~15%

Q  ~25%

H  ~30%
```

---

H ဆိုတာ:

30% ပျက်သွားတောင် scan လုပ်နိုင်တယ်။

---

Example:

QR မှာ Logo ထည့်ချင်ရင်

H သုံးတယ်။

---

# Part 9 — Character Encoding

```java
hints.put(
EncodeHintType.CHARACTER_SET,
"UTF-8"
);
```

---

ဘာကြောင့်?

Myanmar Unicode, English, Special Character support အတွက်။

Example:

```
Aung မရတ်
```

လို data တွေ။

---

# Part 10 — Margin

```java
hints.put(
EncodeHintType.MARGIN,
1
);
```

---

QR အပြင်က white border။

Example:

Without:

```
████████
████████
```

Scanner အခက်တွေ့နိုင်တယ်။

---

With:

```
   white

████████

   white
```

ပို scan လွယ်တယ်။

---

# Part 11 — Convert BitMatrix to Image

```java
MatrixToImageWriter.toBufferedImage(bitMatrix);
```

---

Before:

```
BitMatrix
```

After:

```
BufferedImage
```

---

ဒါနဲ့ Swing JLabel မှာ ပြနိုင်ပြီ။

---

Example:

```java
lblQR.setIcon(
new ImageIcon(image)
);
```

---

# Part 12 — Save QR PNG File

Method:

```java
saveQRCodeToFile()
```

---

Code:

```java
MatrixToImageWriter.writeToPath(
bitMatrix,
"PNG",
destination
);
```

---

Result:

Folder:

```
qrcodes/

    ST001.png

    ST002.png
```

---

# Part 13 — Create Directory Automatically

Code:

```java
File parentDir =
file.getParentFile();

if(!parentDir.exists()){
    parentDir.mkdirs();
}
```

---

Example:

Before:

```
Project

```

---

Need:

```
Project

   qrcodes

       ST001.png
```

---

Program က folder မရှိရင် create လုပ်ပေးတယ်။

---

# Part 14 — Store Token Database

အရေးကြီးဆုံး။

QR Generate ပြီးရင်

Token ကို database ထဲသိမ်းရမယ်။

Example:

Student Table:

|ID|Name|QR Token|
|---|---|---|
|ST001|Aung|f7a82c91|

---

ဘာကြောင့်?

Scanner ပြန်လာတဲ့အခါ:

```
QR Token

       compare

Database Token
```

လုပ်ဖို့။

---

Flow:

```
Generate Token

       |

Save Database

       |

Encrypt Token

       |

Create QR
```

---

# Part 15 — Complete QR Generation Flow

အားလုံးပေါင်းရင်:

```
Student Selected

        |
        v

Generate Token

        |
        v

Create Payload


StudentID:ST001
Token:XYZ123


        |
        v

AES Encrypt


        |
        v

ZXing


        |
        v

QR Image


        |
        v

Save PNG


        |
        v

Save Token Database
```

---

# Scanner Side နဲ့ Compare

## Generator

```
Token Generate

       ↓

Encrypt

       ↓

QR
```

## Scanner

```
QR

       ↓

Decrypt

       ↓

Extract Token

       ↓

Database Compare
```

---

# Security Architecture

ဒီ Project မှာ Authentication Flow:

```
             QR Generator


Student Data

    |
    v

Random Token

    |
    v

AES Encryption

    |
    v

QR Code



====================



             QR Scanner


QR Code

    |
    v

AES Decryption

    |
    v

Extract Token

    |
    v

Database Check

    |
    v

Access Granted
```

---

# Chapter 11 Summary

ဒီ Chapter မှာ:

✅ Student Data ကို QR Payload ပြုလုပ်တယ်။

✅ Random Token generate လုပ်တယ်။

✅ AES Encryption နဲ့ data protect လုပ်တယ်။

✅ ZXing Library နဲ့ QR Code generate လုပ်တယ်။

✅ BitMatrix → BufferedImage ပြောင်းတယ်။

✅ PNG File အဖြစ် save လုပ်တယ်။

✅ Token ကို Database ထဲသိမ်းတယ်။

✅ Scanner ဘက်မှာ Token ပြန်စစ်နိုင်အောင် ပြင်ဆင်တယ်။

---

**Senior Developer Observation**

ဒီ QR System က ရိုးရိုး QR Generator မဟုတ်ဘူး။

ဒါက:

```
QR Code
+
AES Encryption
+
Database Authentication
+
Token Verification
```

ပေါင်းထားတဲ့ **Student Identity Authentication System** ဖြစ်ပါတယ်။

---

နောက် Chapter 12 မှာတော့ ဒီ Project ရဲ့ Architecture ကို ကြည့်မယ်:

**Chapter 12 — Complete System Architecture (DAO + Model + Utils + UI Layer + Data Flow)**

အဲ့ဒီမှာ ဒီ Project ကို Professional Java Application အနေနဲ့ ဘယ်လိုခွဲရေးထားလဲ လေ့လာမယ်။