

အခုရောက်လာတဲ့ **Chapter 8 — `decodeQRCode()`** က QR Scanner ရဲ့ **Vision + Intelligence Layer** ဖြစ်ပါတယ်။

Chapter 7 မှာ

```
Webcam

↓

BufferedImage

↓

decodeQRCode(image)
```

အထိ ရောက်ခဲ့ပါတယ်။

အခု ဒီ Method က

**Camera က ရတဲ့ Image ထဲမှာ QR Code ရှိလား? ရှိရင် ဘာ Data ပါလဲ?**

ဆိုတာကို ရှာပေးပါတယ်။

---

# Chapter 8 Overview

ဒီ Method ကို ကြည့်ရအောင်။

```java
private Result decodeQRCode(BufferedImage image) {

    LuminanceSource source =
        new BufferedImageLuminanceSource(image);


    try {

        BinaryBitmap bitmap =
            new BinaryBitmap(
                new HybridBinarizer(source)
            );

        MultiFormatReader reader =
            new MultiFormatReader();

        reader.setHints(DECODE_HINTS);

        return reader.decodeWithState(bitmap);


    } catch (NotFoundException e) {

    }


    try {

        BinaryBitmap fallbackBitmap =
            new BinaryBitmap(
                new GlobalHistogramBinarizer(source)
            );

        MultiFormatReader reader =
            new MultiFormatReader();

        reader.setHints(DECODE_HINTS);

        return reader.decodeWithState(fallbackBitmap);


    } catch (Exception e) {

        return null;
    }
}
```

---

# Overall Flow

ဒီ Method အလုပ်လုပ်ပုံကို အရင်ကြည့်မယ်။

```
BufferedImage
(Camera Frame)

        |
        v

LuminanceSource
(Convert Image to Brightness)

        |
        v

Binarizer
(Convert Black/White)

        |
        v

BinaryBitmap

        |
        v

ZXing Decoder

        |
        v

QR Result
```

---

# Step 1 — Method Declaration

```java
private Result decodeQRCode(BufferedImage image)
```

---

## private

ဒီ Method ကို

ဒီ Class အတွင်းမှာပဲ သုံးလို့ရတယ်။

အပြင်ကနေ

```java
scanner.decodeQRCode()
```

လုပ်လို့မရဘူး။

---

## Result

Return Type က

```java
Result
```

ဖြစ်တယ်။

ZXing Library ကပေးတဲ့ Object ပါ။

ဥပမာ QR ထဲမှာ

```
StudentID:ST001
Token:ABC123
```

ရှိရင်

Result ထဲမှာ သိမ်းထားမယ်။

---

## Parameter

```java
BufferedImage image
```

ဆိုတာ

Camera က ရတဲ့ Frame ဖြစ်တယ်။

Chapter 7 မှာ

```java
BufferedImage image =
        webcam.getImage();
```

ရခဲ့တာ။

---

# Step 2 — Convert Image

```java
LuminanceSource source =
    new BufferedImageLuminanceSource(image);
```

ဒီ Line က အရေးကြီးတယ်။

Camera Image က

Color Image ဖြစ်တယ်။

ဥပမာ

```
RGB Image

Red
Green
Blue
```

ရှိတယ်။

ဒါကို QR Scanner က တိုက်ရိုက် မဖတ်ဘူး။

---

ZXing က လိုတာက

Brightness Data

ဖြစ်တယ်။

---

Convert လုပ်လိုက်ရင်

```
Before

RGB Image

██████


After

Brightness

255
200
50
0
```

ဖြစ်သွားတယ်။

---

# Luminance ဆိုတာ?

Luminance = Light Intensity

အလင်းရောင် ပမာဏ။

---

ဥပမာ

White Pixel

```
255
```

Black Pixel

```
0
```

---

QR Code က

အခြေခံအားဖြင့်

```
Black Square

+

White Background
```

ဖြစ်တဲ့အတွက်

Brightness Data လိုတယ်။

---

# Step 3 — First Attempt

```java
try {

```

ဘာကြောင့် Try လုပ်တာလဲ?

QR Decode Fail ဖြစ်နိုင်လို့ပါ။

ဥပမာ

Camera Frame:

```
No QR

```

ဆိုရင်

Exception ဖြစ်မယ်။

---

# HybridBinarizer

```java
new HybridBinarizer(source)
```

ဒါက ZXing ရဲ့

အကောင်းဆုံး QR Processing Algorithm တစ်ခု။

---

## Binarization ဆိုတာ?

Color/Brightness Image ကို

Black & White

ပြောင်းတာ။

---

Before:

```
Gray Image


120 130 140

100 90 80

```

After:

```
0 1 1

0 0 1

```

