# Part 3: Café POS Real Implementation Phase

# Lesson 45: Order Management Module (Part 4)

## Advanced POS Features

### Discount + Tax + Multiple Payment + Order History

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Basic Checkout System ကို **Professional Café POS Level** အဖြစ် Upgrade လုပ်ပါမယ်။

အခုအထိ:

```text
Product Management       ✅

Inventory                ✅

Recipe System            ✅

Cart                     ✅

Order Creation           ✅

Payment                  ✅

Receipt                  ✅

```

ရှိပြီးပါပြီ။

Enterprise POS မှာ လိုအပ်တဲ့:

```text
Order

 ↓

Discount

 ↓

Tax

 ↓

Multiple Payment

 ↓

Order History

 ↓

Sales Report

```

ကို တည်ဆောက်ပါမယ်။

---

# 1. Real Café Checkout Calculation

Before:

```text
Latte       5000

Cake        4000

Burger      6000

----------------

Subtotal   15000

```

---

Add Discount:

```text
Discount 10%

-1500

```

---

Tax:

```text
Tax 5%

675

```

---

Final:

```text
Subtotal       15000

Discount       -1500

Tax             675

---------------------

TOTAL          14175 MMK

```

---

# 2. Order Calculation Architecture

Wrong:

```java
double total =
price - discount + tax;

```

UI ထဲမှာ မလုပ်ရပါ။

---

Correct:

```text
OrderPanel

    |

OrderController

    |

OrderService

    |

PricingService

```

---

# 3. Create Pricing Module

Package:

```text
module/pricing


├── model

│
├── Discount.java

├── Tax.java


├── service

│
└── PricingService.java

```

---

# 4. Discount Model

Create:

`Discount.java`

```java
package com.cafe.pos.module.pricing.model;



public record Discount(


String name,


double percentage


){

}

```

---

Example:

```java
new Discount(

"Member Discount",

10

);

```

---

# 5. Tax Model

Create:

`Tax.java`

```java
public record Tax(

String name,

double percentage

){

}

```

---

Example:

```java
Tax tax =
new Tax(

"Commercial Tax",

5

);

```

---

# 6. Pricing Service

Business Rule:

```text
Subtotal

      ↓

Discount

      ↓

Tax

      ↓

Final Total

```

---

Create:

`PricingService.java`

```java
package com.cafe.pos.module.pricing.service;


public class PricingService {



public double calculateDiscount(

double subtotal,

Discount discount

){


return subtotal *
discount.percentage()
/
100;


}



public double calculateTax(

double amount,

Tax tax

){


return amount *
tax.percentage()
/
100;


}



public double calculateTotal(

double subtotal,

Discount discount,

Tax tax

){


double discountAmount =
calculateDiscount(
subtotal,
discount
);



double afterDiscount =
subtotal -
discountAmount;



double taxAmount =
calculateTax(
afterDiscount,
tax
);



return afterDiscount +
taxAmount;


}


}

```

---

# 7. Example

Input:

```text
Subtotal:

20000


Discount:

10%


Tax:

5%

```

Calculation:

```
Discount:

20000 × 10%

= 2000


After Discount:

18000


Tax:

18000 × 5%

= 900


Total:

18900

```

---

# 8. Database Update

Orders table:

Add:

```sql
ALTER TABLE orders

ADD COLUMN discount_amount
DECIMAL(10,2)
DEFAULT 0;



ALTER TABLE orders

ADD COLUMN tax_amount
DECIMAL(10,2)
DEFAULT 0;


```

---

Final:

```text
orders


id

subtotal

discount_amount

tax_amount

total_amount

status

```

---

# 9. Order Entity Update

Before:

```java
Order(

id,

totalAmount,

status

)

```

---

After:

```java
public record Order(


Long id,


Long userId,


double subtotal,


double discountAmount,


double taxAmount,


double totalAmount,


OrderStatus status


){

}

```

---

# 10. Multiple Payment System

Real Customer:

```text
Total:

50000 MMK


Pay:

Cash

30000


Card

20000

```

---

Need:

```text
Order

    |

    |

Payments

    |

    |

Many

```

---

# 11. Database Design

Change:

Before:

```text
orders

   |

payment

```

---

After:

```text
orders

    |

    |

payments


```

---

payments:

```sql
CREATE TABLE payments
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


order_id BIGINT,


method VARCHAR(30),


amount DECIMAL(10,2)


);

```

---

# 12. Payment Entity Update

```java
public record Payment(


Long id,


Long orderId,


PaymentMethod method,


double amount


){

}

```

---

# 13. Split Payment Validation

Example:

Total:

```text
50000

```

Payments:

```text
Cash 30000

Card 15000

```

Wrong:

```text
45000

```

Need:

```java
if(
paidAmount < total
){

throw new PaymentException(
"Payment incomplete"
);

}

```

---

# 14. Order History Module

Cashier / Manager needs:

```text
Today's Orders


Order No

Customer

Amount

Status

Time


1001

45000

PAID

10:30


1002

12000

PAID

11:00

```

---

# 15. Order History Package

Create:

```text
order/history


OrderHistoryPanel.java

OrderHistoryTableModel.java

```

---

# 16. Order History Query

SQL:

```sql
SELECT *

FROM orders

ORDER BY created_at DESC;

```

---

Today:

```sql
SELECT *

FROM orders

WHERE DATE(created_at)
=
CURRENT_DATE;

```

---

# 17. Order History JTable

Columns:

```text
Order ID

Date

Total

Status

Cashier

```

---

TableModel:

```java
public class OrderHistoryTableModel

extends AbstractTableModel {


private String[] columns =
{

"ID",

"Date",

"Total",

"Status"

};


}

```

---

# 18. View Order Details

Double Click:

```text
Order #1001


Items:


Latte x2

Cake x1


Payment:


Cash 30000

Card 20000


```

---

# 19. Complete Advanced Checkout Flow

Now:

```text
Cart


 ↓


PricingService


 ↓


Discount


 ↓


Tax


 ↓


OrderService


 ↓


Transaction


 ├── Save Order

 ├── Save Items

 ├── Save Payments

 ├── Reduce Inventory

 └── Save Stock Movement


 ↓


Receipt

```

---

# 20. Current Café POS Level

Completed:

```text
Login System             ✅

Dashboard                ✅

Product CRUD             ✅

Category System          ✅

Inventory                ✅

Recipe Engine            ✅

Automatic Stock Control  ✅

Cart System              ✅

Order System             ✅

Payment System           ✅

Receipt                  ✅

Discount                 ✅

Tax                      ✅

Order History             🔄

```

---

# Practice Task

Implement:

1. PricingService
    
2. Discount Model
    
3. Tax Model
    
4. Update Order Entity
    
5. Multiple Payment Design
    
6. Order History JTable
    

---

# Next Lesson

# Lesson 46: Reporting Module (Part 1)

## Sales Report + Daily Revenue + Dashboard Analytics

Next we start Management Features:

```text
Manager Dashboard

      ↓

Today's Sales

      ↓

Best Selling Product

      ↓

Revenue Chart

      ↓

Inventory Report

```

ပြီးရင် Café POS က Cashier System မဟုတ်တော့ဘဲ **Complete Business Management System** ဖြစ်လာပါမယ်။