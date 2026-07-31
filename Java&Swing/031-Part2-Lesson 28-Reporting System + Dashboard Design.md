# Part 2: Advanced Java Swing + Café POS Project

# Lesson 28: Reporting System + Dashboard Design

## Enterprise Café POS Analytics

### (Java 25 + Swing + MVC + JDBC + MySQL + Charts + PDF/Excel Export)

ဒီ Lesson မှာ Café POS ကို **Business Intelligence System** အဖြစ် တိုးချဲ့ပါမယ်။

အခုအထိ System မှာ:

✅ Product Management  
✅ Order Management  
✅ Inventory Control  
✅ Authentication & Authorization  
✅ Audit Logging

ရှိပြီးပါပြီ။

ဒါပေမယ့် Owner / Manager အတွက် အရေးကြီးတာက:

> "Business ဘယ်လောက်ရောင်းရလဲ?"  
> "ဘယ် Product က အများဆုံးရောင်းရလဲ?"  
> "Stock ဘာတွေကုန်တော့မလဲ?"

ဆိုတဲ့ Data Analysis ဖြစ်ပါတယ်။

---

# 1. Reporting System Purpose

Normal POS:

```
Order Saved
     |
     |
Database

```

Professional POS:

```
Order Saved

     |

Analytics Engine

     |

Dashboard

     |

Business Decision

```

---

# 2. Dashboard Design

Professional Café Dashboard:

```
+------------------------------------------------+

 Café POS Dashboard


 +------------+ +------------+ +------------+

 | Today's   | | Orders     | | Profit     |

 | Sales     | | Count      | |            |

 | 850,000   | | 120        | | 250,000    |

 +------------+ +------------+ +------------+


 Sales Chart


        /\

       /  \

  ____/    \____


 Top Products


 Coffee       250

 Cake         180

 Tea          120


+------------------------------------------------+

```

---

# 3. Reporting Architecture

MVC:

```
                 Dashboard UI

                      |

               ReportController

                      |

               ReportService

                      |

              ReportRepository

                      |

                  MySQL


```

---

# 4. Database Reporting Concept

Report တွေက New Table မလိုပါဘူး။

Existing Data ကနေ Query လုပ်ပါတယ်။

Example:

```
orders

order_items

products

payments

```

---

# 5. Sales Report Model

Create:

```
model/report

SalesReport.java

```

---

Java 25 Record:

```java
public record SalesReport(

LocalDate date,

BigDecimal totalSales,

long totalOrders

){

}

```

---

# 6. Dashboard Summary Model

```java
public record DashboardSummary(

BigDecimal todaySales,

long todayOrders,

BigDecimal todayProfit,

long lowStockCount

){

}

```

---

# 7. Report Repository

Interface:

```java
public interface ReportRepository {


DashboardSummary getDashboard();



List<SalesReport> getDailySales(
LocalDate start,
LocalDate end
);



List<ProductSalesReport> getTopProducts();



}

```

---

# 8. Today's Sales Query

SQL:

```sql
SELECT

SUM(total_amount)


FROM orders


WHERE DATE(created_at)=CURRENT_DATE();

```

---

Example:

Database:

```
orders


id | total_amount

1     10000

2     15000

3      8000

```

Result:

```
33000

```

---

# 9. Today's Order Count

SQL:

```sql
SELECT COUNT(*)

FROM orders

WHERE DATE(created_at)
=
CURRENT_DATE();

```

---

# 10. Dashboard Service

Business Layer:

```java
public class ReportService {


private final ReportRepository repository;



public DashboardSummary dashboard(){


return repository.getDashboard();


}


}

```

---

# 11. Dashboard Controller

```java
public class ReportController {


private final ReportService service;



public DashboardSummary loadDashboard(){


return service.dashboard();


}

}

```

---

# 12. Swing Dashboard Panel

Structure:

```
view/dashboard


DashboardPanel.java

CardPanel.java

ChartPanel.java

```

---

# 13. Dashboard Layout

Use:

```java
setLayout(
new BorderLayout()
);

```

---

North:

```
Cards

```

Center:

```
Charts

```

South:

```
Tables

```

---

# 14. Summary Cards

Reusable Component:

```
CardPanel

```

---

Example:

```java
public class CardPanel
extends JPanel{


private JLabel valueLabel;



public void setValue(
String value
){

valueLabel.setText(value);

}


}

```

---

Usage:

```java
salesCard.setValue(
"850,000 MMK"
);

```

---

# 15. Sales Chart

Need Library:

## JFreeChart

Maven:

```xml
<dependency>

<groupId>org.jfree</groupId>

<artifactId>jfreechart</artifactId>

<version>1.5.4</version>

</dependency>

```

---

# 16. Line Chart Example

Data:

```
Mon  20000

Tue  30000

Wed  50000

Thu  40000

```

Chart:

```

Sales

 |

50k       *

40k           *

30k    *

20k *

    Mon Tue Wed Thu


```

---

# 17. Create Chart Service

```java
public class ChartService {


public JFreeChart createSalesChart(
List<SalesReport> data
){


return ChartFactory
.createLineChart(

"Daily Sales",

"Date",

"Amount",

dataset

);


}


}

```

