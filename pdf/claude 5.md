## 📘 Step 5: Page Overflow Handling — 2nd Page Auto-Create

### 5.1 — ဘာလို့ လိုအပ်လဲ?

Item အရေအတွက် များလွန်ရင် `yPosition` က page အောက်ဆုံး (y=0 ဒါမှမဟုတ် margin) ကို ကျော်သွားနိုင်တယ်။ ဒါဆို text က page ပြင်ပ ရောက်သွားပြီး **မမြင်ရတော့ဘူး**။ ဒါကို check လုပ်ပြီး လိုအပ်ရင် page အသစ်ဖန်တီးရမယ်။

### 5.2 — Refactor: Multi-Page Support ထည့်ခြင်း

Content stream ကို "current page" ပေါ်မူတည်ပြီး ပြန်ဖန်တီးနိုင်အောင် logic ပြင်ရမယ်:

```java
public void exportReceipt(List<SaleItem> items, String filePath) throws IOException {
    PDDocument document = new PDDocument();

    PDPage page = new PDPage();
    document.addPage(page);
    PDPageContentStream contentStream = new PDPageContentStream(document, page);

    float margin = 50;
    float yStart = 750;
    float bottomLimit = 50;      // ဒီအောက်ကို ရောက်ရင် page ပြောင်းမယ်
    float lineHeight = 20;
    float yPosition = yStart;

    // Header
    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 16);
    contentStream.beginText();
    contentStream.newLineAtOffset(margin, yPosition);
    contentStream.showText("Cafe POS - Receipt");
    contentStream.endText();

    yPosition -= 15;
    contentStream.setLineWidth(1f);
    contentStream.moveTo(margin, yPosition);
    contentStream.lineTo(545, yPosition);
    contentStream.stroke();

    yPosition -= 20;
    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA), 12);

    double grandTotal = 0;

    for (SaleItem item : items) {

        // ⭐ Key logic: bottomLimit ကျော်မလား စစ်မယ်
        if (yPosition < bottomLimit) {
            contentStream.close();               // လက်ရှိ page ကို ပိတ်မယ်

            page = new PDPage();                 // page အသစ်ဖန်တီးမယ်
            document.addPage(page);
            contentStream = new PDPageContentStream(document, page);  // stream အသစ်ချိတ်မယ်
            contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA), 12);

            yPosition = yStart;                  // y ကို page အသစ်ရဲ့ အပေါ်ဆုံးကို ပြန်စမယ်
        }

        double lineTotal = item.quantity * item.unitPrice;
        grandTotal += lineTotal;

        String line = String.format("%-15s x%-3d $%-8.2f $%.2f",
                item.name, item.quantity, item.unitPrice, lineTotal);

        contentStream.beginText();
        contentStream.newLineAtOffset(margin, yPosition);
        contentStream.showText(line);
        contentStream.endText();

        yPosition -= lineHeight;
    }

    // Total (ဒီမှာလည်း overflow check ထပ်လိုအပ်နိုင်တယ်၊ item list အတိုကို assume လုပ်ထားတယ်)
    yPosition -= 10;
    contentStream.moveTo(margin, yPosition);
    contentStream.lineTo(545, yPosition);
    contentStream.stroke();

    yPosition -= 20;
    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 13);
    contentStream.beginText();
    contentStream.newLineAtOffset(margin, yPosition);
    contentStream.showText(String.format("Total: $%.2f", grandTotal));
    contentStream.endText();

    contentStream.close();   // နောက်ဆုံး page ကို ပိတ်မယ်

    document.save(filePath);
    document.close();
}
```

**Key points — ဂရုစိုက်ရမယ့်အချက်များ:**

- Try-with-resources **မသုံးတော့ဘူး** — ဘာလို့ဆိုတော့ content stream ကို loop ထဲမှာ manual ပိတ်/ဖွင့် ပြန်လုပ်ရလို့ (`contentStream.close()` ကို explicit ခေါ်ရတယ်)
- Page ပြောင်းတိုင်း **font ကို ပြန် set ရတယ်** — content stream အသစ်ဆိုရင် font state က reset ဖြစ်တယ်
- `if (yPosition < bottomLimit)` ကို loop ရဲ့ **အစမှာစစ်တာ** — item ရေးပြီးမှ စစ်ရင် text က page ပြင်ပ ထွက်သွားနိုင်တယ်
- `yPosition = yStart` — page အသစ်ရောက်တိုင်း y ကို အပေါ်ဆုံးကို ပြန်စ

⚠️ Production code မှာဆို header/total ကိုလည်း helper method ခွဲထုတ်ပြီး code duplication လျှော့ချသင့်ပေမယ့် concept ရှင်းအောင် ဒီနေရာမှာ inline ထားတယ်။

---

### 🎯 Summary — PDFBox Export Flow

```
PDDocument → PDPage → PDPageContentStream → beginText/showText/endText (text) 
                                          → moveTo/lineTo/stroke (line)
                                          → overflow check → new page if needed
                                          → document.save() → document.close()
```

POI (cell-based, auto layout) နဲ့ PDFBox (coordinate-based, manual layout) ရဲ့ core mindset ကွာခြားချက်ကို အခုသိပြီးဖြစ်လိမ့်မယ်။

ဆက်လက်သင်ချင်တာရှိလား — ဥပမာ:

- Image/logo ကို receipt ထဲ ထည့်နည်း
- Table (grid line ပါတဲ့) ဆွဲနည်း
- JFileChooser နဲ့ Swing button ကနေ ချိတ်နည်း (POI lesson Step 4 အတိုင်း)