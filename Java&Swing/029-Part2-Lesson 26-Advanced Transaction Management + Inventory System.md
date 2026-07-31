# Part 2: Advanced Java Swing + Café POS Project

# Lesson 26: Advanced Transaction Management + Inventory System

## Enterprise Stock Control for Café POS

### (Java 25 + Swing + JDBC + MySQL + MVC + Transaction Architecture)

ဒီ Lesson မှာ Café POS ရဲ့ **အရေးကြီးဆုံး Backend System တစ်ခု** ဖြစ်တဲ့

> Inventory Management System

ကို Enterprise Level နဲ့ Design လုပ်ပါမယ်။

Restaurant POS တွေမှာ Order System ထက်တောင် Inventory က ပိုအရေးကြီးပါတယ်။

ဘာကြောင့်လဲဆိုတော့:

- Stock မှားရင် Loss ဖြစ်မယ်
    
- Ingredient မရှိရင် Order မလုပ်နိုင်ဘူး
    
- Multiple Cashier တွေရှိရင် Data Conflict ဖြစ်နိုင်တယ်
    

---

# 1. Inventory System Real World Example

Café မှာ:

```
Ingredient Stock

Coffee Bean
    5000 g

Milk
    10000 ml

Sugar
    3000 g


```

Customer Order:

```
5 Coffee

```

Recipe:

```
1 Coffee

Coffee Bean 20g
Milk 100ml
Sugar 5g

```

Calculation:

```
Coffee Bean:

20g × 5

=100g


Milk:

100ml ×5

=500ml


Sugar:

5g ×5

=25g

```

After:

```
Coffee Bean

5000g → 4900g


Milk

10000ml → 9500ml

```

---

# 2. Inventory Architecture

Final Architecture:

```
                Order Service

                     |

                     |

             Inventory Service

                     |

          ---------------------

          |                   |

 Stock Repository      Recipe Repository


          |

          |

        MySQL


```

---

# 3. Inventory Database Design

Previous tables:

```
ingredients

product_recipes

inventory

```

Need new tables:

```
stock_movements

suppliers

purchases

purchase_items

```

---

# 4. Ingredient Table

Example:

```
Coffee Bean

Milk

Sugar

Chocolate

```

SQL:

```sql
CREATE TABLE ingredients(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100) NOT NULL,

unit VARCHAR(20) NOT NULL,

created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

```

---

# 5. Inventory Table

Current stock:

```sql
CREATE TABLE inventory(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

ingredient_id BIGINT NOT NULL,

quantity DECIMAL(10,2) DEFAULT 0,


updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(ingredient_id)

REFERENCES ingredients(id)

);

```

---

# 6. Stock Movement History

Why need history?

Example:

```
Today:

Milk

10000 ml


Sold:

-500 ml


Purchase:

+5000 ml


Final:

14500 ml

```

Need audit:

```sql
CREATE TABLE stock_movements(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


ingredient_id BIGINT NOT NULL,


movement_type VARCHAR(30),


quantity DECIMAL(10,2),


reference_id BIGINT,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(ingredient_id)

REFERENCES ingredients(id)

);

```

---

# Movement Types:

```
PURCHASE

SALE

ADJUSTMENT

RETURN

WASTE

```

---

# 7. Inventory Entity

Java 25 Record:

```java
public record Inventory(

Long id,

Long ingredientId,

BigDecimal quantity

){

}

```

---

# 8. Stock Movement Entity

```java
public record StockMovement(

Long id,

Long ingredientId,

String type,

BigDecimal quantity,

Long referenceId

){

}

```

---

# 9. The Big Problem: Race Condition

Imagine two cashiers.

Stock:

```
Coffee Bean = 100g

```

Cashier A:

```
Sell Coffee

Need 80g

```

Cashier B:

```
Sell Coffee

Need 50g

```

Both check:

```
Stock > required

TRUE

```

