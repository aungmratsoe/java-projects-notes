## 📘 Step 2: PDDocument ဖန်တီးပြီး Text ရေးနည်း

Cafe POS App ရဲ့ **Receipt** ကို PDFBox နဲ့ ထုတ်ကြည့်ရအောင်။

### 2.1 — Document & Page ဖန်တီးခြင်း

```java
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.PDPageContentStream;
import org.apache.pdfbox.pdmodel.font.PDType1Font;
import org.apache.pdfbox.pdmodel.font.Standard14Fonts;
import java.io.IOException;

public class ReceiptExporter {

    public void exportReceipt() throws IOException {
        // Step A: PDDocument object ဖန်တီးမယ် (PDF file တစ်ခုလုံးကို ကိုယ်စားပြု)
        PDDocument document = new PDDocument();

        // Step B: Page တစ်ခုဖန်တီးပြီး document ထဲ ထည့်မယ်
        PDPage page = new PDPage();
        document.addPage(page);

        // ... Step 2.2 မှာ text ရေးမယ်

    }
}
```

**မှတ်ချက်:**

- `new PDPage()` က default size (A4) နဲ့ page ဖန်တီးတယ်
- Page ကို document ထဲ `addPage()` နဲ့ ထည့်မှ visible ဖြစ်မယ်

---

### 2.2 — ContentStream နဲ့ Text ရေးခြင်း

```java
        try (PDPageContentStream contentStream = new PDPageContentStream(document, page)) {

            // Font ရွေးမယ် (PDFBox 3.x မှာ Standard14Fonts enum သုံးရတယ်)
            contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 16);

            contentStream.beginText();
            contentStream.newLineAtOffset(50, 750); // x=50, y=750 (page ရဲ့ ဘယ်ဘက်အပေါ်ဆုံးအနီး)
            contentStream.showText("Cafe POS - Receipt");
            contentStream.endText();
        }

        document.save("receipt.pdf");
        document.close();
```

**Logic — POI နဲ့ အဓိကကွာခြားချက်:**

- POI မှာဆို cell ထဲ value ထည့်ရုံပဲ (`setCellValue`) — PDFBox မှာတော့ x/y **coordinate** သတ်မှတ်ပြီး "ဆွဲ" ရတယ်
- PDF coordinate system က **bottom-left က origin (0,0)** ဖြစ်တယ် — y တန်ဖိုးများရင် အပေါ်ဘက်ကို ရွေ့တယ် (Swing/screen coordinate နဲ့ ပြောင်းပြန်!)
- `beginText()` / `endText()` ကြားထဲမှာပဲ text ရေးလို့ရတယ် — line/rectangle ဆွဲမယ်ဆိုရင် ဒီ block အပြင်မှာ ရေးရမယ်
- `PDPageContentStream` ကို try-with-resources ထဲထားတာက auto-close (stream ကို ပိတ်မှ page ပေါ်ရောက်မယ်)

---

### 🎯 Practice

`ReceiptExporter` class ဖန်တီးပြီး ဒီ code ကို `exportReceipt()` method ထဲထည့်ကြည့်ပါ။ Run လုပ်ပြီး `receipt.pdf` ဖွင့်ကြည့်ပါ — "Cafe POS - Receipt" ဆိုတဲ့ bold text ပေါ်နေရမယ်။

Run အောင်မြင်ရင် "ready" ပြောပါ — **Step 3** (loop နဲ့ item list တွေရေးနည်း — y-coordinate ကို line တစ်ကြောင်းချင်း လျှော့သွားရမည့် logic) ကို ဆက်သင်ပေးပါမယ်။