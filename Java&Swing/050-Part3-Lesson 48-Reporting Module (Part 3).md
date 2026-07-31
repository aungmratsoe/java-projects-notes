# Part 3: Café POS Real Implementation Phase

# Lesson 48: Reporting Module (Part 3)

## Advanced Analytics + Profit Calculation + Excel/PDF Export

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Café POS ကို **Business Intelligence Level** သို့ တိုးမြှင့်ပါမယ်။

အခုအထိ:

```text
Product Management        ✅
Inventory                 ✅
Recipe System             ✅
Order System              ✅
Payment System            ✅
Receipt                   ✅
Sales Dashboard           ✅
JFreeChart                ✅
```

ရှိပြီးပါပြီ။

အခု Manager က မေးနိုင်တဲ့ Advanced Questions တွေကို ဖြေရှင်းပါမယ်။

---

# 1. Business Owner Questions

## Question 1

"ရောင်းအားကောင်းပေမယ့် Profit ဘယ်လောက်ရလဲ?"

Sales:

```
10,000,000 MMK
```

ဆိုတာ Profit မဟုတ်ပါ။

---

## Question 2

"Latte တစ်ခွက်ရောင်းရင် အမြတ်ဘယ်လောက်လဲ?"

Example:

Selling Price:

```
5000 MMK
```

Cost:

```
Coffee Bean 500
Milk        300
Sugar       50
----------------
Cost        850
```

Profit:

```
4150 MMK

```

---

# 2. New Reporting Architecture

Before:

```
Order
 |
 |
Sales Report

```

After:

```
                 Report Dashboard


                        |


                Analytics Service


                        |


       --------------------------------

       |              |               |

    Sales       Profit          Inventory


                        |

                  Database

```

---

# 3. New Package Structure

Create:

```
module/report


├── analytics

│
├── ProfitReport.java

│
├── ProfitService.java


├── export

│
├── ExcelExporter.java

│
└── PdfExporter.java

```

---

# 4. Profit Calculation Concept

Formula:

```
Profit = Revenue - Cost

```

Example:

```
Revenue

100,000


-

Ingredient Cost

35,000


=

Profit

65,000

```

---

# 5. Database Requirement

Need product cost information.

Create:

```sql
ALTER TABLE products

ADD COLUMN cost_price

DECIMAL(10,2);

```

---

Product:

```
products


id

name

selling_price

cost_price

```

---

Example:

|Product|Selling|Cost|
|---|---|---|
|Latte|5000|850|
|Cake|4000|1500|

---

# 6. ProfitReport Model

Create:

`ProfitReport.java`

```java
package com.cafe.pos.module.report.analytics;


import java.time.LocalDate;


public record ProfitReport(

LocalDate date,

double revenue,

double cost,

double profit

){

}

```

---

# 7. Profit Repository Query

Revenue:

```sql
SELECT

SUM(total_amount)

FROM orders

WHERE status='PAID';

```

---

Cost:

Need order_items + product:

```sql
SELECT

SUM(

oi.quantity *

p.cost_price

)


FROM order_items oi


JOIN products p


ON oi.product_id=p.id;

```

---

# 8. Profit Service

Business Logic:

```java
public class ProfitService {


public ProfitReport calculate(
double revenue,
double cost
){


double profit =
revenue - cost;


return new ProfitReport(

LocalDate.now(),

revenue,

cost,

profit

);


}

}

```

---

# 9. Profit Dashboard Card

UI:

```
+----------------+

Revenue

10,000,000

+----------------+


+----------------+

Cost

3,500,000

+----------------+


+----------------+

Profit

6,500,000

+----------------+

```

---

# 10. Expense Management

Real Café has expenses:

```
Electricity

Internet

Salary

Rent

Delivery Fee

```

Sales Profit is not final.

Need:

```
Net Profit

=

Sales Profit

-

Expenses

```

---

# 11. Expense Table

Create:

```sql
CREATE TABLE expenses
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


name VARCHAR(100),


amount DECIMAL(10,2),


expense_date DATE


);

```

---

Example:

```
Electricity

200000


Salary

1500000

```

---

# 12. Net Profit

Formula:

```
Gross Profit

-

Expense

=

Net Profit

```

Example:

```
Revenue

10,000,000


Ingredient Cost

3,000,000


Gross Profit

7,000,000


Expense

2,000,000


-----------------

Net Profit

5,000,000

```

---

# 13. Export System

Managers need:

```
Sales Report

        ↓

Excel

        ↓

Send to Accountant


```

and:

```
Monthly Report

        ↓

PDF

        ↓

Print

```

---

# 14. Excel Export Architecture

```
Report Data


     ↓


ExcelExporter


     ↓


.xlsx File

```

---

Library:

Apache POI

Maven:

```xml
<dependency>

<groupId>
org.apache.poi
</groupId>

<artifactId>
poi-ooxml
</artifactId>

<version>
5.3.0
</version>

</dependency>

```

---

# 15. Excel Export Example

```java
Workbook workbook =
new XSSFWorkbook();


Sheet sheet =
workbook
.createSheet(
"Sales Report"
);


Row row =
sheet.createRow(0);


row.createCell(0)
.setCellValue(
"Total Sales"
);


```

---

Save:

```java
FileOutputStream out =
new FileOutputStream(
"sales.xlsx"
);


workbook.write(out);

workbook.close();

```

---

# 16. PDF Export Architecture

```
Report Data


     ↓


PdfExporter


     ↓


PDF File

```

---

Library:

iText / OpenPDF

Maven Example:

```xml
<dependency>

<groupId>
com.github.librepdf
</groupId>

<artifactId>
openpdf
</artifactId>

<version>
2.0.3
</version>

</dependency>

```

---

# 17. PDF Report Example

Generated:

```
========================

       CAFÉ REPORT


Date:

2026-07-31


Sales:

10,000,000


Cost:

3,000,000


Profit:

7,000,000


========================

```

---

# 18. Report Export Service

Create:

```java
public class ReportExportService {



public void exportExcel(
SalesReport report
){

}


public void exportPDF(
ProfitReport report
){

}


}

```

---

# 19. Complete Analytics Flow

Final:

```
Order

 ↓

Payment

 ↓

Sales Data


 ↓


Analytics Service


 ↓


--------------------------------

|              |               |

Revenue      Profit        Expense


 ↓


Dashboard


 ↓


Excel/PDF Export

```

---

# 20. Café POS Current Level

Now:

```
Authentication          ✅

Product CRUD            ✅

Inventory               ✅

Recipe Engine            ✅

Order System             ✅

Payment                  ✅

Receipt                  ✅

Discount/Tax             ✅

Sales Dashboard          ✅

Chart Analytics          ✅

Profit Analysis          ✅

Expense Tracking         🔄

Excel Export             🔄

PDF Export               🔄

```

---

# Practice Task

Implement:

1. Add cost_price column
    
2. Create ProfitReport
    
3. Create ProfitService
    
4. Create Expense Table
    
5. Add Excel Export
    
6. Add PDF Export
    

---

# Next Lesson

# Lesson 49: Reporting Module (Part 4)

## Inventory Analytics + Low Stock Alert + Purchase Recommendation

Next we build:

```
Inventory

   ↓

Stock Analysis

   ↓

Low Stock Alert

   ↓

Purchase Suggestion

   ↓

Supplier Management

```

ပြီးရင် Café POS ရဲ့ Inventory System ကို **Enterprise Level** သို့ တိုးတက်ပါမယ်။