Both update:

```
100 - 80 =20

100 - 50 =50

```

Wrong!

Actual:

```
Need:

80+50=130g


Available:

100g


```

---

# 10. Solution 1: Database Lock

Use:

```
SELECT ... FOR UPDATE

```

Meaning:

"Lock this row until transaction finishes"

---

Example:

```sql
SELECT quantity

FROM inventory

WHERE ingredient_id=1

FOR UPDATE;

```

---

Flow:

```
Transaction A

Lock Inventory


        |

Update


        |

Commit


        |

Release Lock



Transaction B

Wait...


```

---

# 11. JDBC Transaction Deep Dive

Basic:

```java
Connection con =
dataSource.getConnection();


con.setAutoCommit(false);


```

---

Why?

Default:

```
Every SQL statement

= automatic commit

```

Problem:

```
Save Order

Commit


Update Stock

Error


```

Database becomes inconsistent.

---

Transaction:

```
BEGIN


INSERT ORDER


INSERT ITEMS


UPDATE STOCK


COMMIT


```

---

# 12. Rollback

Example:

```java
try{


saveOrder();


updateStock();


con.commit();


}
catch(Exception e){


con.rollback();


}

```

---

Result:

```
Everything cancelled

```

---

# 13. Transaction Isolation Level

Database problem:

Multiple transactions.

Java:

```java
con.setTransactionIsolation(
Connection.TRANSACTION_REPEATABLE_READ
);

```

---

Levels:

## READ_UNCOMMITTED

Fastest.

Problem:

Dirty Read

---

## READ_COMMITTED

Common.

Prevent dirty read.

---

## REPEATABLE_READ

MySQL Default.

Good for POS.

---

## SERIALIZABLE

Safest.

Slow.

---

# Café POS Recommendation:

```
REPEATABLE_READ

+

Row Lock

```

---

# 14. Inventory Repository

Interface:

```java
public interface InventoryRepository {


Inventory findByIngredientId(
Long ingredientId
);



void decreaseStock(
Connection con,

Long ingredientId,

BigDecimal amount

);



}

```

---

# 15. Atomic Stock Update

Bad:

```sql
SELECT quantity


UPDATE quantity

```

---

Better:

One SQL:

```sql
UPDATE inventory

SET quantity = quantity - ?

WHERE ingredient_id = ?

AND quantity >= ?

```

---

Why?

Database handles concurrency.

---

Java:

```java
PreparedStatement ps =
con.prepareStatement(sql);


ps.setBigDecimal(
1,
amount
);


ps.setLong(
2,
ingredientId
);


ps.setBigDecimal(
3,
amount
);

```

---

If affected rows:

```
1

Success


```

If:

```
0

Not enough stock

```

---

# 16. Custom Exception

Create:

```
exception


InsufficientStockException.java

```

```java
public class InsufficientStockException
extends RuntimeException{


public InsufficientStockException(
String message
){

super(message);

}


}

```

---

Usage:

```java
if(updatedRows==0){

throw new InsufficientStockException(

"Stock not enough"

);

}

```

---

# 17. Inventory Service

Business Logic:

```java
public class InventoryService{


private final InventoryRepository repository;



public void decrease(

Connection con,

Long ingredientId,

BigDecimal qty

){


repository.decreaseStock(

con,

ingredientId,

qty

);


}



}

```

---

# 18. Order + Inventory Transaction

Complete flow:

```java
public void createOrder(Order order){


Connection con=null;


try{


con =
database.getConnection();



con.setAutoCommit(false);



Long orderId =
orderRepository.save(
con,
order
);



for(OrderItem item:
order.items()){


inventoryService.reduce(

con,

item

);


}



paymentRepository.save(
con,
payment
);



con.commit();



}

catch(Exception e){


con.rollback();


throw e;


}


}

```

---

# 19. Purchase Management

Inventory also increases.

Example:

Supplier delivers:

