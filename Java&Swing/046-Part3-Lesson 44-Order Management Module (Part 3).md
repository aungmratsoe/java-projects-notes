# Part 3: Café POS Real Implementation Phase

# Lesson 44: Order Management Module (Part 3)

## Complete Checkout + Payment System + Receipt Generation

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Café POS ရဲ့ **Complete Selling Transaction** ကို တည်ဆောက်ပါမယ်။

အခုအထိ:

```text
Product Management       ✅
Inventory Management     ✅
Recipe System            ✅
Cart System              ✅
Order Screen             ✅
```

ရှိပြီးပါပြီ။

ဒီနေ့ပြီးရင်:

```text
Cashier

 ↓

Cart

 ↓

Checkout

 ↓

Create Order

 ↓

Payment

 ↓

Receipt

 ↓

Inventory Update

 ↓

Transaction Complete

```

အထိ ရပါမယ်။

---

# 1. Real POS Checkout Concept

Restaurant မှာ Order တစ်ခုရောင်းတဲ့အခါ:

Example:

Customer:

```
Latte x2
Cake  x1
```

Total:

```
14,000 MMK
```

Cashier:

```
Customer pays:

20,000 MMK
```

System:

```
Change:

6,000 MMK

```

---

# 2. New Module Structure

Create:

```text
module/payment


├── model

│   Payment.java

│   PaymentMethod.java


├── repository

│   PaymentRepository.java


├── service

│   PaymentService.java


└── view

    PaymentDialog.java

```

---

# 3. Database Design

Create Payment Table:

```sql
CREATE TABLE payments
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


order_id BIGINT NOT NULL,


amount DECIMAL(10,2),


payment_method VARCHAR(30),


paid_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


CONSTRAINT fk_payment_order

FOREIGN KEY(order_id)

REFERENCES orders(id)

);

```

---

# 4. Payment Entity

Create:

```java
Payment.java

```

---

Code:

```java
package com.cafe.pos.module.payment.model;


import java.time.LocalDateTime;



public record Payment(


Long id,


Long orderId,


double amount,


PaymentMethod method,


LocalDateTime paidAt


){

}

```

---

# 5. Payment Method Enum

Create:

```java
PaymentMethod.java

```

---

Code:

```java
package com.cafe.pos.module.payment.model;


public enum PaymentMethod {


CASH,

CARD,

MOBILE_PAYMENT


}

```

---

# 6. Payment Flow

Architecture:

```
PaymentDialog

      |

      |

PaymentController

      |

      |

PaymentService

      |

      |

PaymentRepository

      |

      |

MySQL

```

---

# 7. Payment Dialog UI

Design:

```
+--------------------------------+

 Payment


 Total:

 14000 MMK


 Received:

 [20000]


 Method:


 (•) CASH

 ( ) CARD

 ( ) MOBILE


 Change:

 6000 MMK



        [PAY]


+--------------------------------+

```

---

# 8. PaymentDialog Class

Create:

```java
public class PaymentDialog
extends JDialog {


private JTextField amountField;


private JLabel changeLabel;


}

```

---

# 9. Calculate Change

Logic:

```java
double received =
Double.parseDouble(
amountField.getText()
);



double change =
received - total;



changeLabel.setText(

"Change: "
+
change

);

```

---

# 10. Validation

Before payment:

Check:

```
Received >= Total

```

Otherwise:

```java
throw new PaymentException(

"Insufficient payment"

);

```

---

# 11. Custom Payment Exception

Create:

```
exception

   |

   PaymentException.java

```

---

Code:

```java
package com.cafe.pos.exception;



public class PaymentException
extends RuntimeException {


public PaymentException(
String message
){

super(message);

}


}

```

---

# 12. Payment Service

Create:

```java
PaymentService.java

```

---

Code:

```java
public class PaymentService {



private final PaymentRepository repository;



public PaymentService(
PaymentRepository repository
){

this.repository =
repository;

}



public void processPayment(
Payment payment
){


repository.save(payment);


}


}

```

