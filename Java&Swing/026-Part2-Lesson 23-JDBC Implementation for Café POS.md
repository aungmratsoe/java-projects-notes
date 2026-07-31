# Part 2: Advanced Java Knowledge for Swing

# Lesson 23: JDBC Implementation for Café POS

## Building Professional Repository Layer with MySQL

### (Java 25 + Swing + MVC + JDBC + HikariCP)

ဒီ Lesson ကနေစပြီး **Café POS Database ကို Java Code နဲ့ တကယ်ချိတ်ဆက်ပြီး Implementation စတင်ပါမယ်။**

အခုအထိ:

- Database Design ✅
    
- ERD ✅
    
- Entity Model ✅
    
- MVC Architecture ✅
    
- Exception Architecture ✅
    

ပြီးသွားပါပြီ။

အခု Development Phase:

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

ကို Code စရေးပါမယ်။

---

# 1. Project Structure

Professional Structure:

```
CafePOS

src/main/java

com.cafe.pos

├── Application.java
│
├── config
│    └── DatabaseConfig.java
│
├── database
│    └── DatabaseManager.java
│
├── model
│    └── Product.java
│
├── repository
│    └── ProductRepository.java
│
├── repository.impl
│    └── MySQLProductRepository.java
│
├── service
│    └── ProductService.java
│
├── exception
│    └── DatabaseException.java
│
└── view
     └── ProductFrame.java

```

---

# 2. Maven Dependencies

`pom.xml`

Add MySQL Driver:

```xml
<dependency>

    <groupId>com.mysql</groupId>

    <artifactId>mysql-connector-j</artifactId>

    <version>9.4.0</version>

</dependency>
```

---

HikariCP:

```xml
<dependency>

    <groupId>com.zaxxer</groupId>

    <artifactId>HikariCP</artifactId>

    <version>6.3.0</version>

</dependency>
```

---

SLF4J:

```xml
<dependency>

    <groupId>org.slf4j</groupId>

    <artifactId>slf4j-api</artifactId>

    <version>2.0.17</version>

</dependency>


<dependency>

    <groupId>ch.qos.logback</groupId>

    <artifactId>logback-classic</artifactId>

    <version>1.5.18</version>

</dependency>

```

---

# 3. Database Configuration

Never hard-code:

Bad:

```java
String password="12345";

```

---

Create:

```
config/database.properties
```

---

Content:

```properties
db.url=jdbc:mysql://localhost:3306/cafe_pos

db.username=root

db.password=password

db.pool.size=10

```

---

# 4. DatabaseConfig Class

Purpose:

Load configuration.

```java
package com.cafe.pos.config;


import java.io.InputStream;
import java.util.Properties;


public class DatabaseConfig {


private static final Properties properties =
new Properties();



static {


try(InputStream input =
DatabaseConfig.class
.getClassLoader()
.getResourceAsStream(
"database.properties"
)){


properties.load(input);


}
catch(Exception e){

throw new RuntimeException(
"Cannot load database configuration",
e
);


}


}



public static String get(
String key
){

return properties.getProperty(key);

}


}

```

---

# 5. DatabaseManager with HikariCP

Old style:

```
Every query:

open connection

close connection

```

Bad.

---

Professional:

```
Application

 |

Hikari Pool

 |

Connection

 |

MySQL

```

---

Create:

```java
package com.cafe.pos.database;


import com.zaxxer.hikari.*;

import com.cafe.pos.config.DatabaseConfig;


import java.sql.Connection;
import java.sql.SQLException;



public class DatabaseManager {


private final HikariDataSource dataSource;



public DatabaseManager(){


HikariConfig config =
new HikariConfig();


config.setJdbcUrl(
DatabaseConfig.get("db.url")
);


config.setUsername(
DatabaseConfig.get("db.username")
);


config.setPassword(
DatabaseConfig.get("db.password")
);


config.setMaximumPoolSize(
Integer.parseInt(
DatabaseConfig.get("db.pool.size")
)
);



dataSource =
new HikariDataSource(config);


}



public Connection getConnection()
throws SQLException{


return dataSource.getConnection();


}



public void close(){


dataSource.close();


}


}

```

---

# 6. Product Entity

Use Java 25 Record:

```java
package com.cafe.pos.model;


import java.math.BigDecimal;


public record Product(

Long id,

Long categoryId,

String name,

BigDecimal price,

boolean active

){

}

```

---

Why Record?

Because Entity mostly stores data.

No boilerplate:

Before:

```java
class Product {


private Long id;


public Long getId(){

return id;

}

}

```

---

Record automatically gives:

- Constructor
    
- Getter
    
- equals()
    
- hashCode()
    
- toString()
    

---

# 7. Repository Interface

Repository defines:

"What can database do?"

```java
package com.cafe.pos.repository;


import com.cafe.pos.model.Product;

import java.util.List;



public interface ProductRepository {


List<Product> findAll();



Product findById(Long id);



void save(Product product);



void update(Product product);



void delete(Long id);



}

```

---

Important:

Interface does not know MySQL.

---

# 8. MySQL Repository Implementation

Create:

```
repository.impl

MySQLProductRepository.java

```

---

