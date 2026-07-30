**ဤ Java ကုဒ်ကို အသေးစိတ် ရှင်းပြချက် (Burmese Explanation)**

ဤကုဒ်သည် **QR Code** ဖန်တီးရန်နှင့် ပုံရိပ်များ ကိုင်တွယ်ရန်အတွက် အသုံးပြုသော Utility Class တစ်ခုဖြစ်သည်။ အမည်မှာ `QRUtils` ဖြစ်ပြီး ZXing (Google ရဲ့ QR/Barcode Library) ကို အဓိက အသုံးပြုထားသည်။

### ၁. Package နှင့် Import Statements

```java
package com.ams.qrcode.utils;
```

- ဤ Class ကို `com.ams.qrcode.utils` package အောက်တွင် ထားရှိသည်။

**အရေးကြီး Import များ:**

- `java.awt.*` → ပုံရိပ်ဆွဲခြင်း (Graphics2D) နှင့် Image ကိုင်တွယ်ရန်
- `javax.swing.ImageIcon` → Swing UI တွင် ပုံပြရန်
- `com.google.zxing.*` → QR Code ဖန်တီးရန် အဓိက Library (ZXing)
- `java.awt.image.BufferedImage` → ပုံရိပ်ကို မှတ်ဉာဏ်ထဲတွင် သိမ်းဆည်းရန်
- `java.io.File`, `java.nio.file.Path` → ဖိုင်သို့ သိမ်းဆည်းရန်

---

### ၂. Class အဓိက အကြောင်းအရာ

```java
public class QRUtils {
```

ဤ Class သည် QR Code ဆိုင်ရာ လုပ်ဆောင်ချက်များကို စုစည်းထားသော Helper Class ဖြစ်သည်။

---

### ၃. Method ၁: `getHighQualityPNG()`

```java
public ImageIcon getHighQualityPNG(String resourcePath, int width, int height)
```

**ရည်ရွယ်ချက်:**  
Resources folder ထဲက ပုံဖိုင်ကို ဖတ်ပြီး **အရည်အသွေး မြင့်မားစွာ** resize လုပ်ပေးသည်။ Java ရဲ့ ပုံမှန် scaler က မှုန်ဝါးတတ်သဖြင့် ဤ method က အရည်အသွေး ပိုကောင်းအောင် လုပ်ပေးသည်။

**အလုပ်လုပ်ပုံ အဆင့်ဆင့်:**

1. `getClass().getResource(resourcePath)` → Resources ထဲက ပုံကို ရှာယူသည်။
2. `BufferedImage` အသစ်တစ်ခု ဖန်တီးသည် (ARGB အရောင်စနစ်)။
3. `Graphics2D` ဖြင့် ရေးဆွဲရန် ပြင်ဆင်သည်။
4. **Rendering Hints** ကို ဖွင့်ပေးသည်:
    - `BICUBIC` interpolation → ချောမွေ့စွာ resize
    - `RENDER_QUALITY` နှင့် `ANTIALIAS_ON` → အရည်အသွေး မြင့်မားစေရန်
5. ပုံကို ဆွဲပြီး `ImageIcon` အဖြစ် ပြန်ပေးသည်။

**အသုံးပြုမှု:** Swing UI (JLabel စသည်) တွင် အိုင်ကွန်များ ချောချောမွေ့မွေ့ ပြရန်။

---

### ၄. Method ၂: `generateQRCodeImage()`

```java
public BufferedImage generateQRCodeImage(String text, int width, int height)
```

**ရည်ရွယ်ချက်:**  
ရိုးရှင်းစွာ QR Code ပုံ (BufferedImage) တစ်ခု ဖန်တီးပေးသည်။

**အလုပ်လုပ်ပုံ:**

- `QRCodeWriter` ကို သုံး၍ `text` ကို QR Code အဖြစ် encode လုပ်သည်။
- `MatrixToImageWriter.toBufferedImage()` ဖြင့် BitMatrix ကို ပုံအဖြစ် ပြောင်းပေးသည်။

**အားနည်းချက်:** Hints (Error Correction, Margin) များ မထည့်ထားသောကြောင့် ရိုးရှင်းသော အသုံးပြုမှုအတွက်သာ သင့်တော်သည်။

---

### ၅. Method ၃: `saveQRCodeToFile()` (အရေးအကြီးဆုံး)

```java
public void saveQRCodeToFile(String data, String filePath, int width, int height)
```

**ရည်ရွယ်ချက်:**  
QR Code ကို ဖန်တီးပြီး **hard disk** ပေါ်တွင် PNG ဖိုင်အဖြစ် သိမ်းဆည်းပေးသည်။

**အသေးစိတ် လုပ်ဆောင်ချက်များ:**

1. **Hints ပြင်ဆင်ခြင်း** (အရေးကြီးဆုံး အပိုင်း):
    - `CHARACTER_SET = UTF-8` → မြန်မာစာ၊ တရုတ်စာ စသည်တို့ ပါဝင်နိုင်ရန်
    - `ERROR_CORRECTION = H` (High) → ပုံပျက်ရင်လည်း ဖတ်နိုင်အောင် ခိုင်ခံ့စေရန်
    - `MARGIN = 1` → QR ပတ်ပတ်လည် အနားသတ်
2. `MultiFormatWriter().encode()` → အဆင့်မြင့် encoder သုံး၍ BitMatrix ရယူသည်။
3. ဖိုင်သိမ်းမည့် ဖိုလ်ဒါ မရှိသေးပါက `mkdirs()` ဖြင့် အလိုအလျောက် ဖန်တီးပေးသည်။
4. `MatrixToImageWriter.writeToPath()` ဖြင့် တိုက်ရိုက် PNG ဖိုင်အဖြစ် သိမ်းသည်။

**ဥပမာ အသုံးပြုပုံ:**

```java
utils.saveQRCodeToFile("STU12345", "qrcodes/STU12345.png", 300, 300);
```

---

### ၆. Method ၄: `generateQRCode()`

```java
public BufferedImage generateQRCode(String data, int width, int height)
```

`saveQRCodeToFile` နှင့် အလားတူလုပ်ဆောင်ချက်ရှိသော်လည်း ဖိုင်သိမ်းမည့်အစား **BufferedImage** ပြန်ပေးသည်။

**အသုံးပြုမှု:**

- Swing UI တွင် ချက်ချင်း ပြရန်
- PDF သို့ ထည့်ရန်
- အခြား ပုံဆိုင်ရာ လုပ်ဆောင်ချက်များအတွက်

---

### အကျဉ်းချုပ် အားသာချက်များ

- **UTF-8** ပံ့ပိုးမှု → မြန်မာစာ ပါဝင်နိုင်သည်။
- **High Error Correction (Level H)** → ပုံရိပ်ကို ပျက်စီးနည်းနည်း ခံနိုင်ရည်ရှိသည်။
- **High Quality Image Resizing** → UI တွင် ချောမွေ့စွာ ပြသနိုင်သည်။
- Directory auto-creation ပါဝင်သည်။

---

**အကြံပြုချက်:**

- ဤ Class ကို အသုံးပြုရန် **ZXing** dependency (Maven/Gradle) ထည့်ရမည်။
- Production တွင် အမှားကိုင်တွယ်မှု (Exception Handling) ပိုမို ခိုင်ခိုင်မာမာ လုပ်သင့်သည်။
- လုံခြုံရေး ရှုထောင့်မှနေ အရေးကြီးသော အချက်အလက်များ (ဥပမာ စီးပွားရေး လုံခြုံရေး) ကို QR Code တွင် ထည့်ရင် ကုဒ်ကို encrypt လုပ်ထားသင့်သည်။

