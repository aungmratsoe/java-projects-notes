# Part 2: Advanced Java Swing + Café POS Project

# Lesson 25: Café POS Order Management Module

## Shopping Cart + Transaction + Inventory Update

### (Java 25 + Swing + MVC + JDBC + MySQL + Clean Architecture)

ဒီ Lesson မှာ Café POS ရဲ့ **အဓိက Core Feature** ကို တည်ဆောက်ပါမယ်။

Product Management ကနေတစ်ဆင့်:

> Customer က Product ရွေး → Cart ထဲထည့် → Order Create → Payment → Inventory လျှော့ → Receipt ထုတ်

ဆိုတဲ့ **Real POS Workflow** ကို Build လုပ်ပါမယ်။

---

# 1. Real Café POS Order Flow

Restaurant မှာ Customer တစ်ယောက်က:

```
Customer
   |
   |
Cashier selects products
   |
   |
Shopping Cart
   |
   |
Checkout
   |
   |
Create Order
   |
   |
Payment
   |
   |
Update Inventory
   |
   |
Print Receipt

```

---

# 2. Order Module Architecture

Previous Product Module နဲ့ပေါင်းမယ်:

```
                 Swing UI

                    |
                    |

              OrderController

                    |

              OrderService

                    |

        ---------------------

        |                   |

 OrderRepository     InventoryRepository


        |

       JDBC

        |

      MySQL

```

---

# 3. Database Tables Review

Order System အတွက်:

## orders

```
id
customer_id
user_id
total_amount
status
created_at

```

## order_items

```
id
order_id
product_id
quantity
price

```

## payments

```
id
order_id
payment_method
amount

```

---

# 4. Java Domain Model

## Order

Java 25 Record:

```java
package com.cafe.pos.model;


import java.math.BigDecimal;
import java.util.List;


public record Order(

Long id,

Long customerId,

Long userId,

List<OrderItem> items,

BigDecimal totalAmount,

String status

){

}

```

---

## OrderItem

```java
public record OrderItem(

Long productId,

String productName,

int quantity,

BigDecimal price

){

public BigDecimal subtotal(){

return price.multiply(
BigDecimal.valueOf(quantity)
);

}

}

```

---

# 5. Why OrderItem Has subtotal()?

Example:

Coffee:

```
Price = 5000

Quantity = 3

```

Calculation:

```
5000 x 3

= 15000

```

Business logic belongs in Domain Model.

---

# 6. Cart Concept

POS မှာ Cart ဆိုတာ:

> Temporary Order data before checkout

Example:

```
Cart


Coffee     2 x 5000

Cake       1 x 8000

Tea        1 x 3000


Total = 21000

```

---

# 7. Create Cart Class

Package:

```
service/cart

Cart.java

```

Code:

```java
public class Cart {


private final List<OrderItem> items =
new ArrayList<>();



public void addItem(
OrderItem item
){

items.add(item);

}



public void removeItem(
Long productId
){

items.removeIf(
item ->
item.productId()
.equals(productId)
);

}



public BigDecimal getTotal(){


return items.stream()

.map(OrderItem::subtotal)

.reduce(
BigDecimal.ZERO,
BigDecimal::add
);


}



public List<OrderItem> getItems(){

return List.copyOf(items);

}


}

```

---

# 8. Why Separate Cart?

Wrong:

```
OrderService

   |
   |
Temporary cart data

```

Problem:

Business logic mixed.

---

Correct:

```
Cart

 |

OrderService

```

---

# 9. Order UI Design

Professional POS Layout:

```
+------------------------------------------------+

 Product Search

 [ Coffee____________ ] 🔍


+----------------+------------------------------+

| Products       | Cart                         |

|                |                              |

| Coffee 5000    | Coffee x2       10000        |

| Cake 8000      | Cake x1         8000         |

| Tea 3000       |                              |

+----------------+------------------------------+


              Total: 18000


          [Checkout]


+------------------------------------------------+

```

---

# 10. OrderPanel Structure

Create:

```
view/order


OrderPanel.java

CartTableModel.java

PaymentDialog.java

```

---

# 11. OrderPanel Components

```java
public class OrderPanel extends JPanel {


private JTable productTable;


private JTable cartTable;


private Cart cart;


private OrderController controller;



}

```

---

# 12. Product Selection

User double click:

```
Coffee

```

Add:

```java
productTable.addMouseListener(
new MouseAdapter(){


public void mouseClicked(MouseEvent e){


if(e.getClickCount()==2){


addToCart();


}


}


});

```

---

# 13. Add Product To Cart

Flow:

```
Table

 |

Selected Product

 |

Cart

 |

Refresh Cart JTable


```

---

Code:

```java
private void addToCart(Product product){


OrderItem item =
new OrderItem(

product.id(),

product.name(),

1,

product.price()

);



cart.addItem(item);


refreshCart();


}

```

---

# 14. Cart JTable Model

Columns:

```
Product

Quantity

Price

Subtotal


```

---

Example:

```
Coffee | 2 | 5000 | 10000

```

---

# 15. Quantity Update

Professional POS:

```
+

-

buttons

```

---

Increase:

```java
quantity++;

```

Decrease:

```java
quantity--;

```

---

# 16. Checkout Button

Button:

```
[ Checkout ]

```

Flow:

