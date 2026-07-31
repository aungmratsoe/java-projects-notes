# Part 3: Café POS Real Implementation Phase

# Lesson 47: Reporting Module (Part 2)

## JFreeChart Integration + Sales Dashboard + Revenue Analytics

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Café POS Dashboard ကို **Professional Management Dashboard** အဖြစ် ပြောင်းပါမယ်။

အခုအထိ:

```text
Sales Report Model        ✅
Product Sales Model       ✅
Report Repository         ✅
Report Service            ✅
Dashboard Concept         ✅
```

ရှိပြီးပါပြီ။

ဒီနေ့:

```text
Database

   ↓

Report Service

   ↓

Chart Data

   ↓

JFreeChart

   ↓

Swing Dashboard

```

ကို တည်ဆောက်ပါမယ်။

---

# 1. Why Chart Dashboard?

Manager က Database Table မကြည့်ချင်ပါဘူး။

သူလိုချင်တာ:

```text
Today Revenue

850,000 MMK


Orders

120


Top Product

Latte


Stock Warning

5 Items

```

Visual နဲ့ မြင်ချင်ပါတယ်။

---

# 2. Dashboard Architecture Update

Before:

```text
Dashboard

    |

 Labels

```

After:

```text
DashboardPanel


   |

   |

ReportController


   |

   |

ReportService


   |

   |

ReportRepository


   |

   |

Database


```

---

# 3. Add JFreeChart Dependency

Maven:

```xml
<dependency>

    <groupId>org.jfree</groupId>

    <artifactId>jfreechart</artifactId>

    <version>1.5.4</version>

</dependency>

```

---

# 4. Chart Package Structure

Create:

```text
module/report/chart


├── SalesChart.java

├── ProductChart.java

└── ChartFactory.java

```

---

# 5. Revenue Chart Design

Example:

```text
Daily Revenue


100k |

 80k |        █

 60k |    █   █

 40k | █

     ----------------

      Mon Tue Wed Thu

```

---

# 6. Create SalesChart Class

```java
package com.cafe.pos.module.report.chart;


import org.jfree.chart.*;

import org.jfree.chart.plot.*;

import org.jfree.data.category.*;

import javax.swing.*;



public class SalesChart {



public static ChartPanel create(
CategoryDataset dataset
){


JFreeChart chart =

ChartFactory
.createLineChart(

"Daily Revenue",

"Day",

"Amount",

dataset

);



return new ChartPanel(chart);


}


}

```

---

# 7. Create Dataset

JFreeChart မသိတဲ့သူတွေ အတွက်:

Chart ကို Database တိုက်ရိုက်မပေးပါဘူး။

Flow:

```text
Database Result


      ↓


Dataset


      ↓


Chart


```

---

Create:

```java
DefaultCategoryDataset dataset =
new DefaultCategoryDataset();


dataset.addValue(

50000,

"Sales",

"Mon"

);


dataset.addValue(

80000,

"Sales",

"Tue"

);

```

---

Output:

```
Mon  50000
Tue  80000

```

Chart က ဆွဲပေးပါမယ်။

---

# 8. Report Chart Service

Chart UI ထဲမှာ SQL မရေးရပါဘူး။

Create:

```text
ReportChartService.java

```

---

Code:

```java
public class ReportChartService {



public CategoryDataset
createDailySalesDataset(){


DefaultCategoryDataset dataset =

new DefaultCategoryDataset();



dataset.addValue(
50000,
"Sales",
"Mon"
);


dataset.addValue(
90000,
"Sales",
"Tue"
);


dataset.addValue(
120000,
"Sales",
"Wed"
);



return dataset;

}


}

```

---

# 9. Dashboard Panel Layout

Professional:

```
+------------------------------------------------+

                 CAFÉ POS DASHBOARD


+-------------+  +-------------+  +-------------+

| SALES TODAY |  | ORDERS      |  | PRODUCTS    |

|850,000 MMK  |  |120          |  |150          |

+-------------+  +-------------+  +-------------+


+------------------------------------------------+

                 Revenue Chart


+------------------------------------------------+


+----------------------+-------------------------+

 Best Seller           Low Stock


 Latte                 Milk

 Burger                Coffee Bean


+----------------------+-------------------------+

```

---

# 10. DashboardPanel

```java
public class DashboardPanel

extends JPanel {



private JLabel salesLabel;


private JLabel orderLabel;



private JPanel chartPanel;



public DashboardPanel(){


setLayout(
new BorderLayout()
);


initialize();


}


}

```

---

# 11. Add Summary Cards

Create:

```java
private JPanel createCard(
String title
){


JPanel panel =
new JPanel();


panel.add(
new JLabel(title)
);


return panel;


}

```

---

Usage:

```java
add(
createCard(
"Today's Sales"
)
);

```

---

# 12. Add Chart

Example:

```java
CategoryDataset dataset =

chartService
.createDailySalesDataset();



ChartPanel chart =

SalesChart
.create(dataset);



add(
chart,
BorderLayout.CENTER
);

```

---

# 13. Best Selling Product Widget

Database:

```sql
SELECT

product_name,

SUM(quantity)


FROM order_items


GROUP BY product_id


ORDER BY SUM(quantity)
DESC


LIMIT 5;

```

---

Display:

```
TOP PRODUCTS


1. Latte        500

2. Burger       300

3. Cake         200

```

---

# 14. Low Stock Widget

Query:

```sql
SELECT

name,

quantity


FROM inventory


WHERE quantity < minimum_stock;

```

---

Result:

```
LOW STOCK


⚠ Milk

⚠ Coffee Bean

```

---

# 15. Dashboard Refresh

Data ပြောင်းသွားရင်:

```java
public void refresh(){


SalesReport report =

controller.today();


salesLabel
.setText(

report.totalSales()
+
" MMK"

);


}

```

---

# 16. Timer Auto Refresh

POS Dashboard:

```java
Timer timer =

new Timer(

60000,

e -> refresh()

);


timer.start();

```

---

Every:

```
60 seconds

```

update ဖြစ်မယ်။

---

# 17. Color Status Logic

Low Stock:

```java
if(
stock < minimum
){

status =
"LOW";

}

```

---

Normal:

```
NORMAL

```

---

# 18. Dashboard Final Flow

Complete:

```
MySQL


 ↓


ReportRepository


 ↓


ReportService


 ↓


ReportController


 ↓


DashboardPanel


 ↓


JFreeChart


```

---

# 19. Current Café POS Level

Completed:

```
Java 25                 ✅

MVC Architecture        ✅

Database                ✅

Product                 ✅

Inventory               ✅

Recipe                  ✅

Order                   ✅

Payment                 ✅

Receipt                 ✅

Reporting System        ✅

Dashboard Analytics     🔄

Charts                  🔄

```

---

# Practice Task

Implement:

1. Add JFreeChart dependency
    
2. Create SalesChart
    
3. Create Dataset
    
4. Add ChartPanel
    
5. Create Dashboard Cards
    
6. Add Refresh Timer
    

---

# Next Lesson

# Lesson 48: Reporting Module (Part 3)

## Advanced Analytics + Profit Calculation + Export Excel/PDF

Next we build:

```
Sales Data

    ↓

Profit Analysis

    ↓

Expense Tracking

    ↓

Excel Export

    ↓

PDF Report

```

ဒီအဆင့်ပြီးရင် Café POS က **Small Business ERP Level** ကို ရောက်လာပါမယ်။