---

# 18. Top Product Report

Question:

"Which product sells most?"

SQL:

```sql
SELECT

p.name,

SUM(oi.quantity) total


FROM order_items oi


JOIN products p

ON oi.product_id=p.id


GROUP BY p.id


ORDER BY total DESC


LIMIT 10;

```

---

Result:

```
Coffee     500

Cake       300

Tea        200

```

---

# 19. Product Sales Model

```java
public record ProductSalesReport(

String productName,

long quantitySold,

BigDecimal revenue

){

}

```

---

# 20. Report JTable

Example:

```
+--------------------------------+

Top Selling Products


Product     Qty     Revenue


Coffee      500     2500000

Cake        300     1500000


+--------------------------------+

```

---

# 21. Date Range Filter

Manager wants:

```
From:

01-07-2026


To:

31-07-2026


[Search]

```

---

Java:

```java
LocalDate start =
startPicker.getDate();


LocalDate end =
endPicker.getDate();

```

---

# 22. Export PDF Report

Architecture:

```
Report Data

     |

PDF Service

     |

PDF File

```

---

Library:

## Apache PDFBox

Maven:

```xml
<dependency>

<groupId>org.apache.pdfbox</groupId>

<artifactId>pdfbox</artifactId>

<version>3.0.5</version>

</dependency>

```

---

Example:

```java
public void exportPDF(
SalesReport report
){


PDDocument doc =
new PDDocument();


}


```

---

# 23. Export Excel

Use:

## Apache POI

Maven:

```xml
<dependency>

<groupId>org.apache.poi</groupId>

<artifactId>poi-ooxml</artifactId>

<version>5.4.1</version>

</dependency>

```

---

Excel:

```
Date        Sales

01/07       50000

02/07       70000


```

---

# 24. Report Threading

Reports can be heavy:

Bad:

```
Click Report

      |

Database Query

      |

UI Freeze

```

---

Correct:

```
Swing EDT

    |

Virtual Thread

    |

Database

```

---

Java 25:

```java
Thread.startVirtualThread(
()->{


loadReport();


SwingUtilities.invokeLater(
()->updateUI()
);


});

```

---

# 25. Query Optimization

Large database:

```
orders

10 million rows

```

Need Index.

---

Example:

```sql
CREATE INDEX idx_order_date

ON orders(created_at);

```

---

Product sales:

```sql
CREATE INDEX idx_order_product

ON order_items(product_id);

```

---

# 26. Dashboard Refresh

Option:

Manual:

```
[Refresh]

```

Automatic:

```java
ScheduledExecutorService scheduler =
Executors.newScheduledThreadPool(1);


scheduler.scheduleAtFixedRate(

this::refreshDashboard,

0,

5,

TimeUnit.MINUTES

);

```

---

# 27. Audit Report

Admin can see:

```
User Activity


Admin

Deleted Product

10:30


Cashier

Created Order

10:35


```

---

Query:

```sql
SELECT *

FROM audit_logs

ORDER BY created_at DESC;

```

---

# 28. Complete Reporting Package

Structure:

```
report


├── DashboardPanel.java

├── ReportController.java

├── ReportService.java

├── ReportRepository.java

├── SalesReport.java

├── ChartService.java

├── PdfExportService.java

└── ExcelExportService.java


```

---

# 29. Professional Dashboard Features

Add:

✅ Sales KPI Cards

✅ Sales Charts

✅ Top Products

✅ Low Stock Warning

✅ Profit Analysis

✅ PDF Export

✅ Excel Export

✅ Date Filter

✅ Auto Refresh

---

# 30. Final Café POS Architecture

Now:

```
                    Swing UI

                       |

              ------------------

              |                |

          Dashboard        POS Modules


                       |

                  Controllers


                       |

                   Services


                       |

                 Repositories


                       |

                    MySQL


```

---

# Practice Task

Build:

## Dashboard Module

Implement:

1. Dashboard Frame
    
2. Summary Cards
    
3. Sales Chart
    
4. Top Product Table
    
5. Date Filter
    
6. PDF Export
    
7. Excel Export
    
8. Report Service
    
9. Report Repository
    

---

# Lesson 28 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Enterprise Dashboard Design  
✅ Reporting Architecture  
✅ KPI Cards  
✅ Sales Analytics  
✅ JFreeChart Integration  
✅ Top Product Query  
✅ PDF Export Architecture  
✅ Excel Export Architecture  
✅ Query Optimization  
✅ Background Refresh  
✅ Report MVC Structure

---

# Next Lesson

# Lesson 29: Café POS Final Architecture Upgrade

## Enterprise Application Design & Deployment

Topics:

- Complete Project Refactoring
    
- Package Architecture
    
- Design Patterns
    
- Logging System
    
- Configuration Management
    
- Application Packaging
    
- Database Migration
    
- Backup & Restore
    
- Production Deployment
    

ဒီ Lesson မှာ Café POS ကို **Senior Java Developer Level Production Application Structure** အဖြစ် ပြင်ဆင်ပါမယ်။