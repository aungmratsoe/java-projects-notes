## 📘 Step 5: Currency/Number Format ထည့်နည်း + Exception Handling ပြင်ဆင်ခြင်း

### 5.1 — ဘာလို့ Number Format လိုအပ်လဲ?

အခု Step 3 အတိုင်းဆိုရင် `unitPrice` (3.50) နဲ့ `total` ကို plain number အနေနဲ့ပဲ Excel ထဲ ပြထားတယ် — currency symbol ($) မပါဘူး၊ decimal place ထိန်းမထားဘူး။ ဒါကို `CellStyle` + `DataFormat` သုံးပြီး ပြင်မယ်။

### 5.2 — Currency Style ဖန်တီးခြင်း

```java
// Header style အောက်မှာ ဒါကိုထပ်ထည့်ပါ
CellStyle currencyStyle = workbook.createCellStyle();
DataFormat format = workbook.createDataFormat();
currencyStyle.setDataFormat(format.getFormat("$#,##0.00"));
```

**Logic:**

- `"$#,##0.00"` က Excel format code — `$` symbol, thousand separator (`,`), 2 decimal places
- ဒီ format code ကို Excel ရဲ့ "Format Cells" dialog ထဲက custom format code တွေနဲ့ တူတယ်

### 5.3 — Data Row ထည့်တဲ့နေရာမှာ Style ချပေးခြင်း

Step 4 ရဲ့ data-row loop ကို ဒီလိုပြင်ပါ:

```java
int rowNum = 1;
for (SaleItem item : items) {
    Row row = sheet.createRow(rowNum++);

    row.createCell(0).setCellValue(item.name);
    row.createCell(1).setCellValue(item.quantity);

    Cell priceCell = row.createCell(2);
    priceCell.setCellValue(item.unitPrice);
    priceCell.setCellStyle(currencyStyle);   // ← currency style ချမယ်

    Cell totalCell = row.createCell(3);
    totalCell.setCellValue(item.quantity * item.unitPrice);
    totalCell.setCellStyle(currencyStyle);   // ← ဒီမှာလည်းချမယ်
}
```

---

### 5.4 — Exception Handling ကို User-Friendly ဖြစ်အောင် ပြင်ခြင်း

Step 4 ရဲ့ `e.printStackTrace()` က developer အတွက်ပဲ အသုံးဝင်တယ်၊ user ဆီ error ဘာမှမပြဘူး။ Real app မှာ ဒီလိုပြင်ပါ:

```java
try (FileOutputStream fileOut = new FileOutputStream(filePath)) {
    workbook.write(fileOut);
    JOptionPane.showMessageDialog(null, "Export successful!\n" + filePath);
} catch (IOException e) {
    JOptionPane.showMessageDialog(null,
        "Export failed: " + e.getMessage(),
        "Error",
        JOptionPane.ERROR_MESSAGE);
} finally {
    try {
        workbook.close();
    } catch (IOException e) {
        // close error က user ကို ပြဖို့မလို — log ထားရုံပဲ
        System.err.println("Failed to close workbook: " + e.getMessage());
    }
}
```

**Logic:**

- Write error (disk full, permission denied, file already open in Excel) → user ကို popup ပြပေးတယ်
- Close error → user အတွက် အရေးမကြီးလို့ console log ပဲထားလိုက်တယ်
- "Export successful" message ကို button listener ထဲ မထားတော့ဘဲ export method အောင်မြင်ရင်ချည်း ပြသအောင် ဒီမှာထားတယ်

---

### 🎯 Summary — Apache POI Excel Export Flow

```
Workbook → Sheet → Header Row (bold style) → Data Rows (currency style) → FileOutputStream → workbook.write() → workbook.close()
```

ဒါဆိုရင် Apache POI ရဲ့ core flow (Workbook → Sheet → Row → Cell, style, save) ကို coverage ပြီးပါပြီ။

ဆက်လက်သင်ချင်တာရှိလား — ဥပမာ:

- Excel ကနေ data **ပြန်ဖတ်** နည်း (import)
- Multiple sheets (e.g. sheet တစ်ခုစီ per day/month)
- Chart/graph ထည့်နည်း

ဒါမှမဟုတ် PDF export (iText) ကို ဆက်စမလား?