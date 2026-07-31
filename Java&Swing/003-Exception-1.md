ကောင်းပါတယ်။ Part 2 မစခင် **Custom Exception ကို Deep Level (Professional Java Developer Level)** အထိ လေ့လာသင့်ပါတယ်။

Real-world Application (Café POS, Banking, E-commerce, ERP) တွေမှာ **Custom Exception Design** က အရေးကြီးပါတယ်။ Beginner တွေက `try-catch` သာ သိတတ်ပေမယ့် Senior Developer တွေက **Exception Architecture** ကို design လုပ်တတ်ရပါတယ်။

ဒီ Lesson မှာ:

1. Why Custom Exception?
    
2. Custom Exception Types
    
3. Checked Custom Exception
    
4. Unchecked Custom Exception
    
5. Exception Hierarchy Design
    
6. Real-world Layered Architecture
    
7. Error Code System
    
8. Global Exception Handler
    
9. Logging
    
10. Best Practices
    

---

# 1. Why Do We Need Custom Exception?

Java မှာ built-in exceptions ရှိပါတယ်။

Example:

```java
NullPointerException
SQLException
IOException
NumberFormatException
```

ဒါပေမယ့် Business Logic ကို မဖော်ပြနိုင်ပါဘူး။

ဥပမာ:

Café POS System မှာ:

User က Coffee 10 ခွက် ဝယ်မယ်။

Database:

```
Product:
Coffee

Stock:
5
```

Error:

```java
throw new Exception("Error");
```

ဒီ message က ဘာဖြစ်တာလဲ မသိပါဘူး။

Better:

```java
throw new InsufficientStockException(
"Coffee stock is only 5"
);
```

အဓိပ္ပါယ်ရှင်းပါတယ်။

---

# 2. Custom Exception Basic Structure

Java Exception ကို inherit လုပ်ရပါတယ်။

Structure:

```
Throwable
    |
 Exception
    |
 MyCustomException
```

Example:

```java
public class InsufficientStockException 
extends Exception {


}
```

ဒါဆို Java ရဲ့ Exception တစ်ခုဖြစ်သွားပါပြီ။

---

# 3. Creating Professional Custom Exception

Beginner version:

```java
public class StockException 
extends Exception {

}
```

ဒါက မလုံလောက်သေးပါဘူး။

Professional version:

```java
package com.pos.exception;


public class StockException 
extends Exception {


    public StockException(){

        super();

    }


    public StockException(String message){

        super(message);

    }


    public StockException(
        String message,
        Throwable cause
    ){

        super(message,cause);

    }


}
```

---

ဘာကြောင့် Constructor ၃ ခုလိုလဲ?

## 1. Default

```java
new StockException();
```

## 2. Message

```java
new StockException(
"Stock is empty"
);
```

## 3. Cause chaining

```java
new StockException(
"Stock checking failed",
sqlException
);
```

Database error ကို မပျောက်စေပါဘူး။

---

# 4. Checked Custom Exception

Checked Exception ဆိုတာ:

Compiler က handle ခိုင်းပါတယ်။

အသုံးပြုသင့်တဲ့နေရာ:

- Business rules
    
- External systems
    
- Database operations
    

Example:

```java
public class PaymentException 
extends Exception {


public PaymentException(String message){

super(message);

}

}
```

---

Usage:

```java
public void pay(double amount)
throws PaymentException{


if(amount <=0){

throw new PaymentException(
"Invalid payment amount"
);

}


}
```

---

Call:

```java
try{

paymentService.pay(0);

}
catch(PaymentException e){

System.out.println(
e.getMessage()
);

}
```

---

# 5. Unchecked Custom Exception

အများဆုံး Enterprise Application တွေမှာ ဒီနည်းကို သုံးပါတယ်။

RuntimeException ကို extend လုပ်ပါတယ်။

Structure:

```
RuntimeException
       |
BusinessException
```

Example:

```java
public class ProductNotFoundException
extends RuntimeException{


public ProductNotFoundException(String message){

super(message);

}

}
```

---

အသုံးပြုခြင်း:

```java
public Product findById(int id){


Product product =
repository.find(id);


if(product == null){

throw new ProductNotFoundException(
"Product not found id="+id
);

}


return product;


}
```

---

ဘာကြောင့် RuntimeException သုံးလဲ?

Code cleaner ဖြစ်ပါတယ်။

မလိုအပ်တဲ့:

```java
throws Exception
```

တွေ မရေးရတော့ပါဘူး။

---

# 6. Checked vs Unchecked Custom Exception

||Checked|Unchecked|
|---|---|---|
|Extend|Exception|RuntimeException|
|Compile check|Yes|No|
|Use|Recoverable error|Business error|
|Code|More verbose|Cleaner|
|Enterprise|Less common|Very common|

---

# 7. Professional Exception Hierarchy Design

Large application မှာ Exception အများကြီးရှိပါတယ်။

မလုပ်သင့်:

