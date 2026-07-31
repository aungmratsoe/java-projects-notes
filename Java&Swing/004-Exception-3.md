# Java Exception Handling Advanced Topics

## Global Handler, Custom Error Code, Logging, Debugging, Best Practices

ဒီ Lesson က **Junior → Senior Java Developer Level** အတွက် ဖြစ်ပါတယ်။

ယခင် Lesson မှာ:

- Custom Exception
    
- Exception Hierarchy
    
- MVC Exception Flow
    

ကို လေ့လာခဲ့ပါတယ်။

ဒီတစ်ခါမှာ Production Application တွေမှာ တကယ်အသုံးပြုတဲ့:

1. Global Exception Handler
    
2. Custom Error Code System
    
3. Logging System
    
4. Debugging Strategy
    
5. Exception Design Best Practices
    
6. Café POS Production Example
    

ကို လေ့လာပါမယ်။

---

# 1. Why Advanced Exception Handling?

Beginner Application:

```java
try {

}
catch(Exception e){

    System.out.println(e);

}
```

Problem:

- Error ဘယ်နေရာက ဖြစ်လဲမသိ
    
- User ကို technical message ပြ
    
- Log မရှိ
    
- Debug ခက်
    

---

Enterprise Application:

```
Exception
    |
    |
Global Handler
    |
    |
------------------
|                |
User Message    Developer Log

```

---

# 2. Global Exception Handler Concept

## Problem

Swing Application မှာ Frame တိုင်းမှာ:

```java
try {

    service.save();

}
catch(Exception e){

}

```

ရေးနေရမယ်။

Example:

```
LoginFrame.java
ProductFrame.java
OrderFrame.java
ReportFrame.java
```

အားလုံးမှာ duplicate code ဖြစ်ပါတယ်။

---

## Solution

Central Exception Handler တစ်ခုထားမယ်။

Structure:

```
util

 └── GlobalExceptionHandler.java

```

---

# 3. Create Global Exception Handler

```java
package util;


import exception.AppException;


public class GlobalExceptionHandler {


    public static void handle(
            Exception e
    ){


        if(e instanceof AppException){


            AppException appException =
            (AppException)e;


            showUserMessage(
                appException
            );


        }
        else{


            showSystemError();


        }


        LoggerUtil.log(e);

    }



    private static void showUserMessage(
            AppException e
    ){

        JOptionPane.showMessageDialog(
            null,
            e.getMessage(),
            "Application Error",
            JOptionPane.ERROR_MESSAGE
        );

    }



    private static void showSystemError(){

        JOptionPane.showMessageDialog(
            null,
            "Unexpected error occurred"
        );

    }


}
```

---

အသုံးပြုခြင်း:

```java
try{

    productService.save(product);

}
catch(Exception e){

    GlobalExceptionHandler.handle(e);

}

```

---

# 4. Custom Error Code System

Professional Application တွေမှာ Message တစ်ခုတည်းမသုံးပါဘူး။

Bad:

```
Product error
```

Good:

```
PRODUCT_001
Product not found
```

---

## Why Error Code?

Support Team:

```
Customer:
"I cannot sell coffee"

Developer:
"Show error code"

Customer:
"STOCK_001"

Developer:
"Stock validation problem"

```

---

# 5. Design Error Code

Example:

```
AUTH
Authentication


USER
User Management


PRODUCT
Product


STOCK
Inventory


ORDER
Order


PAY
Payment


DB
Database

```

---

Format:

```
MODULE_NUMBER

Example:

STOCK_001

```

---

# 6. ErrorCode Class

မဖြန့်ကျက်ထားဘဲ Constant အနေနဲ့သိမ်းပါ။

```java
public class ErrorCode {


    public static final String DB_ERROR =
            "DB_001";


    public static final String STOCK_ERROR =
            "STOCK_001";


    public static final String PRODUCT_NOT_FOUND =
            "PRODUCT_001";


    public static final String INVALID_LOGIN =
            "AUTH_001";


}

```

---

အသုံးပြု:

```java
throw new StockException(
        ErrorCode.STOCK_ERROR,
        "Not enough stock"
);

```

---

# 7. Better Custom Exception Design

Previous:

```java
public class StockException
extends RuntimeException

```

Professional:

```java
public class StockException
extends AppException {


public StockException(
String code,
String message
){

super(code,message);

}


}

```

---

# 8. Exception Response Object Pattern

Large System တွေမှာ Exception ကို Object အဖြစ်ပြောင်းပါတယ်။

Example:

```java
public class ErrorResponse {


private String code;

private String message;

private LocalDateTime time;


}

```

---

Output:

```json
{
 "code":"STOCK_001",
 "message":"Not enough stock",
 "time":"2026-07-31T10:00"
}

```

---

ဒီ Pattern ကို:

- REST API
    
- Microservice
    
- Enterprise System
    

တွေမှာသုံးပါတယ်။

---

# 9. Logging System

Exception ဖြစ်ရင်:

User ကို:

```
Cannot complete sale
```

Developer ကို:

```
SQLException:
Connection timeout
Database URL...
Line 45

```

လိုပါတယ်။

---

# 10. Java Logging

Java built-in:

```
java.util.logging

```

Example:

```java
import java.util.logging.Logger;


public class LoggerUtil {


private static final Logger logger =
Logger.getLogger("POS");


public static void error(
Exception e
){

logger.severe(
e.getMessage()
);


}


}

```

---

Usage:

```java
catch(Exception e){

LoggerUtil.error(e);

}

```

