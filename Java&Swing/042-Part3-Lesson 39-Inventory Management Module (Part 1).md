# Part 3: Café POS Real Implementation Phase

# Lesson 39: Inventory Management Module (Part 1)

## Ingredient Entity + Stock Management + Inventory Transaction System

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Café POS ရဲ့ **အရေးကြီးဆုံး Core Module တစ်ခု** ဖြစ်တဲ့ Inventory System ကို စတင်ပါမယ်။

Restaurant / Café POS မှာ Product ရောင်းတာထက် ပိုအရေးကြီးတာက:

> "ဘယ်လောက် Stock ကျန်သလဲ?"  
> "ဘယ်အချိန် Reorder လုပ်ရမလဲ?"  
> "ရောင်းပြီးရင် Ingredient ဘယ်လောက် လျော့သွားသလဲ?"

ဆိုတာတွေပါ။

---

# 1. Inventory System Real World Example

Customer က:

```
Order:

2 x Latte
1 x Burger
```

ဆိုရင် System က:

```
Latte Recipe

Coffee Bean 20g
Milk 100ml
Sugar 5g


× 2

-----------------

Coffee Bean -40g
Milk       -200ml
Sugar      -10g


```

ပြီးရင် Inventory update လုပ်ရမယ်။

---

# 2. Inventory Module Architecture

Professional Design:

```
                 Order Module

                      |
                      |
              Inventory Service

                      |
          ------------------------

          |                      |

   InventoryRepository    RecipeRepository


          |

       MySQL Database

```

---

# 3. Inventory Module Package Structure

Create:

```
module

└── inventory


    ├── model

    │
    │── Ingredient.java
    │── Inventory.java
    │── StockMovement.java


    ├── repository

    │
    │── IngredientRepository.java
    │── InventoryRepository.java
    │── StockMovementRepository.java


    ├── service

    │
    │── InventoryService.java


    ├── controller

    │
    │── InventoryController.java


    └── view

        └── InventoryPanel.java

```

---

# 4. Database Review

Already created:

## ingredients

```
id

name

unit

```

Example:

|id|name|unit|
|---|---|---|
|1|Coffee Bean|gram|
|2|Milk|ml|
|3|Sugar|gram|

---

## inventory

```
id

ingredient_id

quantity

updated_at

```

Example:

|Ingredient|Quantity|
|---|---|
|Coffee Bean|5000g|
|Milk|20000ml|

---

## stock_movements

```
id

ingredient_id

type

quantity

reference_id

created_at

```

---

# 5. Why Stock Movement Table?

Many beginners:

```
inventory.quantity = 5000

```

ပြီးရင် ဘာဖြစ်သွားလဲ မသိ။

---

Professional:

Every change record:

```
Stock Movement


+5000 Purchase

-20 Sale

-50 Waste

+100 Return


```

---

Example:

```
Coffee Bean


10:00

PURCHASE

+5000g


12:00

SALE

-40g


15:00

WASTE

-100g

```

---

# 6. Ingredient Entity

Create:

```
Ingredient.java

```

---

Code:

```java
package com.cafe.pos.module.inventory.model;


public record Ingredient(

Long id,

String name,

String unit

){

}

```

---

# 7. Inventory Entity

Create:

```
Inventory.java

```

---

Code:

```java
package com.cafe.pos.module.inventory.model;


public record Inventory(

Long id,

Ingredient ingredient,

double quantity

){

}

```

---

# 8. Stock Movement Entity

Create:

```
StockMovement.java

```

---

Code:

```java
package com.cafe.pos.module.inventory.model;


import java.time.LocalDateTime;


public record StockMovement(

Long id,

Ingredient ingredient,

MovementType type,

double quantity,

Long referenceId,

LocalDateTime createdAt

){

}

```

---

# 9. Movement Type Enum

Create:

```
MovementType.java

```

---

Code:

```java
package com.cafe.pos.module.inventory.model;


public enum MovementType {


PURCHASE,

SALE,

ADJUSTMENT,

WASTE,

RETURN


}

```

---

# 10. Repository Design

## IngredientRepository

```java
public interface IngredientRepository {


void save(
Ingredient ingredient
);



List<Ingredient> findAll();



Ingredient findById(
Long id
);



}

```

---

# 11. InventoryRepository

```java
public interface InventoryRepository {


Inventory findByIngredientId(
Long id
);



void updateQuantity(
Long ingredientId,
double quantity
);



}

```

