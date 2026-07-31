# Part 2: Advanced Java Knowledge for Swing

# Lesson 21: Enterprise Application Architecture

## Building Complete Café POS System Structure

### (Java 25 + Swing + MVC + Service + Repository + Clean Architecture)

ဒီ Lesson ကနေစပြီး ကျွန်တော်တို့ဟာ **Café POS System ကို Senior Java Developer တစ်ယောက် Design လုပ်သလို** စဉ်းစားပါမယ်။

အရင် Lesson တွေမှာ:

- Exception Handling
    
- Logging
    
- JDBC
    
- Threading
    
- Design Patterns
    

တွေကို လေ့လာပြီးပါပြီ။

အခု အဲဒီအရာတွေကို **တစ်ခုတည်းသော Enterprise Architecture** အဖြစ် ပေါင်းစည်းပါမယ်။

---

# 1. What is Software Architecture?

Architecture ဆိုတာ:

> Application ရဲ့ အကြီးစားဖွဲ့စည်းပုံ (Structure) ဖြစ်ပါတယ်။

ဥပမာ အိမ်တစ်လုံးဆောက်ရင်:

```text
Foundation
 |
Structure
 |
Rooms
 |
Electric System
 |
Decoration

```

Software မှာ:

```text
Database
 |
Data Access
 |
Business Logic
 |
UI
```

ဖြစ်ပါတယ်။

---

# 2. Why Architecture is Important?

Small Application:

```text
Main.java

Product.java

Database.java

```

အဆင်ပြေပါတယ်။

---

Large Application:

```text
1000 Classes

100 Features

20 Developers

```

Architecture မရှိရင်:

```text
Bug Fix Difficult

New Feature Difficult

Testing Difficult

```

ဖြစ်ပါတယ်။

---

# 3. Café POS Enterprise Requirements

System မှာ ပါမယ့် Features:

## User Management

- Login
    
- Role
    
- Permission
    

## Product Management

- Add Product
    
- Update Product
    
- Delete Product
    

## Inventory

- Stock Control
    
- Stock History
    

## Order

- Create Order
    
- Cancel Order
    
- Receipt
    

## Payment

- Cash
    
- Card
    
- Mobile Pay
    

## Reporting

- Daily Sales
    
- Monthly Report
    

---

# 4. Bad Architecture Example

Beginner style:

```text
Swing Button

     |
     |
SQL Query

     |
     |
MySQL

```

Example:

```java
button.addActionListener(e -> {


Connection con =
DriverManager.getConnection();


PreparedStatement ps =
con.prepareStatement(
"INSERT INTO orders..."
);


});

```

Problem:

Button knows:

- Database
    
- SQL
    
- Business Logic
    

Everything is coupled.

---

# 5. Professional Architecture

We separate:

```text
Presentation Layer

        |

Application Layer

        |

Domain Layer

        |

Infrastructure Layer

```

---

# 6. Complete Café POS Architecture

```text
                 Swing UI
                    |
                    |
              Controller
                    |
                    |
              Service Layer
                    |
                    |
          Repository Interface
                    |
                    |
       Repository Implementation
                    |
                    |
                 JDBC
                    |
                    |
                 MySQL

```

---

# 7. Package Structure

Professional package:

```
com.cafe.pos

|
├── app
│
├── config
│
├── model
│
├── dto
│
├── controller
│
├── service
│
├── repository
│
├── repository.impl
│
├── database
│
├── exception
│
├── security
│
├── logging
│
├── util
│
└── view

```

---

# 8. Layer Explanation

# 1. View Layer

Location:

```
view

```

Contains:

- JFrame
    
- JPanel
    
- JTable
    
- Dialog
    

Example:

```java
public class ProductFrame
extends JFrame {


private JTable table;


}

```

---

View does NOT:

❌ SQL

❌ Database

❌ Business Rules

---

# 9. Controller Layer

Controller receives UI actions.

Example:

User clicks:

```text
Save Product Button

```

Controller:

```java
public class ProductController {


private ProductService service;


public void save(Product product){


service.save(product);


}


}

```

---

Controller responsibility:

- Receive request
    
- Call service
    
- Update view
    

---

# 10. Service Layer

Most important layer.

Contains:

- Business Logic
    
- Validation
    
- Transaction Rules
    

Example:

```java
public class OrderService {


private OrderRepository repository;


public void createOrder(Order order){


validate(order);


repository.save(order);


}


}

```

---

Service decides:

"Can this order happen?"

---

# 11. Repository Layer

Database communication.

Interface:

```java
public interface ProductRepository {


List<Product> findAll();


void save(Product product);


}

```

---

Implementation:

```java
public class MySQLProductRepository
implements ProductRepository {


public void save(Product product){


 // JDBC


}


}

```

---

# 12. Why Repository Interface?

Without:

```
Service

 |
 |
MySQL Code

```

Service depends on MySQL.

---

With:

```
Service

 |
 |
Repository Interface

 |
 |
MySQL Repository

```

Flexible.

---

# 13. Dependency Injection Concept

Old:

```java
class ProductService {


ProductRepository repo =
new MySQLProductRepository();


}

```

Problem:

Hard dependency.

---

Better:

```java
class ProductService {


private ProductRepository repo;


ProductService(
ProductRepository repo
){

this.repo=repo;

}

}

```

---

Now:

```java
ProductRepository

        |
        |
 MySQLRepository


```

can change.

---

# 14. Application Startup

Need a place to create objects.

Example:

```
Application.java

```

---

Flow:

```text
main()

 |
 |
Create Database

 |
 |
Create Repository

 |
 |
Create Service

 |
 |
Create Controller

 |
 |
Open Swing UI

```

---

Example:

