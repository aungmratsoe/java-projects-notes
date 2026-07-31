# Part 2: Advanced Java Knowledge for Swing

# Lesson 16: Java Design Patterns for Professional Swing Applications

## Building Enterprise Café POS Architecture (Java 25 + Swing + MVC)

ဒီ Lesson က **Professional Java Developer Level** အတွက် အရေးကြီးပါတယ်။

အခုကစပြီး ကျွန်တော်တို့က Java syntax မဟုတ်တော့ဘဲ **Software Architecture Design** ကို လေ့လာပါမယ်။

Real-world Café POS System တစ်ခုမှာ code ကို စနစ်တကျ ခွဲမထားရင်:

- Class 100+ ဖြစ်လာမယ်
    
- Bug ရှာရခက်မယ်
    
- Feature အသစ်ထည့်ရခက်မယ်
    
- Team အများကြီးနဲ့ develop မလုပ်နိုင်ဘူး
    

ဒါကြောင့် Design Pattern တွေသုံးပါတယ်။

---

# 1. What is Design Pattern?

Design Pattern ဆိုတာ:

> Software Design မှာ အမြဲတွေ့ရတဲ့ ပြဿနာတွေကို ဖြေရှင်းဖို့ အသုံးပြုတဲ့ proven solution ဖြစ်ပါတယ်။

Pattern က code မဟုတ်ပါ။

ဒါက:

```
Problem

   |

Design Idea

   |

Reusable Solution

```

ဖြစ်ပါတယ်။

---

# 2. Why Design Patterns?

Without Pattern:

```
Button Click

     |

SQL Query

     |

Database

     |

Calculate

     |

Print Receipt

```

Button class ထဲမှာ အကုန်ထည့်ထားရင်:

```
OrderButton.java

5000 lines

```

ဖြစ်နိုင်ပါတယ်။

---

With Pattern:

```
View

 |

Controller

 |

Service

 |

Repository

 |

Database

```

---

# 3. Design Pattern Categories

Java မှာ အဓိက 3 မျိုးရှိပါတယ်။

---

## 1. Creational Patterns

Object creation ကို manage လုပ်တာ။

Examples:

- Singleton
    
- Factory
    
- Builder
    

---

## 2. Structural Patterns

Class/Object relationship design.

Examples:

- Adapter
    
- Decorator
    

---

## 3. Behavioral Patterns

Object communication.

Examples:

- Observer
    
- Strategy
    
- Command
    

---

# 4. MVC Pattern (Most Important for Swing)

Swing Application အတွက် MVC က အခြေခံ architecture ဖြစ်ပါတယ်။

MVC:

```
        User

         |
         v

+----------------+

|      View      |

|   (Swing UI)   |

+----------------+

         |

         v

+----------------+

|  Controller    |

+----------------+

         |

         v

+----------------+

|     Model      |

| Business Data  |

+----------------+

```

---

# 5. Café POS MVC Example

## Model

Data:

```
Product
Order
Customer
Payment

```

Example:

```java
public record Product(
    int id,
    String name,
    double price
) {}

```

---

## View

Swing UI:

```java
public class ProductFrame
extends JFrame {


JTable table;

JButton saveButton;


}

```

---

## Controller

Between View and Model:

```java
public class ProductController {


private ProductService service;


public void saveProduct(Product p){

service.save(p);

}


}

```

---

Flow:

```
User Click Save

      |

View

      |

Controller

      |

Service

      |

Repository

      |

Database

```

---

# 6. Singleton Pattern

## Problem

Database connection object:

```
Application

    |
    |
Connection 1

Connection 2

Connection 3

```

Unnecessary resources.

---

Need:

```
One shared instance

```

---

Singleton:

```java
public class DatabaseManager {


private static DatabaseManager instance;


private DatabaseManager(){

}


public static DatabaseManager getInstance(){

if(instance == null){

instance =
new DatabaseManager();

}

return instance;

}

}

```

---

Usage:

```java
DatabaseManager db =
DatabaseManager.getInstance();

```

---

# 7. Singleton in Café POS

Examples:

Good candidates:

```
DatabaseManager

Logger

ConfigurationManager

PrinterManager

```

---

Architecture:

```
Application

    |

DatabaseManager

    |

MySQL

```

---

# 8. Singleton Thread Safety

Problem:

Two threads:

```
Thread A

checks instance null


Thread B

checks instance null

```

Two objects created.

---

Solution:

```java
public static synchronized
DatabaseManager getInstance(){

}

```

---

Modern Java:

Use Holder Pattern:

```java
public class DatabaseManager {


private DatabaseManager(){}


private static class Holder{


static final DatabaseManager INSTANCE =
new DatabaseManager();


}


public static DatabaseManager getInstance(){

return Holder.INSTANCE;

}


}

```

---

# 9. Factory Pattern

## Problem

Object creation logic everywhere.

Bad:

```java
if(type.equals("CASH")){

payment =
new CashPayment();

}

else if(type.equals("CARD")){

payment =
new CardPayment();

}

```

---

Solution:

Factory

```java
public class PaymentFactory {


public static Payment create(
String type
){


return switch(type){


case "CASH" ->
new CashPayment();


case "CARD" ->
new CardPayment();


default ->
throw new IllegalArgumentException();


};


}

}

```

---

Usage:

```java
Payment payment =
PaymentFactory.create("CARD");

```

---

# 10. Café POS Factory Example

Payment Types:

```
Payment

 |

----------------

Cash

Card

MobilePay

```

Factory controls creation.

---

Benefits:

- Easy extension
    
