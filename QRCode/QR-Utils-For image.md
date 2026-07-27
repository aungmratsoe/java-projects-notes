ဒီ Class က **QRUtils** လို့ခေါ်ပြီး **QR Code နဲ့ Image ကို စီမံခန့်ခွဲတဲ့ Utility Class** ဖြစ်ပါတယ်။

ဒီ Class ထဲမှာ အဓိက Function (၄) ခုရှိတယ်။

|Method|အလုပ်လုပ်ပုံ|
|---|---|
|`getHighQualityPNG()`|Image ကို Quality မကျဘဲ Resize လုပ်တယ်|
|`generateQRCodeImage()`|String ကနေ QR Image ထုတ်တယ်|
|`saveQRCodeToFile()`|QR Code ကို PNG File အဖြစ် Save လုပ်တယ်|
|`generateQRCode()`|UI မှာပြဖို့ BufferedImage ပြန်ပေးတယ်|

---

# Overall Architecture

```
                 QRUtils
                    │
      ┌─────────────┼─────────────┐
      │             │             │
      ▼             ▼             ▼
 Resize Image   Generate QR   Save QR File
      │             │             │
      ▼             ▼             ▼
 ImageIcon    BufferedImage      PNG File
```

ဒီ Class က Database မထိဘူး။

Encryption မလုပ်ဘူး။

QR Scanner မလုပ်ဘူး။

သူ့တာဝန်က

> "QR Image တွေ Generate လုပ်ပေးတာ"

---

# Package

```java
package com.ams.qrcode.utils;
```

Project Structure

```
com
 └── ams
      └── qrcode
            └── utils
                  QRUtils.java
```

---

# Imports

## Graphics2D

```java
import java.awt.Graphics2D;
```

Image ပေါ်မှာ Drawing လုပ်ဖို့။

ဥပမာ

```
Image

↓

Resize

↓

Rotate

↓

Draw Logo

↓

Crop
```

ဒီလိုအလုပ်တွေ လုပ်နိုင်တယ်။

---

## Image

```java
import java.awt.Image;
```

Java ရဲ့ Image Object

ဥပမာ

```
PNG

↓

Image Object
```

---

## RenderingHints

```java
import java.awt.RenderingHints;
```

Image Quality ကို မြှင့်ဖို့။

Java Default

```
Image

↓

Resize

↓

Blur
```

RenderingHints သုံးရင်

```
Image

↓

Resize

↓

Sharp
```

---

## ImageIcon

```java
import javax.swing.ImageIcon;
```

Swing JLabel မှာ

```java
label.setIcon(...)
```

ထည့်ဖို့။

---

# ZXing Library

ZXing = Zebra Crossing

Google ရဲ့ QR Library

```
String

↓

ZXing

↓

QR Code
```

---

# QRUtils Class

```java
public class QRUtils {
```

Utility Class

Object ဆောက်ပြီး

```java
QRUtils qr = new QRUtils();
```

သုံးနိုင်တယ်။

---

# Method (1)

## getHighQualityPNG()

```java
public ImageIcon getHighQualityPNG(
String resourcePath,
int width,
int height)
```

သူ့တာဝန်က

```
PNG

↓

Resize

↓

High Quality

↓

ImageIcon
```

---

# Step 1

```java
URL imgUrl =
getClass().getResource(resourcePath);
```

Project Resource ထဲက

ဥပမာ

```
/icons/user.png
```

ကို Load လုပ်တယ်။

```
Resources

↓

user.png

↓

URL
```

---

# Step 2

```java
Image img =
new ImageIcon(imgUrl).getImage();
```

URL

↓

Image Object

---

# Step 3

```java
BufferedImage resizedImg =
new BufferedImage(
width,
height,
TYPE_INT_ARGB
);
```

Image အသစ်တစ်ခု ဆောက်တယ်။

ဥပမာ

Original

```
500 x 500
```

အသစ်

```
80 x 80
```

---

### TYPE_INT_ARGB

ARGB ဆိုတာ

```
A = Alpha

R = Red

G = Green

B = Blue
```

Alpha = Transparency

PNG Background မပျောက်ဘူး။

---

# Step 4

```java
Graphics2D g2 =
resizedImg.createGraphics();
```

