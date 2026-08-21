## 📘 Step 4: Swing Button ကနေ Export + JFileChooser နဲ့ Save Location ရွေးနည်း

### 4.1 — Export Method ကို Reusable ဖြစ်အောင် ပြင်ခြင်း

Step 3 က code တွေကို hardcode file path အစား, **user ရွေးတဲ့ location** ကို parameter အနေနဲ့ လက်ခံအောင် ပြင်မယ်:

```java
public void exportSalesReport(List<SaleItem> items, String filePath) {
    Workbook workbook = new XSSFWorkbook();
    Sheet sheet = workbook.createSheet("Daily Sales");

    // Header row + style (Step 2, 3.3 အတိုင်း)
    Row headerRow = sheet.createRow(0);
    String[] headers = {"Item Name", "Quantity", "Unit Price", "Total"};

    CellStyle headerStyle = workbook.createCellStyle();
    Font headerFont = workbook.createFont();
    headerFont.setBold(true);
    headerStyle.setFont(headerFont);

    for (int i = 0; i < headers.length; i++) {
        Cell cell = headerRow.createCell(i);
        cell.setCellValue(headers[i]);
        cell.setCellStyle(headerStyle);
    }

    // Data rows (Step 3.2 အတိုင်း)
    int rowNum = 1;
    for (SaleItem item : items) {
        Row row = sheet.createRow(rowNum++);
        row.createCell(0).setCellValue(item.name);
        row.createCell(1).setCellValue(item.quantity);
        row.createCell(2).setCellValue(item.unitPrice);
        row.createCell(3).setCellValue(item.quantity * item.unitPrice);
    }

    for (int i = 0; i < headers.length; i++) {
        sheet.autoSizeColumn(i);
    }

    // filePath ကို parameter ကနေယူသုံးမယ် (hardcode မလုပ်တော့ဘူး)
    try (FileOutputStream fileOut = new FileOutputStream(filePath)) {
        workbook.write(fileOut);
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            workbook.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### 4.2 — JFileChooser နဲ့ Save Location ရွေးနည်း

Swing button ရဲ့ `actionPerformed` (or lambda) ထဲမှာ:

```java
JButton exportButton = new JButton("Export to Excel");

exportButton.addActionListener(e -> {
    JFileChooser fileChooser = new JFileChooser();
    fileChooser.setDialogTitle("Save Sales Report");
    fileChooser.setSelectedFile(new java.io.File("daily_sales_report.xlsx"));

    int userSelection = fileChooser.showSaveDialog(null);

    if (userSelection == JFileChooser.APPROVE_OPTION) {
        String filePath = fileChooser.getSelectedFile().getAbsolutePath();

        // .xlsx extension မပါရင် auto ထည့်ပေးမယ်
        if (!filePath.toLowerCase().endsWith(".xlsx")) {
            filePath += ".xlsx";
        }

        List<SaleItem> items = getTodaySales(); // သင့် DB/logic ကနေ data ယူတဲ့ method

        SalesReportExporter exporter = new SalesReportExporter();
        exporter.exportSalesReport(items, filePath);

        JOptionPane.showMessageDialog(null, "Export successful!\n" + filePath);
    }
});
```

**Logic:**

- `JFileChooser.showSaveDialog(null)` → save dialog ပြပြီး user ရွေးတဲ့ file path ကို return
- `APPROVE_OPTION` check လုပ်တာက user "Save" နှိပ်မှလား "Cancel" နှိပ်မှလား စစ်တာ
- `.xlsx` extension auto append လုပ်တာက user manually မထည့်ရင် error မဖြစ်အောင်

---

### 🎯 Practice

ဒီ button code ကို သင့် POS App ရဲ့ report screen ထဲ ထည့်ကြည့်ပါ (`getTodaySales()` method ကိုတော့ သင့် logic နဲ့ ပြင်ပေးရမယ်)။

Run လုပ်ပြီး file save dialog ပေါ်လာလား၊ export ပြီးရင် Excel file ပွင့်ကြည့်ကြည့်ပါ။

ပြီးရင် "ready" ပြောပါ — **Step 5** (Exception handling ကောင်းအောင် ပြင်နည်း + Try-with-resources ပြင် refine လုပ်နည်း, ဒါမှမဟုတ် Excel formula/currency format ထည့်နည်း) ကို ဆက်သင်ပေးမယ်။