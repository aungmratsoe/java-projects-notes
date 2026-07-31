# Part 3: Café POS Real Implementation Phase

# Lesson 50: Inventory Management Advanced

# Supplier Module + Purchase Order + Stock Receiving System

## (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson က Café POS Project ရဲ့ **Inventory Management ကို Professional ERP Level** တင်မယ့် အရေးကြီးတဲ့ Lesson ဖြစ်ပါတယ်။

အရင် Lesson မှာ:

```text
Inventory

 ↓

Low Stock Detection

 ↓

Purchase Recommendation

```

အထိလုပ်ခဲ့ပါတယ်။

ဒါပေမယ့် Real Restaurant/Café မှာ:

> "Stock နည်းနေပြီ" ဆိုတာသိရုံနဲ့ မလုံလောက်ပါဘူး။

Manager က:

- ဘယ် Supplier ဆီက ဝယ်မလဲ?
    
- Purchase Order ဘယ်လိုလုပ်မလဲ?
    
- ပစ္စည်းရောက်လာရင် ဘယ်လိုလက်ခံမလဲ?
    
- Stock ဘယ်လိုတိုးမလဲ?
    
- ဘယ်သူက ဘယ်နေ့မှာ Receive လုပ်ခဲ့လဲ?
    

ဆိုတာတွေ လိုအပ်ပါတယ်။

---

# 1. Real Inventory Purchase Flow

Restaurant Example:

Coffee Bean လက်ကျန်:

```
Current Stock:

2 kg

Minimum:

5 kg

```

System:

```
LOW STOCK

        ↓

Create Purchase Order

        ↓

Send To Supplier

        ↓

Supplier Delivers

        ↓

Receive Stock

        ↓

Update Inventory

        ↓

Create Stock Movement

```

---

# 2. Complete Inventory Architecture

Professional ERP Flow:

```
                 Inventory Module


                        |


        ---------------------------------

        |               |               |


   Supplier        Purchase          Stock

   Management      Order             Receiving


                        |

                        |

                Inventory Update


                        |

                        |

                 Stock Movement


```

---

# 3. New Module Structure

Create:

```
module/inventory


├── supplier

│
├── model

│    Supplier.java

│
├── repository

│    SupplierRepository.java

│
├── service

│    SupplierService.java



├── purchase

│
├── model

│    PurchaseOrder.java

│    PurchaseOrderItem.java


├── repository

│    PurchaseOrderRepository.java


├── service

│    PurchaseOrderService.java



└── receiving

     ├── StockReceiving.java

     └── StockReceivingService.java

```

---

# PART 1

# Supplier Management System

---

# 4. Supplier Database Design

Supplier ဆိုတာ:

```
Coffee Bean Supplier

Milk Supplier

Cake Ingredient Supplier

```

Table:

```sql
CREATE TABLE suppliers
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


name VARCHAR(100) NOT NULL,


phone VARCHAR(30),


email VARCHAR(100),


address TEXT,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP


);

```

---

Database:

```
suppliers


id

name

phone

email

address

created_at

```

---

# 5. Supplier Entity

Create:

`Supplier.java`

```java
package com.cafe.pos.module.inventory.supplier.model;



public record Supplier(


Long id,


String name,


String phone,


String email,


String address


){

}

```

---

# 6. Why use Record?

Java 25 မှာ:

```java
record

```

က immutable data object အတွက် အကောင်းဆုံးပါ။

Example:

```java
Supplier supplier =
new Supplier(

1L,

"ABC Coffee Supplier",

"091234567",

"abc@gmail.com",

"Yangon"

);

```

---

# 7. Supplier Repository

Database Access Layer:

```java
public interface SupplierRepository {


void save(
Supplier supplier
);



List<Supplier> findAll();



Supplier findById(
Long id
);



void update(
Supplier supplier
);



void delete(
Long id
);



}

```

---

# 8. Supplier Service

Business Layer:

```java
public class SupplierService {



private final SupplierRepository repository;



public SupplierService(
SupplierRepository repository
){

this.repository =
repository;

}



public void create(
Supplier supplier
){


validate(supplier);


repository.save(
supplier
);


}



private void validate(
Supplier supplier
){


if(
supplier.name()
.isBlank()
){


throw new RuntimeException(
"Supplier name required"
);


}


}


}

```

---

# 9. Supplier Validation Rules

Professional Application မှာ:

Supplier Name:

```
Cannot be empty

```

Phone:

```
Valid format

```

Email:

```
Valid email

```

---

Example:

Wrong:

```
Supplier:

(empty)

```

Throw:

```
Supplier name is required

```

---

# PART 2

# Purchase Order System

---

# 10. What is Purchase Order?

Purchase Order (PO):

Restaurant က Supplier ကို ပေးတဲ့ Request Document ဖြစ်ပါတယ်။

Example:

```
PURCHASE ORDER


Supplier:

ABC Coffee Supplier


Items:


Coffee Bean

20 kg


Milk

50 L


Total:

500,000 MMK

```

---

# 11. Purchase Order Database

