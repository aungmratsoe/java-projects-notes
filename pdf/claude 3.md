## 📘 Step 3: Item List ရေးနည်း — Y-Coordinate Loop Logic

### 3.1 — Sample Receipt Items

Step 3 (POI lesson) မှာသုံးခဲ့တဲ့ `SaleItem` class ကိုပဲ ပြန်သုံးမယ်:

```java
List<SaleItem> items = new ArrayList<>();
items.add(new SaleItem("Espresso", 2, 3.50));
items.add(new SaleItem("Latte", 1, 4.00));
items.add(new SaleItem("Croissant", 3, 2.50));
```

### 3.2 — Y-Coordinate ကို Line တစ်ကြောင်းချင်း လျှော့သွားနည်း

PDF မှာ text ရေးတိုင်း **manual **ပဲ y-position ကို ချိန်ရတယ် (POI လို auto row-increment မရှိဘူး):

```java
try (PDPageContentStream contentStream = new PDPageContentStream(document, page)) {

    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 16);
    contentStream.beginText();
    contentStream.newLineAtOffset(50, 750);
    contentStream.showText("Cafe POS - Receipt");
    contentStream.endText();

    // Item list အတွက် font ပြောင်းမယ် (regular, size 12)
    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA), 12);

    float yPosition = 700;      // header အောက်ကနေ စတင်မယ်
    float lineHeight = 20;      // line တစ်ကြောင်းချင်း ဘယ်လောက်ကွာမလဲ
    double grandTotal = 0;

    for (SaleItem item : items) {
        double lineTotal = item.quantity * item.unitPrice;
        grandTotal += lineTotal;

        String line = String.format("%-15s x%-3d $%-8.2f $%.2f",
                item.name, item.quantity, item.unitPrice, lineTotal);

        contentStream.beginText();
        contentStream.newLineAtOffset(50, yPosition);
        contentStream.showText(line);
        contentStream.endText();

        yPosition -= lineHeight;   // ← key logic: ကြောင်းတိုင်း y ကို လျှော့သွားရမယ်
    }

    // Grand total ကို bold ပြန်ပြောင်းပြီးရေးမယ်
    yPosition -= 10; // spacing တစ်ခု ခြားထားမယ်
    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 13);
    contentStream.beginText();
    contentStream.newLineAtOffset(50, yPosition);
    contentStream.showText(String.format("Total: $%.2f", grandTotal));
    contentStream.endText();
}
```

**Logic — အဓိက key point:**

- `yPosition -= lineHeight` — PDF coordinate က bottom-left origin ဖြစ်လို့ **အောက်ကို ရေးချင်ရင် y ကို လျှော့ရတယ်** (Swing/HTML နဲ့ ပြောင်းပြန်)
- `beginText()`/`endText()` ကို item **တစ်ခုချင်းစီအတွက် ပြန်ခေါ်ရတယ်** — POI မှာဆို row loop ထဲက cell.setCellValue ပဲခေါ်ရတာနဲ့ ကွာတယ်
- Font ကို loop အပြင်ဘက်မှာ တစ်ခါပဲ set လိုက်ရင် ရတယ် (setFont က content stream state ကို ပြောင်းလိုက်တာ၊ text ရေးတိုင်း ပြန်ခေါ်စရာမလို)
- `String.format("%-15s...")` သုံးထားတာက column alignment လုပ်ဖို့ (monospace font မဟုတ်ရင် perfectly align မဖြစ်နိုင်ပေမယ့် receipt-style ပုံစံအတွက် လုံလောက်တယ်)

⚠️ **သတိထားရမည့်အချက်:** Item list များလွန်းရင် `yPosition` က 0 ဒါမှမဟုတ် negative ဖြစ်သွားနိုင်တယ် (page အောက်ကို ကျော်သွားတာ) — ဒါကို Step 4/5 မှာ **page overflow handling** (2nd page auto-create) သင်ပေးမယ်။

---

### 🎯 Practice

Step 2 code ထဲကို ဒီ item loop ကို ထည့်ပေါင်းပြီး run ကြည့်ပါ။ `receipt.pdf` ထဲမှာ item list ၃ခု + Total line ပေါ်နေရမယ်။

Run အောင်မြင်ရင် "ready" ပြောပါ — **Step 4** (line/border ဆွဲနည်း — `moveTo`/`lineTo`, receipt ကို visually ပိုကောင်းအောင် divider line ထည့်နည်း) ကို ဆက်သင်ပေးပါမယ်။