ဤကုဒ်ကို နားလည်ရလွယ်ကူအောင် ခွဲပြီး ရှင်းပြပြီးပါပြီ။ ဘယ်အပိုင်းကို ပိုမို အသေးစိတ် ရှင်းပြရမလဲ သို့မဟုတ် ဥပမာ အသုံးပြုပုံ လိုချင်ပါသလား ပြောပါ။

---

**ဤ Java Class ရှိ တစ်ခုချင်းစီ Method များကို အသေးစိတ် ရှင်းပြချက်**

### ၁. `getHighQualityPNG()` Method

```java
public ImageIcon getHighQualityPNG(String resourcePath, int width, int height)
```

**အသေးစိတ်ရှင်းပြချက်:**

- **Function:** Resources folder ထဲမှ ပုံဖိုင်ကို ဖတ်ယူ၍ လိုချင်သော အရွယ်အစား (width, height) အတိုင်း **အရည်အသွေးမြင့်** resize လုပ်ပြီး `ImageIcon` အဖြစ် ပြန်ပေးသည်။

**ပါရာမီတာ (Parameters):**

- `String resourcePath` → ပုံရှိတည်နေရာ (ဥပမာ: `/icons/logo.png`)
- `int width` → ချုံ့/ချဲ့မည့် အကျယ် (pixel)
- `int height` → ချုံ့/ချဲ့မည့် အမြင့် (pixel)

**အတွင်းလုပ်ဆောင်ချက် အဆင့်ဆင့်:**

1. `getClass().getResource(resourcePath)` ဖြင့် ပုံ URL ရယူသည်။
2. ပုံဟောင်းကို `Image` အဖြစ် ဖတ်ယူသည်။
3. အသစ် `BufferedImage` (TYPE_INT_ARGB) တစ်ခု ဖန်တီးသည်။
4. `Graphics2D` ကို ရယူ၍ ရေးဆွဲပြင်ဆင်သည်။
5. **Rendering Hints** သုံး၍ အရည်အသွေး မြှင့်တင်သည်:
    - `KEY_INTERPOLATION` → `VALUE_INTERPOLATION_BICUBIC` (အကောင်းဆုံး resize algorithm)
    - `KEY_RENDERING` → `VALUE_RENDER_QUALITY`
    - `KEY_ANTIALIASING` → `VALUE_ANTIALIAS_ON` (အစွန်းများ ချောမွေ့စေရန်)
6. ပုံကို ဆွဲပြီး `ImageIcon` အဖြစ် ပြန်ပေးသည်။

**အမှားဖြစ်ရင်:** Console တွင် error ပြပြီး `null` ပြန်ပေးသည်။

---

### ၂. `generateQRCodeImage()` Method

```java
public BufferedImage generateQRCodeImage(String text, int width, int height) throws Exception
```

**ရည်ရွယ်ချက်:** QR Code ကို ရိုးရိုးရှင်းရှင်း ဖန်တီးပြီး `BufferedImage` အဖြစ် ပြန်ပေးသည်။

**ပါရာမီတာ:**

- `String text` → QR Code ထဲတွင် ထည့်မည့် စာသား (URL, ကုဒ်၊ အချက်အလက် စသည်)
- `int width`, `int height` → QR ပုံ အရွယ်အစား

**လုပ်ဆောင်ပုံ:**

- `QRCodeWriter` ကို အသုံးပြု၍ `encode()` လုပ်သည်။
- ရလဒ် BitMatrix ကို `MatrixToImageWriter.toBufferedImage()` ဖြင့် ပုံအဖြစ် ပြောင်းပေးသည်။

**မှတ်ချက်:** ဤ method တွင် **Hints** (Error Correction, Margin, Charset) မထည့်ထားသောကြောင့် ရိုးရှင်းသော အသုံးပြုမှုအတွက်သာ သင့်တော်သည်။

---

### ၃. `saveQRCodeToFile()` Method (အကောင်းဆုံး နှင့် အသုံးအများဆုံး)

```java
public void saveQRCodeToFile(String data, String filePath, int width, int height) throws Exception
```

**ရည်ရွယ်ချက်:** QR Code ဖန်တီး၍ သတ်မှတ်ထားသော ဖိုင်လမ်းကြောင်းတွင် **PNG** အဖြစ် သိမ်းဆည်းပေးသည်။

**ပါရာမီတာ:**

- `String data` → QR ထဲတွင် ထည့်မည့် အချက်အလက်
- `String filePath` → သိမ်းမည့် ဖိုင်လမ်းကြောင်း (ဥပမာ: `"qrcodes/student123.png"`)
- `int width`, `int height` → ပုံအရွယ်အစား

**အတွင်း အသေးစိတ် လုပ်ဆောင်ချက်:**

1. **Hints Map** ဖန်တီးသည်:
    - `CHARACTER_SET = "UTF-8"` → မြန်မာစာ ပါဝင်နိုင်ရန်
    - `ERROR_CORRECTION = ErrorCorrectionLevel.H` → အဆင့်မြင့် ပြန်လည်ရယူနိုင်မှု (30% အထိ ပျက်စီးခံနိုင်)
    - `MARGIN = 1` → QR ပတ်လည် အနားသတ်
2. `MultiFormatWriter().encode()` ဖြင့် BitMatrix ရယူသည်။
3. ဖိုင်သိမ်းမည့် ဖိုလ်ဒါ မရှိသေးပါက `parentDir.mkdirs()` ဖြင့် အလိုအလျောက် ဖန်တီးပေးသည်။
4. `MatrixToImageWriter.writeToPath()` ဖြင့် ဖိုင်ကို တိုက်ရိုက် ရေးသားသိမ်းဆည်းသည်။

**အားသာချက်:** ဖိုလ်ဒါ မရှိရင်လည်း အလုပ်လုပ်နိုင်သည်။

---

### ၄. `generateQRCode()` Method

```java
public BufferedImage generateQRCode(String data, int width, int height) throws Exception
```

**ရည်ရွယ်ချက်:** QR Code ကို ဖန်တီးပြီး ဖိုင်သိမ်းမည့်အစား **BufferedImage** အဖြစ် ပြန်ပေးသည်။ UI တွင် ချက်ချင်း ပြရန်အတွက် အသုံးဝင်ဆုံး။

**ပါရာမီတာ:** `saveQRCodeToFile` နှင့် အတူတူပင်။

**အလုပ်လုပ်ပုံ:**  
`saveQRCodeToFile` နှင့် အလုံးစုံ တူညီသော Hints နှင့် encoding လုပ်ဆောင်ချက်ရှိသည်။ ကွဲပြားချက်မှာ ဖိုင်သိမ်းမည့်အစား `MatrixToImageWriter.toBufferedImage(bitMatrix)` ဖြင့် ပုံကို တိုက်ရိုက်ပြန်ပေးခြင်း ဖြစ်သည်။

---

**နှိုင်းယှဉ်ချက် (အကျဉ်းချုပ်):**

