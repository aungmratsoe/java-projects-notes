# Part 3: Café POS Real Implementation Phase

# Lesson 42: Order Management Module (Part 1)

## Order Entity + Cart System + JTable + Checkout Architecture

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson ကနေ Café POS ရဲ့ **အဓိက POS Selling System** ကို စတင်ပါမယ်။

အခုအထိ:

```text
Authentication
        ✅

Dashboard
        ✅

Product Management
        ✅

Inventory
        ✅

Recipe + Stock Deduction
        ✅
```

ရှိပြီးပါပြီ။

အခု:

```text
Cashier

   ↓

Select Product

   ↓

Add To Cart

   ↓

Calculate Total

   ↓

Checkout

   ↓

Payment

   ↓

Inventory Deduction

   ↓

Receipt

```

ကို တည်ဆောက်ပါမယ်။

---

# 1. Real POS Screen Concept

Cashier က မြင်ရမယ့် Screen:

```text
+------------------------------------------------+

 Products                         Cart


 Latte                            Latte x2

 Cappuccino                       Cake x1

 Burger


 [Search Product]


                                  Total

                                  14,000 MMK


                                  [CHECKOUT]

+------------------------------------------------+

```

---

# 2. Order Module Architecture

Professional Flow:

```text
                 OrderPanel

                     |

                     |

             OrderController

                     |

                     |

              OrderService

                     |

        -----------------------

        |                     |

 OrderRepository       InventoryService


        |

      MySQL

```

---

# 3. Database Review

Already created:

## orders

```sql
id

user_id

total_amount

status

created_at

```

---

## order_items

```sql
id

order_id

product_id

quantity

price

```

---

Relationship:

```text
orders

   1

   |

   |

   Many

order_items

```

---

# 4. Package Structure

Create:

```text
module/order


├── model

│
├── Order.java

├── OrderItem.java

├── OrderStatus.java


├── repository

│
├── OrderRepository.java

├── OrderItemRepository.java


├── service

│
├── OrderService.java


├── controller

│
├── OrderController.java


└── view

    ├── OrderPanel.java

    ├── CartTableModel.java

    └── CheckoutDialog.java

```

---

# 5. Order Entity

Database:

```text
orders

id

user_id

total_amount

status

created_at

```

---

Create:

`Order.java`

```java
package com.cafe.pos.module.order.model;


import java.time.LocalDateTime;


public record Order(

Long id,

Long userId,

double totalAmount,

OrderStatus status,

LocalDateTime createdAt

){

}

```

---

# 6. Order Status Enum

Create:

`OrderStatus.java`

```java
package com.cafe.pos.module.order.model;


public enum OrderStatus {


PENDING,

PAID,

CANCELLED


}

```

---

# 7. Order Item Entity

Customer Cart Item:

```text
Latte

Quantity:

2

Price:

5000

```

---

Create:

`OrderItem.java`

```java
package com.cafe.pos.module.order.model;


public record OrderItem(

Long id,

Long orderId,

Long productId,

String productName,

int quantity,

double price

){

}

```

---

# 8. Cart Concept

Cart ≠ Order

Important:

Before Checkout:

```text
Cart

Temporary Data

Memory Only

```

After Checkout:

```text
Order

Saved Database

```

---

Flow:

```text
Product

   ↓

Cart

   ↓

Checkout

   ↓

Order

   ↓

Database

```

---

# 9. Cart Model

Create:

```text
model

CartItem.java

```

---

Code:

```java
package com.cafe.pos.module.order.model;


public record CartItem(

Long productId,

String productName,

double price,

int quantity

){



public double subtotal(){

return price * quantity;

}


}

```

---

# 10. Cart Management Class

Create:

```text
service

CartService.java

```

---

Code:

```java
package com.cafe.pos.module.order.service;


import java.util.*;

import com.cafe.pos.module.order.model.CartItem;



public class CartService {



private final List<CartItem> items =
new ArrayList<>();



public void addItem(
CartItem item
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



public List<CartItem> getItems(){

return items;

}



public double getTotal(){


return items.stream()

.mapToDouble(
CartItem::subtotal
)

.sum();


}



public void clear(){

items.clear();

}


}

```