---

QR Code က

Binary Pattern ဖြစ်လို့

ဒီအဆင့်လိုတယ်။

---

# HybridBinarizer ဘာကောင်းလဲ?

Hybrid ဆိုတာ

Local + Global

ပေါင်းထားတာ။

---

ဥပမာ

Image:

```
Left side Dark

Right side Bright
```

ဆိုရင်

တစ်ခုလုံးကို Threshold တစ်ခုတည်း မသုံးဘူး။

နေရာအလိုက် တွက်တယ်။

---

ဒါကြောင့်

- Camera Quality မကောင်းတာ
    
- အလင်းမညီတာ
    
- Background ရှုပ်တာ
    

တွေမှာ ပိုကောင်းတယ်။

---

# Step 4 — BinaryBitmap

```java
BinaryBitmap bitmap =
    new BinaryBitmap(
        new HybridBinarizer(source)
    );
```

ဒီ Object က

ZXing Decoder ဖတ်နိုင်တဲ့ Format ဖြစ်တယ်။

---

Flow:

```
BufferedImage

↓

LuminanceSource

↓

Binarizer

↓

BinaryBitmap

```

---

# Step 5 — Create Reader

```java
MultiFormatReader reader =
        new MultiFormatReader();
```

ဒီဟာက

ZXing Decoder Engine ဖြစ်တယ်။

---

သူက

QR Code

Barcode

တွေကို ဖတ်နိုင်တယ်။

---

ဒါပေမယ့် ဒီ Project မှာ

QR ပဲ သုံးမယ်။

ဒါကြောင့် Hint ထည့်ထားတယ်။

---

# Step 6 — Set Decode Hints

```java
reader.setHints(DECODE_HINTS);
```

ဒီ Variable က

အပေါ်မှာ ရေးထားတယ်။

```java
private static final Map<DecodeHintType,Object>
DECODE_HINTS
```

---

Contents:

```java
DECODE_HINTS.put(
 DecodeHintType.TRY_HARDER,
 Boolean.TRUE
);
```

---

# TRY_HARDER

ပုံမှန် Scan မရရင်

ပိုကြိုးစားဖတ်ပါ

ဆိုတဲ့ Meaning။

---

ဥပမာ

Normal:

```
Try once
```

TRY_HARDER:

```
Try multiple methods
```

---

ဒါကြောင့်

Quality နည်းတဲ့ QR

ဖတ်နိုင်မှု တိုးတယ်။

---

# POSSIBLE_FORMATS

```java
DECODE_HINTS.put(
DecodeHintType.POSSIBLE_FORMATS,
EnumSet.of(BarcodeFormat.QR_CODE)
);
```

---

ZXing ကို ပြောတာ:

"Barcode အမျိုးအစား အားလုံးမရှာနဲ့၊ QR ပဲရှာ"

---

မထည့်ရင်

```
QR

EAN

UPC

CODE128

DataMatrix

```

အားလုံးစဉ်းစားမယ်။

---

ထည့်လိုက်ရင်

```
Camera Image

↓

Only QR Search

```

ပိုမြန်တယ်။

---

# CHARACTER_SET

```java
DECODE_HINTS.put(
DecodeHintType.CHARACTER_SET,
"UTF-8"
);
```

QR Data Encoding သတ်မှတ်တာ။

---

ဥပမာ

```
ကျောင်းသားအမည်
```

လို Unicode Text ပါရင်

UTF-8 လိုတယ်။

---

# Step 7 — Decode

```java
return reader.decodeWithState(bitmap);
```

ဒီ Line က

အဓိကအလုပ်။

---

Flow

```
BinaryBitmap

↓

ZXing Engine

↓

Find QR Pattern

↓

Extract Data

↓

Return Result
```

---

အောင်မြင်ရင်

```
Result Object
```

ပြန်မယ်။

---

မအောင်မြင်ရင်

```
NotFoundException
```

ဖြစ်မယ်။

---

# Step 8 — Catch First Attempt

```java
catch(NotFoundException e){

}
```

QR မတွေ့ဘူးဆိုတာပဲ။

ဒါကို Error ကြီး မသတ်မှတ်ဘူး။

ဘာကြောင့်?

Camera Frame တိုင်းမှာ QR မရှိနိုင်ဘူး။

---

ဥပမာ

Frame 100 ခုမှာ

```
Frame 1
No QR

Frame 2
No QR

Frame 3
QR Found
```

ဖြစ်နိုင်တယ်။

---

# Step 9 — Fallback Method

ပထမနည်းနဲ့ မရရင်

ဒီကို သွားတယ်။

```java
new GlobalHistogramBinarizer(source)
```

---