|Method|ပြန်ပေးသော Type|ဖိုင်သိမ်း|Hints ပါမှု|အဓိက အသုံး|
|---|---|---|---|---|
|getHighQualityPNG|ImageIcon|မသိမ်း|-|UI Icon|
|generateQRCodeImage|BufferedImage|မသိမ်း|မပါ|ရိုးရှင်း|
|saveQRCodeToFile|void|သိမ်း|ပါ|ဖိုင်သိမ်း|
|generateQRCode|BufferedImage|မသိမ်း|ပါ|UI ပြရန်|

---

လိုချင်ရင် တစ်ခုချင်းစီကို ပိုမို နက်ရှိုင်းစွာ ရှင်းပြပေးနိုင်ပါတယ် (ဥပမာ: Hints တွေ ဘာကြောင့် အရေးကြီးလဲ၊ ဘယ်လို ပြောင်းလဲရမလဲ စသည်)။ ဘယ် method ကို ပိုအသေးစိတ် လိုချင်ပါသလဲ ပြောပါ။

---

ဒီ Method က **PNG Image တစ်ခုကို Project Resource ထဲကနေ Load လုပ်ပြီး၊ High Quality နဲ့ Resize လုပ်ပြီး ImageIcon အဖြစ် Return ပြန်ပေးတဲ့ Utility Method** ဖြစ်ပါတယ်။

အတိုချုပ် Flow ကို အရင်ကြည့်ရအောင်။

```text
Resource Path
      │
      ▼
getResource()
      │
      ▼
Load Original Image
      │
      ▼
Create Empty BufferedImage
      │
      ▼
Create Graphics2D
      │
      ▼
Enable High Quality Rendering
      │
      ▼
Draw (Resize) Image
      │
      ▼
Dispose Graphics
      │
      ▼
Return ImageIcon
```

အခု Line by Line အသေးစိတ်ရှင်းပါမယ်။

---

# Method Declaration

```java
public ImageIcon getHighQualityPNG(String resourcePath, int width, int height)
```

ဒီ Method မှာ Parameter (၃) ခုရှိပါတယ်။

```java
String resourcePath
```

Image ရဲ့ Path

ဥပမာ

```java
"/icons/user.png"
```

---

```java
int width
```

Image Width

ဥပမာ

```java
64
```

---

```java
int height
```

Image Height

ဥပမာ

```java
64
```

---

Return Type

```java
ImageIcon
```

နောက်ဆုံးမှာ

```java
ImageIcon
```

ပြန်ပေးမယ်။

Swing မှာ

```java
JLabel
JButton
```

တွေမှာ တိုက်ရိုက်သုံးနိုင်တယ်။

ဥပမာ

```java
label.setIcon(getHighQualityPNG(...));
```

---

# try block

```java
try {
```

Image Load လုပ်တဲ့အချိန်

- File မရှိတာ
    
- Path မှားတာ
    

ဖြစ်နိုင်လို့

Exception ကို Handle လုပ်ထားတာ။

---

# Step 1

```java
java.net.URL imgUrl =
        getClass().getResource(resourcePath);
```

ဒီ Line က အရမ်းအရေးကြီးပါတယ်။

## getClass()

ဆိုတာ

လက်ရှိ Class ကိုဆိုလိုတာ။

ဥပမာ

```java
public class Home
```

ဆိုရင်

```java
getClass()
```

=

```java
Home.class
```

---

## getResource()

Resource Folder ထဲက File ကိုရှာတယ်။

ဥပမာ

Project

```text
src
 ├── icons
 │      user.png
 │      home.png
```

ခေါ်ရင်

```java
getResource("/icons/user.png");
```

Java က

```text
icons/user.png
```

ကိုရှာမယ်။

တွေ့ရင်

```java
URL
```

Return ပြန်ပေးတယ်။

ဥပမာ

```text
file:/C:/Project/build/classes/icons/user.png
```

---

Diagram

```text
"/icons/user.png"

        │

        ▼

getResource()

        │

        ▼

URL
```

---

# Step 2

```java
Image img =
    new ImageIcon(imgUrl).getImage();
```

ဒီမှာ

URL ကနေ

Image Object ပြောင်းလိုက်တာ။

Flow

```text
URL

↓

ImageIcon

↓

Image
```

ဘာကြောင့် Image လိုတာလဲ?

ဘာဖြစ်လို့လဲဆိုတော့

Graphics2D က

```java
drawImage()
```

လုပ်နိုင်ဖို့

Image Object လိုတယ်။

---

# Step 3

```java
BufferedImage resizedImg =
        new BufferedImage(
            width,
            height,
            BufferedImage.TYPE_INT_ARGB
        );
```

ဒီ Line က

Image အသစ်တစ်ခုဖန်တီးတာ။

ဒါပေမယ့်

Blank Image ဖြစ်တယ်။

ဥပမာ

```java
width = 100

height =100
```

ဆိုရင်

Memory ထဲမှာ

```text
□□□□□□□□□□

□□□□□□□□□□

□□□□□□□□□□
```

လို Empty Canvas တစ်ခုဆောက်လိုက်တာ။

---

## TYPE_INT_ARGB

ဒီ Format က

```text
A

R

G

B
```

ဖြစ်တယ်။

A

↓

Alpha

Transparency

R

↓

Red

G

↓

Green

B

↓

Blue

ဒါကြောင့်

PNG Transparency မပျောက်ဘူး။

ဥပမာ

```text
Transparent Background
```

ကို ထိန်းထားနိုင်တယ်။

---

# Step 4

```java
Graphics2D g2 =
        resizedImg.createGraphics();
```

ဒီ Line က

BufferedImage ပေါ်မှာ

ဆွဲဖို့ Pen ယူတာနဲ့တူတယ်။

Diagram

```text
Blank Image

□□□□□□□□

□□□□□□□□

      │

createGraphics()

      │

      ▼

Graphics2D
```

Graphics2D က

- Draw Image
    
- Draw Text
    
- Draw Shape
    
- Rotate
    
- Scale
    

အကုန်လုပ်နိုင်တယ်။

---

# Step 5

```java
g2.setRenderingHint(
    RenderingHints.KEY_INTERPOLATION,
    RenderingHints.VALUE_INTERPOLATION_BICUBIC
);
```

ဒီအပိုင်းက

Image Quality ကို ထိန်းတာ။

---

Interpolation ဆိုတာဘာလဲ?

Image ကို

```text
32x32
```

ကနေ

```text
128x128
```

ချဲ့လိုက်ရင်

Pixel တွေကြီးသွားတယ်။

ဥပမာ

Original

```text
■■■■
■■■■
■■■■
```

Nearest Neighbor

```text
■■■■■■■■

■■■■■■■■

■■■■■■■■
```

Pixel တွေ

တုံးတုံးကြီးဖြစ်တယ်။

---

Bicubic

```text
■■▒▒░░

▒▒▒▒░░

░░▒▒▒▒
```

Pixel တွေကို

အနီးအနား Pixel တွေနဲ့

တွက်ချက်ပြီး

Smooth ဖြစ်အောင်လုပ်တယ်။

---

ဒါကြောင့်

Blur မဖြစ်ဘဲ

Professional Quality ရတယ်။

---

# Step 6

```java
g2.setRenderingHint(
    RenderingHints.KEY_RENDERING,
    RenderingHints.VALUE_RENDER_QUALITY
);
```

ဒီ Hint က

Java ကို

```text
Speed မဦးစားပေးနဲ့

Quality ကိုဦးစားပေး
```

လို့ ပြောတာ။