```
Milk

5000 ml

```

Purchase:

```
Supplier

+

Purchase

+

Stock Increase

```

---

Tables:

## suppliers

```sql
CREATE TABLE suppliers(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100),

phone VARCHAR(30)

);

```

---

## purchases

```sql
CREATE TABLE purchases(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

supplier_id BIGINT,

total_amount DECIMAL(10,2),

created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

```

---

# 20. Purchase Items

```sql
CREATE TABLE purchase_items(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

purchase_id BIGINT,

ingredient_id BIGINT,

quantity DECIMAL(10,2),

price DECIMAL(10,2)

);

```

---

# 21. Stock Increase Flow

```
Purchase


 |

PurchaseService


 |

InventoryService


 |

Increase Stock


 |

Stock Movement

```

---

# 22. Inventory UI Design

Professional POS:

```
+------------------------------------------------+

Inventory Management


Search Ingredient

[ Milk________ ]


+-----------------------------------------------+

Ingredient | Stock | Unit | Status

-----------------------------------------------

Milk       |5000   | ml   | OK

Coffee     |2000   | g    | OK


+-----------------------------------------------+


[ Purchase Stock ]

[ Adjustment ]

[ History ]

```

---

# 23. Stock Status

Calculate:

```java
if(quantity.compareTo(
minimumStock
)<0)

{

status="LOW STOCK";

}

```

---

Display:

```
Coffee Bean

LOW STOCK

```

---

# 24. Background Stock Monitoring

Use Java 25 Virtual Thread:

```java
Thread.startVirtualThread(
()->{


while(true){


checkLowStock();


Thread.sleep(
60000
);


}


});

```

---

Every minute:

```
Check inventory

Send alert

```

---

# 25. Inventory Logging

Every change:

```
WHO

WHAT

WHEN

HOW MUCH

WHY


```

Example:

```
User:

Admin


Action:

Stock Adjustment


Ingredient:

Milk


Quantity:

+1000ml


Time:

10:30 AM

```

---

# 26. Inventory Exception Architecture

```
Exception

 |

AppException

 |

BusinessException

 |

InsufficientStockException

 |

InventoryUpdateException


```

---

# 27. Final Inventory Architecture

```
              Order Service

                   |

             Inventory Service

                   |

        ----------------------

        |                    |

 Inventory Repository   Recipe Repository


        |

        |

 Stock Movement


        |

        |

      MySQL


```

---

# 28. Production Checklist

Inventory System:

✅ Transaction Support

✅ Row Locking

✅ Atomic Update

✅ Stock History

✅ Recipe Calculation

✅ Purchase System

✅ Supplier Management

✅ Low Stock Alert

✅ Exception Handling

---

# Practice Task

Implement:

## Inventory Module

Create:

1. Inventory Entity
    
2. StockMovement Entity
    
3. InventoryRepository
    
4. InventoryService
    
5. Stock Transaction
    
6. Low Stock Checker
    
7. Inventory Swing Panel
    

---

# Lesson 26 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Enterprise Inventory Design  
✅ JDBC Transaction  
✅ Commit / Rollback  
✅ Isolation Level  
✅ Race Condition Prevention  
✅ SELECT FOR UPDATE  
✅ Atomic Stock Update  
✅ Stock Movement History  
✅ Purchase System  
✅ Supplier Module  
✅ Inventory Exception System  
✅ Virtual Thread Monitoring

---

# Next Lesson

# Lesson 27: Authentication & Authorization System

## Secure Login for Café POS

Topics:

- User Login UI
    
- Password Hashing
    
- BCrypt
    
- Role Based Access Control (RBAC)
    
- Admin / Cashier Permission
    
- Session Management
    
- Security Exception
    
- Audit Logging
    

ဒီ Lesson ပြီးရင် Café POS ကို **Multi User Secure Application** အဖြစ် ပြောင်းပါမယ်။