Canvas ပေါ်မှာ

Painter တစ်ယောက်လို

ဆွဲဖို့ Object ယူတယ်။

---

# Step 5

```java
g2.setRenderingHint(...)
```

ဒီဟာက Quality အတွက် အရေးကြီးဆုံး။

---

## Interpolation

```
VALUE_INTERPOLATION_BICUBIC
```

Java မှာ

Interpolation (၃) မျိုးရှိတယ်။

```
Nearest Neighbor
```

မြန်တယ်

Quality အဆိုးဆုံး

---

```
Bilinear
```

Quality အလယ်အလတ်

---

```
Bicubic
```

Quality အကောင်းဆုံး

ဒါကြောင့်

```
Blur

↓

Sharp
```

ဖြစ်သွားတယ်။

---

## Rendering Quality

```
VALUE_RENDER_QUALITY
```

Speed ထက်

Quality ဦးစားပေး။

---

## Anti Aliasing

```
VALUE_ANTIALIAS_ON
```

အစွန်းတွေကို Smooth ဖြစ်စေတယ်။

ဥပမာ

မသုံးရင်

```
██████
```

သုံးရင်

```
▓████▓
```

အစွန်းတွေ ချောလာတယ်။

---

# Step 6

```java
g2.drawImage(...)
```

Original Image

↓

Resize

↓

New Image

---

# Step 7

```java
g2.dispose();
```

Graphics Memory ပြန်လွှတ်တယ်။

ဒါမလုပ်ရင်

Memory Leak ဖြစ်နိုင်တယ်။

---

# Step 8

```java
return new ImageIcon(resizedImg);
```

BufferedImage

↓

ImageIcon

↓

JLabel

---

# Catch

```java
return null;
```

Image မတွေ့ရင်

```
Could not load icon
```

ထုတ်တယ်။

---

# Method (2)

## generateQRCodeImage()

```java
public BufferedImage
generateQRCodeImage(...)
```

ဒီ Method က

```
String

↓

QR Image

↓

BufferedImage
```

---

# Step 1

```java
QRCodeWriter qrCodeWriter =
new QRCodeWriter();
```

QR Generator Object

---

# Step 2

```java
BitMatrix bitMatrix =
qrCodeWriter.encode(...)
```

ဒီဟာ အရမ်းအရေးကြီးတယ်။

BitMatrix ဆိုတာ

QR Code ရဲ့

Black / White Matrix

ဥပမာ

```
1 0 0 1

0 1 1 0

1 1 0 1
```

1 = Black

0 = White

---

# Step 3

```java
return MatrixToImageWriter
.toBufferedImage(bitMatrix);
```

BitMatrix

↓

Image

---

# Method (3)

## saveQRCodeToFile()

ဒီ Method က

```
String

↓

QR

↓

PNG File
```

---

# Step 1

Hints

```java
Map<EncodeHintType,Object>
```

Hints ဆိုတာ

QR Generator ကို

Setting ပေးတာ။

---

### UTF-8

```
CHARACTER_SET
```

မြန်မာစာ

Chinese

Japanese

Emoji

အားလုံး Support။

---

### Error Correction

```
ErrorCorrectionLevel.H
```

QR မှာ

Level (၄) ခုရှိတယ်။

|Level|Recovery|
|---|---|
|L|7%|
|M|15%|
|Q|25%|
|H|30%|

H က အမြင့်ဆုံး။

ဥပမာ

QR Code

```
███████
██  ██
███████
```

၃၀% လောက် ပျက်သွားရင်တောင်

Scan ရသေးတယ်။

---

### Margin

```
MARGIN = 1
```

QR ပတ်လည်

Border

```
□□□□□□

██████

□□□□□□
```

Margin မရှိရင်

Scanner တချို့

မဖတ်နိုင်ဘူး။

---

# Step 2

```java
BitMatrix bitMatrix =
new MultiFormatWriter()
.encode(...)
```

String

↓

QR Matrix

---

# Step 3

```java
File file =
new File(filePath);
```

ဥပမာ

```java
qrcodes/STU1001.png
```

---

# Step 4

```java
parentDir.mkdirs();
```

Folder မရှိရင်

အလိုအလျောက် ဆောက်တယ်။

