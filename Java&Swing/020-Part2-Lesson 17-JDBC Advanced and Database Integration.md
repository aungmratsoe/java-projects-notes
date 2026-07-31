# Part 2: Advanced Java Knowledge for Swing

# Lesson 17: JDBC Advanced & Database Integration

## Building Professional MySQL Layer for Café POS (Java 25 + MVC Architecture)

ဒီ Lesson မှာ Java Application ကို **MySQL Database** နဲ့ professional level ချိတ်ဆက်ပုံကို လေ့လာပါမယ်။

Café POS System တစ်ခုမှာ Database Layer က အရေးကြီးဆုံးအပိုင်းတွေထဲက တစ်ခုဖြစ်ပါတယ်။

Real-world flow:

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
Repository / DAO
   |
   |
JDBC
   |
   |
MySQL Database
```

---

# 1. What is JDBC?

JDBC ဆိုတာ:

> Java Database Connectivity

Java Application နဲ့ Database ကြား communication လုပ်ပေးတဲ့ API ဖြစ်ပါတယ်။

---

Example:

Java:

```java
Product product;
```

↓

JDBC:

```sql
SELECT *
FROM products;
```

↓

MySQL:

```
products table
```

---

# 2. JDBC Architecture

Complete architecture:

```
Java Application

       |
       |
   JDBC API

       |
       |
 JDBC Driver

       |
       |
 MySQL Database

```

---

Components:

## 1. JDBC API

Java interfaces:

```java
Connection

Statement

PreparedStatement

ResultSet

```

---

## 2. JDBC Driver

Database vendor ပေးတဲ့ driver.

Example:

```text
MySQL Connector/J
```

---

## 3. Database

Example:

```
cafe_pos

 |
 |-- products

 |-- orders

 |-- order_items

 |-- customers

```

---

# 3. Adding MySQL Driver (Maven)

Java 25 project:

`pom.xml`

```xml
<dependency>

    <groupId>
        com.mysql
    </groupId>

    <artifactId>
        mysql-connector-j
    </artifactId>

    <version>
        9.4.0
    </version>

</dependency>
```

---

# 4. JDBC Connection

Basic connection:

```java
import java.sql.Connection;
import java.sql.DriverManager;


public class DatabaseConnection {


public static Connection getConnection()
throws Exception {


return DriverManager.getConnection(

"jdbc:mysql://localhost:3306/cafe_pos",

"root",

"password"

);


}


}

```

---

Flow:

```
Java

 |
DriverManager

 |
MySQL Driver

 |
Database

```

---

# 5. Connection Object

Connection ဆိုတာ:

> Database session တစ်ခုကို represent လုပ်တဲ့ object ဖြစ်ပါတယ်။

Example:

```java
Connection con =
DatabaseConnection.getConnection();

```

---

Through connection:

- Execute SQL
    
- Commit
    
- Rollback
    
- Close
    

လုပ်နိုင်ပါတယ်။

---

# 6. Statement vs PreparedStatement

## Statement

Example:

```java
Statement stmt =
connection.createStatement();


stmt.executeQuery(
"SELECT * FROM products"
);

```

---

Problem:

SQL Injection ဖြစ်နိုင်ပါတယ်။

---

Example:

User input:

```
' OR '1'='1
```

SQL:

```sql
SELECT *
FROM users
WHERE name=''
OR '1'='1'
```

Security problem.

---

# 7. PreparedStatement

Professional way:

```java
String sql =
"""
SELECT *
FROM products
WHERE id=?
""";


PreparedStatement ps =
connection.prepareStatement(sql);


ps.setInt(1,100);


ResultSet rs =
ps.executeQuery();

```

---

Advantages:

✅ SQL Injection Protection

✅ Faster

✅ Cleaner

---

# 8. Café POS Product Repository

Architecture:

```
ProductController

        |

ProductService

        |

ProductRepository

        |

JDBC

        |

MySQL

```

---

Interface:

```java
public interface ProductRepository {


List<Product> findAll();


Product findById(int id);


void save(Product product);


void delete(int id);


}

