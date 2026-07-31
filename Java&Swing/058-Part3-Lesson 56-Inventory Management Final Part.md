# Part 3: Café POS Real Implementation Phase

# Lesson 56: Inventory Management Final Part

## Inventory Audit + Stock Adjustment + Complete Inventory Dashboard

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson က Inventory Module ရဲ့ **Final Professional Level** ဖြစ်ပါတယ်။

အခုအထိ ကျွန်တော်တို့ တည်ဆောက်ပြီးတာ:

```text
Supplier Management              ✅

Purchase Order                    ✅

Stock Receiving                   ✅

Inventory Increase                ✅

Stock Movement History            ✅

Transaction Management            ✅

```

ဒါပေမယ့် Real Café / Restaurant ERP မှာ နောက်ထပ် အရေးကြီးတဲ့ Feature တစ်ခု ရှိပါတယ်။

---

# Inventory Audit & Stock Adjustment

---

## 1. Why Inventory Audit Needed?

Real World Problem:

System မှာ:

```text
Coffee Bean

System Stock:

25 kg

```

ဒါပေမယ့် Warehouse မှာ Count လုပ်ကြည့်:

```text
Physical Stock:

23 kg

```

Difference:

```text
-2 kg

```

ဘာကြောင့်?

Possible Reasons:

```text
✔ Spillage

✔ Damaged Product

✔ Wrong Entry

✔ Theft

✔ Measurement Error

```

---

ဒါကြောင့် System မှာ:

```text
Physical Count

        ↓

Compare

        ↓

Adjustment

        ↓

Stock Movement

```

လိုအပ်ပါတယ်။

---

# 2. Final Inventory Architecture

Complete ERP Inventory:

```
                Inventory System


                      |

 ------------------------------------------------

 |                    |                         |


Purchase          Sales Consume             Audit


 |                    |                         |


Receiving          Stock Deduction        Adjustment


 |                    |                         |


      -------- Stock Movement History --------


                      |


              Inventory Dashboard

```

---

# PART 1

# Inventory Audit Database Design

---

# 3. Inventory Audit Table

Create:

```sql
CREATE TABLE inventory_audits
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


audit_date DATE,


created_by BIGINT,


status VARCHAR(30),


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP


);

```

---

Example:

```
Audit ID:

5001


Date:

2026-07-31


Status:

COMPLETED

```

---

# 4. Audit Items Table

Because one audit has many ingredients.

```sql
CREATE TABLE inventory_audit_items
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


audit_id BIGINT,


ingredient_id BIGINT,


system_quantity DECIMAL(10,2),


physical_quantity DECIMAL(10,2),


difference DECIMAL(10,2),


FOREIGN KEY(audit_id)

REFERENCES inventory_audits(id)

);

```

---

Example:

```
Ingredient:

Coffee Bean


System:

25 kg


Physical:

23 kg


Difference:

-2 kg

```

---

# PART 2

# Audit Model Design

---

## 5. InventoryAudit

```java
public record InventoryAudit(


Long id,


LocalDate date,


Long createdBy,


String status


){

}

```

---

## 6. AuditItem

```java
public record AuditItem(


Long ingredientId,


String ingredientName,


double systemQuantity,


double physicalQuantity


){



public double difference(){


return physicalQuantity
-
systemQuantity;


}


}

```

---

Example:

```java
AuditItem item =


new AuditItem(

10L,

"Coffee Bean",

25,

23

);


System.out.println(
item.difference()
);

```

Output:

```
-2

```

---

# PART 3

# Inventory Audit Swing UI

---

# 7. Audit Screen Design

```
+------------------------------------------------+

              INVENTORY AUDIT


Date:

31-07-2026


-----------------------------------------------


Ingredient       System      Physical     Diff


Coffee Bean       25kg        23kg        -2kg


Milk              50L         50L          0L


Sugar             10kg        12kg        +2kg



-----------------------------------------------


              [ SAVE AUDIT ]


+------------------------------------------------+

```

---

# 8. Main Components

```java
public class InventoryAuditPanel

extends JPanel {



private JTable table;


private AuditTableModel model;


private JButton saveButton;


}

```

---

# PART 4

# Audit JTable Model

---

# 9. Editable Physical Quantity

Important:

System Quantity:

```
Read Only

```

Physical:

```
Editable

```

---

Code:

```java
@Override

public boolean isCellEditable(

int row,

int column

){


return column == 2;


}

```

---

# 10. Calculate Difference

Example:

```java
System:

25


Physical:

23


Difference:

-2

```

---

Table Model:

```java
@Override

public Object getValueAt(
int row,
int col
){


AuditItem item =
items.get(row);



return switch(col){


case 0 ->
item.ingredientName();



case 1 ->
item.systemQuantity();



case 2 ->
item.physicalQuantity();



case 3 ->
item.difference();



default ->
null;


};


}

```