```
qrcodes/

↓

မရှိ

↓

Create
```

---

# Step 5

```java
MatrixToImageWriter
.writeToPath(...)
```

BitMatrix

↓

PNG File

---

# Method (4)

## generateQRCode()

ဒီ Method က

UI အတွက်။

```
Student Data

↓

QR

↓

BufferedImage
```

ပြီးရင်

```java
label.setIcon(
new ImageIcon(image)
);
```

ပြနိုင်တယ်။

---

# ဒီ Method နဲ့ generateQRCodeImage() ဘာကွာလဲ?

အခု Code မှာ

```java
generateQRCodeImage()
```

နဲ့

```java
generateQRCode()
```

နှစ်ခုလုံးက **BufferedImage** ပြန်ပေးပြီး QR Code Generate လုပ်တဲ့အလုပ်ကိုပဲ လုပ်ပါတယ်။

ကွာတာက

### generateQRCodeImage()

```
QRCodeWriter
```

သုံးထားတယ်။

```
Hints မရှိ
```

---

### generateQRCode()

```
MultiFormatWriter
```

သုံးထားတယ်။

```
UTF-8 ✔

Margin ✔

Error Correction ✔
```

---

Professional Project မှာတော့

```
generateQRCode()
```

ကိုပဲ သုံးရင် လုံလောက်ပါတယ်။

---

# ဒီ Class ရဲ့ Flow

```
Student Information
        │
        ▼
QRUtils.generateQRCode()
        │
        ▼
Hints
(UTF-8, Margin, Error Correction)
        │
        ▼
MultiFormatWriter
        │
        ▼
BitMatrix
        │
        ▼
MatrixToImageWriter
        │
        ▼
BufferedImage
        │
        ├──────────────► JLabel
        │
        ▼
saveQRCodeToFile()
        │
        ▼
PNG File
```

# Senior Developer အမြင်နဲ့ Review

ဒီ Code ကိုကြည့်ရင် Structure က ရှင်းလင်းပြီး ZXing Library ကို မှန်မှန်ကန်ကန် အသုံးပြုထားပါတယ်။ `ErrorCorrectionLevel.H` ကို သုံးထားတာကြောင့် Logo ထည့်တဲ့ QR Code သို့မဟုတ် အနည်းငယ်ပျက်စီးနိုင်တဲ့ QR Code တွေအတွက်လည်း ပိုအဆင်ပြေပါတယ်။

ဒါပေမယ့် နည်းနည်း Refactor လုပ်လို့ရတဲ့ အချက်တွေရှိပါတယ်။

1. **`generateQRCodeImage()` နဲ့ `generateQRCode()` က လုပ်ဆောင်ချက်ထပ်နေပါတယ်။** Hints ပါတဲ့ `generateQRCode()` တစ်ခုတည်းထားပြီး အခြားတစ်ခုကို ဖျက်နိုင်ပါတယ်။
2. **`getHighQualityPNG()` က `static` ဖြစ်နိုင်ပါတယ်။** Class မှာ Instance Variable မရှိတဲ့အတွက် `QRUtils.getHighQualityPNG(...)` လိုခေါ်လို့ရရင် ပိုသန့်ပါတယ်။
3. **`getResource()` က `null` ပြန်လာနိုင်ပါတယ်။** `new ImageIcon(imgUrl)` မလုပ်ခင် `imgUrl == null` ကို စစ်ပေးရင် `NullPointerException` ကို ပိုကောင်းကောင်း ကိုင်တွယ်နိုင်ပါတယ်။
4. **RenderingHints** တွေကို သုံးထားတာက အလွန်ကောင်းပါတယ်။ Java Swing မှာ Image Resize လုပ်တဲ့အခါ Default `getScaledInstance()` ထက် ဒီနည်းလမ်းက ပိုပြီး ကြည်လင်တဲ့ Result ရပါတယ်။

ဒီ Class က Java Swing Student Management System တစ်ခုမှာ **QR Code Generate**, **UI Preview**, **PNG Export**, **High-quality Icon Resize** ဆိုတဲ့ တာဝန်တွေကို သပ်သပ်ရပ်ရပ် စုစည်းထားတဲ့ Utility Class ကောင်းတစ်ခုလို့ ပြောလို့ရပါတယ်။