# Part 3: Café POS Real Implementation Phase

# Lesson 32: Database Layer Implementation

## MySQL + HikariCP + Connection Management + Transaction Manager

### (Java 25 + JDBC + MVC + Enterprise Database Architecture)

ဒီ Lesson မှာ ကျွန်တော်တို့ Café POS ရဲ့ **Database Foundation Layer** ကို တည်ဆောက်ပါမယ်။

အခုအချိန်ထိ:

```
Swing UI
   |
Application
```

ပဲရှိသေးပါတယ်။

ဒီ Lesson ပြီးရင်:

```
Swing UI

    |

Service

    |

Repository

    |

Database Layer

    |

MySQL

```

အထိ ချိတ်ဆက်နိုင်ပါမယ်။

---

# 1. Why Database Layer Important?

Beginner Code:

```java
Connection con =
DriverManager.getConnection(
"url",
"user",
"password"
);

```

Problem:

- Connection management မရှိ
    
- Resource leak ဖြစ်နိုင်
    
- Configuration hard coded
    
- Transaction control ခက်
    

Production Application မှာ:

```
Application

     |

Connection Pool

     |

Database Connection

     |

MySQL

```

သုံးပါတယ်။

---

# 2. Database Architecture

Our Design:

```
database


├── DatabaseConfig

├── ConnectionPool

├── DatabaseManager

├── TransactionManager

└── MigrationManager


```

---

# 3. Add HikariCP Concept

HikariCP က:

> High Performance JDBC Connection Pool

ဖြစ်ပါတယ်။

---

Without Pool:

```
Request 1

Create Connection
     |
Close


Request 2

Create Connection
     |
Close


```

Problem:

Connection creation က expensive ဖြစ်ပါတယ်။

---

With Pool:

```
Application Start


Create 10 Connections


     |
     |
     |


Reuse Connections


```

---

# 4. Database Properties

Already created:

```
resources/database.properties

```

Example:

```properties
db.url=jdbc:mysql://localhost:3306/cafe_pos

db.username=root

db.password=password

db.pool.size=10

```

---

# 5. Create DatabaseConfig Class

Package:

```
config

   DatabaseConfig.java

```

Purpose:

Read database settings.

Code:

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

# 6. Test Configuration

Example:

```java
System.out.println(
DatabaseConfig.get(
"db.url"
)
);

```

Output:

```
jdbc:mysql://localhost:3306/cafe_pos

```

---

# 7. Create ConnectionPool

Package:

```
database

    ConnectionPool.java

```

Code:

```java
package com.cafe.pos.database;


import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import com.cafe.pos.config.DatabaseConfig;


import java.sql.Connection;
import java.sql.SQLException;



public class ConnectionPool {


private static HikariDataSource dataSource;



static {


HikariConfig config =
new HikariConfig();



config.setJdbcUrl(
DatabaseConfig.get(
"db.url"
)
);



config.setUsername(
DatabaseConfig.get(
"db.username"
)
);



config.setPassword(
DatabaseConfig.get(
"db.password"
)
);



config.setMaximumPoolSize(
Integer.parseInt(
DatabaseConfig.get(
"db.pool.size"
)
)
);



dataSource =
new HikariDataSource(
config
);


}



public static Connection getConnection()
throws SQLException{


return dataSource.getConnection();


}



}

```

---

# 8. Connection Flow

Now:

```
Repository


   |

ConnectionPool.getConnection()


   |

HikariCP


   |

MySQL


```

---

# 9. Create DatabaseManager

Why?

Central database control.

Package:

```
database

 DatabaseManager.java

```

---

Code:

```java
package com.cafe.pos.database;


import java.sql.Connection;
import java.sql.SQLException;



public class DatabaseManager {



private DatabaseManager(){}



public static Connection getConnection()
throws SQLException{


return ConnectionPool
.getConnection();


}



}

```

---

# 10. Test Database Connection

Create:

```
app/DatabaseTest.java

```

Code:

```java
package com.cafe.pos.app;


import com.cafe.pos.database.DatabaseManager;

import java.sql.Connection;



public class DatabaseTest {


public static void main(String[] args)
throws Exception{


try(Connection con =
DatabaseManager.getConnection()){


System.out.println(
"Database Connected Successfully"
);


}


}

}

```

