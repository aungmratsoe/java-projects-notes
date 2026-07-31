# Part 2: Advanced Java Knowledge for Swing

# Lesson 19: Enterprise Exception Handling & Logging Architecture

## Building Production Error Management System for Café POS

### (Java 25 + MVC + JDBC + Enterprise Practice)

ဒီ Lesson က အရေးကြီးပါတယ်။

Beginner Developer တွေက Exception ကို ဒီလိုပဲ သုံးတတ်ပါတယ်။

```java
try {

    saveOrder();

}
catch(Exception e){

    e.printStackTrace();

}
```

ဒါက Production Application အတွက် မလုံလောက်ပါဘူး။

Enterprise Application မှာ Error ဖြစ်ရင်:

- User ကို ဘာပြောမလဲ?
    
- Developer က ဘယ်လိုရှာမလဲ?
    
- Log ဘယ်မှာသိမ်းမလဲ?
    
- Database Error နဲ့ Network Error ကို ဘယ်လိုခွဲမလဲ?
    
- Error Code ဘယ်လိုပေးမလဲ?
    

ဆိုတာတွေ စဉ်းစားရပါတယ်။

---

# 1. Production Exception Handling Architecture

Café POS မှာ Error Flow:

```
User

 |
 |
Swing UI

 |
 |
Controller

 |
 |
Service Layer

 |
 |
Repository

 |
 |
Database


Exception

 |
 |
Global Exception Handler

 |
 +----------------+
 |
User Message
 |
Log File
 |
Database Audit

```

---

# 2. Exception Types in Real Applications

Java Exception ကို ၂ မျိုးခွဲပါတယ်။

## 1. Checked Exception

Compiler က handle လုပ်ခိုင်းပါတယ်။

Example:

```java
IOException
SQLException
```

Example:

```java
try {

    Files.readString(path);

}
catch(IOException e){

}

```

---

## 2. Unchecked Exception

`RuntimeException` ကို extend လုပ်ထားတာ။

Example:

```java
NullPointerException
IllegalArgumentException
```

---

Enterprise Application တွေမှာ:

```
Business Error

        |
        |
 RuntimeException

```

ကို များစွာသုံးပါတယ်။

---

# 3. Why Custom Exception?

Problem:

JDBC Exception:

```java
SQLException
```

User ကိုပြရင်:

```
Communications link failure
Access denied
Duplicate entry
```

User နားမလည်ပါ။

---

Better:

```
Order cannot be completed.

Error Code:
ORDER_001

Please contact administrator.

```

---

# 4. Application Exception Base Class

Architecture:

```
AppException

     |
     |
 --------------------

DatabaseException

BusinessException

NetworkException

ValidationException

```

---

Create:

```java
public abstract class AppException
extends RuntimeException {


private final String errorCode;


public AppException(
String errorCode,
String message
){

super(message);

this.errorCode = errorCode;

}


public String getErrorCode(){

return errorCode;

}


}

```

---

Now all application errors have:

- Code
    
- Message
    
- Stack Trace
    

---

# 5. Database Exception

```java
public class DatabaseException
extends AppException {


public DatabaseException(
String message,
Throwable cause
){

super(
"DB_001",
message
);

initCause(cause);

}

}

```

---

Usage:

```java
try {


save();


}
catch(SQLException e){


throw new DatabaseException(
"Cannot save product",
e
);


}

```

---

# 6. Business Exception

Business rule:

Example:

```
Stock = 0

Cannot sell product

```

---

Create:

```java
public class BusinessException
extends AppException {


public BusinessException(
String code,
String message
){

super(code,message);

}

}

```

---

Usage:

```java
if(product.stock()==0){

throw new BusinessException(

"STOCK_001",

"Product out of stock"

);

}

```

---

# 7. Validation Exception

Example:

Customer name empty.

```java
throw new ValidationException(

"VAL_001",

"Customer name required"

);

```

---

# 8. Error Code Design

