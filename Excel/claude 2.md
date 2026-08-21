## 📘 Step 2: Workbook ဖန်တီးပြီး Data ထည့်နည်း

Cafe POS App ရဲ့ **Daily Sales Report** ကို sample data (Item name, Quantity, Price) နဲ့ Excel ထုတ်ကြည့်ရအောင်။

### 2.1 — Workbook & Sheet ဖန်တီးခြင်း

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileOutputStream;
import java.io.IOException;

public class SalesReportExporter {

    public void exportSalesReport() {
        // Step A: Workbook object ဖန်တီးမယ် (.xlsx format အတွက် XSSFWorkbook သုံးမယ်)
        Workbook workbook = new XSSFWorkbook();

        // Step B: Sheet တစ်ခုဖန်တီးမယ် (tab name ပေးလို့ရတယ်)
        Sheet sheet = workbook.createSheet("Daily Sales");

        // ... Step 3 မှာ Row/Cell ထည့်မယ်

    }
}
```

**မှတ်ချက်:**

- `XSSFWorkbook` = .xlsx format (Excel 2007+) အတွက်
- `HSSFWorkbook` ဆိုတာလည်းရှိတယ် — ဒါက old .xls format အတွက် (မသုံးတော့ပါဘူး၊ XSSF ကိုပဲသုံးပါ)

---

### 2.2 — Header Row ထည့်ခြင်း

```java
        // Row 0 (ပထမဆုံး row) ကို header အဖြစ်သုံးမယ်
        Row headerRow = sheet.createRow(0);

        String[] headers = {"Item Name", "Quantity", "Unit Price", "Total"};

        for (int i = 0; i < headers.length; i++) {
            Cell cell = headerRow.createCell(i);
            cell.setCellValue(headers[i]);
        }
```

**Logic:**

- `sheet.createRow(0)` → row index 0 (Excel ရဲ့ row 1) ဖန်တီးတယ်
- `for` loop နဲ့ column တစ်ခုချင်းစီအတွက် `createCell(i)` ခေါ်ပြီး header text ထည့်တယ်
- Cell index က 0-based (0,1,2,3 = A,B,C,D column)

---

### 🎯 Practice

`SalesReportExporter` class ဖန်တီးပြီး ဒီ code ၂ခုကို `exportSalesReport()` method ထဲထည့်ကြည့်ပါ။ Compile error မရှိအောင် import တွေ မှန်မှန်ထည့်ထားလား စစ်ပါ။

ပြီးရင် "ready" ပြောပါ — Step 3 (data rows တွေထည့်ပြီး, cell format/style လုပ်နည်း, file ကို save လုပ်နည်း) ကို ဆက်သင်ပေးပါမယ်။