- Less if/else
    
- Cleaner code
    

---

# 11. Builder Pattern

## Problem

Object has many fields.

Example:

```java
Order order =
new Order(
id,
customer,
items,
discount,
tax,
payment,
date
);

```

Hard to read.

---

Builder:

```java
Order order =
new Order.Builder()

.customer("John")

.addItem(coffee)

.discount(10)

.build();

```

---

# 12. Café POS Order Builder

Example:

```java
public class Order {


private String customer;

private double discount;


private Order(Builder b){

this.customer=b.customer;
this.discount=b.discount;

}


public static class Builder{


private String customer;

private double discount;


public Builder customer(
String c
){

customer=c;

return this;

}


public Builder discount(
double d
){

discount=d;

return this;

}


public Order build(){

return new Order(this);

}


}

}

```

---

Usage:

```java
Order order =
new Order.Builder()

.customer("Alice")

.discount(5)

.build();

```

---

# 13. Strategy Pattern

## Problem

Different algorithms.

Example:

Payment calculation:

```
Cash

Card

Mobile Payment

```

Each has different logic.

---

Wrong:

```java
if(payment=="CARD")

calculateCard();


else if(payment=="CASH")

calculateCash();

```

---

Strategy:

```
PaymentStrategy

       |

----------------

CashStrategy

CardStrategy

```

---

Interface:

```java
public interface PaymentStrategy {


void pay(double amount);


}

```

---

Cash:

```java
public class CashStrategy
implements PaymentStrategy{


public void pay(double amount){

System.out.println(
"Cash payment"
);

}

}

```

---

Use:

```java
PaymentStrategy strategy =
new CashStrategy();


strategy.pay(5000);

```

---

# 14. Observer Pattern

Used for event notification.

Swing itself uses Observer concept.

Example:

```
New Order Created

        |

 ----------------

Kitchen

Printer

Inventory

```

---

Interface:

```java
interface Observer {


void update();


}

```

---

Example:

When sale happens:

```
SaleService

   |

Notify

   |

Printer

Inventory

Report

```

---

# 15. Repository Pattern

Database access separation.

Without:

```
Controller

 |
 |
SQL

```

Bad.

---

With:

```
Controller

 |

Service

 |

Repository

 |

Database

```

---

Example:

```java
public interface ProductRepository {


List<Product> findAll();


void save(Product p);


}

```

---

Implementation:

```java
public class MySQLProductRepository
implements ProductRepository{


public void save(Product p){

// JDBC code

}

}

```

---

# 16. Service Layer Pattern

Business logic location.

Example:

```
ProductController

        |

ProductService

        |

ProductRepository

```

---

Service:

```java
public class OrderService {


private OrderRepository repo;


public void createOrder(Order order){


validate(order);

repo.save(order);


}

}

```

---

# 17. Dependency Injection Concept

Problem:

Hard dependency:

```java
class Service{


Database db =
new Database();


}

```

---

Better:

```java
class Service{


private Database db;


Service(Database db){

this.db=db;

}

}

```

---

Now:

```
Service

receives

Database

```

---

Benefits:

- Testing easy
    
- Flexible
    
- Loose coupling
    

---

# 18. Complete Café POS Enterprise Architecture

Final design:

```
                 Swing View

                    |

               Controller

                    |

              Service Layer

                    |

             Repository Layer

                    |

              Database Layer


                    |

               MySQL


```

Supporting:

```
Singleton

Factory

Builder

Strategy

Observer

Exception Handler

Logging

Thread Pool

```

---

# 19. Java 25 + Patterns Example

Modern Model:

```java
public record OrderDTO(
int id,
double total
){}
```

---

Payment:

```java
sealed interface Payment
permits Cash,Card
{}

```

---

Factory:

```java
PaymentFactory.create();

```

---

Async:

```java
Thread.startVirtualThread(
orderService::process
);

```

---

# 20. Interview Questions

## Q1: Why use Design Patterns?

To provide reusable and maintainable solutions.

---

## Q2: Difference between Factory and Singleton?

Singleton:

```
One object

```

Factory:

```
Creates objects

```

---

## Q3: Why MVC in Swing?

To separate:

- UI
    
- Logic
    
- Data
    

---

## Q4: What is Repository Pattern?

Separates database operations from business logic.

---

# Practice Project

Design Café POS Architecture:

Create packages:

```
com.cafe.pos

 |
 |--model

 |--view

 |--controller

 |--service

 |--repository

 |--factory

 |--strategy

 |--exception

 |--util

```

Create:

```
ProductService

OrderService

PaymentFactory

DatabaseManager

ProductRepository

```

---

# Lesson 16 Summary

Today we learned:

✅ Design Pattern Concept  
✅ MVC Architecture  
✅ Singleton Pattern  
✅ Factory Pattern  
✅ Builder Pattern  
✅ Strategy Pattern  
✅ Observer Pattern  
✅ Repository Pattern  
✅ Service Layer  
✅ Dependency Injection  
✅ Enterprise Café POS Architecture

---

# Next Lesson

# Lesson 17: JDBC Advanced & Database Integration

## Building Professional MySQL Layer for Café POS

Topics:

- JDBC Architecture
    
- Connection Pool
    
- PreparedStatement
    
- Transaction Management
    
- Batch Processing
    
- DAO vs Repository
    
- MySQL Optimization
    
- Database Exception Handling
    
- Real Café POS Database Integration
    

ဒီ Lesson ပြီးရင် Java Swing + MySQL POS System ကို production architecture နဲ့ connect လုပ်နိုင်ပါမယ်။