Java မှာ

Rendering Mode

၂ ခုရှိတယ်။

```text
Fast
```

↓

Performance

```text
Quality
```

↓

ပိုလှ

ဒီ Method က

Quality ကိုရွေးထားတာ။

---

# Step 7

```java
g2.setRenderingHint(
    RenderingHints.KEY_ANTIALIASING,
    RenderingHints.VALUE_ANTIALIAS_ON
);
```

Anti-Aliasing ဆိုတာ

Edge တွေကို

Smooth လုပ်တာ။

မဖွင့်ထားရင်

```text
########
########
########
```

လို

စောင်းမျဉ်းတွေမှာ လှေကားထစ်လို (Jagged) ဖြစ်နိုင်တယ်။

ဖွင့်ထားရင်

```text
██████

▓████▓

▒▓██▓▒
```

လို

အနားသတ်တွေကို ပိုချောမွေ့စေတယ်။

> မှတ်ချက် - Anti-Aliasing က Shape နဲ့ Text တွေအတွက် အကျိုးသက်ရောက်မှု အများဆုံးရှိပြီး၊ Image Resize Quality ကိုတော့ `KEY_INTERPOLATION` က အဓိက ထိန်းပေးပါတယ်။

---

# Step 8

```java
g2.drawImage(
        img,
        0,
        0,
        width,
        height,
        null
);
```

ဒီ Line က

Original Image ကို

အသစ်ဖန်တီးထားတဲ့

BufferedImage ထဲကို

Resize လုပ်ပြီး ဆွဲထည့်တာ။

Diagram

Original

```text
500 x 500
```

↓

drawImage()

↓

```text
100 x 100
```

ဒါက တကယ် Resize ဖြစ်တဲ့နေရာပါ။

---

Parameter တွေ

```java
img
```

ဆွဲမယ့် Image

---

```java
0
```

X Position

---

```java
0
```

Y Position

---

```java
width
```

အသစ် Width

---

```java
height
```

အသစ် Height

---

```java
null
```

ImageObserver

အများအားဖြင့်

null ပဲသုံးတယ်။

---

# Step 9

```java
g2.dispose();
```

Graphics Object ကို

Memory ကနေ Release လုပ်တာ။

ဥပမာ

Pen ငှားထားတာကို

ပြန်အပ်တာနဲ့တူတယ်။

မလုပ်ရင်

Memory Leak ဖြစ်နိုင်တယ်။

---

# Step 10

```java
return new ImageIcon(resizedImg);
```

Resize လုပ်ပြီးသား

BufferedImage ကို

ImageIcon ပြောင်းပြီး Return ပြန်ပေးတယ်။

Flow

```text
BufferedImage

↓

ImageIcon

↓

JLabel

JButton
```

ဥပမာ

```java
JLabel label = new JLabel();

label.setIcon(
    getHighQualityPNG(
        "/icons/user.png",
        80,
        80
    )
);
```

---

# catch Block

```java
catch (Exception e)
```

Image မတွေ့ရင်

ဥပမာ

```java
"/icons/abc.png"
```

မရှိဘူးဆိုရင်

```java
System.err.println(
    "Could not load icon path: "
        + resourcePath
);
```

Console မှာ

```text
Could not load icon path:
/icons/abc.png
```

ထုတ်မယ်။

ပြီးရင်

```java
return null;
```

ပြန်မယ်။

---

# Method တစ်ခုလုံး Flow

```text
User calls

getHighQualityPNG("/icons/user.png",100,100)

                │
                ▼
        getResource()
                │
                ▼
          Load Image
                │
                ▼
     Create BufferedImage
                │
                ▼
      Create Graphics2D
                │
                ▼
 Enable High Quality Rendering
                │
                ▼
 Resize using drawImage()
                │
                ▼
      Dispose Graphics
                │
                ▼
 Return ImageIcon
```

## ဒီ Method ရဲ့ အားသာချက်များ

- **Resource Folder** ထဲက Image ကို လုံခြုံစွာ Load လုပ်နိုင်တယ်။
    
- **PNG Transparency (Alpha Channel)** ကို `TYPE_INT_ARGB` ကြောင့် မပျောက်ဘဲ ထိန်းထားနိုင်တယ်။
    
- `BICUBIC` Interpolation ကြောင့် Resize လုပ်တဲ့အခါ ပုံအရည်အသွေးက `Image.getScaledInstance()` ထက် ပိုကောင်းလေ့ရှိတယ်။
    
- `Graphics2D` နဲ့ Rendering Hints တွေကို အသုံးပြုထားလို့ High Quality Output ရရှိတယ်။
    
- `ImageIcon` ကို Return ပြန်ပေးတဲ့အတွက် `JLabel`, `JButton`, `JMenuItem` စတဲ့ Swing Components တွေမှာ ချက်ချင်း အသုံးပြုနိုင်တယ်။
    

**နောက်ထပ် အကြံပြုချက်** အနေနဲ့ `imgUrl == null` ကို အရင်စစ်ပြီးမှ `new ImageIcon(imgUrl)` လုပ်တာက ပိုလုံခြုံပါတယ်။ ဒါ့အပြင် `catch (IOException | IllegalArgumentException e)` လိုမျိုး လိုအပ်တဲ့ Exception အမျိုးအစားတွေကိုသာ ဖမ်းတာကလည်း Code Quality ပိုကောင်းစေပါတယ်။


---


ဒီ Method က **Text တစ်ခုကို QR Code Image (BufferedImage) အဖြစ် ပြောင်းပေးတဲ့ Method** ဖြစ်ပါတယ်။

ဒီ Code က **ZXing (Zebra Crossing)** Library ကို အသုံးပြုထားတာဖြစ်ပြီး Java Swing Project တွေမှာ QR Code Generate လုပ်တဲ့အခါ အများဆုံးအသုံးပြုတဲ့ Library တစ်ခုပါ။

အရင်ဆုံး Method ရဲ့ Flow ကိုကြည့်ရအောင်။

```text
User Input (Text)
        │
        ▼
QRCodeWriter
        │
        ▼
Encode Text
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
        ▼
JLabel / Save as PNG
```

---

# Method Declaration

```java
public BufferedImage generateQRCodeImage(String text, int width, int height) throws Exception
```

ဒီ Method မှာ

Parameter (၃) ခုရှိပါတယ်။

```java
String text
```

QR Code ထဲမှာ သိမ်းမယ့် Data

ဥပမာ

```text
https://google.com
```

ဒါမှမဟုတ်

```text
123456789
```

ဒါမှမဟုတ်

```text
Student ID : 2025001
```

QR Code Scan လုပ်လိုက်ရင် ဒီ Text ကို ပြန်ရမယ်။

---

ဒုတိယ Parameter

```java
int width
```

QR Code ရဲ့ Width

ဥပမာ

```text
300
```

---

တတိယ Parameter

```java
int height
```

QR Code ရဲ့ Height

ဥပမာ

```text
300
```

---

Return Type

```java
BufferedImage
```

ဒါက

QR Code Image ကို Return ပြန်ပေးတာ။

Swing မှာ

```java
JLabel
```

ထဲထည့်နိုင်တယ်။

ဒါမှမဟုတ်

PNG File အဖြစ် Save လုပ်နိုင်တယ်။

---

# throws Exception

```java
throws Exception
```

ဘာကိုဆိုလိုတာလဲ?

QR Generate လုပ်တဲ့အချိန်

