

Professional Java Developer တစ်ယောက်က Code ကိုဖတ်တဲ့အခါ **Method ကစပြီး မဖတ်ဘူး။ Import ကနေ စဖတ်တယ်။**

ဘာကြောင့်လဲ?

Import တွေကြည့်လိုက်တာနဲ့

> **"ဒီ Program က ဘာလုပ်မလဲ?"**

ဆိုတာ 70% လောက် ခန့်မှန်းနိုင်သွားတယ်။

ဥပမာ

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
```

ကြည့်လိုက်တာနဲ့

> Database သုံးတဲ့ Program

လို့သိသွားပြီ။

---

# Import Section Overview

သင့် Project ရဲ့ Import တွေကို Category ခွဲလိုက်ရင်

```text
                    QRScanner Imports
                           │
 ┌──────────────┬──────────────┬──────────────┬─────────────┐
 │              │              │              │
 ▼              ▼              ▼              ▼
 Project      Swing GUI      Webcam        ZXing QR
 Classes                       API          Library
 │              │              │              │
 ▼              ▼              ▼              ▼
DAO         JFrame        Webcam      MultiFormatReader
Model       JLabel        Resolution  BinaryBitmap
Crypto      Timer         Camera      Result
```

ဒီလို Group (၄) ခု ခွဲလို့ရတယ်။

---

# Group 1 — Project Classes

ဒါတွေက ကိုယ်ရေးထားတဲ့ Class တွေ။

```java
import com.ams.qrcode.dao.StudentDAO;
import com.ams.qrcode.dao.StudentDAOInterface;
import com.ams.qrcode.model.Student;
import com.ams.qrcode.utils.CryptoUtils;
```

---

## StudentDAO

```java
StudentDAO studentDAO = new StudentDAO();
```

ဒီ Class က

Database ထဲက

Student ကိုရှာပေးတယ်။

Diagram

```text
QRScanner

      │
      ▼

StudentDAO

      │
      ▼

MySQL Database
```

ဥပမာ

```java
studentDAO.getStudentByStudentId("STU1001");
```

↓

Database ထဲက

```text
Student

ID : STU1001

Name : Aung
```

ပြန်လာမယ်။

---

## StudentDAOInterface

```java
StudentDAOInterface
```

ဒါက

Interface ဖြစ်တယ်။

Professional Project တွေမှာ

ဒီလိုရေးတယ်။

```text
QRScanner

      │

      ▼

Interface

      │

      ▼

StudentDAO
```

ဘာကြောင့် Interface သုံးတာလဲ?

နောက်ပိုင်း

MySQL အစား

Oracle

PostgreSQL

MongoDB

ပြောင်းရင်

QRScanner ကို မပြင်ရတော့ဘူး။

---

## Student

ဒါက Model Class

ဥပမာ

```java
Student student;
```

Memory ထဲမှာ

```text
Student

ID

Name

Email

Department

Phone

QR Token
```

သိမ်းထားတယ်။

---

## CryptoUtils

ဒါက

AES Encryption

လုပ်တဲ့ Class

Workflow

```text
QR Code

↓

Encrypted String

↓

CryptoUtils.decrypt()

↓

Plain Text

↓

StudentID

↓

Database
```

---

# Group 2 — FlatLaf

```java
import com.formdev.flatlaf.FlatLightLaf;
```

FlatLaf ဆိုတာ

Look & Feel Library

Java Swing Default

```text
Windows 98
```

လိုမျိုး ဖြစ်နေတယ်။

FlatLaf

↓

```text
Modern UI
```

ဖြစ်သွားတယ်။

---

## FlatSVGIcon

```java
FlatSVGIcon
```

SVG Icon ကို

Quality မကျဘဲ

Resize လုပ်ပေးတယ်။

PNG ဆိုရင်

```text
100px

↓

20px

↓

Blur
```

SVG

```text
100px

↓

20px

↓

Sharp
```

---

# Group 3 — Webcam Library

ဒါက အရေးကြီးတယ်။

```java
import com.github.sarxos.webcam.Webcam;
```

Java Standard Library မှာ

Webcam API မရှိဘူး။

ဒါကြောင့်

Third Party Library

သုံးထားတာ။

Diagram

```text
Java

↓

Webcam Library

↓

USB Camera

↓

Image
```

---

## Webcam

```java
Webcam webcam;
```

Camera Object

---

ဥပမာ

```java
webcam.open();
```

↓

Camera ON

---

```java
webcam.close();
```

↓

Camera OFF

---

```java
webcam.getImage();
```

↓

Current Frame

---

## WebcamResolution

```java
WebcamResolution.VGA
```

Resolution Ready-made Enum

ဥပမာ

```text
QQVGA

QVGA

VGA

HD

Full HD
```

---

VGA

=

```text
640 x 480
```

---

# Group 4 — ZXing Library

ဒါက

QR Scanner Engine

Google ရေးထားတဲ့ Library

---

## BarcodeFormat

```java
BarcodeFormat.QR_CODE
```

QR ပဲဖတ်မယ်။

Barcode မဖတ်ဘူး။

---

## BinaryBitmap

QR Scanner က

Color Image ကို

မဖတ်ဘူး။

သူဖတ်တာ

```text
Black

White
```

ပဲ။

ဒါကြောင့်

```text
Image

↓

BinaryBitmap

↓

Decode
```

---

## LuminanceSource

Luminance

=

Brightness

ပဲ။

Image

↓

Gray Scale

ပြောင်းတယ်။

ဥပမာ

```text
RGB Image

↓