Professional system:

```
CATEGORY_NUMBER

```

Example:

```
AUTH_001

DB_001

ORDER_001

PAY_001

INV_001

NET_001

```

---

Meaning:

|Code|Meaning|
|---|---|
|AUTH|Login|
|DB|Database|
|ORDER|Order|
|PAY|Payment|
|INV|Inventory|
|NET|Network|

---

# 9. Café POS Error Example

Scenario:

Customer orders coffee.

Flow:

```
Create Order

     |

Check Stock

     |

Stock = 0

     |

BusinessException

```

Error:

```
Code:
INV_001


Message:
Coffee is unavailable

```

---

# 10. Global Exception Handler

Problem:

Every Controller:

```java
try {

}
catch(Exception e){

}

```

Bad.

---

Need:

```
One Central Handler

```

---

Example:

```java
public class GlobalExceptionHandler {


public static void handle(
Exception e
){


if(e instanceof AppException app){


showUserMessage(
app.getMessage()
);


}


else {


showUserMessage(
"Unexpected error"
);


}


}

}

```

---

# 11. Swing Integration

Button:

```java
saveButton.addActionListener(e -> {


try{


controller.saveOrder();


}
catch(Exception ex){


GlobalExceptionHandler.handle(ex);


}


});

```

---

Better:

```java
saveButton.addActionListener(
new SaveOrderAction()
);

```

---

# 12. User Friendly Error Message

Never show:

```
java.sql.SQLException:
Duplicate entry '5'

```

---

Show:

```
Unable to save order.

Please try again.

```

---

Developer sees:

```
DB_001

Duplicate key

Stack trace

```

---

# 13. Logging Concept

Logging means:

> Recording application events for debugging and monitoring.

---

Difference:

System.out:

```java
System.out.println(
"Order saved"
);

```

Not professional.

---

Logging:

```java
logger.info(
"Order saved"
);

```

---

Benefits:

- File storage
    
- Levels
    
- Search
    
- Rotation
    

---

# 14. Logging Levels

Common:

## TRACE

Very detailed.

## DEBUG

Developer information.

## INFO

Normal operation.

Example:

```
Order #100 created

```

## WARN

Potential problem.

Example:

```
Low stock

```

## ERROR

Failure.

Example:

```
Database connection failed

```

---

# 15. SLF4J + Logback

Professional Java uses:

```
Application

    |

SLF4J API

    |

Logback Implementation

```

---

Maven:

```xml
<dependency>

<groupId>
org.slf4j
</groupId>

<artifactId>
slf4j-api
</artifactId>

</dependency>


<dependency>

<groupId>
ch.qos.logback
</groupId>

<artifactId>
logback-classic
</artifactId>

</dependency>

```

---

# 16. Creating Logger

Example:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


public class OrderService {


private static final Logger log =
LoggerFactory.getLogger(
OrderService.class
);


}

```

---

# 17. Logging Example

```java
public void createOrder(Order order){


log.info(
"Creating order {}",
order.id()
);


}

```

---

Output:

```
INFO OrderService:
Creating order 100

```

---

# 18. Exception Logging

Wrong:

```java
catch(Exception e){

log.error(
e.getMessage()
);

}

```

---

You lose stack trace.

---

Correct:

```java
catch(Exception e){


log.error(
"Order creation failed",
e
);


}

```

---

Output:

```
ERROR Order creation failed

java.sql.SQLException
 at ProductRepository.java:50

```

---

# 19. Log Configuration

File:

```
logback.xml

```

Example:

```xml
<configuration>


<appender name="FILE">

</appender>


<root level="INFO">

<appender-ref
ref="FILE"/>

</root>


</configuration>

```

---

Production:

```
logs/

 |
 |
app.log

 |
 |
error.log

```

---

# 20. Audit Logging

Normal Log:

```
Database connection failed

```

Audit Log:

```
USER:
admin


ACTION:
DELETE PRODUCT