---

# 12. StockMovementRepository

```java
public interface StockMovementRepository {


void save(
StockMovement movement
);



List<StockMovement> findByIngredient(
Long ingredientId
);


}

```

---

# 13. Inventory Service

Business Logic:

```
Increase Stock

        +

Create Movement Log


```

---

Create:

```
InventoryService.java

```

---

Code:

```java
package com.cafe.pos.module.inventory.service;


import com.cafe.pos.module.inventory.model.*;



public class InventoryService {



private final InventoryRepository inventoryRepository;


private final StockMovementRepository movementRepository;




public InventoryService(

InventoryRepository inventoryRepository,

StockMovementRepository movementRepository

){


this.inventoryRepository =
inventoryRepository;


this.movementRepository =
movementRepository;


}



public void increaseStock(

Ingredient ingredient,

double amount

){



Inventory inventory =
inventoryRepository
.findByIngredientId(
ingredient.id()
);



double newQuantity =
inventory.quantity()
+
amount;



inventoryRepository
.updateQuantity(
ingredient.id(),
newQuantity
);



StockMovement movement =
new StockMovement(

null,

ingredient,

MovementType.PURCHASE,

amount,

null,

null

);



movementRepository.save(
movement
);



}



}

```

---

# 14. Transaction Problem

Current:

```
Update Inventory

        |

Save Movement


```

What if:

```
Update Inventory SUCCESS


Save Movement FAILED


```

Problem:

Inventory wrong.

---

# 15. Use Database Transaction

Correct:

```
BEGIN TRANSACTION


Update Inventory


Insert Movement


COMMIT


```

or:

```
ROLLBACK

```

---

# 16. Inventory Transaction Flow

Example Purchase:

```
User

 |
 |
InventoryPanel

 |
 |
InventoryController

 |
 |
InventoryService

 |
 |
TransactionManager

 |
 |
Database


```

---

# 17. Stock Deduction Logic

Later Order Module:

When order completed:

```
Order Paid


        |

Find Product Recipe


        |

Calculate Ingredient Usage


        |

Decrease Stock


        |

Create SALE Movement


```

---

# 18. Low Stock Alert

Professional POS needs:

Example:

```
Coffee Bean

Current:

500g


Minimum:

1000g


Status:

LOW STOCK

```

---

Database update:

Add:

```sql
ALTER TABLE ingredients

ADD COLUMN minimum_stock DECIMAL(10,2);

```

---

Entity:

```java
public record Ingredient(

Long id,

String name,

String unit,

double minimumStock

){

}

```

---

# 19. Inventory UI Design

Later:

```
+------------------------------------------------+

 Inventory Management


 Search: [Coffee]


+-----------------------------------------------+

| Ingredient | Unit | Stock | Status             |

|-----------------------------------------------|

| Coffee    | gram |5000   | OK                 |

| Milk      | ml   |500    | LOW STOCK          |


+-----------------------------------------------+


[Add Stock]

[Adjustment]


+------------------------------------------------+

```

---

# 20. Inventory Module Rules

Never:

❌ Direct update quantity from UI

❌ Delete stock history

❌ Ignore transaction

Always:

✅ Service Layer

✅ Stock Movement Log

✅ Transaction

✅ Audit Trail

---

# 21. Current Project Status

Completed:

```
Java 25 Setup             ✅

Database Architecture     ✅

Migration System          ✅

Authentication            ✅

Dashboard                 ✅

Product Module            ✅

Category                  ✅

Inventory Design          ✅

Ingredient Entity         ✅

Stock Movement Design     ✅

```

---

# Practice Task

Create:

1. Ingredient.java
    
2. Inventory.java
    
3. StockMovement.java
    
4. MovementType.java
    
5. Repository Interfaces
    
6. InventoryService
    

---

# Next Lesson

# Lesson 40: Inventory Management Module (Part 2)

## Complete Stock CRUD + JTable + Stock Adjustment + Low Stock Alert

Next we implement:

✅ Inventory Swing UI  
✅ Ingredient Management  
✅ Add Stock Dialog  
✅ Stock Adjustment  
✅ Stock History JTable  
✅ Low Stock Warning System  
✅ Database Transaction Integration

ပြီးရင် Café POS မှာ **Stock ကို Real Time Control လုပ်နိုင်ပါမယ်။**