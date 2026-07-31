# Part 3: Café POS Real Implementation Phase

# Lesson 46: Reporting Module (Part 1)

## Sales Report + Daily Revenue + Dashboard Analytics

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson ကနေ Café POS ကို **Cashier Application ကနေ Business Management System** အဖြစ် Upgrade လုပ်ပါမယ်။

အခုအထိ:

```text
Authentication              ✅

Dashboard                   ✅

Product Management          ✅

Inventory                   ✅

Recipe System               ✅

Order System                ✅

Payment System              ✅

Discount + Tax              ✅

Order History               🔄

```

ရှိပြီးပါပြီ။

Manager / Owner အတွက်လိုအပ်တဲ့:

```text
Sales Report

Daily Revenue

Monthly Revenue

Best Selling Product

Inventory Report

Profit Analysis

Dashboard Chart

```

တွေကို တည်ဆောက်ပါမယ်။

---

# 1. Reporting System Real World Purpose

Owner မေးနိုင်တဲ့ Question:

### Q1:

"ဒီနေ့ ဘယ်လောက်ရောင်းရလဲ?"

Answer:

```text
Today's Sales:

850,000 MMK

```

---

### Q2:

"ဘယ် Product အရောင်းအများဆုံးလဲ?"

Answer:

```text
1. Latte

500 cups sold


2. Burger

300 sold

```

---

### Q3:

"Stock ဘယ်လောက်ကျန်လဲ?"

Answer:

```text
Coffee Bean

LOW STOCK

```

---

# 2. Reporting Architecture

Professional Design:

```text
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

# 3. Package Structure

Create:

```text
module/report


├── model


│    SalesReport.java

│    ProductSales.java


├── repository


│    ReportRepository.java


├── service


│    ReportService.java


├── controller


│    ReportController.java


└── view


     ReportPanel.java

     ChartPanel.java

```

---

# 4. Report Module Models

## SalesReport

Purpose:

Daily summary.

Create:

```java
package com.cafe.pos.module.report.model;



import java.time.LocalDate;



public record SalesReport(


LocalDate date,


long totalOrders,


double totalSales


){

}

```

---

Example:

```text
Date:

2026-07-31


Orders:

120


Sales:

850000 MMK

```

---

# 5. Product Sales Model

For best seller.

Create:

```java
public record ProductSales(


Long productId,


String productName,


long quantitySold,


double revenue


){

}

```

---

Example:

```text
Latte

500 cups

2,500,000 MMK

```

---

# 6. Database Report Query

## Today's Revenue

SQL:

```sql
SELECT


COUNT(id),

SUM(total_amount)


FROM orders


WHERE status='PAID'


AND DATE(created_at)
=
CURRENT_DATE;

```

---

Result:

```text
COUNT

120


SUM

850000

```

---

# 7. Report Repository Interface

Create:

```java
public interface ReportRepository {



SalesReport getTodaySales();



List<ProductSales>
getBestSellingProducts();



}

```

---

# 8. Report Service

Business Layer:

Create:

```java
public class ReportService {



private final ReportRepository repository;



public ReportService(
ReportRepository repository
){

this.repository =
repository;

}




public SalesReport today(){

return repository
.getTodaySales();

}




public List<ProductSales>
bestProducts(){


return repository
.getBestSellingProducts();


}



}

```

---

# 9. Best Selling Product Query

SQL:

```sql
SELECT


p.id,


p.name,


SUM(oi.quantity),


SUM(
oi.quantity *
oi.price
)


FROM order_items oi


JOIN products p


ON oi.product_id=p.id


JOIN orders o


ON oi.order_id=o.id


WHERE o.status='PAID'


GROUP BY p.id


ORDER BY SUM(oi.quantity)
DESC


LIMIT 10;

```

---

Result:

```text
Product       Qty


Latte         500

Burger        300

Cake          250

```

---

# 10. Manager Dashboard Design

UI:

```text
+------------------------------------------------+

             CAFÉ POS DASHBOARD


+----------------+  +----------------+

| TODAY SALES    |  | TOTAL ORDERS   |

| 850,000 MMK    |  | 120            |

+----------------+  +----------------+



+-----------------------------------------------+

|             SALES CHART                       |

|                                               |

|   █                                           |

|   █       █                                   |

|   █       █       █                           |

|                                               |

+-----------------------------------------------+



Best Selling:


1. Latte

2. Burger

3. Cake



+------------------------------------------------+

```

---

# 11. ReportPanel

Create:

```java
public class ReportPanel

extends JPanel {


private JLabel salesLabel;


private JLabel orderLabel;



}

```

---

# 12. Loading Dashboard Data

Example:

```java
SalesReport report =

controller
.getTodaySales();



salesLabel
.setText(

report.totalSales()
+
" MMK"

);

```

---

# 13. Chart System

Swing မှာ Default Chart မရှိပါ။

Professional Options:

### Option 1

JFreeChart

```text
Database

 ↓

Data

 ↓

JFreeChart

 ↓

ChartPanel

```

---

### Option 2

Custom Java2D

```java
Graphics2D

drawLine()

drawRect()

```

---

Café POS အတွက်:

အသုံးများတာ:

```text
JFreeChart

```

---

# 14. Add JFreeChart Dependency

Maven:

```xml
<dependency>

<groupId>
org.jfree
</groupId>

<artifactId>
jfreechart
</artifactId>

<version>
1.5.4
</version>

</dependency>

```

---

# 15. Sales Chart Example

Daily:

```text
Mon 50000

Tue 80000

Wed 120000

Thu 90000

Fri 150000

```

Chart:

```text
Revenue


150k |        █

100k |    █   █

 50k | █


       M T W T F

```

---

# 16. Report Controller

Create:

```java
public class ReportController {



private final ReportService service;



public SalesReport today(){

return service.today();

}



public List<ProductSales>
bestProducts(){

return service.bestProducts();

}


}

```

---

# 17. Dashboard Auto Refresh

Professional:

Every 5 minutes:

```java
Timer timer =
new Timer(

300000,

e -> refresh()

);


timer.start();

```

---

# 18. Cache Report Data

Large POS:

Don't query every second.

Use:

```text
Report Cache


5 minutes


```

---

# 19. Report Types

Final System:

```text
Sales Report

        |

        |

---------------------------

|            |             |

Daily     Monthly      Yearly


```

---

# 20. Current Café POS Level

Completed:

```text
Login System              ✅

Product Management        ✅

Inventory                 ✅

Recipe Engine              ✅

Order System              ✅

Payment                   ✅

Receipt                   ✅

Discount                  ✅

Tax                       ✅

Reporting Architecture    ✅

Sales Analytics            🔄

```

---

# Practice Task

Implement:

1. SalesReport.java
    
2. ProductSales.java
    
3. ReportRepository Interface
    
4. ReportService
    
5. Today's Revenue Query
    
6. Dashboard Cards
    

---

# Next Lesson

# Lesson 47: Reporting Module (Part 2)

## JFreeChart Integration + Sales Dashboard + Revenue Analytics

Next:

```text
Database Data

       ↓

Report Service

       ↓

Charts

       ↓

Professional Dashboard

```

ကို တည်ဆောက်ပါမယ်။