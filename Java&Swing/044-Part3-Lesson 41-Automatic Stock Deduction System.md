# Part 3: Café POS Real Implementation Phase

# Lesson 41: Automatic Stock Deduction System

## Product Recipe + Order Integration + Database Transaction Management

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson က Café POS ရဲ့ **အရေးကြီးဆုံး Business Logic** တစ်ခုဖြစ်ပါတယ်။

အခုအထိ:

```text
Product
   |
   |
Recipe
   |
   |
Ingredient
   |
   |
Inventory
```

Design ရှိပြီးပါပြီ။

ဒီနေ့မှာ Customer က Order တင်ပြီး Payment ပြီးသွားတဲ့အချိန်:

```text
Order Completed

        ↓

Read Product Recipe

        ↓

Calculate Required Ingredients

        ↓

Decrease Inventory

        ↓

Save Stock Movement

        ↓

Commit Transaction

```

ကို Implement လုပ်ပါမယ်။

---

# 1. Real Café Example

Customer Order:

```
2 x Latte
1 x Cappuccino
```

Database:

## Product Recipe

Latte:

|Ingredient|Quantity|
|---|---|
|Coffee Bean|20g|
|Milk|100ml|
|Sugar|5g|

Cappuccino:

|Ingredient|Quantity|
|---|---|
|Coffee Bean|25g|
|Milk|120ml|

---

Calculation:

Latte × 2:

```
Coffee Bean = 40g
Milk        = 200ml
Sugar       = 10g

```

Cappuccino × 1:

```
Coffee Bean = 25g
Milk        = 120ml

```

Total:

```
Coffee Bean = 65g
Milk        = 320ml
Sugar       = 10g

```

---

Inventory:

Before:

```
Coffee Bean 5000g
Milk        20000ml

```

After:

```
Coffee Bean 4935g
Milk        19680ml

```

---

# 2. Architecture Update

Before:

```text
Order

   ↓

Payment

```

After:

```text
OrderService


       |

       |

InventoryService


       |

       |

RecipeRepository


       |

       |

InventoryRepository


```

---

# 3. New Package Structure

Create:

```
module/order


├── model

│    Order.java

│    OrderItem.java


├── repository

│    OrderRepository.java


├── service

│    OrderService.java



module/recipe


├── model

│    Recipe.java


├── repository

│    RecipeRepository.java

```

---

# 4. Recipe Entity

Database:

```
product_recipes


id

product_id

ingredient_id

quantity

```

Java:

`Recipe.java`

```java
package com.cafe.pos.module.recipe.model;


public record Recipe(

Long productId,

Long ingredientId,

double quantity

){

}

```

---

# 5. Recipe Repository

Purpose:

Get ingredients needed for product.

Interface:

```java
public interface RecipeRepository {


List<Recipe> findByProductId(
Long productId
);


}

```

---

# 6. SQL Query

Example:

Product Latte ID = 1

```sql
SELECT

product_id,

ingredient_id,

quantity


FROM product_recipes


WHERE product_id=?;

```

---

Result:

```
product_id | ingredient_id | quantity

1             1              20

1             2              100

1             3              5

```

---

# 7. Order Item Entity

Customer buys:

```
Latte x2

```

Java:

```java
public record OrderItem(

Long productId,

int quantity

){

}

```

---

# 8. Inventory Service Update

Need:

```
consumeProduct()

```

---

Method:

```java
public void consumeProduct(

Long productId,

int quantity

)

```

---

Flow:

```
Product ID

      ↓

Find Recipe

      ↓

Calculate Ingredient Usage

      ↓

Decrease Stock

```

---

# 9. Implement Stock Consumption

InventoryService:

```java
public void consumeProduct(

Long productId,

int orderQuantity

){


List<Recipe> recipes =

recipeRepository
.findByProductId(productId);



for(Recipe recipe: recipes){



double required =

recipe.quantity()
*
orderQuantity;



decreaseStock(

recipe.ingredientId(),

required

);



createMovement(

recipe.ingredientId(),

required

);



}


}

```