---

# 11. Log Levels

Java Logging:

|Level|Meaning|
|---|---|
|SEVERE|Critical Error|
|WARNING|Warning|
|INFO|Normal Information|
|CONFIG|Configuration|
|FINE|Debug|

---

Example:

```java
logger.info(
"User logged in"
);


logger.warning(
"Stock level low"
);


logger.severe(
"Database crashed"
);

```

---

# 12. Professional Logging Library

Enterprise တွေမှာ:

- Log4j2
    
- Logback
    
- SLF4J
    

အသုံးများပါတယ်။

Architecture:

```
Application

    |
    |
 SLF4J

    |
    |
Logback

    |
    |
File

```

---

Log file:

```
logs

 └── application.log

```

Example:

```
2026-07-31 10:00 ERROR

DB_001

Connection failed

SQLException

```

---

# 13. Debugging Exception

Exception Stack Trace:

```java
NullPointerException

at ProductService.java:45

at ProductController.java:20

at ProductFrame.java:100

```

---

Read from bottom to top:

```
User Click
   |
Frame
   |
Controller
   |
Service
   |
Error Line

```

---

# 14. Debugging Strategy

## Step 1

Read Exception Type

Example:

```
NullPointerException

```

Question:

"Which object is null?"

---

## Step 2

Find Line Number

```
ProductService.java:45

```

---

## Step 3

Check Data Flow

Example:

```
Database

 ↓

DAO

 ↓

Service

 ↓

Controller

 ↓

UI

```

---

# 15. Common Real-world Exceptions

## NullPointerException

Bad:

```java
product.getName();

```

product null ဖြစ်နိုင်ပါတယ်။

---

Good:

```java
if(product == null){

throw new ProductNotFoundException(
"Product missing"
);

}

```

---

# NumberFormatException

User:

```
Quantity:
abc

```

Code:

```java
Integer.parseInt("abc");

```

Solution:

```java
try{

int qty =
Integer.parseInt(text);


}
catch(NumberFormatException e){

throw new ValidationException(
"Invalid quantity"
);

}

```

---

# SQLException

DAO:

```java
catch(SQLException e){

throw new DatabaseException(
"Database error",
e
);

}

```

---

# 16. Exception Handling in Swing Event

Bad:

```java
button.addActionListener(e->{


service.save();


});

```

Error ဖြစ်ရင် application crash ဖြစ်နိုင်ပါတယ်။

---

Good:

```java
button.addActionListener(e->{


try{


service.save();



}
catch(Exception ex){


GlobalExceptionHandler.handle(ex);


}


});

```

---

# 17. Transaction Exception Example

Order Process:

```
Create Order

     |
Save Order

     |
Reduce Stock

     |
Payment

```

Problem:

Payment fail ဖြစ်ရင်?

Order save ပြီးသားဖြစ်နေမယ်။

---

Solution:

Database Transaction:

```
BEGIN TRANSACTION


Save Order

Update Stock

Payment


SUCCESS

COMMIT


ERROR

ROLLBACK

```

Exception Handling နဲ့ Database Transaction တွဲသုံးရပါတယ်။

---

# 18. Exception Best Practices

## Rule 1: Never Catch and Ignore

Bad:

```java
catch(Exception e){

}

```

---

Good:

```java
catch(Exception e){

logger.error(e);

throw e;

}

```

---

# Rule 2: Catch Specific Exception

Bad:

```java
catch(Exception e)

```

Good:

```java
catch(SQLException e)

```

---

# Rule 3: Don't Use Exception for Normal Flow

Bad:

```java
try{

findUser();

}
catch(UserNotFoundException e){

createUser();

}

```

---

Better:

```java
if(userExists()){

}

```

---

# Rule 4: Preserve Original Cause

Bad:

```java
throw new DatabaseException(
"Error"
);

```

Original error ပျောက်သွားပါတယ်။

---

Good:

```java
throw new DatabaseException(
"Database failed",
e
);

```

---

# Rule 5: User Message vs Developer Message

User:

```
Unable to save product

```

Developer Log:

```
SQLException:
Duplicate key PRODUCT_ID
Table products
Line 65

```

---

# 19. Final Café POS Exception Architecture

```
                 Swing UI

                    |

              Controller

                    |

        GlobalExceptionHandler

                    |

              Service Layer

                    |

          ------------------

          |                |

   Business Error     Database Error


          |                |

 StockException    DatabaseException


                    |

                  DAO

                    |

                MySQL


                    |

                 Logger

                    |

             application.log

```

---

# What You Should Practice Now

Create this mini system:

## Project: Bank Account

Classes:

```
Account

AccountService

AccountController

```

Exceptions:

```
AppException

InsufficientBalanceException

InvalidAccountException

DatabaseException

```

Features:

```
Deposit

Withdraw

Transfer

Login

```

Add:

```
Error Code

Logger

Global Handler

```

---

ဒီ Lesson ပြီးရင် သင်က **Professional Java Application Exception Architecture** ကို နားလည်ပြီးပါပြီ။

နောက်တစ်ဆင့်အနေနဲ့:

# Part 2: Advanced Java Knowledge for Swing

## Lesson 1: Collections Framework Deep Dive

မှာ:

- ArrayList Internals
    
- ArrayList vs LinkedList
    
- HashMap Internals
    
- HashSet
    
- List / Set / Map
    
- Café POS Inventory Data Handling
    
- JTable Integration
    

ကို စတင်ပါမယ်။