Gray Image
```

---

## BufferedImageLuminanceSource

Java BufferedImage

↓

ZXing LuminanceSource

ပြောင်းပေးတယ်။

---

## HybridBinarizer

ဒါက

Image Processing Algorithm

Professional Scanner တွေ

အသုံးများတယ်။

သူက

Shadow

Brightness

Contrast

ကို

Adaptive လုပ်ပေးတယ်။

ဥပမာ

```text
█████░░░░

↓

████████
```

လိုမျိုး။

---

## GlobalHistogramBinarizer

Hybrid မအောင်မြင်ရင်

Fallback

အနေနဲ့သုံးတယ်။

သူက

Image တစ်ခုလုံးရဲ့

Average Brightness

နဲ့

Black/White ခွဲတယ်။

---

## MultiFormatReader

ဒါက

QR Decoder

Engine

Diagram

```text
BinaryBitmap

↓

MultiFormatReader

↓

Decoded Text
```

---

## Result

Decode ပြီးရင်

ဒီ Object ထဲမှာ

Result ရတယ်။

ဥပမာ

```java
Result result;
```

↓

```text
StudentID

Token

Text

Barcode Type
```

---

## NotFoundException

QR မတွေ့ရင်

Throw လုပ်တဲ့ Exception

ဒါကို

```java
catch(NotFoundException e)
```

နဲ့ဖမ်းတယ်။

---

# Group 5 — Java Concurrency

ဒါက Senior Level Topic ပါ။

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicBoolean;
```

---

## ExecutorService

Thread Manager

Diagram

```text
Program

↓

Executor

↓

Worker Thread
```

သူက

Thread ကို

Create

Reuse

Destroy

လုပ်ပေးတယ်။

---

## Executors

ဒါက

Executor Factory

ဥပမာ

```java
Executors.newSingleThreadExecutor();
```

↓

Thread (၁) ခုပဲ

ဖန်တီးတယ်။

---

## AtomicBoolean

ဒါက

Boolean ပေမယ့်

Thread Safe

ဖြစ်တယ်။

ပုံမှန်

```java
boolean dialogOpen;
```

Thread နှစ်ခုက

တစ်ပြိုင်တည်းပြင်ရင်

Race Condition ဖြစ်နိုင်တယ်။

AtomicBoolean က

CPU Level မှာ Atomic Operation သုံးလို့

ပိုလုံခြုံတယ်။

---

# Group 6 — Swing

```java
import javax.swing.*;
```

ဒါတွေက

GUI Components

```text
JFrame

JLabel

Timer

JOptionPane
```

---

## SwingUtilities

ဒါက

GUI Thread ပေါ်မှာ

Code Run ဖို့။

ဥပမာ

```java
SwingUtilities.invokeLater(...)
```

Background Thread က

JLabel ကို

တိုက်ရိုက်ပြင်ရင်

Bug ဖြစ်နိုင်တယ်။

ဒါကြောင့်

GUI Thread ဆီ

ပြန်ပို့တာ။

---

## Timer

Swing Animation

အတွက်သုံးတာ။

---

## JOptionPane

Message Box

```text
Information

Warning

Error
```

ပြဖို့။

---

# Group 7 — AWT

```java
import java.awt.Image;
```

Image Object

---

```java
BufferedImage
```

Memory ထဲက

Pixel Image

---

# Group 8 — Utility Classes

```java
EnumMap

EnumSet

Map
```

ဒီဟာတွေကို

ZXing Hint တွေ

သိမ်းဖို့

သုံးထားတာ။

ဥပမာ

```java
Map<DecodeHintType,Object>
```

↓

```text
TRY_HARDER = true

CHARACTER_SET = UTF-8

FORMAT = QR
```

---

# Group 9 — Logger

```java
import java.util.logging.Level;
```

Error တွေကို

Console ထဲမှာ

Professional ပုံစံနဲ့

Record လုပ်ဖို့။

ဥပမာ

```java
logger.log(Level.SEVERE,
    "Webcam initialization error", e);
```

ဒါက `e.printStackTrace()` ထက် ပိုကောင်းပါတယ်။ ဘာ Error ဖြစ်လဲ၊ Severity ဘယ်လောက်ရှိလဲဆိုတာကို စနစ်တကျ မှတ်တမ်းတင်ပေးပါတယ်။

---

# Import Section ကို တစ်ကြည့်နဲ့ ဘာသိနိုင်လဲ?

Import တွေကိုပဲ ကြည့်လိုက်ရင် ဒီ Class က

- ✅ Java Swing GUI Application ဖြစ်တယ်။
    
- ✅ Webcam ကို အသုံးပြုပြီး Live Camera Feed ရယူမယ်။
    
- ✅ Google ZXing Library နဲ့ QR Code Decode လုပ်မယ်။
    
- ✅ AES နဲ့ Encrypt လုပ်ထားတဲ့ QR Data ကို `CryptoUtils` နဲ့ Decrypt လုပ်မယ်။
    
- ✅ `StudentDAO` ကတစ်ဆင့် Database ကို စစ်မယ်။
    
- ✅ Background Thread (`ExecutorService`) နဲ့ Camera Processing လုပ်ပြီး GUI (`SwingUtilities`) ကို သီးခြားကိုင်တွယ်ထားတယ်။
    
- ✅ Modern UI အတွက် FlatLaf နဲ့ SVG Icon တွေ အသုံးပြုထားတယ်။
    

ဒီလို Import Section ကို နားလည်ထားရင် Method တွေဖတ်တဲ့အခါ **"ဒီ Class က ဘာအတွက်ရေးထားတာလဲ"** ဆိုတာကို အရင်ဆုံး မြင်နိုင်ပြီး Code Flow ကို နားလည်ဖို့ အများကြီးလွယ်သွားပါတယ်။