ဥပမာ

- Invalid Data
    
- Width = 0
    
- Height = 0
    

ဖြစ်ရင် Error တက်နိုင်တယ်။

ဒီ Method က

Error ကို မကိုင်ဘူး။

Method ကိုခေါ်တဲ့သူက Handle လုပ်ရမယ်။

ဥပမာ

```java
try{
    generateQRCodeImage(...);
}
catch(Exception e){
}
```

---

# Step 1

```java
QRCodeWriter qrCodeWriter = new QRCodeWriter();
```

ဒီ Line က

QR Code Generator Object တစ်ခုဖန်တီးတာ။

Diagram

```text
QRCodeWriter

↓

Encoder
```

ဒီ Object က

Text ကို

QR Code Pattern အဖြစ် ပြောင်းပေးတယ်။

---

ဥပမာ

Input

```text
Hello
```

QRCodeWriter က

```text
████ █ ███

██  ██ ███

██████ ███
```

လို Pattern တွေတွက်ပေးတယ်။

---

# Step 2

```java
BitMatrix bitMatrix =
    qrCodeWriter.encode(
        text,
        BarcodeFormat.QR_CODE,
        width,
        height
    );
```

ဒီ Line က

Method တစ်ခုလုံးရဲ့ အရေးကြီးဆုံးအပိုင်းပါ။

---

## encode()

```java
encode(...)
```

ဆိုတာ

Text ကို

QR Code Data အဖြစ် Encode လုပ်တာ။

---

Parameter (၄) ခုရှိတယ်။

---

### Parameter 1

```java
text
```

ဥပမာ

```text
Hello World
```

---

### Parameter 2

```java
BarcodeFormat.QR_CODE
```

ဘာ Barcode အမျိုးအစား Generate လုပ်မလဲ။

ZXing က

Barcode အမျိုးအစားအများကြီးရှိတယ်။

ဥပမာ

```text
QR_CODE

CODE_128

EAN_13

PDF_417

DATA_MATRIX

AZTEC
```

ဒီမှာ

```java
BarcodeFormat.QR_CODE
```

သုံးထားလို့

QR Code Generate လုပ်တာ။

---

### Parameter 3

```java
width
```

ဥပမာ

```text
300
```

---

### Parameter 4

```java
height
```

ဥပမာ

```text
300
```

---

ပြီးရင်

Return ပြန်လာတာက

```java
BitMatrix
```

---

# BitMatrix ဆိုတာဘာလဲ?

ဒီဟာကို Beginner တွေ အများဆုံးနားမလည်ကြဘူး။

BitMatrix ဆိုတာ

Image မဟုတ်ဘူး။

Pixel Data ပဲ။

ဥပမာ

QR Code

```text
██  ██

██████

  ████
```

ကို

Memory ထဲမှာ

```text
1 1 0 0

1 1 1 1

0 0 1 1
```

လို Boolean Matrix အဖြစ် သိမ်းထားတာ။

---

ဥပမာ

```text
1 = Black

0 = White
```

Diagram

```text
BitMatrix

1 1 0 0

1 0 0 1

1 1 1 1

0 0 1 0
```

ဒီ Matrix က

Image မဟုတ်သေးဘူး။

---

# Step 3

```java
return MatrixToImageWriter.toBufferedImage(bitMatrix);
```

ဒီ Line က

BitMatrix ကို

BufferedImage ပြောင်းတာ။

Diagram

```text
BitMatrix

↓

MatrixToImageWriter

↓

BufferedImage
```

---

ဘာဖြစ်လို့ BufferedImage ပြောင်းရတာလဲ?

Swing က

```java
BitMatrix
```

ကို မပြနိုင်ဘူး။

ဒါပေမယ့်

```java
BufferedImage
```

ကိုတော့ ပြနိုင်တယ်။

---

ဥပမာ

```java
BufferedImage qr =
    generateQRCodeImage(
        "Hello",
        300,
        300
    );
```

ပြီးရင်

```java
JLabel label =
        new JLabel(
            new ImageIcon(qr)
        );
```

ဒီလို UI မှာ ပြနိုင်တယ်။

---

# MatrixToImageWriter ဘာလုပ်တာလဲ?

သူက

ဒီလို

```text
1 1 0 0

1 0 1 0

1 1 1 0
```

ကို

ဒီလို Image ဖြစ်အောင် ဆွဲပေးတာ။

```text
██  ██

██  ██

██████
```

သူက

Loop ပတ်ပြီး

Black Pixel

White Pixel

ဆွဲပေးနေတဲ့ Library ဖြစ်တယ်။

သင်ကိုယ်တိုင် ရေးရင်

```java
for(int y=0;y<height;y++){

    for(int x=0;x<width;x++){

        if(bitMatrix.get(x,y)){

            image.setRGB(...);

        }

    }

}
```

လို Code တွေအများကြီး ရေးရမယ်။

`MatrixToImageWriter` က ဒါတွေကို အဆင်သင့်လုပ်ပေးထားတာပါ။

---

# Method တစ်ခုလုံး Flow

```text
Input Text

"Student ID : 1001"

          │

          ▼

QRCodeWriter

          │

          ▼

encode()

          │

          ▼

BitMatrix

(Black / White Data)

          │

          ▼

MatrixToImageWriter

          │

          ▼

BufferedImage

          │

          ▼

Swing JLabel

or

Save PNG File
```

---

# Example

```java
BufferedImage qr = generateQRCodeImage(
        "https://openai.com",
        250,
        250
);

JLabel label = new JLabel(new ImageIcon(qr));
```

အလုပ်လုပ်ပုံက

```text
"https://openai.com"

        │

        ▼

QR Code Generate

        │

        ▼

BufferedImage

        │

        ▼

ImageIcon

        │

        ▼

JLabel

        │

        ▼

Swing Window မှာ QR Code ပြသ
```

---

# ဒီ Method မှာ အသုံးပြုထားတဲ့ Java Concepts

| Code                                    | အလုပ်လုပ်ပုံ                                                                                     |
| --------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `QRCodeWriter`                          | QR Code Encoder Object ဖန်တီးတယ်။                                                                |
| `encode()`                              | Text ကို QR Code Data (`BitMatrix`) အဖြစ် Encode လုပ်တယ်။                                        |
| `BarcodeFormat.QR_CODE`                 | QR Code အမျိုးအစား Generate လုပ်မယ်လို့ သတ်မှတ်တယ်။                                              |
| `BitMatrix`                             | QR Code ရဲ့ Black/White Pixel Pattern ကို Boolean Matrix ပုံစံနဲ့ သိမ်းထားတယ်။                   |
| `MatrixToImageWriter.toBufferedImage()` | `BitMatrix` ကို Swing အသုံးပြုနိုင်တဲ့ `BufferedImage` အဖြစ် ပြောင်းပေးတယ်။                      |
| `BufferedImage`                         | Swing UI မှာ `ImageIcon` အဖြစ်ပြသနိုင်သလို PNG အဖြစ်လည်း Save လုပ်နိုင်တဲ့ Image Object ဖြစ်တယ်။ |

---

ဒီ Method က **QR Code ကို Generate လုပ်ပြီး PNG File အဖြစ် Computer ထဲမှာ Save လုပ်ပေးတဲ့ Method** ဖြစ်ပါတယ်။

အရင်တုန်းက `generateQRCodeImage()` Method က

