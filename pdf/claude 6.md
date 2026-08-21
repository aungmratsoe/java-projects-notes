## 📘 Bonus Lesson: Image/Logo ကို Receipt ထဲ ထည့်နည်း

### 1 — PDImageXObject Class

PDFBox မှာ image ထည့်ဖို့ `PDImageXObject` သုံးရတယ် — PNG, JPG နှစ်မျိုးလုံး support လုပ်တယ်:

```java
import org.apache.pdfbox.pdmodel.graphics.image.PDImageXObject;
import java.io.File;
```

### 2 — Image ကို Load ပြီး Draw လုပ်နည်း

```java
// Image file ကနေ PDImageXObject ဖန်တီးမယ်
PDImageXObject logo = PDImageXObject.createFromFile("logo.png", document);

// Image ကို page ပေါ်ဆွဲမယ်
contentStream.drawImage(logo, x, y, width, height);
```

### 3 — Receipt Header ထဲ Logo ထည့်ပြီး ဥပမာပြင်ခြင်း

Step 5 ရဲ့ header code ကို ဒီလိုပြင်ပါ:

```java
// Logo ကို document ဆီ တစ်ခါတည်းလုပ် (loop ထဲမှာ ထပ်ခေါ်စရာမလို)
PDImageXObject logo = PDImageXObject.createFromFile("logo.png", document);

float logoWidth = 60;
float logoHeight = 60;

// Logo ကို page ရဲ့ ဘယ်ဘက်ထိပ်မှာ ထည့်မယ်
contentStream.drawImage(logo, margin, yPosition - logoHeight + 15, logoWidth, logoHeight);

// Shop name ကို logo ဘေးမှာ ရေးမယ် (x offset ခြားထားတယ်)
contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 16);
contentStream.beginText();
contentStream.newLineAtOffset(margin + logoWidth + 15, yPosition);
contentStream.showText("Cafe POS - Receipt");
contentStream.endText();

// Logo height အလိုက် yPosition ကို ချိန်ညှိပေးရမယ် (logo ကနေ အောက်ဆက်ရေးလို့ရအောင်)
yPosition -= (logoHeight + 5);
```

**Logic — အရေးကြီးတဲ့အချက်များ:**

- `drawImage(image, x, y, width, height)` — `x,y` က image ရဲ့ **bottom-left corner**ဖြစ်တယ် (text position logic အတိုင်းပဲ, bottom-left origin)
- Logo ကို **loop အပြင်ဘက်မှာ တစ်ခါပဲ** `createFromFile()` ခေါ်ရမယ် — page overflow အတွက် loop ထဲက page ပြောင်းတဲ့နေရာမှာ logo ကို ပြန်ထည့်ချင်ရင် (e.g. page 2 ရဲ့ header မှာလည်း logo ပြထားချင်ရင်) ဒီ `logo` object ကိုပဲ ပြန်သုံးရမယ် — ပြန်ဖန်တီးစရာမလို
- Image size (width/height) ကို logo ရဲ့ actual aspect ratio အတိုင်း ချိန်ထားရမယ် (မဟုတ်ရင် stretch/squeeze ဖြစ်နိုင်)
- `logo.png` path က relative ဆိုရင် project working directory ကို depend လုပ်တယ် — production မှာဆို `/mnt/resources/logo.png` စတဲ့ absolute path ဒါမှမဟုတ် classpath resource (`getClass().getResourceAsStream(...)`) သုံးတာက ပိုစိတ်ချရတယ်

⚠️ **JAR ထဲ Resource အဖြစ် Logo Bundle လုပ်ချင်ရင်** (production build):

```java
InputStream is = getClass().getResourceAsStream("/images/logo.png");
byte[] imageBytes = is.readAllBytes();
PDImageXObject logo = PDImageXObject.createFromByteArray(document, imageBytes, "logo");
```

---

### 🎯 Practice

`logo.png` file တစ်ခု project folder ထဲ ထည့်ပြီး Step 5 code ထဲ ဒီ logic ကို ပေါင်းထည့်ကြည့်ပါ။ Receipt ရဲ့ ထိပ်ဆုံးမှာ logo + shop name side-by-side ပေါ်နေရမယ်။

ဆက်သင်ချင်တာရှိလား — Table/grid line ဆွဲနည်း (item list ကို ပိုကောင်းအောင် column border ထည့်ချင်ရင်)၊ ဒါမှမဟုတ် JFileChooser နဲ့ button ချိတ်နည်း (POI lesson အတိုင်း) ကို ဆက်သင်ပေးရမလား?