```

---

# 9. MySQL Repository Implementation

Example:

```java
public class MySQLProductRepository
implements ProductRepository {


private Connection connection;


public MySQLProductRepository(
Connection connection
){

this.connection = connection;

}


@Override
public List<Product> findAll(){

List<Product> products =
new ArrayList<>();


String sql =
"SELECT * FROM products";


try(
PreparedStatement ps =
connection.prepareStatement(sql);

ResultSet rs =
ps.executeQuery()

){


while(rs.next()){


products.add(

new Product(

rs.getInt("id"),

rs.getString("name"),

rs.getDouble("price")

)

);


}


}
catch(Exception e){

throw new DatabaseException(
"Cannot load products",
e
);

}


return products;

}

}

```

---

# 10. ResultSet

SQL Result:

```
products


id | name   | price

1  | Coffee | 5000

2  | Cake   | 7000

```

---

Java:

```java
while(rs.next()){


String name =
rs.getString("name");


}

```

---

ResultSet cursor:

```
Before first row

       |
       v

Row 1

       |
       v

Row 2

       |
       v

End

```

---

# 11. Insert Data

Example:

```java
String sql =
"""
INSERT INTO products
(name,price)
VALUES (?,?)
""";


PreparedStatement ps =
connection.prepareStatement(sql);


ps.setString(
1,
"Coffee"
);


ps.setDouble(
2,
5000
);


ps.executeUpdate();

```

---

# 12. Update Data

```java
String sql =
"""
UPDATE products

SET price=?

WHERE id=?

""";


PreparedStatement ps =
connection.prepareStatement(sql);


ps.setDouble(
1,
6000
);


ps.setInt(
2,
1
);


ps.executeUpdate();

```

---

# 13. Delete Data

```java
String sql =
"""
DELETE FROM products
WHERE id=?
""";


PreparedStatement ps =
connection.prepareStatement(sql);


ps.setInt(
1,
5
);


ps.executeUpdate();

```

---

# 14. Database Exception Handling

Create custom exception:

```java
public class DatabaseException
extends RuntimeException {


public DatabaseException(
String message,
Throwable cause
){

super(message,cause);

}


}

```

---

Usage:

```java
catch(SQLException e){


throw new DatabaseException(

"Database operation failed",

e

);


}

```

---

Why?

Because Service Layer doesn't need JDBC details.

---

# 15. Transaction Management

Important concept.

Example:

Customer order:

```
Order

+

Order Items

+

Stock Update

```

All must succeed.

---

Problem:

```
Save Order ✔

Save Items ✔

Update Stock ❌


Database inconsistent

```

---

Solution:

Transaction.

---

# 16. JDBC Transaction

Default:

```java
autoCommit=true
```

Every SQL commits immediately.

---

Disable:

```java
connection.setAutoCommit(false);

```

---

Example:

```java
try{


connection.setAutoCommit(false);



saveOrder();


saveItems();


updateStock();



connection.commit();


}
catch(Exception e){


connection.rollback();


}

```

---

Flow:

```
START TRANSACTION

      |

Insert Order

      |

Insert Items

      |

Update Stock

      |

COMMIT

```

---

If error:

```
ROLLBACK

Return original state

```

---

# 17. Café POS Order Transaction Example

User buys:

```
Coffee x2

Cake x1

```

Process:

```
1. Create Order

2. Create Order Items

3. Reduce Inventory

4. Calculate Payment

5. Commit

```

---

If inventory fails:

```
Rollback everything

```

---

# 18. Batch Processing

Problem:

Insert 10,000 sales:

Bad:

```java
for(Sale s:sales){

insert(s);

}

```

Slow.

---

Batch:

```java
PreparedStatement ps =
connection.prepareStatement(sql);


for(Sale s:sales){


ps.setInt(
1,
s.id()
);


ps.addBatch();


}


ps.executeBatch();

```

---

Advantages:

- Faster
    
- Less network communication
    

---

# 19. Connection Pool

Problem:

Every request:

```
Open Connection