---

Run:

Expected:

```
Database Connected Successfully

```

---

# 11. Create TransactionManager

Why?

POS requires transactions.

Example:

Order:

```
Save Order

      +

Save Order Items

      +

Decrease Inventory

      +

Save Payment


```

All success:

```
COMMIT

```

Any error:

```
ROLLBACK

```

---

Package:

```
database

 TransactionManager.java

```

---

Code:

```java
package com.cafe.pos.database;


import java.sql.Connection;
import java.sql.SQLException;



public class TransactionManager {



public static void begin(
Connection con
)
throws SQLException{


con.setAutoCommit(false);


}



public static void commit(
Connection con
)
throws SQLException{


con.commit();


}



public static void rollback(
Connection con
)
{


try{


con.rollback();


}
catch(SQLException e){


e.printStackTrace();


}


}



}

```

---

# 12. Transaction Example

Order Service:

```java
Connection con =
DatabaseManager
.getConnection();



try{


TransactionManager.begin(con);


// Save Order

// Save Items

// Update Stock



TransactionManager.commit(con);



}
catch(Exception e){


TransactionManager.rollback(con);


}

```

---

# 13. Auto Commit Explanation

Default JDBC:

```
INSERT

↓

COMMIT automatically

```

Problem:

Example:

```
Insert Order

SUCCESS


Update Stock

FAILED


```

Database:

```
Order exists

Stock not updated


```

Wrong.

---

Transaction:

```
BEGIN


Insert Order


Update Stock


Payment


COMMIT


```

or:

```
ROLLBACK

Everything cancelled

```

---

# 14. Database Exception

Create:

```
exception


DatabaseException.java

```

Code:

```java
package com.cafe.pos.exception;



public class DatabaseException
extends RuntimeException{


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

# 15. Replace Runtime Exception

Bad:

```java
throw new RuntimeException(
"Database Error"
);

```

Good:

```java
throw new DatabaseException(
"Cannot connect database",
e
);

```

---

# 16. Connection Closing

Old:

```java
Connection con =
getConnection();


con.close();

```

Problem:

If exception happens:

Connection leak.

---

Correct:

```java
try(Connection con =
DatabaseManager.getConnection()){


}

```

Java automatically closes.

---

# 17. Database Layer Final Structure

Now:

```
database


├── ConnectionPool.java

├── DatabaseManager.java

├── TransactionManager.java


config


└── DatabaseConfig.java


exception


└── DatabaseException.java


```

---

# 18. Professional Database Flow

Complete:

```
ProductController


       |

ProductService


       |

ProductRepository


       |

DatabaseManager


       |

HikariCP


       |

MySQL


```

---

# 19. Testing Checklist

Before moving forward:

Database:

✅ MySQL running

✅ Database created

✅ Properties loaded

✅ Connection successful

✅ Pool working

✅ Transaction working

---

# 20. Common Production Mistakes

Avoid:

❌ Connection inside JFrame

❌ Hard coded password

❌ No transaction

❌ No connection pool

❌ No rollback

❌ No exception wrapping

---

# Lesson 32 Summary

ဒီနေ့ Build လုပ်ခဲ့တာ:

✅ Database Layer Architecture  
✅ HikariCP Connection Pool  
✅ Database Configuration  
✅ Database Manager  
✅ Transaction Manager  
✅ Commit / Rollback  
✅ JDBC Resource Management  
✅ Database Exception Handling  
✅ Enterprise Database Foundation

---

# Next Lesson

# Lesson 33: Database Migration System + Initial Schema Creation

## Creating Café POS MySQL Database Automatically

Next we build:

✅ Migration Runner  
✅ V1__init.sql  
✅ Roles Table  
✅ Users Table  
✅ Product Tables  
✅ Inventory Tables  
✅ Order Tables  
✅ Foreign Key Design  
✅ Index Optimization

ပြီးရင် Café POS Database ကို **Professional Deployment Ready Database** ဖြစ်အောင် တည်ဆောက်ပါမယ်။