```java
public class Application {


public static void main(String[] args){


var repository =
new MySQLProductRepository();


var service =
new ProductService(repository);


var controller =
new ProductController(service);


new MainFrame(controller)
.setVisible(true);


}

}

```

---

# 15. Configuration Management

Never:

```java
String password="12345";

```

inside code.

---

Use:

```
application.properties

```

Example:

```properties
db.url=jdbc:mysql://localhost/cafe_pos

db.username=root

db.password=password

```

---

Java:

```java
Properties config =
new Properties();

```

---

# 16. Database Manager

Create:

```
database

    |
    |
DatabaseManager

```

---

Example:

```java
public class DatabaseManager {


private DataSource dataSource;


public Connection getConnection()
throws SQLException{


return dataSource.getConnection();


}

}

```

---

Use:

- HikariCP
    
- Connection Pool
    

---

# 17. Domain Model

Modern Java 25:

Instead of:

```java
class Product {


private int id;

private String name;


}

```

Use:

```java
public record Product(

int id,

String name,

double price

){

}

```

---

# 18. DTO Layer

DTO:

Data Transfer Object

Purpose:

Move data between layers.

Example:

```java
public record ProductDTO(

String name,

double price

){

}

```

---

Flow:

```
View

 |

DTO

 |

Service

 |

Entity

 |

Database

```

---

# 19. Exception Architecture

Package:

```
exception


AppException

DatabaseException

BusinessException

ValidationException

GlobalHandler

```

---

Flow:

```text
Database Error

       |

DatabaseException

       |

Global Handler

       |

User Message + Log

```

---

# 20. Logging Architecture

Package:

```
logging

    |
    |
LoggerManager

```

---

Example:

```java
log.info(
"Order created {}",
order.id()
);

```

---

Logs:

```
logs/

 app.log

 error.log

 audit.log

```

---

# 21. Security Layer

Package:

```
security

```

Contains:

- Authentication
    
- Password Hashing
    
- Permission
    

Example:

```text
Admin

 |
 +-- Product Management


Cashier

 |
 +-- Sales Only

```

---

# 22. MVC + Clean Architecture

Final:

```
              Swing View

                  |
                  |

             Controller

                  |
                  |

             Application

                  |
                  |

              Services

                  |
                  |

              Domain

                  |
                  |

            Repository

                  |
                  |

             Database

```

---

# 23. Order Processing Example

Customer buys coffee.

## Step 1

View:

```text
Click Checkout

```

---

## Step 2

Controller:

```java
orderController.checkout(order);

```

---

## Step 3

Service:

```java
orderService.create(order);

```

---

## Step 4

Service:

```
Validate

Check Stock

Calculate Total

Save Order

Update Inventory

```

---

## Step 5

Repository:

```sql
INSERT INTO orders

```

---

# 24. Multi-thread Integration

Order:

```text
UI Thread

     |

Virtual Thread

     |

Order Service

     |

Database

```

---

Printer:

```text
Virtual Thread

     |

Print Receipt

```

---

# 25. Testing Architecture

Good architecture makes testing easy.

Example:

Service Test:

```java
ProductServiceTest

```

can use:

```java
FakeProductRepository

```

instead of MySQL.

---

# 26. Production Deployment

Desktop POS:

```text
Café Computer

 |
 |
Java Application

 |
 |
MySQL Server

```

---

Multi Branch:

```text
Branch POS

     |

API Server

     |

Cloud Database

```

---

# 27. Senior Developer Checklist

A professional Café POS should have:

Architecture:

✅ MVC

✅ Service Layer

✅ Repository Pattern

✅ Dependency Injection

Database:

✅ JDBC

✅ Connection Pool

✅ Transaction

Error:

✅ Custom Exception

✅ Global Handler

Performance:

✅ Thread Pool

✅ Virtual Thread

✅ Cache

Security:

✅ Authentication

✅ Authorization

---

# 28. Final Café POS Project Structure

```
CafePOS

src/main/java


com.cafe.pos


├── Application.java


├── view

├── controller

├── service

├── repository

├── repository.impl

├── model

├── dto

├── database

├── exception

├── security

├── logging

├── util

└── config

```

---

# Interview Questions

## Q1: Why separate Service and Repository?

Service:

Business rules

Repository:

Database access

---

## Q2: Why use Dependency Injection?

To reduce coupling and improve testing.

---

## Q3: What is Clean Architecture?

Separating business logic from external systems.

---

## Q4: MVC purpose?

Separate:

- UI
    
- Logic
    
- Data
    

---

# Practice Project

Create initial Café POS skeleton:

Create packages:

```
view

controller

service

repository

model

exception

database

config

```

Implement:

1. Product Model
    
2. ProductRepository Interface
    
3. MySQLProductRepository
    
4. ProductService
    
5. ProductController
    
6. Application Launcher
    

---

# Lesson 21 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Enterprise Architecture  
✅ Layered Architecture  
✅ MVC  
✅ Service Layer  
✅ Repository Pattern  
✅ Dependency Injection  
✅ DTO Concept  
✅ Configuration Management  
✅ Application Lifecycle  
✅ Production Structure  
✅ Senior Java Project Organization

---

# Next Lesson

# Lesson 22: Building Café POS Database Schema + Entity Design

## Professional ERD → MySQL → Java Model Mapping

Topics:

- Database Design
    
- Normalization
    
- ERD
    
- Entity Relationship
    
- PK/FK Design
    
- Product Recipe System
    
- Inventory Model
    
- Order Model
    
- Payment Model
    
- Java Record Mapping
    

ဒီ Lesson ကနေ **ကျွန်တော်တို့ Café POS Project ကို တကယ် Build စတင်ပါမယ်။**