Query

Close Connection

```

Expensive.

---

Connection Pool:

```
Application

     |

Connection Pool

[Conn]
[Conn]
[Conn]

     |

Database

```

---

Popular:

- HikariCP
    

Spring Boot uses HikariCP by default.

---

# 20. HikariCP Example

Maven:

```xml
<dependency>

<groupId>com.zaxxer</groupId>

<artifactId>HikariCP</artifactId>

<version>6.3.0</version>

</dependency>

```

---

Configuration:

```java
HikariConfig config =
new HikariConfig();


config.setJdbcUrl(
"jdbc:mysql://localhost/cafe_pos"
);


config.setUsername(
"root"
);


config.setPassword(
"password"
);


HikariDataSource dataSource =
new HikariDataSource(config);

```

---

Get connection:

```java
Connection con =
dataSource.getConnection();

```

---

# 21. DAO vs Repository

DAO:

```
Database focused

ProductDAO
OrderDAO

```

---

Repository:

```
Domain focused

ProductRepository

OrderRepository

```

---

Modern applications prefer:

```
Repository Pattern
```

---

# 22. Complete Café POS Database Layer

```
model

 |
 |
Product.java


repository

 |
 |
ProductRepository.java


repository.impl

 |
 |
MySQLProductRepository.java


service

 |
 |
ProductService.java


controller

 |
 |
ProductController.java


```

---

# 23. Java 25 Modern JDBC Style

Use:

Record:

```java
public record Product(

int id,

String name,

double price

){}

```

---

Repository:

```java
List<Product> findAll();

```

---

Exception:

```java
sealed class AppException

```

---

Async:

```java
Thread.startVirtualThread(
() -> loadProducts()
);

```

---

# 24. JDBC Best Practices

Always:

✅ Use PreparedStatement

✅ Close resources

✅ Use transactions

✅ Use connection pool

✅ Separate database layer

✅ Handle SQL exceptions

---

Avoid:

❌ SQL inside Swing button

❌ Global Connection object

❌ String concatenated SQL

❌ Auto commit for complex operations

---

# 25. Interview Questions

## Q1: What is JDBC?

API for Java Database communication.

---

## Q2: Statement vs PreparedStatement?

Statement:

- Simple
    
- SQL Injection risk
    

PreparedStatement:

- Secure
    
- Faster
    
- Parameterized
    

---

## Q3: What is Transaction?

A group of database operations executed as one unit.

---

## Q4: Why Connection Pool?

To reuse database connections and improve performance.

---

## Q5: Commit vs Rollback?

Commit:

```
Save changes
```

Rollback:

```
Undo changes
```

---

# Practice Project

Build Café POS JDBC Layer:

Create:

```
database

 |
DatabaseManager


repository

 |
ProductRepository


repository.impl

 |
MySQLProductRepository


service

 |
ProductService

```

Implement:

1. Product CRUD
    
2. Order Transaction
    
3. Inventory Update
    
4. Custom DatabaseException
    
5. HikariCP Connection Pool
    

---

# Lesson 17 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ JDBC Architecture  
✅ MySQL Connection  
✅ PreparedStatement  
✅ ResultSet  
✅ Repository Pattern  
✅ CRUD Operations  
✅ Custom Database Exception  
✅ Transaction Management  
✅ Commit / Rollback  
✅ Batch Processing  
✅ Connection Pool  
✅ HikariCP  
✅ Enterprise Database Layer Design

---

# Next Lesson

# Lesson 18: Advanced SQL + JDBC Optimization

## Designing High Performance Café POS Database Access

Topics:

- Index Optimization
    
- Query Performance
    
- Pagination
    
- Lazy Loading
    
- Caching
    
- Batch Insert Optimization
    
- Database Locking
    
- Isolation Levels
    
- Deadlock Handling
    
- Production Database Strategy
    

ဒီ Lesson ပြီးရင် Java + MySQL POS System ကို Enterprise Performance Level နဲ့ optimize လုပ်နိုင်ပါမယ်။