```text
Text
   │
   ▼
BufferedImage
```

ကို Return ပြန်ပေးတယ်။

ဒါပေမယ့် ဒီ Method ကတော့

```text
Text
   │
   ▼
PNG File
```

အဖြစ် တိုက်ရိုက် Save လုပ်ပေးတယ်။

---

# Overall Flow

```text
User Input
      │
      ▼
Configure QR Settings
      │
      ▼
Encode Text
      │
      ▼
BitMatrix
      │
      ▼
Check Folder Exists?
      │
      ▼
Create Folder (if needed)
      │
      ▼
Save PNG File
```

---

# Method Declaration

```java
public void saveQRCodeToFile(String data,
                             String filePath,
                             int width,
                             int height)
        throws Exception
```

## Parameter (၁)

```java
String data
```

QR Code ထဲမှာ သိမ်းမယ့် Data

ဥပမာ

```text
Student ID : 2025001
```

ဒါမှမဟုတ်

```text
https://google.com
```

---

## Parameter (၂)

```java
String filePath
```

QR Code သိမ်းမယ့် File Path

ဥပမာ

```text
C:/QR/student001.png
```

ဒါမှမဟုတ်

```text
D:/Images/qr.png
```

---

## Parameter (၃)

```java
int width
```

QR Code Width

ဥပမာ

```text
300
```

---

## Parameter (၄)

```java
int height
```

QR Code Height

ဥပမာ

```text
300
```

---

## Return Type

```java
void
```

ဘာမှ Return မပြန်ဘူး။

သူ့အလုပ်က

File Save လုပ်ပြီး

ပြီးဆုံးသွားတာ။

---

# throws Exception

```java
throws Exception
```

File Save လုပ်တဲ့အချိန်

ဥပမာ

- Permission မရှိ
    
- Disk Full
    
- Invalid Path
    

ဖြစ်နိုင်လို့

Exception ကို

Caller ဆီ ပြန်ပို့ထားတာ။

---

# Step 1

```java
Map<EncodeHintType, Object> hints = new HashMap<>();
```

ဒီ Line က

QR Code Settings သိမ်းမယ့် Map တစ်ခုဆောက်တာ။

Diagram

```text
HashMap

+-----------------------------+
| CHARACTER_SET -> UTF-8      |
| ERROR_CORRECTION -> H       |
| MARGIN -> 1                 |
+-----------------------------+
```

---

## Map ဆိုတာဘာလဲ?

Map ဆိုတာ

**Key → Value**

အတွဲလိုက် သိမ်းတဲ့ Collection ဖြစ်တယ်။

ဥပမာ

```text
Name → Aung Aung

Age → 20

City → Yangon
```

ဒီမှာလည်း

```text
EncodeHintType

↓

Setting Value
```

ဖြစ်တယ်။

---

# Step 2

```java
hints.put(
    EncodeHintType.CHARACTER_SET,
    "UTF-8"
);
```

ဒီ Setting က

QR Code ထဲမှာ

စာတွေကို

UTF-8 Encoding နဲ့ သိမ်းမယ်။

---

UTF-8 ဘာကြောင့်လိုတာလဲ?

ဥပမာ

```text
မြန်မာ
```

ဒါမှမဟုတ်

```text
日本
```

ဒါမှမဟုတ်

```text
한국
```

Unicode စာတွေ

မှန်မှန်ဖတ်နိုင်အောင်။

UTF-8 မသုံးရင်

```text
?????
```

ဖြစ်နိုင်တယ်။

---

# Step 3

```java
hints.put(
    EncodeHintType.ERROR_CORRECTION,
    ErrorCorrectionLevel.H
);
```

ဒီ Setting က

QR Code Damage ဖြစ်ရင်

ဘယ်လောက် Recovery လုပ်နိုင်မလဲ

သတ်မှတ်တာ။

---

QR Code မှာ

Error Correction Level

၄ မျိုးရှိတယ်။

|Level|Recovery|
|---|---|
|L|7%|
|M|15%|
|Q|25%|
|H|30%|

---

ဒီ Code မှာ

```java
ErrorCorrectionLevel.H
```

သုံးထားတယ်။

အဓိပ္ပါယ်

QR Code ရဲ့

30% လောက်

ပျက်နေရင်တောင်

Scan လုပ်လို့ရသေးတယ်။

ဥပမာ

```text
██████████

██████

██    ██

██████████
```

Sticker ကပ်သွားတာ

ခြစ်မိတာ

နည်းနည်းစိုတာ

ဖြစ်ရင်တောင်

ဖတ်နိုင်သေးတယ်။

---

ဒါပေမယ့်

Error Correction မြင့်လေ

QR Code က

ပိုရှုပ်လာတယ်။

---

# Step 4

```java
hints.put(
    EncodeHintType.MARGIN,
    1
);
```

Margin ဆိုတာ

QR Code ပတ်လည်

White Border

ဖြစ်တယ်။

ဥပမာ

Margin = 4

```text
□□□□□□□

□█████□

□█████□

□□□□□□□
```

Margin = 1

```text
□█████□
```

ဒီ Code က

Border နည်းနည်းပဲထားတယ်။

---

# Step 5

```java
BitMatrix bitMatrix =
        new MultiFormatWriter().encode(
```

ဒီ Line က

Text ကို

QR Code Pattern အဖြစ်

ပြောင်းတာ။

---

## MultiFormatWriter

သူက

Barcode အမျိုးအစားအများကြီး

Generate လုပ်နိုင်တယ်။

ဥပမာ

```text
QR_CODE

CODE_128

EAN_13

PDF417

AZTEC
```

ဒီမှာ

QR Code Generate လုပ်တာ။

---

# encode()

```java
encode(
        data,
        BarcodeFormat.QR_CODE,
        width,
        height,
        hints
)
```

Parameter

### data

```text
Student001
```

---

### BarcodeFormat

```java
BarcodeFormat.QR_CODE
```

QR Code Generate

---

### width

```text
300
```

---

### height

```text
300
```

---

### hints

QR Settings

---

ပြီးရင်

Return ပြန်လာတာက

```java
BitMatrix
```

Diagram

```text
Student001

        │

        ▼

encode()

        │

        ▼

BitMatrix

1011010

0010111

1110011
```

---

# Step 6

```java
File file = new File(filePath);
```

ဒီမှာ

File Object တစ်ခုဖန်တီးတယ်။

ဥပမာ

```text
C:/QR/student001.png
```

ဒါပေမယ့်

ဒီအချိန် File မဖန်တီးရသေးဘူး။

File Path ကို ကိုယ်စားပြုတဲ့ Object ပဲရှိသေးတယ်။

---

# Step 7

```java
File parentDir = file.getParentFile();
```

ဒီ Line က

Folder ကိုယူတာ။

ဥပမာ

```text
C:/QR/student001.png
```

Parent Folder

```text
C:/QR
```

---

# Step 8

```java
if (parentDir != null &&
    !parentDir.exists())
```

ဒီမှာ

Folder ရှိလား

စစ်တယ်။

ဥပမာ

```text
C:/QR
```

မရှိရင်

---

# Step 9

```java
parentDir.mkdirs();
```

Folder ဆောက်တယ်။

ဥပမာ

မရှိသေးရင်

```text
C:
 └── QR
```

ကို အလိုအလျောက်ဖန်တီးပေးတယ်။

`mkdirs()` က

Intermediate Folder တွေပါ

