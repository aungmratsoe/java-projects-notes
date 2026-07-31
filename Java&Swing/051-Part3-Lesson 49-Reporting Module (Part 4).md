# Part 3: Café POS Real Implementation Phase

# Lesson 49: Reporting Module (Part 4)

## Inventory Analytics + Low Stock Alert + Purchase Recommendation

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Café POS ရဲ့ Inventory System ကို **Smart Inventory Management System** အဖြစ် Upgrade လုပ်ပါမယ်။

အခုအထိ:

```text
Product Management        ✅
Inventory Management      ✅
Recipe Engine             ✅
Order System              ✅
Payment System            ✅
Sales Dashboard           ✅
Profit Analysis           ✅
```

ရှိပြီးပါပြီ။

ဒီနေ့:

```text
Inventory Data

      ↓

Stock Analysis

      ↓

Low Stock Detection

      ↓

Alert System

      ↓

Purchase Recommendation

      ↓

Supplier Management

```

ကို တည်ဆောက်ပါမယ်။

---

# 1. Real Café Inventory Problem

Normal System:

```text
Coffee Bean

Current:

5 kg

```

Manager က မသိနိုင်တာ:

```
ဘယ်နှစ်ရက်လောက် လုံလောက်မလဲ?

ဘယ်နေ့မှာ ဝယ်ရမလဲ?

ဘယ် Supplier ဆီက ဝယ်ရမလဲ?

```

Smart System:

```
Coffee Bean

Current Stock:

5 kg


Average Usage:

1 kg/day


Remaining:

5 days


Recommendation:

Order 20 kg

```

---

# 2. Inventory Analytics Architecture

Professional:

```text
                    Dashboard


                       |


              InventoryController


                       |


              InventoryAnalyticsService


                       |


             InventoryRepository


                       |


                   MySQL

```

---

# 3. Package Structure

Create:

```
module/inventory


├── analytics

│
├── StockReport.java

├── LowStockItem.java

├── InventoryAnalyticsService.java


├── alert

│
├── StockAlertService.java


└── recommendation

    └── PurchaseRecommendationService.java

```

---

# 4. Stock Report Model

Create:

`StockReport.java`

```java
package com.cafe.pos.module.inventory.analytics;


public record StockReport(

Long ingredientId,

String ingredientName,

double currentQuantity,

double minimumQuantity,

String status

){

}

```

---

Example:

```
Coffee Bean

Current:

5kg


Minimum:

10kg


Status:

LOW

```

---

# 5. Low Stock Model

Create:

`LowStockItem.java`

```java
package com.cafe.pos.module.inventory.analytics;


public record LowStockItem(

Long id,

String name,

double quantity,

double minimumStock

){

}

```

---

# 6. Database Design Update

Inventory table:

Before:

```sql
inventory


id

ingredient_id

quantity

```

---

After:

```sql
inventory


id

ingredient_id

quantity

minimum_stock

updated_at

```

---

Add:

```sql
ALTER TABLE inventory

ADD COLUMN minimum_stock
DECIMAL(10,2)
DEFAULT 0;

```

---

# 7. Low Stock Query

Find items:

```sql
SELECT


i.ingredient_id,

ing.name,

i.quantity,

i.minimum_stock


FROM inventory i


JOIN ingredients ing


ON i.ingredient_id = ing.id


WHERE i.quantity < i.minimum_stock;

```

---

Result:

```
Ingredient       Qty     Min


Milk             500ml   1000ml

Coffee Bean      2kg     5kg

```

---

# 8. Inventory Repository

Create:

```java
public interface InventoryAnalyticsRepository {


List<LowStockItem>
findLowStockItems();



}

```

---

# 9. Inventory Analytics Service

Business Logic:

```java
public class InventoryAnalyticsService {



private final InventoryAnalyticsRepository repository;



public InventoryAnalyticsService(
InventoryAnalyticsRepository repository
){

this.repository =
repository;

}



public List<LowStockItem>
getLowStock(){


return repository
.findLowStockItems();


}


}

```

---

# 10. Low Stock Alert System

Purpose:

When Login:

```
Manager Dashboard


        ↓


Check Stock


        ↓


Show Warning

```

---

Create:

`StockAlertService.java`

```java
public class StockAlertService {



private final InventoryAnalyticsService service;



public void check(){


List<LowStockItem> items =

service.getLowStock();



if(!items.isEmpty()){


System.out.println(
"LOW STOCK ALERT"
);


}



}

}

```

---

# 11. Swing Alert UI

Example:

```text
+--------------------------------+

⚠ LOW STOCK WARNING


Coffee Bean

Current: 2kg


Milk

Current: 500ml


Please Purchase


+--------------------------------+

```

---

Java:

```java
JOptionPane.showMessageDialog(

null,

message,

"Stock Alert",

JOptionPane.WARNING_MESSAGE

);

```

---

# 12. Purchase Recommendation

Problem:

Low stock only says:

```
Milk LOW

```

But Manager needs:

```
How much should I buy?

```

---

Formula:

```
Recommended Quantity

=

(Max Stock)

-

(Current Stock)

```

---

Example:

```
Maximum Stock:

20,000 ml


Current:

5,000 ml


Need:

15,000 ml

```

---

# 13. Recommendation Model

Create:

```java
public record PurchaseRecommendation(


Long ingredientId,


String ingredientName,


double currentStock,


double recommendedQuantity


){

}

```

---

# 14. Purchase Recommendation Service

```java
public class PurchaseRecommendationService {


public PurchaseRecommendation calculate(


String name,

double current,

double maximum


){



double need =

maximum - current;



return new PurchaseRecommendation(

1L,

name,

current,

need

);


}


}

```

---

# 15. Smart Stock Calculation

Better:

Use sales history.

Example:

Last 30 days:

```
Coffee Usage:

60 kg


Average:

2 kg/day

```

---

Formula:

```
Safety Stock

=

Average Daily Usage × Days

```

Example:

```
2kg × 7 days

=

14kg

```

---

# 16. Inventory Dashboard Widget

Design:

```
+--------------------------------+

Inventory Status


Total Ingredients:

50


Low Stock:

5


Out Of Stock:

2


+--------------------------------+

```

---

# 17. Out Of Stock Detection

Query:

```sql
SELECT *

FROM inventory

WHERE quantity <= 0;

```

---

Result:

```
Coffee Bean

OUT OF STOCK

```

---

# 18. Integration With Order System

Important:

When Sale:

```
Order Complete


        ↓


Inventory Update


        ↓


Check Stock


        ↓


Alert Manager

```

---

Code:

```java
inventoryService.consumeProduct();


stockAlertService.check();

```

---

# 19. Supplier Management Preview

Next system:

```
Ingredient


       |

Supplier


       |

Purchase Order


       |

Receive Stock


       |

Inventory Update

```

---

# 20. Complete Inventory Intelligence Flow

Final:

```
             Sale


              ↓


        Inventory Change


              ↓


       Analytics Service


              ↓


   ------------------------


   |          |            |


 Low      Forecast     Purchase


 Stock    Usage        Suggestion


   |          |            |


 Dashboard  Chart       Supplier


```

---

# 21. Current Café POS Level

Completed:

```
Authentication             ✅

Product CRUD               ✅

Inventory CRUD             ✅

Recipe System              ✅

Automatic Stock Deduction  ✅

Order Management           ✅

Payment                    ✅

Receipt                    ✅

Sales Dashboard            ✅

Profit Analysis            ✅

Inventory Analytics        ✅

Low Stock Alert            ✅

Purchase Recommendation    🔄

```

---

# Practice Task

Implement:

1. Add minimum_stock column
    
2. Create LowStockItem
    
3. Create InventoryAnalyticsService
    
4. Create StockAlertService
    
5. Create Dashboard Warning Panel
    
6. Create Purchase Recommendation Logic
    

---

# Next Lesson

# Lesson 50: Inventory Management Advanced

## Supplier Module + Purchase Order + Stock Receiving System

Next we build:

```
Supplier

   ↓

Purchase Order

   ↓

Receive Goods

   ↓

Update Inventory

   ↓

Stock History

```

ဒီအဆင့်ပြီးရင် Café POS ရဲ့ Inventory System က **Real Restaurant ERP Level** ဖြစ်လာပါမယ်။