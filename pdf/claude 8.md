## 📘 Bonus Lesson: Table (Grid Line ပါ) ဆွဲနည်း

### 1 — Table ဆွဲဖို့ Strategy

PDFBox မှာ built-in table class မရှိဘူး (POI လို row/cell auto-layout မရှိ) — column position တွေကို **manual x-coordinate array** နဲ့ သတ်မှတ်ပြီး, line တွေကို grid အဖြစ် ဆွဲရတယ်။

### 2 — Column Layout ကို Define လုပ်ခြင်း

```java
// Column x-position တွေ (margin ကနေစပြီး)
float[] columnX = {50, 250, 330, 430, 545};  // 4 column အတွက် boundary 5ခု
String[] headers = {"Item", "Qty", "Price", "Total"};

float tableTop = 700;
float rowHeight = 20;
```

**Logic:** `columnX[]` က column **boundary** တွေကို ကိုယ်စားပြုတယ် — column 4 ခုအတွက် boundary 5 ခု လိုတယ် (fence-post logic: item 4ခု ခြားဖို့ fence 5ခု)

---

### 3 — Header Row + Vertical/Horizontal Line များ ဆွဲခြင်း

```java
float yPosition = tableTop;

// Header background အောက် line (top border)
contentStream.setLineWidth(1f);
contentStream.moveTo(columnX[0], yPosition);
contentStream.lineTo(columnX[columnX.length - 1], yPosition);
contentStream.stroke();

// Header text ရေးမယ်
contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA_BOLD), 11);
for (int i = 0; i < headers.length; i++) {
    contentStream.beginText();
    contentStream.newLineAtOffset(columnX[i] + 5, yPosition - 15);  // +5 padding
    contentStream.showText(headers[i]);
    contentStream.endText();
}

yPosition -= rowHeight;

// Header အောက် line (header/data separator)
contentStream.moveTo(columnX[0], yPosition);
contentStream.lineTo(columnX[columnX.length - 1], yPosition);
contentStream.stroke();
```

---

### 4 — Data Rows + Grid Lines ဆွဲခြင်း

```java
contentStream.setFont(new PDType1Font(Standard14Fonts.FontName.HELVETICA), 10);
double grandTotal = 0;

for (SaleItem item : items) {
    double lineTotal = item.quantity * item.unitPrice;
    grandTotal += lineTotal;

    String[] rowData = {
        item.name,
        String.valueOf(item.quantity),
        String.format("$%.2f", item.unitPrice),
        String.format("$%.2f", lineTotal)
    };

    // Row data ရေးမယ်
    for (int i = 0; i < rowData.length; i++) {
        contentStream.beginText();
        contentStream.newLineAtOffset(columnX[i] + 5, yPosition - 15);
        contentStream.showText(rowData[i]);
        contentStream.endText();
    }

    yPosition -= rowHeight;

    // Row အောက် horizontal line
    contentStream.moveTo(columnX[0], yPosition);
    contentStream.lineTo(columnX[columnX.length - 1], yPosition);
    contentStream.stroke();
}

// ⭐ Vertical lines ဆွဲမယ် — table height ကို သိပြီးမှသာ ဆွဲလို့ရတယ်
float tableBottom = yPosition;
for (float x : columnX) {
    contentStream.moveTo(x, tableTop);
    contentStream.lineTo(x, tableBottom);
    contentStream.stroke();
}
```

**Key points — အရေးကြီးဆုံး logic:**

- **Horizontal line** တွေကို row ရေးတိုင်း "အောက်ခြေ" မှာ ချက်ချင်းဆွဲလို့ရတယ် (loop ထဲမှာပဲ)
- **Vertical line** တွေကိုတော့ **loop ပြီးမှ တစ်ခါတည်း ဆွဲရတယ်** — ဘာလို့ဆိုတော့ table ရဲ့ အမြင့် (`tableTop` ကနေ `tableBottom` ထိ) ကို item အားလုံးရေးပြီးမှသာ သိနိုင်လို့ (loop ထဲမှာ item တစ်ခုချင်းစီအတွက် vertical line တိုတိုတွေ ဆွဲရင် junction point တွေမညီဖြစ်နိုင်)
- `columnX[i] + 5` — text ကို column boundary ကနေ 5pt padding ခြားပြီးရေးတာ (border line နဲ့ text မထိအောင်)

---

### 5 — Page Overflow နဲ့ ပေါင်းစပ်ခြင်း (Step 5 logic ကို ပြန်သုံးရင်)

Table ကို page overflow logic နဲ့ ပေါင်းချင်ရင် — loop ထဲက overflow check ပြီးရင် **vertical line ကို segment အလိုက် ခွဲဆွဲရမယ်** (page 1 ရဲ့ vertical line တွေကို page 1 ချင်းသာ ဆွဲရမယ်၊ page ကူးလို့မရဘူး)။ ဒါက advanced topic ဖြစ်လို့ — item list ကို fixed length (e.g. receipt တစ်ခုမှာ item 15ခုအောက်) လို့ assume ပြီးရင် ယာယီ ဒီ complexity ကို ကျော်ထားလို့ရတယ်။

---

### 🎯 Practice

Step 5 ရဲ့ item loop ကို ဒီ table version နဲ့ အစားထိုးကြည့်ပါ။ `receipt.pdf` ထဲမှာ Excel-style grid table (header + rows + vertical/horizontal border) ပေါ်နေရမယ်။

ဆက်သင်ချင်တာရှိလား — page-overflow + table ပေါင်းစပ်တဲ့ advanced version၊ ဒါမှမဟုတ် POI/PDFBox နှစ်ခုလုံးကို Swing button တစ်ခုတည်းကနေ (PDF/Excel ရွေးလို့ရအောင်) ချိတ်နည်း ကို ဆက်ပြောပြရမလား?