ဆောက်ပေးနိုင်တယ်။

ဥပမာ

```text
C:/Images/QR/Student
```

အားလုံး မရှိရင်

တစ်ခါတည်းဆောက်ပေးတယ်။

---

# Step 10

```java
Path destination =
        file.toPath();
```

File ကို

Java NIO

Path Object ပြောင်းတာ။

Diagram

```text
File

↓

Path
```

ဘာကြောင့်?

`MatrixToImageWriter`

က

Path ကိုလိုချင်လို့။

---

# Step 11

```java
MatrixToImageWriter.writeToPath(
        bitMatrix,
        "PNG",
        destination
);
```

ဒီဟာက

နောက်ဆုံး Step ဖြစ်တယ်။

သူက

BitMatrix ကို

PNG File အဖြစ်

ရေးသိမ်းပေးတယ်။

Parameter တွေက

```java
bitMatrix
```

Generate လုပ်ထားတဲ့ QR Data

---

```java
"PNG"
```

Image Format

PNG

---

```java
destination
```

သိမ်းမယ့်နေရာ

---

Diagram

```text
BitMatrix

        │

        ▼

MatrixToImageWriter

        │

        ▼

student001.png
```

---

# ဒီ Method တစ်ခုလုံး Flow

```text
Input Data

"Student001"

        │

        ▼

Create QR Settings

UTF-8

Error Correction H

Margin 1

        │

        ▼

Encode

        │

        ▼

BitMatrix

        │

        ▼

Check Folder

        │

 ┌──────┴───────┐
 │              │
 ▼              ▼
Exists?       Not Exists
 │              │
 │         mkdirs()
 │              │
 └──────┬───────┘
        ▼
Convert File → Path
        │
        ▼
Write PNG
        │
        ▼
student001.png
```

# ဒီ Method ရဲ့ အားသာချက်

- ✅ UTF-8 သုံးထားလို့ မြန်မာစာ၊ ဂျပန်စာ၊ တရုတ်စာ စတဲ့ Unicode စာသားတွေကို မှန်မှန် Encode လုပ်နိုင်တယ်။
    
- ✅ `ErrorCorrectionLevel.H` ကြောင့် QR Code ရဲ့ အစိတ်အပိုင်းတချို့ ပျက်စီးသွားရင်တောင် Scan လုပ်နိုင်တဲ့ အခွင့်အရေးပိုများတယ်။
    
- ✅ `mkdirs()` ကို အသုံးပြုထားလို့ Target Folder မရှိသေးရင်လည်း အလိုအလျောက် ဖန်တီးပေးတယ်။
    
- ✅ `MatrixToImageWriter.writeToPath()` က `BitMatrix` ကို PNG File အဖြစ် တိုက်ရိုက်ရေးသိမ်းပေးတာကြောင့် `BufferedImage` ကို အရင်ဖန်တီးပြီး `ImageIO.write()` လုပ်စရာ မလိုဘဲ Code က ပိုတိုပြီး ထိရောက်ပါတယ်။

---

ဒီ Method က **String Data တစ်ခုကို QR Code Image (`BufferedImage`) အဖြစ် Generate လုပ်ပေးတဲ့ Method** ဖြစ်ပါတယ်။

အရင် Method (`saveQRCodeToFile()`) နဲ့ မတူတာက ဒီ Method က **File မသိမ်းဘူး**။ QR Code Image ကို **Memory ထဲမှာ `BufferedImage` အဖြစ် Return ပြန်ပေးတယ်**။

---

# Method တစ်ခုလုံး Flow

```text
Input Data (String)
        │
        ▼
Create QR Settings (Hints)
        │
        ▼
Encode String
        │
        ▼
BitMatrix
        │
        ▼
Convert to BufferedImage
        │
        ▼
Return BufferedImage
```

ဥပမာ

```text
"Student ID : 1001"

        │

        ▼

████ ███ ███
███  █ █  ██
█ ████ ███ █

        │

        ▼

BufferedImage

        │

        ▼

JLabel / JButton / Save File
```

---

# Method Declaration

```java
public BufferedImage generateQRCode(
        String data,
        int width,
        int height
) throws Exception
```

ဒီ Method မှာ

Parameter (၃) ခုရှိပါတယ်။

---

## Parameter (1)

```java
String data
```

QR Code ထဲမှာ Encode လုပ်မယ့် Data ဖြစ်ပါတယ်။

ဥပမာ

```text
"Hello World"
```

ဒါမှမဟုတ်

```text
"Student ID : 2025001"
```

ဒါမှမဟုတ်

```text
"https://openai.com"
```

QR Scanner နဲ့ Scan လုပ်လိုက်ရင် ဒီ String ကို ပြန်ရမယ်။

---

## Parameter (2)

```java
int width
```

QR Code ရဲ့ Width

ဥပမာ

```text
300
```

---

## Parameter (3)

```java
int height
```

QR Code ရဲ့ Height

ဥပမာ

```text
300
```

---

## Return Type

```java
BufferedImage
```

ဒီ Method က

```text
QR Code Image
```

ကို Return ပြန်ပေးတယ်။

ဒါကို

```java
JLabel

ImageIcon

ImageIO.write()
```

စတာတွေမှာ အသုံးပြုနိုင်တယ်။

---

# throws Exception

```java
throws Exception
```

QR Generate လုပ်တဲ့အချိန်

- Width = 0
    
- Height = 0
    
- Data မမှန်
    
- ZXing Error
    

ဖြစ်ရင် Exception ကို

Method ခေါ်တဲ့နေရာဆီ ပြန်ပို့တာ။

ဥပမာ

```java
try{
    BufferedImage qr =
        generateQRCode(...);
}
catch(Exception e){
    e.printStackTrace();
}
```

---

# Step 1

```java
Map<EncodeHintType, Object> hints = new HashMap<>();
```

ဒီ Line က

QR Code Generate လုပ်တဲ့အချိန်

အသုံးပြုမယ့်

Settings တွေကို သိမ်းဖို့

HashMap တစ်ခု ဖန်တီးတာ။

Diagram

```text
HashMap

+------------------------------+
| CHARACTER_SET → UTF-8        |
| ERROR_CORRECTION → H         |
| MARGIN → 1                   |
+------------------------------+
```

---

## HashMap ဆိုတာဘာလဲ?

HashMap က

**Key → Value**

အတွဲလိုက် သိမ်းတယ်။

ဥပမာ

```text
Name

↓

Aung Aung

Age

↓

20
```

ဒီမှာတော့

```text
EncodeHintType

↓

Setting Value
```

ဖြစ်ပါတယ်။

---

# Step 2

```java
hints.put(
        EncodeHintType.CHARACTER_SET,
        "UTF-8"
);
```

ဒီ Setting က

QR Code ထဲမှာ

String ကို

UTF-8 Encoding နဲ့ Encode လုပ်မယ်လို့

ပြောတာ။

---

ဘာကြောင့် UTF-8 သုံးတာလဲ?

ဥပမာ

```text
မြန်မာစာ
```

ဒါမှမဟုတ်

```text
日本語
```

ဒါမှမဟုတ်

```text
한국어
```

Unicode Language အားလုံးကို

မှန်မှန် Encode လုပ်နိုင်တယ်။

UTF-8 မသုံးရင်

```text
?????
```

လို ဖြစ်နိုင်တယ်။

---

# Step 3