```
Click

 |

Validate Cart

 |

Create Order

 |

Save Database

 |

Payment

 |

Print Receipt


```

---

# 17. OrderController

```java
public class OrderController {


private final OrderService service;



public void checkout(
Order order
){

service.createOrder(order);


}


}

```

---

# 18. OrderService

Most important.

```java
public class OrderService {


private final OrderRepository orderRepository;


private final InventoryRepository inventoryRepository;



public void createOrder(
Order order
){


validate(order);



orderRepository.save(order);



inventoryRepository
.reduceStock(order.items());



}

}

```

---

# 19. Database Transaction

Important:

Order creation has multiple steps:

```
INSERT orders


INSERT order_items


UPDATE inventory


INSERT payment

```

---

If one fails:

Example:

```
Order saved

Inventory failed


```

Problem:

Customer order exists but stock wrong.

---

Solution:

Transaction.

---

# 20. Transaction Flow

```
BEGIN TRANSACTION


      |

Create Order


      |

Create Items


      |

Reduce Stock


      |

Create Payment


      |

COMMIT



```

---

Failure:

```
ROLLBACK

```

---

# 21. JDBC Transaction Code

```java
Connection con =
database.getConnection();


try{


con.setAutoCommit(false);



saveOrder(con);


saveItems(con);


updateInventory(con);



con.commit();



}
catch(Exception e){


con.rollback();


throw e;


}

```

---

# 22. Inventory Deduction Logic

Example:

Before:

```
Milk

1000 ml

```

Coffee recipe:

```
Milk 100ml

```

Order:

```
Coffee x5

```

Need:

```
100ml x 5

=

500ml

```

After:

```
500ml

```

---

# 23. Recipe Based Inventory

Important Café feature.

Flow:

```
Order Item

      |

Product Recipe

      |

Ingredient

      |

Inventory

```

---

Example:

```
Coffee

 |

Coffee Bean 20g

Milk 100ml


```

---

# 24. Inventory Service

```java
public class InventoryService {


public void deduct(
List<OrderItem> items
){



for(OrderItem item:items){


List<Recipe> recipes =
recipeRepository
.findByProduct(
item.productId()
);



for(Recipe r:recipes){


inventoryRepository
.decrease(

r.ingredientId(),

r.quantity()
.multiply(
item.quantity()
)

);


}


}



}


}

```

---

# 25. Payment Model

```java
public record Payment(

Long orderId,

String method,

BigDecimal amount

){

}

```

---

Payment methods:

```
CASH

CARD

KBZ_PAY

WAVE_PAY


```

---

# 26. Payment Dialog UI

```
+--------------------+

Total:

18000


Payment:


( ) Cash

( ) Card

( ) Mobile


[Confirm]


+--------------------+

```

---

# 27. Receipt Generation

After success:

```
Café POS

----------------

Coffee     10000

Cake        8000


Total      18000


Thank You!


```

---

Use:

```
PrinterService

```

---

# 28. Threading Integration

Printing should not block UI.

Wrong:

```
Checkout

 |

Print

 |

UI Freeze


```

---

Correct:

```
Checkout

 |

Virtual Thread

 |

Printer


```

---

Example:

```java
Thread.startVirtualThread(
()->{


printer.print(order);


});

```

---

# 29. Exception Handling

Possible errors:

## Empty Cart

```
CartEmptyException

```

## Out of Stock

```
InsufficientStockException

```

## Payment Failed

```
PaymentException

```

---

# 30. Order Module Package Structure

```
order


├── OrderPanel.java

├── Cart.java

├── OrderController.java

├── OrderService.java

├── OrderRepository.java

├── OrderItem.java

├── PaymentService.java

└── ReceiptService.java


```

---

# 31. Complete POS Workflow

```
Cashier Login

      |

Open Order Screen

      |

Select Products

      |

Add To Cart

      |

Checkout

      |

Payment

      |

Save Order

      |

Update Inventory

      |

Print Receipt


```

---

# 32. Professional POS Features

Future:

✅ Barcode Scanner

✅ Customer Display

✅ Discount System

✅ Loyalty Points

✅ Split Payment

✅ Kitchen Display System

✅ Offline Mode

---

# Lesson 25 Practice Task

Implement:

### Order Module

Create:

1. Cart Class
    
2. Order Model
    
3. OrderItem Model
    
4. OrderPanel UI
    
5. Cart JTable
    
6. Checkout Button
    
7. Order Transaction
    
8. Inventory Deduction
    

---

# Lesson 25 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ POS Order Workflow  
✅ Shopping Cart Design  
✅ Order Domain Model  
✅ JTable Cart UI  
✅ Checkout Flow  
✅ Order Service  
✅ Database Transaction  
✅ Inventory Deduction  
✅ Recipe Based Stock System  
✅ Payment Architecture  
✅ Receipt Processing  
✅ Virtual Thread Printing

---

# Next Lesson

# Lesson 26: Advanced Transaction Management + Inventory System

## Enterprise Stock Control for Café POS

Topics:

- JDBC Transaction Deep Dive
    
- Isolation Level
    
- Locking
    
- Inventory Race Condition
    
- Atomic Stock Update
    
- Stock Movement History
    
- Purchase Management
    
- Supplier Module
    

ဒီ Lesson မှာ **Café POS ကို Production Level Inventory System** အဖြစ် တိုးချဲ့ပါမယ်။