---

# 10. Stock Checking

Before decrease:

Example:

Need:

```
Milk 300ml

```

Available:

```
Milk 200ml

```

Cannot sell.

---

Code:

```java
if(
inventory.quantity()
<
required
){


throw new StockException(

"Insufficient stock"

);


}

```

---

# 11. Custom Stock Exception

Create:

```
exception

 └── StockException.java

```

---

Code:

```java
package com.cafe.pos.exception;


public class StockException
extends RuntimeException{


public StockException(
String message
){

super(message);

}


}

```

---

# 12. Transaction Management

Important.

Wrong:

```
Decrease Stock

        ↓

Save Order

        ↓

Payment Failed


```

Problem:

Stock already removed.

---

Correct:

```
BEGIN TRANSACTION


Create Order


Create Order Items


Decrease Inventory


Create Stock Movement


Create Payment


COMMIT



```

---

Failure:

```
ROLLBACK

```

---

# 13. JDBC Transaction Example

```java
Connection con =
DatabaseManager.getConnection();


try{


con.setAutoCommit(false);



orderRepository.save(order);



inventoryService
.consumeProduct();



paymentRepository.save(payment);



con.commit();



}
catch(Exception e){


con.rollback();



throw e;


}

```

---

# 14. Transaction Responsibility

Professional Design:

Create:

```
TransactionManager.java

```

---

Purpose:

```text
BEGIN

COMMIT

ROLLBACK

```

---

Example:

```java
public class TransactionManager {


public void execute(
Runnable action
){

try{


action.run();


}

catch(Exception e){


throw e;

}


}


}

```

---

# 15. Complete Sale Flow

Now:

```
Cashier


 ↓


Create Order


 ↓


Add Items


 ↓


Checkout


 ↓


Payment


 ↓


OrderService


 ↓


InventoryService


 ↓


RecipeRepository


 ↓


Inventory Update


 ↓


Stock Movement


 ↓


Receipt

```

---

# 16. Database Consistency

Before:

```
Order

DONE


Inventory

NOT UPDATED

```

Bad ❌

---

After:

```
Order

DONE


Inventory

UPDATED


Stock Movement

CREATED

```

Good ✅

---

# 17. Low Stock After Sale

Example:

Before:

```
Milk

1200ml

```

Sale:

```
-500ml

```

After:

```
700ml

```

Minimum:

```
1000ml

```

System:

```
⚠ LOW STOCK

```

---

# 18. Audit Log

Every stock change:

```
User:

cashier1


Action:

SALE


Ingredient:

Milk


Quantity:

-500ml


Time:

10:30

```

Stored:

```
audit_logs

```

---

# 19. Final Architecture

Full Café POS Logic:

```
             Swing UI


                |

          OrderController


                |

          OrderService


                |

        -------------------

        |                 |

 InventoryService     PaymentService


        |

 RecipeRepository


        |

 InventoryRepository


        |

      MySQL


```

---

# 20. Current Project Status

Completed:

```
Java 25 Setup             ✅

MVC Architecture         ✅

Database Migration       ✅

Authentication           ✅

Dashboard                ✅

Product Management       ✅

Category System          ✅

Inventory System         ✅

Stock Movement           ✅

Recipe System             ✅

Automatic Stock Deduction ✅

Transaction Design        ✅

```

---

# Practice Task

Implement:

1. Recipe Entity
    
2. RecipeRepository
    
3. StockException
    
4. InventoryService.consumeProduct()
    
5. Transaction Flow
    

---

# Next Lesson

# Lesson 42: Order Management Module (Part 1)

## Order Entity + Cart System + JTable + Checkout Architecture

Next we build the core POS screen:

```
Cashier Screen

Products

    ↓

Add To Cart

    ↓

Calculate Total

    ↓

Checkout

    ↓

Payment

    ↓

Receipt

```

ဒီ Lesson ကနေ Café POS က **တကယ်ရောင်းလို့ရတဲ့ POS Application** ပုံစံကို စတင်ဝင်ပါမယ်။