---

# PART 5

# Stock Adjustment System

---

# 11. What is Adjustment?

When audit finds difference:

Example:

System:

```
25 kg

```

Physical:

```
23 kg

```

Need:

```
Inventory = 23 kg

```

---

Adjustment:

```
-2 kg

```

---

# 12. Adjustment Table

```sql
CREATE TABLE stock_adjustments
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


ingredient_id BIGINT,


old_quantity DECIMAL(10,2),


new_quantity DECIMAL(10,2),


difference DECIMAL(10,2),


reason VARCHAR(255),


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

```

---

# 13. Adjustment Example

```
Ingredient:

Coffee Bean


Old:

25


New:

23


Difference:

-2


Reason:

Damaged

```

---

# PART 6

# Adjustment Service

Flow:

```
Audit Completed


        ↓


Find Difference


        ↓


Update Inventory


        ↓


Create Adjustment Record


        ↓


Create Stock Movement


```

---

# 14. Service Code

```java
public class InventoryAdjustmentService {



private final InventoryRepository inventoryRepository;



public void adjust(

AuditItem item,

String reason

){


double diff =

item.difference();



inventoryRepository
.updateQuantity(

item.ingredientId(),

item.physicalQuantity()

);



saveAdjustment(
item,
reason
);



saveMovement(
item,
diff
);



}


}

```

---

# PART 7

# Stock Movement Integration

---

Every adjustment creates movement.

Example:

Before:

```
Coffee Bean

25kg

```

After:

```
23kg

```

Movement:

```
TYPE:

ADJUSTMENT


Quantity:

-2


Reason:

Damaged

```

---

# 15. Movement Enum Update

```java
public enum StockMovementType {


PURCHASE_RECEIVE,


SALE_CONSUME,


ADJUSTMENT,


RETURN,


WASTE


}

```

---

# PART 8

# Inventory Dashboard

---

# 16. Why Dashboard?

Manager wants quick view:

```
Inventory Status


Total Ingredients:

120


Low Stock:

15


Out Of Stock:

3


Today's Movement:

+250 kg


```

---

# 17. Dashboard Design

```
+------------------------------------------------+

 INVENTORY DASHBOARD


+--------------+


 Total Items

    120


+--------------+


 Low Stock

    15


+--------------+


 Out Of Stock

     3


+--------------+



Recent Movements:


PURCHASE +20kg Coffee


SALE -2kg Milk


ADJUSTMENT -1kg Sugar



+------------------------------------------------+

```

---

# 18. Dashboard Service

```java
public class InventoryDashboardService {



public long totalIngredients(){

return repository.count();

}



public long lowStockCount(){

return repository.countLowStock();

}



public List<StockMovement>

recentMovements(){


return movementRepository
.findLatest(10);


}


}

```

---

# PART 9

# Low Stock Algorithm

SQL:

```sql
SELECT *

FROM ingredients

WHERE quantity <= minimum_quantity;

```

---

Example:

```
Coffee Bean

Stock:

2kg


Minimum:

5kg


Result:

LOW STOCK

```

---

# PART 10

# Complete Inventory Module Architecture

Now:

```
                 Inventory


                     |


 ------------------------------------------------


Supplier       Purchase       Receiving       Audit


   |              |               |              |


   |              |               |              |


                Inventory


                     |


              Stock Movement


                     |


              Dashboard Report

```

---

# Current Café POS Level

After Lesson 56:

```
Java 25                         ✅

Swing MVC                       ✅

JDBC Architecture               ✅

Supplier System                 ✅

Purchase Order                  ✅

Stock Receiving                 ✅

Inventory Audit                 ✅

Stock Adjustment                ✅

Stock Movement                  ✅

Inventory Dashboard             ✅

ERP Inventory Workflow          ✅

```

---

# Practice Task

Implement:

## Database

1. inventory_audits
    
2. inventory_audit_items
    
3. stock_adjustments
    

## Java

4. InventoryAudit Model
    
5. AuditTableModel
    
6. Adjustment Service
    

## Swing

7. Inventory Audit Panel
    
8. Dashboard Panel
    

---

# Next Lesson

# Lesson 57: Café POS Final Integration

## Connecting Inventory + Sales + Recipe + Order + Payment System

Next we connect the complete system:

```
Customer Order


       ↓


Recipe


       ↓


Ingredient Consumption


       ↓


Inventory Deduction


       ↓


Low Stock Alert


       ↓


Purchase Recommendation


       ↓


Supplier Order


       ↓


Receiving


       ↓


Inventory Restore


```

ဒီ Lesson ကနေ Café POS ကို **Complete Enterprise Application Architecture** အဖြစ် Integration စတင်ပါမယ်။