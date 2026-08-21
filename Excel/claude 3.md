## 📘 Step 3: Data Rows ထည့်ခြင်း + Style + File Save

### 3.1 — Sample Data List ပြင်ဆင်ခြင်း

Real app မှာတော့ database/ArrayList ကနေ data လာမှာဖြစ်ပေမယ့် ခုလို sample data နဲ့ practice လုပ်ကြည့်ရအောင်:

```java
import java.util.ArrayList;
import java.util.List;

// Sample data class (POJO)
class SaleItem {
    String name;
    int quantity;
    double unitPrice;

    public SaleItem(String name, int quantity, double unitPrice) {
        this.name = name;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
    }
}
```

### 3.2 — Data Rows ကို Sheet ထဲ ထည့်ခြင်း

```java
        // Sample data list (mock data)
        List<SaleItem> items = new ArrayList<>();
        items.add(new SaleItem("Espresso", 5, 3.50));
        items.add(new SaleItem("Latte", 3, 4.00));
        items.add(new SaleItem("Croissant", 2, 2.50));

        // Row 0 က header ဖြစ်နေလို့ data က row 1 ကနေစမယ်
        int rowNum = 1;

        for (SaleItem item : items) {
            Row row = sheet.createRow(rowNum++);

            row.createCell(0).setCellValue(item.name);
            row.createCell(1).setCellValue(item.quantity);
            row.createCell(2).setCellValue(item.unitPrice);
            row.createCell(3).setCellValue(item.quantity * item.unitPrice); // Total
        }
```

**Logic:**

- `rowNum++` — row ရေတွက်ပြီး Excel row တစ်ခုချင်းစီကို auto increment လုပ်တယ်
- `setCellValue()` က overload ထားလို့ String, int, double, boolean, Date စတာတွေကို တိုက်ရိုက်ထည့်လို့ရတယ်

---

### 3.3 — Header ကို Bold Style ထည့်ခြင်း (Optional but ကောင်းတယ်)

```java
        // Header style ဖန်တီးမယ်
        CellStyle headerStyle = workbook.createCellStyle();
        Font headerFont = workbook.createFont();
        headerFont.setBold(true);
        headerStyle.setFont(headerFont);

        // headerRow ရဲ့ cell တွေအားလုံးကို style ချပေးမယ်
        for (Cell cell : headerRow) {
            cell.setCellStyle(headerStyle);
        }
```

⚠️ ဒီ code ကို **header row ဖန်တီးပြီးနောက်၊ data rows မတိုင်ခင်** ထားရမယ် (order အရေးကြီးတယ်)

---

### 3.4 — Column Width Auto-size (Optional)

```java
        for (int i = 0; i < headers.length; i++) {
            sheet.autoSizeColumn(i);
        }
```

---

### 3.5 — File ကို Save လုပ်ခြင်း (အရေးကြီးဆုံး Step)

```java
        // File output stream ဖန်တီးပြီး workbook ကို disk ပေါ်ရေးမယ်
        try (FileOutputStream fileOut = new FileOutputStream("daily_sales_report.xlsx")) {
            workbook.write(fileOut);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                workbook.close(); // memory leak မဖြစ်အောင် workbook ကို ပိတ်ပေးရမယ်
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

**⚠️ အရေးကြီးတဲ့အချက်:**

- `try-with-resources` (FileOutputStream အတွက်) သုံးထားလို့ auto-close ဖြစ်တယ်
- `workbook.close()` ကိုတော့ manual close လုပ်ပေးရမယ် (Workbook က AutoCloseable ဖြစ်ပေမယ့် finally block မှာချပေးတာက safer)

---

### 🎯 Practice

အားလုံး Step 1-3 ကို ပေါင်းပြီး run ကြည့်ပါ။ Project folder ထဲမှာ `daily_sales_report.xlsx` file ပေါ်လာရင် ဖွင့်ကြည့်ပါ — header က bold ဖြစ်နေပြီး data ၃ row ပါနေရမယ်။

Run လုပ်ပြီးရင် "ready" ပြောပါ — **Step 4** (Swing GUI ထဲက button ကနေ click လုပ်ပြီး export လုပ်နည်း, save location ကို JFileChooser နဲ့ user ကိုယ်တိုင်ရွေးခိုင်းနည်း) ကို ဆက်သင်ပေးပါမယ်။