```java
package com.cafe.pos.repository.impl;


import com.cafe.pos.model.Product;

import com.cafe.pos.repository.ProductRepository;

import com.cafe.pos.database.DatabaseManager;

import java.sql.*;

import java.util.*;


public class MySQLProductRepository
implements ProductRepository {



private final DatabaseManager database;



public MySQLProductRepository(
DatabaseManager database
){

this.database=database;

}



@Override
public List<Product> findAll(){


List<Product> products =
new ArrayList<>();


String sql =
"""
SELECT *

FROM products

ORDER BY name

""";



try(
Connection con =
database.getConnection();

PreparedStatement ps =
con.prepareStatement(sql);

ResultSet rs =
ps.executeQuery()

){



while(rs.next()){


products.add(

new Product(

rs.getLong("id"),

rs.getLong("category_id"),

rs.getString("name"),

rs.getBigDecimal("price"),

rs.getBoolean("status")

)

);


}



}
catch(SQLException e){


throw new RuntimeException(
"Cannot load products",
e
);


}



return products;


}

}

```

---

# 9. Why PreparedStatement?

Never:

```java
String sql =
"SELECT * FROM products WHERE id="
+id;

```

---

Problem:

SQL Injection.

---

Use:

```java
WHERE id=?

```

---

Then:

```java
ps.setLong(
1,
id
);

```

---

# 10. findById()

Implementation:

```java
@Override
public Product findById(Long id){


String sql =
"""
SELECT *

FROM products

WHERE id=?

""";



try(
Connection con =
database.getConnection();

PreparedStatement ps =
con.prepareStatement(sql)

){


ps.setLong(
1,
id
);


ResultSet rs =
ps.executeQuery();



if(rs.next()){


return new Product(

rs.getLong("id"),

rs.getLong("category_id"),

rs.getString("name"),

rs.getBigDecimal("price"),

rs.getBoolean("status")

);


}



return null;


}
catch(SQLException e){


throw new DatabaseException(
"Cannot find product",
e
);


}

}

```

---

# 11. Custom DatabaseException

```java
package com.cafe.pos.exception;


public class DatabaseException
extends RuntimeException {


public DatabaseException(
String message,
Throwable cause
){

super(
message,
cause
);

}


}

```

---

Now:

Before:

```java
catch(SQLException e){

e.printStackTrace();

}

```

---

After:

```java
throw new DatabaseException(
"Database error",
e
);

```

---

# 12. Service Layer

Service contains business rules.

```java
package com.cafe.pos.service;


import com.cafe.pos.model.Product;

import com.cafe.pos.repository.ProductRepository;



public class ProductService {


private final ProductRepository repository;



public ProductService(
ProductRepository repository
){

this.repository=repository;

}



public void create(Product product){



if(product.price()
.compareTo(
java.math.BigDecimal.ZERO
)<=0){


throw new IllegalArgumentException(
"Price must be positive"
);


}



repository.save(product);



}



}

```

---

# 13. Dependency Injection Flow

Application:

```java
DatabaseManager db =
new DatabaseManager();



ProductRepository repo =
new MySQLProductRepository(db);



ProductService service =
new ProductService(repo);



```

---

Dependency:

```
ProductService

       |
       |
ProductRepository

       |
       |
MySQLProductRepository

       |
       |
DatabaseManager

```

---

# 14. Application Startup

```java
public class Application {


public static void main(String[] args){


DatabaseManager database =
new DatabaseManager();



ProductRepository repository =
new MySQLProductRepository(database);



ProductService service =
new ProductService(repository);



System.out.println(
service.findAll()
);



}

}

```

---

# 15. Testing Database Connection

Test:

```java
public class DatabaseTest {


public static void main(String[] args)
throws Exception{


DatabaseManager db =
new DatabaseManager();



try(Connection con =
db.getConnection()){


System.out.println(
"Database Connected"
);


}



}

}

```

---

Output:

```
Database Connected

```

---

# 16. Transaction Example

Order creation requires:

```
Order

+

Order Items

+

Inventory Update

```

One transaction.

---

Service:

```java
connection.setAutoCommit(false);


try{


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

# 17. Repository Best Practices

Always:

✅ Interface first

✅ PreparedStatement

✅ Try-with-resources

✅ Custom Exception

✅ Connection Pool

✅ Transaction in Service

---

Avoid:

❌ SQL in Controller

❌ SQL in Swing Button

❌ Static Connection

❌ Catch and ignore Exception

---

# 18. Current Café POS Architecture

Now:

```
Swing

 |
Controller

 |
Service

 |
Repository Interface

 |
MySQL Repository

 |
HikariCP

 |
MySQL

```

---

# Practice Task

Implement:

### Product Module

Create:

```
ProductRepository

findAll()

findById()

save()

update()

delete()

```

Add:

```
ProductService

ProductController

ProductFrame

```

---

# Lesson 23 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ JDBC Architecture Implementation  
✅ HikariCP Connection Pool  
✅ Database Configuration  
✅ Repository Pattern Coding  
✅ Java 25 Record Entity  
✅ PreparedStatement  
✅ Custom DatabaseException  
✅ Service Layer  
✅ Dependency Injection  
✅ Application Startup

---

# Next Lesson

# Lesson 24: Building Complete Product Management Module

## CRUD + Swing JTable + MVC Integration

Topics:

- Product Controller
    
- Product Service
    
- Repository CRUD Complete
    
- Swing JTable
    
- Form Validation
    
- Add / Update / Delete Product
    
- Exception Handling in UI
    
- Threading for Database Operations
    

ဒီ Lesson မှာ **Café POS ရဲ့ ပထမဆုံး Functional Feature (Product Management)** ကို တကယ် Build စပါမယ်။