```
StockException
PaymentException
UserException
DatabaseException
LoginException

(all extends Exception)
```

ပိုကောင်း:

```
AppException
      |
      |
 -----------------------
 |          |           |
Business   Database   Security
Exception  Exception  Exception

 |
 |
----------------
|              |
Stock       Payment
Exception   Exception
```

---

# 8. Base Application Exception

Create:

```
exception
 |
 AppException.java
```

Code:

```java
public class AppException
extends RuntimeException{


private String errorCode;


public AppException(
String errorCode,
String message
){

super(message);

this.errorCode=errorCode;

}


public String getErrorCode(){

return errorCode;

}


}
```

---

Now all exceptions can have:

```
Error Code
Message
Cause
```

---

# 9. Business Exception Example

## Stock Exception

```java
public class StockException
extends AppException{


public StockException(String message){

super(
"STOCK_001",
message
);

}


}
```

---

Usage:

```java
if(quantity > stock){

throw new StockException(
"Not enough product stock"
);

}
```

---

Output:

```
Code:
STOCK_001

Message:
Not enough product stock
```

---

# 10. Error Code System (Real Enterprise Style)

Instead of:

```
Something went wrong
```

Use:

```
USER_001
USER_002

PRODUCT_001
PRODUCT_002

STOCK_001

PAYMENT_001

DB_001
```

Example:

```java
throw new StockException(
"STOCK_001",
"Coffee stock unavailable"
);
```

---

Why?

Because:

- Support team can identify problem
    
- Logging easier
    
- Debugging faster
    
- API response easier
    

---

# 11. Exception in Layered Architecture

Professional POS:

```
VIEW
 |
 |
CONTROLLER
 |
 |
SERVICE
 |
 |
DAO
 |
 |
DATABASE
```

Example:

Database:

```java
SQLException
```

DAO catches:

```java
catch(SQLException e){

throw new DatabaseException(
"Database connection failed",
e
);

}
```

Service:

```java
catch(DatabaseException e){

throw new AppException(
"Cannot process order",
e
);

}
```

Controller:

```java
catch(AppException e){


JOptionPane.showMessageDialog(
null,
e.getMessage()
);


}
```

---

# 12. Exception Chaining (VERY IMPORTANT)

Bad:

```java
catch(SQLException e){

throw new DatabaseException(
"Database Error"
);

}
```

Original error ပျောက်သွားပါတယ်။

Good:

```java
catch(SQLException e){

throw new DatabaseException(
"Database Error",
e
);

}
```

Now:

```
DatabaseException

    |
    |
 SQLException
```

ရှိနေပါသေးတယ်။

---

# 13. Global Exception Handler

Swing Application မှာ:

User Action တိုင်းမှာ:

```java
try{

}
catch(Exception e){

}
```

မရေးချင်ပါဘူး။

Solution:

Central Handler

```
ExceptionHandler

       |
       |
Show Message
Log Error
Save Report
```

Example:

```java
public class ExceptionHandler{


public static void handle(Exception e){


JOptionPane.showMessageDialog(
null,
e.getMessage()
);


}


}
```

---

အသုံးပြု:

```java
try{

service.saveProduct();

}
catch(Exception e){

ExceptionHandler.handle(e);

}
```

---

# 14. Real Café POS Exception List

သင့် Project အတွက်:

```
exception

├── AppException

├── DatabaseException

├── AuthenticationException

├── AuthorizationException

├── ProductNotFoundException

├── DuplicateProductException

├── InsufficientStockException

├── InvalidOrderException

├── PaymentException

├── ReportGenerationException

```

---

# 15. Senior Developer Rules

## Rule 1

Never:

```java
catch(Exception e){

}
```

---

## Rule 2

Don't expose database error to user.

Database:

```
Duplicate entry 'coffee'
```

User:

```
Product already exists
```

---

## Rule 3

Every Exception should answer:

1. What happened?
    
2. Where happened?
    
3. Why happened?
    
4. How to solve?
    

---

# Mini Project Practice

ဒီအဆင့်မှာ လေ့ကျင့်ရန်:

Create:

```
Bank Application

Classes:

Account
AccountService

Custom Exceptions:

InsufficientBalanceException
InvalidAccountException
```

Requirements:

```java
withdraw(amount)

if balance < amount

throw InsufficientBalanceException
```

---

# Next Step Before Part 2

Custom Exception ကို ပိုနားလည်ဖို့ နောက် Lesson:

**"Build Complete Exception Handling System for Café POS using Java MVC Architecture"**

မှာ:

- Exception Package Design
    
- DAO Exception
    
- Service Exception
    
- Controller Handling
    
- Logger
    
- User Friendly Message
    
- Real Swing JOptionPane Handling
    

ကို တကယ့် Project ပုံစံနဲ့ တည်ဆောက်ပေးပါမယ်။

ပြီးရင်မှ **Part 2: Collections Framework Deep Dive** ကို စပါမယ်။