## purchase_orders

```sql
CREATE TABLE purchase_orders
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


supplier_id BIGINT,


status VARCHAR(30),


total_amount DECIMAL(10,2),


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(supplier_id)

REFERENCES suppliers(id)


);

```

---

Status:

```java
public enum PurchaseStatus {


CREATED,

SENT,

RECEIVED,

CANCELLED


}

```

---

# 12. Purchase Order Items

One Purchase Order:

```
PO #100


Coffee Bean

20kg


Milk

50L

```

---

Table:

```sql
CREATE TABLE purchase_order_items
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


purchase_order_id BIGINT,


ingredient_id BIGINT,


quantity DECIMAL(10,2),


cost DECIMAL(10,2),



FOREIGN KEY(purchase_order_id)

REFERENCES purchase_orders(id)

);

```

---

# 13. Purchase Order Entity

```java
public record PurchaseOrder(


Long id,


Long supplierId,


PurchaseStatus status,


double totalAmount


){

}

```

---

# 14. Purchase Order Item

```java
public record PurchaseOrderItem(


Long id,


Long purchaseOrderId,


Long ingredientId,


double quantity,


double cost


){

}

```

---

# 15. Purchase Order Flow

```
Manager


 ↓


Select Supplier


 ↓


Add Ingredients


 ↓


Create PO


 ↓


Save Database


 ↓


Send Supplier


```

---

# PART 3

# Stock Receiving System

---

# 16. Why Separate Receiving?

Wrong Design:

```
Create Purchase Order

        ↓

Immediately Add Stock

```

Problem:

Supplier hasn't delivered yet.

---

Correct:

```
Purchase Order


        ↓


Waiting


        ↓


Goods Arrived


        ↓


Receive


        ↓


Increase Stock

```

---

# 17. Receiving Table

```sql
CREATE TABLE stock_receiving
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


purchase_order_id BIGINT,


received_by BIGINT,


received_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP


);

```

---

# 18. Receiving Flow

Example:

PO:

```
Coffee Bean

20kg

```

Supplier delivers:

```
Received:

18kg

```

System:

```
Inventory +18kg

```

---

# 19. Stock Receiving Service

```java
public class StockReceivingService {



private final InventoryService inventoryService;



public void receive(
PurchaseOrderItem item
){



inventoryService
.addStock(

item.ingredientId(),

item.quantity()

);


}

}

```

---

# 20. Transaction Requirement

Receiving must be transaction:

```
BEGIN


Create Receiving Record


Update Inventory


Create Stock Movement


Update Purchase Order Status


COMMIT


```

---

Failure:

```
ROLLBACK

```

---

# 21. Stock Movement Example

Before:

```
Coffee Bean

2kg

```

Receive:

```
+20kg

```

After:

```
22kg

```

Stock Movement:

```
TYPE:

PURCHASE_RECEIVE


Quantity:

+20kg


User:

Manager


Date:

2026-07-31

```

---

# 22. Complete Inventory ERP Flow

Now:

```
Supplier


   ↓


Purchase Order


   ↓


Supplier Delivery


   ↓


Stock Receiving


   ↓


Inventory Increase


   ↓


Stock Movement


   ↓


Analytics Update


```

---

# 23. Swing UI Design

## Supplier Management

```
+--------------------------------+

Supplier Management


Name:

[____________]


Phone:

[____________]


[Save]


--------------------------------


ID   Name       Phone


1    ABC        09123


+--------------------------------+

```

---

## Purchase Order Screen

```
+--------------------------------+

Create Purchase Order


Supplier:

[ABC Supplier]


Ingredient:

[Coffee Bean]


Quantity:

[20]


[ADD]


-------------------------------

Items


Coffee Bean 20kg


-------------------------------


[CREATE PO]


+--------------------------------+

```

---

# 24. Current Café POS Level

After Lesson 50:

```
Authentication              ✅

Product Management           ✅

Inventory CRUD               ✅

Recipe System                ✅

Stock Deduction              ✅

Order Management             ✅

Payment                      ✅

Receipt                      ✅

Reporting                    ✅

Supplier Management          ✅

Purchase Order               ✅

Stock Receiving              ✅


```

---

# Practice Task

Implement:

### Database

1. suppliers table
    
2. purchase_orders table
    
3. purchase_order_items table
    
4. stock_receiving table
    

### Java

5. Supplier Entity
    
6. Supplier Repository
    
7. Supplier Service
    
8. Purchase Order Service
    
9. Receiving Service
    

---

# Next Lesson

# Lesson 51: Inventory Management Advanced Part 2

## Complete Supplier UI + Purchase Order Swing Module + Transaction Implementation

Next we will build:

```
Swing Supplier Management

        ↓

Purchase Order UI

        ↓

Receive Stock UI

        ↓

Database Transaction

        ↓

Real Inventory Workflow

```

ဒီ Lesson က Café POS ကို **Professional Restaurant ERP Software Architecture** အဆင့်ရောက်စေမယ့် Core Module ဖြစ်ပါတယ်။