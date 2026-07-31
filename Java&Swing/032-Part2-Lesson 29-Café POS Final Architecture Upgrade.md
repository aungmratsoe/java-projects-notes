# Part 2: Advanced Java Swing + Café POS Project

# Lesson 29: Café POS Final Architecture Upgrade

## Enterprise Application Design & Production Deployment

### (Java 25 + Swing + MVC + JDBC + MySQL + Design Patterns + Deployment)

ဒီ Lesson မှာ ကျွန်တော်တို့ တည်ဆောက်နေတဲ့ **Café POS System** ကို

> "Student Project" → "Enterprise Desktop Application"

အဖြစ် Upgrade လုပ်ပါမယ်။

ဒီအဆင့်က Senior Java Developer / Software Architect Level မှာ အရေးကြီးတဲ့ အပိုင်းပါ။

---

# 1. Current Café POS Architecture Review

အခု System:

```text
Cafe POS

├── Product Management
├── Order Management
├── Inventory System
├── Authentication
├── Authorization
├── Reporting
└── Audit Logging

```

Architecture:

```text
              Swing UI

                 |

            Controller

                 |

             Service

                 |

          Repository

                 |

              JDBC

                 |

              MySQL

```

ဒါက Layered Architecture ဖြစ်ပါတယ်။

---

# 2. Enterprise Package Architecture

Beginner:

```
src

 ├── model

 ├── view

 ├── controller

 └── database

```

Production:

```
com.cafe.pos

│
├── app
│    └── Application.java
│
├── config
│    ├── DatabaseConfig.java
│    └── AppConfig.java
│
├── database
│    ├── ConnectionPool.java
│    └── MigrationManager.java
│
├── security
│    ├── Session.java
│    ├── PasswordEncoder.java
│    └── PermissionService.java
│
├── exception
│    ├── AppException.java
│    ├── DatabaseException.java
│    └── ValidationException.java
│
├── common
│    ├── Constants.java
│    ├── DateUtils.java
│    └── Formatter.java
│
├── module
│
│    ├── product
│    │     ├── Product.java
│    │     ├── ProductRepository.java
│    │     ├── ProductService.java
│    │     └── ProductPanel.java
│
│    ├── order
│    │     ├── Order.java
│    │     ├── OrderService.java
│    │     └── OrderPanel.java
│
│    ├── inventory
│    │     └── InventoryService.java
│
│    └── report
│          └── DashboardPanel.java
│
└── resources

     ├── database.properties

     ├── logback.xml

     └── images


```

---

# 3. Why Module Based Architecture?

Old style:

```text
Product

Order

Inventory


All mixed together

```

Problem:

- Hard to maintain
    
- Hard to test
    
- Large files
    

---

Module:

```text
product module


order module


inventory module


```

Each module owns:

- Model
    
- Repository
    
- Service
    
- UI
    

---

# 4. Dependency Rule

Important Architecture Rule:

```
UI

↓

Controller

↓

Service

↓

Repository

↓

Database

```

Never:

```
UI

↓

Database

```

Example Bad:

```java
// JButton action

Connection con =
DriverManager.getConnection();


SELECT * FROM products;

```

---

Why bad?

- UI knows database
    
- Hard testing
    
- Hard changing DB
    

---

# 5. Dependency Injection

Old:

```java
ProductService service =
new ProductService();

```

Problem:

Service creates everything.

---

Better:

```java
ProductRepository repo =
new MySQLProductRepository();


ProductService service =
new ProductService(repo);

```

---

Flow:

```
Application

creates objects


↓

inject dependencies


↓

run application


```

---

# 6. Application Bootstrap Class

Create:

```
app/Application.java

```

Example:

```java
public class Application {


public static void main(String[] args){


initialize();


start();


}



private static void initialize(){


DatabaseManager.initialize();


Logger.initialize();


}



private static void start(){


SwingUtilities.invokeLater(
()->{


new LoginFrame()
.setVisible(true);


});


}


}

```

---

# 7. Configuration Management

Never:

```java
String url =
"jdbc:mysql://localhost/cafe";

```

Problem:

Change server = modify code.

---

Use:

```
resources


database.properties

```

Example:

```properties
database.url=jdbc:mysql://localhost:3306/cafe_pos

database.username=root

database.password=password


application.name=Cafe POS

version=1.0

```

---

# 8. Environment Configuration

Professional:

Development:

```
application-dev.properties

```

Production:

```
application-prod.properties

```

Example:

Development:

```
localhost

```

Production:

```
server.company.com

```

---

# 9. Logging System

Never:

```java
System.out.println(
"Error"
);

```

Production needs:

```
INFO

WARN

ERROR

DEBUG

```

---

# 10. Logback Setup

Dependency:

```xml
<dependency>

<groupId>ch.qos.logback</groupId>

<artifactId>logback-classic</artifactId>

</dependency>

```

---

File:

```
resources/logback.xml

```

Example:

```xml
<configuration>


<appender name="FILE"
class="ch.qos.logback.core.FileAppender">


<file>
logs/cafe-pos.log
</file>


</appender>


</configuration>

```

---

# 11. Logger Usage

Instead of:

```java
e.printStackTrace();

```

Use:

```java
private static final Logger log =
LoggerFactory.getLogger(
ProductService.class
);



log.error(
"Product save failed",
e
);

```

---