```java
hints.put(
        EncodeHintType.ERROR_CORRECTION,
        ErrorCorrectionLevel.H
);
```

ဒီ Setting က

QR Code ပျက်သွားရင်

ဘယ်လောက် Recover လုပ်နိုင်မလဲ

သတ်မှတ်တာ။

QR Code မှာ

၄ မျိုးရှိတယ်။

|Level|Recover|
|---|---|
|L|7%|
|M|15%|
|Q|25%|
|H|30%|

ဒီမှာ

```java
ErrorCorrectionLevel.H
```

သုံးထားတယ်။

အဓိပ္ပါယ်က

QR Code ရဲ့

30%

လောက်ပျက်နေရင်တောင်

Scanner က

ဖတ်နိုင်သေးတယ်။

ဥပမာ

```text
██████

██  ██

█

██████
```

Sticker ကပ်တာ

ခြစ်မိတာ

နည်းနည်းရေစိုတာ

ဖြစ်ရင်တောင်

Scan လုပ်နိုင်တယ်။

---

# Step 4

```java
hints.put(
        EncodeHintType.MARGIN,
        1
);
```

Margin ဆိုတာ

QR Code ပတ်လည်က

White Border ဖြစ်ပါတယ်။

ဥပမာ

Margin = 4

```text
□□□□□□

□████□

□████□

□□□□□□
```

Margin = 1

```text
□████□
```

ဒီ Code မှာ

Border ကို

သေးသေးပဲထားထားတယ်။

---

# Step 5

```java
BitMatrix bitMatrix =
        new MultiFormatWriter().encode(
```

ဒီ Line က

Method ရဲ့

အဓိကအလုပ်လုပ်တဲ့နေရာပါ။

---

## MultiFormatWriter ဆိုတာဘာလဲ?

ZXing Library ထဲက

Barcode Generator ဖြစ်တယ်။

သူက

QR Code တင်မဟုတ်ဘူး။

Barcode အမျိုးမျိုး Generate လုပ်နိုင်တယ်။

ဥပမာ

```text
QR_CODE

CODE_128

EAN_13

UPC_A

PDF417

AZTEC
```

ဒီမှာ

QR Code Generate လုပ်နေတယ်။

---

# encode()

```java
encode(
        data,
        BarcodeFormat.QR_CODE,
        width,
        height,
        hints
)
```

ဒီ Method မှာ

Parameter (၅) ခုရှိတယ်။

---

## Parameter 1

```java
data
```

Encode လုပ်မယ့် String

ဥပမာ

```text
Student001
```

---

## Parameter 2

```java
BarcodeFormat.QR_CODE
```

ဘယ် Barcode အမျိုးအစား

Generate လုပ်မလဲ။

ဒီမှာ

QR Code

---

## Parameter 3

```java
width
```

ဥပမာ

```text
300
```

---

## Parameter 4

```java
height
```

ဥပမာ

```text
300
```

---

## Parameter 5

```java
hints
```

QR Code Settings

---

ပြီးရင်

Return ပြန်လာတာက

```java
BitMatrix
```

---

# BitMatrix ဆိုတာဘာလဲ?

ဒီဟာကို

Beginner တွေ

အများဆုံးနားမလည်ကြဘူး။

BitMatrix က

Image မဟုတ်ဘူး။

သူက

QR Code ရဲ့

Black

White

Pattern ကို

Memory ထဲမှာ

Boolean Matrix အဖြစ် သိမ်းထားတာ။

ဥပမာ

QR Code

```text
██  ██

██████

██  ██
```

Memory ထဲမှာ

```text
1 1 0 0

1 1 1 1

1 0 1 0
```

လိုသိမ်းထားတယ်။

Diagram

```text
BitMatrix

1 1 0 1

0 1 1 0

1 1 1 1
```

---

# Step 6

```java
return MatrixToImageWriter.toBufferedImage(bitMatrix);
```

ဒီ Line က

BitMatrix ကို

BufferedImage ပြောင်းတာ။

Diagram

```text
BitMatrix

↓

MatrixToImageWriter

↓

BufferedImage
```

---

ဘာကြောင့် ပြောင်းရတာလဲ?

Swing က

```java
BitMatrix
```

ကို

တိုက်ရိုက် မပြနိုင်ဘူး။

ဒါပေမယ့်

```java
BufferedImage
```

ကို

```java
new ImageIcon(bufferedImage)
```

လုပ်ပြီး

JLabel မှာ ပြနိုင်တယ်။

ဥပမာ

```java
BufferedImage qr = generateQRCode(
        "Student001",
        250,
        250
);

JLabel label = new JLabel(new ImageIcon(qr));
```

---

# `MatrixToImageWriter` အလုပ်လုပ်ပုံ

အတွင်းမှာ အကြမ်းဖျင်းအားဖြင့် ဒီလို Logic နဲ့ အလုပ်လုပ်ပါတယ်။

```java
for (int y = 0; y < bitMatrix.getHeight(); y++) {

    for (int x = 0; x < bitMatrix.getWidth(); x++) {

        if (bitMatrix.get(x, y)) {
            // Black Pixel
        } else {
            // White Pixel
        }

    }

}
```

ဆိုလိုတာက `BitMatrix` ထဲက `(x, y)` တစ်ခုချင်းစီကို စစ်ပြီး

- `true` → အနက်ရောင် Pixel
    
- `false` → အဖြူရောင် Pixel
    

အဖြစ် `BufferedImage` ပေါ်မှာ ဆွဲပေးတာပါ။ Library က ဒီအလုပ်ကို အဆင်သင့်လုပ်ပေးထားတဲ့အတွက် သင်ကိုယ်တိုင် Pixel တွေ Loop ပတ်ရေးစရာ မလိုတော့ပါဘူး။

---

# Method တစ်ခုလုံး Flow

```text
Input Data

"Student ID : 2025001"

        │
        ▼
Create QR Settings
(UTF-8, Error Correction, Margin)
        │
        ▼
MultiFormatWriter.encode()
        │
        ▼
BitMatrix
(Black / White Pattern)
        │
        ▼
MatrixToImageWriter
        │
        ▼
BufferedImage
        │
        ▼
Return
```

# ဒီ Method ရဲ့ အားသာချက်များ

- ✅ `UTF-8` သုံးထားလို့ မြန်မာစာ၊ တရုတ်စာ၊ ဂျပန်စာ စတဲ့ Unicode စာသားတွေကို မှန်မှန် Encode လုပ်နိုင်တယ်။
    
- ✅ `ErrorCorrectionLevel.H` ကြောင့် QR Code ရဲ့ 30% လောက်ပျက်နေရင်တောင် Scan လုပ်နိုင်တဲ့ အခွင့်အရေးရှိတယ်။
    
- ✅ `BufferedImage` ကို Return ပြန်ပေးတာကြောင့် Swing UI မှာ `ImageIcon` အဖြစ်ပြနိုင်သလို `ImageIO.write()` နဲ့ PNG File အဖြစ်လည်း Save လုပ်နိုင်တယ်။
    
- ✅ `MultiFormatWriter` ကို သုံးထားတဲ့အတွက် လိုအပ်ရင် `BarcodeFormat` ကိုပြောင်းပြီး QR Code အစား CODE_128, PDF417, Data Matrix စတဲ့ အခြား Barcode အမျိုးအစားတွေကိုလည်း Generate လုပ်နိုင်ပါတယ်။