PRODUCT:
Coffee


TIME:
2026-07-31 10:30

```

---

Used for:

- Security
    
- Tracking
    
- Business history
    

---

# 21. Café POS Audit Example

When cashier deletes order:

Database table:

```sql
CREATE TABLE audit_logs(

id BIGINT PRIMARY KEY,

user_id INT,

action VARCHAR(100),

created_at TIMESTAMP

);

```

---

Insert:

```
DELETE_ORDER

USER=5

TIME=...

```

---

# 22. Exception + Logging Architecture

Complete:

```
Exception

    |

AppException

    |

Global Handler

    |

+-------------+

|             |

User UI     Logger

              |

           Log File

              |

          Audit DB

```

---

# 23. Service Layer Example

```java
public void createOrder(Order order){


try{


repository.save(order);


log.info(
"Order created {}",
order.id()
);


}
catch(SQLException e){


log.error(
"Database failure",
e
);


throw new DatabaseException(

"Cannot create order",

e

);


}


}

```

---

# 24. Never Catch Everything

Bad:

```java
catch(Exception e){

}

```

Why?

You hide bugs.

---

Better:

```java
catch(SQLException e){

}


catch(IOException e){

}

```

---

# 25. Debugging Strategy

When bug happens:

Step 1:

Read error code.

Example:

```
ORDER_001

```

Step 2:

Search logs.

```
app.log

```

---

Step 3:

Find stack trace.

```
OrderService.java:55

```

---

Step 4:

Reproduce.

---

# 26. Production Error Handling Rules

Always:

✅ Custom Exceptions

✅ Error Codes

✅ Central Handler

✅ Logging

✅ Stack Trace

✅ User Friendly Messages

✅ Audit Important Actions

Avoid:

❌ printStackTrace()

❌ Empty catch blocks

❌ Showing technical errors

❌ Logging passwords

---

# 27. Java 25 Modern Exception Design

Using sealed hierarchy:

```java
public sealed class AppException
extends RuntimeException

permits
DatabaseException,
BusinessException,
NetworkException {

}

```

---

Modern pattern matching:

```java
switch(exception){

case DatabaseException d ->
handleDatabase(d);


case BusinessException b ->
handleBusiness(b);


default ->
handleUnknown();

}

```

---

# 28. Café POS Final Exception Architecture

Package:

```
com.cafe.pos.exception


AppException.java

DatabaseException.java

BusinessException.java

ValidationException.java

NetworkException.java

GlobalExceptionHandler.java

ErrorCode.java

```

---

# 29. Practice Project

Implement:

## ErrorCode Enum

```java
public enum ErrorCode {


DB_001,

ORDER_001,

INV_001,

PAY_001


}

```

---

## Create:

```
AppException

DatabaseException

InventoryException

PaymentException

```

---

## Add:

```
GlobalExceptionHandler

SLF4J Logger

AuditLogger

```

---

# Lesson 19 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Enterprise Exception Architecture  
✅ Custom Exception Hierarchy  
✅ Error Code System  
✅ Business Exception  
✅ Database Exception  
✅ Global Exception Handler  
✅ Logging Architecture  
✅ SLF4J  
✅ Logback  
✅ Audit Logging  
✅ Debugging Strategy  
✅ Production Error Handling Best Practices

---

# Next Lesson

# Lesson 20: Advanced Multithreading for Café POS

## Java 25 Concurrency Architecture

Topics:

- Thread Safety
    
- ExecutorService
    
- Thread Pool
    
- CompletableFuture
    
- Virtual Threads
    
- Atomic Classes
    
- Concurrent Collections
    
- Race Condition
    
- Deadlock Prevention
    
- Async Order Processing
    

ဒီ Lesson မှာ သင်အရင်မေးခဲ့တဲ့ **ExecutorService, AtomicBoolean, Thread Pool** တွေကို Café POS real-world example နဲ့ deep dive လုပ်ပါမယ်။