# 12. Global Exception Handler

Swing Application:

Many errors possible.

Create:

```
exception

GlobalExceptionHandler.java

```

---

Code:

```java
public class GlobalExceptionHandler {


public static void handle(
Exception e
){


JOptionPane.showMessageDialog(

null,

e.getMessage(),

"Error",

JOptionPane.ERROR_MESSAGE

);


Logger.error(e);


}


}

```

---

# 13. Design Patterns Used

Professional applications use patterns.

Our POS:

---

## MVC Pattern

Already used:

```
Model

View

Controller

```

---

## Repository Pattern

Database abstraction:

```
Service

 |

Repository

 |

Database

```

---

## Factory Pattern

Example:

Create Payment:

```
PaymentFactory


CashPayment


CardPayment


MobilePayment

```

---

Example:

```java
Payment payment =
PaymentFactory.create(
"CASH"
);

```

---

## Singleton Pattern

Examples:

- Database Manager
    
- Session
    
- Configuration
    

Example:

```java
public class Session {


private static Session instance;


private Session(){}



public static Session getInstance(){


if(instance==null)

instance=new Session();


return instance;


}


}

```

---

# 14. Database Migration System

Production problem:

Version changes.

Example:

Version 1:

```
products

name

price

```

Version 2:

Add:

```
barcode

```

Need migration.

---

Use:

```
database/migration


V1__init.sql

V2__add_barcode.sql

V3__add_discount.sql

```

---

# 15. Backup System

Important for POS.

Daily:

```
02:00 AM


Database Backup


```

---

Command:

```bash
mysqldump

cafe_pos

>

backup.sql

```

---

Restore:

```bash
mysql

cafe_pos

<

backup.sql

```

---

# 16. Application Packaging

Java Application:

Need:

```
CafePOS.exe

```

---

Use:

## jpackage

Java 25 includes:

```
jpackage

```

---

Example:

```bash
jpackage

--name CafePOS

--input target

--main-jar cafe-pos.jar

--type exe

```

---

Result:

```
CafePOS.exe

```

---

# 17. Database Deployment

Customer PC:

Install:

```
MySQL Server

+

CafePOS Application


```

---

Better:

```
CafePOS

 |

Embedded MySQL / Server Database

```

---

# 18. Backup Strategy

Professional:

```
Daily Backup

Weekly Full Backup

Monthly Archive

```

---

# 19. Security Checklist

Application:

✅ BCrypt Password

✅ RBAC

✅ Audit Log

✅ SQL Injection Protection

✅ Configuration Encryption

Database:

✅ User Permission

✅ Backup

✅ Migration

---

# 20. Performance Optimization

Large Café:

100,000 Orders.

Need:

## Database Index

Example:

```sql
CREATE INDEX idx_order_created

ON orders(created_at);

```

---

## Connection Pool

HikariCP:

```
10-20 connections

```

---

## Async Operations

Use Java 25:

```
Virtual Threads

```

---

# 21. Testing Strategy

Professional:

## Unit Test

Test Service:

```
ProductServiceTest

OrderServiceTest

```

---

## Integration Test

Test:

```
Java

↓

Database

```

---

## UI Test

Test:

```
Button

Form

Validation

```

---

# 22. Final Café POS Architecture

Complete System:

```
                Swing UI

                   |

              Controllers

                   |

              Application Services

                   |

        -------------------------

        |                       |

   Repositories            External Services


        |

        |

      MySQL


        |

        |

     Backup System


```

---

# 23. Final Technology Stack

Our Café POS:

|Layer|Technology|
|---|---|
|Language|Java 25|
|UI|Swing + FlatLaf|
|Architecture|MVC + Layered|
|Database|MySQL|
|Connection|HikariCP|
|Security|BCrypt + RBAC|
|Logging|SLF4J + Logback|
|Reports|JFreeChart|
|PDF|PDFBox|
|Excel|Apache POI|
|Threading|Virtual Threads|
|Build|Maven|
|Deployment|jpackage|

---

# 24. Senior Developer Checklist

Before Production:

Architecture:

✅ Clean Package Structure  
✅ Dependency Injection  
✅ Separation of Responsibility

Database:

✅ Transaction  
✅ Index  
✅ Backup  
✅ Migration

Security:

✅ Authentication  
✅ Authorization  
✅ Audit

Performance:

✅ Connection Pool  
✅ Async Processing  
✅ Query Optimization

Deployment:

✅ Installer  
✅ Configuration  
✅ Logs

---

# Lesson 29 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Enterprise Package Architecture  
✅ Dependency Injection  
✅ Configuration Management  
✅ Logging System  
✅ Global Exception Handling  
✅ Design Patterns  
✅ Database Migration  
✅ Backup Strategy  
✅ Deployment with jpackage  
✅ Testing Strategy  
✅ Production Checklist

---

# Next Lesson

# Lesson 30: Café POS Final Project Implementation Roadmap

## From UML → Database → Code → Deployment

Topics:

- Complete Development Plan
    
- Coding Order
    
- Sprint Planning
    
- Git Workflow
    
- Team Development
    
- Final Project Structure
    
- Implementation Checklist
    

ဒီ Lesson ကနေစပြီး **Café POS ကို အစမှအဆုံး တကယ် Build လုပ်မယ့် Professional Roadmap** ဖြစ်ပါမယ်။