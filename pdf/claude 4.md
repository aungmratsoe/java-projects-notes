## 📘 Step 4: Line/Border ဆွဲနည်း — Divider Line ထည့်ခြင်း

### 4.1 — Line ဆွဲတဲ့ Method များ

Text ရေးတာနဲ့ မတူဘဲ line ဆွဲဖို့ဆိုရင် `beginText()`/`endText()` **အပြင်ဘက်**မှာ ဒီ method တွေသုံးရတယ်:

```java
contentStream.moveTo(startX, startY);   // pen ကို starting point ရွှေ့မယ် (မဆွဲသေးဘူး)
contentStream.lineTo(endX, endY);       // starting point ကနေ ဒီ point ထိ line ဆွဲမယ်
contentStream.stroke();                 // အထက်က path ကို actual ဆွဲပြသမယ်
```

**Logic:** `moveTo`/`lineTo` က path ကို "ရေးဆွဲ" ရုံသက်သက်ပါ — `stroke()` ခေါ်မှ page ပေါ်မှာ တကယ်ပေါ်လာမယ်။

---

### 4.2 — Receipt ထဲ Divider Line ထည့်ခြင်း

Step 3 ရဲ့ code ကို ဒီလို ထပ်ဖြည့်ပါ (header အောက်နဲ့ item list အောက်မှာ line ဆွဲမယ်):

```java
try (PDPageContentStream contentStream = new PDPageContentStream(document, page)) {

    // Header
    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 16);
    contentStream.beginText();
    contentStream.newLineAtOffset(50, 750);
    contentStream.showText("Cafe POS - Receipt");
    contentStream.endText();

    // Header အောက်မှာ line ဆွဲမယ်
    contentStream.setLineWidth(1f);
    contentStream.moveTo(50, 735);
    contentStream.lineTo(545, 735);   // page width ~595 (A4), margin 50 ခြားထားလို့ 545 ထိ
    contentStream.stroke();

    // Item list
    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA), 12);
    float yPosition = 715;
    float lineHeight = 20;
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

        yPosition -= lineHeight;
    }

    // Total အပေါ်မှာ line ထပ်ဆွဲမယ်
    yPosition -= 5;
    contentStream.moveTo(50, yPosition);
    contentStream.lineTo(545, yPosition);
    contentStream.stroke();

    // Grand Total
    yPosition -= 20;
    contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 13);
    contentStream.beginText();
    contentStream.newLineAtOffset(50, yPosition);
    contentStream.showText(String.format("Total: $%.2f", grandTotal));
    contentStream.endText();
}
```

**Key points:**

- Line ဆွဲတာနဲ့ text ရေးတာကို **ရောနှောသုံးလို့ရတယ်** — တစ်ခုနဲ့တစ်ခု block အနေနဲ့ သီးသန့်ပဲ (beginText ထဲမှာ line ဆွဲလို့မရ)
- `setLineWidth(1f)` — line thickness (point unit)
- Divider line တွေကို `yPosition` variable အတိုင်းလိုက် dynamic ဆွဲထားလို့ item အရေအတွက် ပြောင်းလဲရင်လည်း အလိုအလျောက် ညီညွတ်နေမယ်

---

### 🎯 Practice

ဒီ code ကို run ကြည့်ပြီး `receipt.pdf` ထဲမှာ header အောက်နဲ့ total အပေါ်မှာ horizontal line ၂ကြောင်း ပေါ်နေလား စစ်ပါ။

Run အောင်မြင်ရင် "ready" ပြောပါ — **Step 5** (Page overflow handling — item list များလွန်ရင် 2nd page auto-create နည်း) ကို ဆက်သင်ပေးပါမယ်။