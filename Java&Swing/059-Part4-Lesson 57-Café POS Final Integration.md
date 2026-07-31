# Part 4: Café POS Enterprise Integration Phase

# Lesson 57: Café POS Final Integration

## Connecting Inventory + Sales + Recipe + Order + Payment System

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson က Café POS Project ရဲ့ **System Integration Level** ဖြစ်ပါတယ်။

အခုအထိ Module တစ်ခုချင်းစီကို သီးခြားတည်ဆောက်ခဲ့ပါတယ်။

```text
Product Management          ✅

Recipe System               ✅

Inventory System            ✅

Supplier Management         ✅

Purchase Order              ✅

Stock Receiving             ✅

Payment System              ✅

Reporting                   ✅
```

ဒါပေမယ့် Enterprise Application မှာ Module တစ်ခုနဲ့တစ်ခု **Data Flow ချိတ်ဆက်မှု** က အရေးကြီးပါတယ်။

---

# 1. Complete Café POS Business Flow

Real Restaurant Flow:

```text
Customer


   ↓


Create Order


   ↓


Select Menu Items


   ↓


Recipe Lookup


   ↓


Calculate Required Ingredients


   ↓


Deduct Inventory


   ↓


Payment


   ↓


Receipt


   ↓


Sales Report


   ↓


Stock Analysis


   ↓


Low Stock Alert


   ↓


Purchase Order


   ↓


Supplier


   ↓


Receiving


   ↓


Inventory Increase

```

---

# 2. Enterprise Architecture

Final Architecture:

```
                    Café POS System


                         UI Layer

                            |
                            |

                    Swing Application


                            |

                    Controller Layer


                            |

                    Service Layer


                            |

        --------------------------------------

        |                 |                  |

    Order Service    Inventory Service   Payment Service


        |                 |                  |


        --------------------------------------


                         Repository Layer


                            |

                         JDBC


                            |

                         MySQL

```

---

# PART 1

# Order → Recipe → Inventory Integration

---

# 3. Current Problem

Before:

Customer Order:

```
Latte x 2

```

Sales System:

```
Order Saved

Payment Complete

```

But:

```
Inventory unchanged ❌

```

---

Real System:

Latte Recipe:

```
Coffee Bean     20g

Milk            200ml

Sugar           10g

```

Order:

```
Latte x 2

```

Need:

```
Coffee Bean -40g

Milk        -400ml

Sugar       -20g

```

---

# 4. Integration Flow

```
OrderService


      ↓


RecipeService


      ↓


IngredientCalculator


      ↓


InventoryService


      ↓


StockMovementService


```

---

# 5. Create Ingredient Usage Model

New Model:

```java
public record IngredientUsage(


Long ingredientId,


String ingredientName,


double quantity


){

}

```

---

Example:

```java
IngredientUsage usage =

new IngredientUsage(

1L,

"Coffee Bean",

40

);

```

Meaning:

```
Coffee Bean

REMOVE

40g

```

---

# PART 2

# Recipe Calculation Engine

---

# 6. Recipe Example

Database:

recipe_items:

```
menu_id | ingredient | quantity


1          Coffee Bean   20


1          Milk          200


1          Sugar         10

```

---

Order:

```
Latte x 3

```

Calculation:

```
Coffee:

20 * 3

=

60g


Milk:

200 * 3

=

600ml


```

---

# 7. Recipe Calculator Service

```java
public class RecipeCalculatorService {



private final RecipeRepository repository;



public List<IngredientUsage>

calculate(

Long menuId,

double quantity

){



List<RecipeItem> recipes =

repository.findByMenuId(
menuId
);



return recipes.stream()

.map(item ->

new IngredientUsage(

item.ingredientId(),

item.ingredientName(),

item.quantity()
*
quantity

)

)

.toList();



}


}

```

---

# PART 3

# Inventory Deduction Integration

---

# 8. InventoryService Enhancement

Before:

```java
addStock()

```

Now add:

```java
deductStock()

```

---

Code:

```java
public void deductStock(

Long ingredientId,

double quantity

){



Inventory inventory =

repository.findByIngredientId(
ingredientId
);



double current =

inventory.quantity();



if(current < quantity){


throw new InventoryException(

"Insufficient stock"

);


}



repository.updateQuantity(

ingredientId,

current - quantity

);



}

```

---

# 9. Stock Validation Example

Current:

```
Coffee Bean

30g

```

Order:

```
Latte x2


Need:

40g

```

Result:

```
ERROR


Not enough Coffee Bean

```

---

# PART 4

# Order Transaction Integration

---

# 10. Order Complete Transaction

Important:

Order Save + Inventory Deduction must be ONE Transaction.

Correct:

```
BEGIN


Create Order


      ↓


Create Order Items


      ↓


Calculate Recipe


      ↓


Deduct Inventory


      ↓


Create Stock Movement


      ↓


COMMIT


```

---

If failed:

```
ROLLBACK

```

---

# 11. OrderService Final Version

```java
public void createOrder(

Order order,

List<OrderItem> items

){


transactionManager.execute(con -> {



Long orderId =

orderRepository.save(
con,
order
);



orderItemRepository.saveAll(

con,

orderId,

items

);



for(OrderItem item:items){



List<IngredientUsage> usage =

recipeCalculator.calculate(

item.menuId(),

item.quantity()

);



inventoryService
.deductAll(

con,

usage

);



stockMovementService
.saveSaleMovement(

con,

usage

);



}



});


}

```

---

# PART 5

# Payment Integration

---

# 12. Payment Flow

```
Order


 ↓


Calculate Total


 ↓


Payment


 ↓


Receipt


 ↓


Complete Sale


```

---

Payment Status:

```java
public enum PaymentStatus {


PENDING,


PAID,


FAILED,


REFUNDED


}

```

---

# 13. Payment Transaction

Example:

```
Order:

1001


Total:

25000 MMK


Payment:

CASH


Status:

PAID

```

---

# PART 6

# Receipt Generation Integration

---

# 14. Receipt Data

Receipt needs:

```
Café Name

Date

Order Number


Items:


Latte     2     10000


Total:

20000


Payment:

Cash

```

---

Model:

```java
public record Receipt(


Long orderId,


List<OrderItem> items,


double total,


PaymentMethod method


){

}

```

---

# PART 7

# Low Stock Alert Integration

---

After Sale:

Example:

Before:

```
Coffee Bean

10kg

```

Sale:

```
-8kg

```

After:

```
2kg

```

Minimum:

```
5kg

```

System:

```
LOW STOCK ALERT

```

---

# 15. Alert Service

```java
public class StockAlertService {



public List<Ingredient>

checkLowStock(){



return repository
.findLowStock();


}


}

```

---

# PART 8

# Complete Event Flow

Professional System:

```
Customer Order Created


        ↓


Order Service


        ↓


Recipe Engine


        ↓


Inventory Deduction


        ↓


Stock Movement


        ↓


Payment


        ↓


Receipt


        ↓


Sales Report


        ↓


Low Stock Detection


        ↓


Purchase Recommendation


```

---

# PART 9

# Database Relationship Final View

```
CUSTOMER

   |

   |

ORDERS

   |

   |

ORDER_ITEMS

   |

   |

MENU_ITEMS

   |

   |

RECIPE_ITEMS

   |

   |

INGREDIENTS

   |

   |

INVENTORY

   |

   |

STOCK_MOVEMENTS

```

---

# PART 10

# Current Café POS Level

After Lesson 57:

```
Java 25 Architecture              ✅

Swing MVC                         ✅

Database Design                   ✅

Inventory ERP                     ✅

Recipe Engine                     ✅

Order Integration                 ✅

Automatic Stock Deduction         ✅

Payment Integration               ✅

Receipt Flow                      ✅

Low Stock Alert                   ✅

Purchase Workflow                 ✅

```

---

# Practice Task

Implement:

### Integration

1. OrderService → RecipeService
    
2. RecipeService → InventoryService
    
3. InventoryService → StockMovement
    

### Transaction

4. Complete Order Transaction
    
5. Rollback Handling
    

### UI

6. Display Low Stock Alert
    
7. Inventory Notification Panel
    

---

# Next Lesson

# Lesson 58: Advanced Database Architecture & System Design

## Senior Database Engineer / System Architect Level

Next we move from application coding into architecture:

```
Database Scaling

        ↓

Index Strategy

        ↓

Query Optimization

        ↓

Database Transaction Design

        ↓

Concurrency Control

        ↓

Enterprise POS Architecture

```

ဒီ Lesson ကနေ Café POS ကို **Senior System Architect Level** နဲ့ Design လုပ်သွားပါမယ်။