# Hybrid vs Global

## HybridBinarizer

ကောင်းတဲ့အရာ:

```
Normal Camera

Good Lighting

High Contrast
```

---

## GlobalHistogramBinarizer

ကောင်းတဲ့အရာ:

```
Screen Reflection

Glare

Uneven Light
```

---

ဥပမာ

Phone Screen ကို Scan လုပ်တဲ့အခါ

အလင်းပြန်နိုင်တယ်။

အဲဒီအချိန်

Global Histogram က ကူညီတယ်။

---

# Dual Strategy

ဒီ Method ရဲ့ Design က

အရမ်းကောင်းတယ်။

ဘာလဲဆိုတော့

```
Try Best Method

        |

Fail

        |

Try Backup Method

        |

Fail

        |

Return null
```

---

Professional Scanner တွေမှာလည်း

ဒီလို Fallback Strategy သုံးတတ်တယ်။

---

# Complete Decode Pipeline

အကုန်ပေါင်းကြည့်မယ်။

```
Camera Frame

      |
      v

BufferedImage

      |
      v

BufferedImageLuminanceSource

      |
      v

Brightness Data

      |
      v

HybridBinarizer

      |
      v

BinaryBitmap

      |
      v

MultiFormatReader

      |
      v

QR Detection

      |
      v

Result

```

---

# Example

Camera က ဒီ QR ကို ဖမ်းတယ်ဆိုပါစို့။

```
████████
█ QR   █
█ CODE █
████████
```

ZXing Process:

```
Image

↓

Brightness

↓

Black/White

↓

Find Finder Pattern

↓

Decode Modules

↓

Read Text

↓

Result
```

---

# Why not directly read BufferedImage?

မရဘူး။

ဘာကြောင့်?

ZXing Reader က

```java
BinaryBitmap
```

လိုတယ်။

---

ဒါကြောင့်

Conversion Chain

လိုတယ်။

```
BufferedImage

↓

LuminanceSource

↓

BinaryBitmap

↓

Reader

```

---

# Error Handling Design

ဒီ Method မှာ

Exception ကို

မပစ်ဘူး။

ဘာကြောင့်?

Scanner Loop က

တစ်စက္ကန့် 25 Frame လောက် Run နေတယ်။

Frame တစ်ခု Fail ဖြစ်တိုင်း

Application Crash ဖြစ်ရင် မကောင်းဘူး။

---

ဒါကြောင့်

```
Frame Fail

↓

Ignore

↓

Next Frame
```

လုပ်ထားတယ်။

---

# Senior Developer Analysis

ဒီ Method မှာ Professional Concepts တွေပါပါတယ်။

### 1. Separation of Responsibility

Webcam:

```
Capture Image
```

decodeQRCode():

```
Analyze Image
```

onQrCodeScanned():

```
Business Logic
```

---

### 2. Fallback Algorithm

```
Primary Algorithm

↓

Backup Algorithm
```

---

### 3. Performance Optimization

```java
POSSIBLE_FORMATS = QR_CODE
```

ထည့်ထားလို့

Search Space လျော့တယ်။

---

### 4. Robust Error Handling

```
Bad Frame

↓

No Crash

↓

Continue Scanning
```

---

# Chapter 8 Summary

ဒီ Chapter မှာ သင်လေ့လာခဲ့တာတွေက

✅ `decodeQRCode()` က Camera Image ထဲက QR Code ကို ရှာတဲ့ Engine ဖြစ်တယ်။

✅ `BufferedImage` ကို ZXing ဖတ်နိုင်အောင် `LuminanceSource` ပြောင်းတယ်။

✅ `Binarizer` က Image ကို Black/White Pattern ပြောင်းတယ်။

✅ `HybridBinarizer` က ပထမဆုံး ကြိုးစားတဲ့ High Accuracy Algorithm ဖြစ်တယ်။

✅ `GlobalHistogramBinarizer` က Backup Decoder ဖြစ်တယ်။

✅ `MultiFormatReader` က QR Data ကို Extract လုပ်ပေးတယ်။

✅ `TRY_HARDER` က Scan Accuracy တိုးစေတယ်။

✅ `POSSIBLE_FORMATS` က QR ပဲရှာအောင် Optimization လုပ်တယ်။

✅ Decode မရရင် Application မပျက်ဘဲ `null` ပြန်ပြီး နောက် Frame ဆက်စစ်တယ်။

---

**Chapter 9 မှာတော့ `onQrCodeScanned()` — AES Decryption + Database Verification + Student Authentication Flow** ကို သွားမယ်။

ဒီအပိုင်းက ဒီ Project ရဲ့ **Security Brain** ဖြစ်ပါတယ်။