---

# 11. Cart Flow Example

Add Latte:

```java
cart.addItem(

new CartItem(

1L,

"Latte",

5000,

2

)

);

```

---

Total:

```java
double total =
cart.getTotal();

```

Result:

```
10000
```

---

# 12. Order Repository Design

Interface:

```java
public interface OrderRepository {


Long save(Order order);


}

```

---

Why return ID?

Because:

Need:

```text
Order ID

        ↓

Order Items Insert

```

---

Example:

```text
Order

ID = 100


Order Items:


100 Latte

100 Cake

```

---

# 13. Order Creation Flow

Checkout:

```text
Cart

 ↓

Create Order

 ↓

Get Order ID

 ↓

Save Items

 ↓

Deduct Stock

 ↓

Payment

 ↓

Commit

```

---

# 14. Order Service

Create:

`OrderService.java`

```java
package com.cafe.pos.module.order.service;



public class OrderService {



public void checkout(
CartService cart
){


double total =
cart.getTotal();



System.out.println(
"Total: "
+
total
);



}


}

```

---

Later we add:

```text
Transaction

Inventory

Payment

Receipt

```

---

# 15. Order JTable Model

Cart Display:

```text
+--------------------------------+

Product | Qty | Price | Subtotal


Latte     2    5000    10000


Cake      1    4000    4000


+--------------------------------+

```

---

Create:

`CartTableModel.java`

```java
public class CartTableModel
extends AbstractTableModel {



private List<CartItem> items =
new ArrayList<>();


private String[] columns =
{
"Product",
"Qty",
"Price",
"Subtotal"
};



public void setItems(
List<CartItem> items
){

this.items=items;

fireTableDataChanged();

}


@Override

public int getRowCount(){

return items.size();

}



@Override

public int getColumnCount(){

return columns.length;

}



@Override

public Object getValueAt(
int row,
int col
){


CartItem item =
items.get(row);



return switch(col){

case 0 ->
item.productName();


case 1 ->
item.quantity();


case 2 ->
item.price();


case 3 ->
item.subtotal();


default ->
null;

};


}

}

```

---

# 16. OrderPanel Layout

Professional:

```text
+------------------------------------------------+

 Search Product:

 [____________]


 Products Table


+----------------------+-------------------------+

| Product              | Cart                    |

|----------------------|-------------------------|

| Latte                | Latte x2                |

| Cake                 | Cake x1                 |

| Burger               |                         |


+----------------------+-------------------------+


                    Total: 14000


                    [CHECKOUT]


+------------------------------------------------+

```

---

# 17. Add To Cart Logic

User clicks product:

```java
productTable
.addMouseListener(
new MouseAdapter(){


public void mouseClicked(
MouseEvent e
){


CartService
.addItem(product);


}


});

```

---

# 18. Checkout Button

Flow:

```text
CHECKOUT


    ↓


Validate Cart


    ↓


OrderService


    ↓


Create Order


    ↓


Payment Dialog


```

---

# 19. Important Design Rule

Cart:

```text
Temporary

RAM

No Database

```

Order:

```text
Permanent

Database

```

---

# 20. Current Progress

Completed:

```text
Database Design          ✅

Authentication           ✅

Dashboard                ✅

Product Module           ✅

Inventory                ✅

Recipe System            ✅

Cart Architecture        ✅

Order Entity             ✅

Order Item               ✅

Checkout Design          ✅

```

---

# Practice Task

Create:

1. Order.java
    
2. OrderItem.java
    
3. CartItem.java
    
4. CartService.java
    
5. CartTableModel.java
    
6. OrderPanel UI skeleton
    

---

# Next Lesson

# Lesson 43: Order Management Module (Part 2)

## Product Selection + Cart UI + Checkout Transaction

Next we implement:

✅ Product Search Panel  
✅ Add Product To Cart  
✅ Quantity Update  
✅ Remove Cart Item  
✅ Total Calculation  
✅ Checkout Button  
✅ Database Order Save

ပြီးရင် Cashier က **တကယ် Order ရိုက်ပြီး ရောင်းနိုင်တဲ့ Screen** ဖြစ်လာပါမယ်။