---

# 13. Payment Repository

Interface:

```java
public interface PaymentRepository {


void save(
Payment payment
);


}

```

---

# 14. Checkout Transaction

Important Part.

Order + Payment + Inventory

must be ONE transaction.

Wrong:

```
Save Order

COMMIT


Payment Failed


```

Problem:

Order exists but no payment.

---

Correct:

```
BEGIN TRANSACTION


Save Order


Save Order Items


Decrease Stock


Save Payment


COMMIT


```

---

# 15. OrderService Complete Checkout

Update:

```java
public void checkout(
List<CartItem> items,
Payment payment
){


Connection con =
database.getConnection();



try{


con.setAutoCommit(false);



Long orderId =
orderRepository
.save(items);



inventoryService
.consume(items);



payment.setOrderId(
orderId
);



paymentRepository
.save(payment);



con.commit();



}


catch(Exception e){


con.rollback();


throw e;


}



}

```

---

# 16. Receipt System

After payment:

Generate:

```
=======================

       JAVA CAFÉ


Order No: 10001


Latte        2 x 5000

Cake         1 x 4000


-----------------------

TOTAL:

14000 MMK


Payment:

CASH


Received:

20000


Change:

6000


=======================

Thank You

=======================

```

---

# 17. Receipt Model

Create:

```
receipt

   |

Receipt.java

```

---

Code:

```java
public record Receipt(


Long orderId,


List<CartItem> items,


double total,


Payment payment


){

}

```

---

# 18. Receipt Service

Create:

```
ReceiptService.java

```

---

Purpose:

```text
Order Data

    ↓

Format Text

    ↓

Print

```

---

Code:

```java
public class ReceiptService {



public String generate(
Receipt receipt
){


StringBuilder sb =
new StringBuilder();



sb.append(
"JAVA CAFE\n"
);



for(
CartItem item:
receipt.items()
){


sb.append(
item.productName()
)
.append("\n");


}



sb.append(
"TOTAL: "
)
.append(
receipt.total()
);



return sb.toString();

}


}

```

---

# 19. Print Receipt

Swing:

```java
JTextArea area =
new JTextArea();



area.setText(
receiptText
);



area.print();

```

---

# 20. Complete POS Flow

Final:

```
Customer


 ↓


Cashier


 ↓


OrderPanel


 ↓


Cart


 ↓


Checkout


 ↓


PaymentDialog


 ↓


OrderService


 ↓


Transaction


 ├── Order Save

 ├── Item Save

 ├── Inventory Deduction

 ├── Stock Movement

 └── Payment Save


 ↓


Receipt


 ↓


Print

```

---

# 21. Current System Progress

Completed:

```
Java 25 Setup              ✅

MVC Architecture           ✅

Database Architecture      ✅

Authentication             ✅

Dashboard                  ✅

Product Management         ✅

Inventory                  ✅

Recipe System              ✅

Cart                       ✅

Order Module               ✅

Payment System             ✅

Receipt Design             ✅

```

---

# 22. Professional Improvements Later

Production POS needs:

```
Barcode Scanner

Discount System

Tax System

Multiple Payment

Refund

Return Order

Receipt Template

Printer Integration

Daily Sales Report

```

---

# Practice Task

Implement:

1. Payment Entity
    
2. Payment Dialog
    
3. Payment Validation
    
4. Payment Repository
    
5. Checkout Transaction
    
6. Receipt Generator
    

---

# Next Lesson

# Lesson 45: Order Management Module (Part 4)

## Advanced POS Features

### Discount + Tax + Multiple Payment + Order History

Next we upgrade from simple cashier system to **Enterprise Café POS Level**:

```
Order

 +
Discount

 +
Tax

 +
Payment Split

 +
Sales History

```

ပြီးရင် Reporting Module ကို စတင်ပါမယ်။