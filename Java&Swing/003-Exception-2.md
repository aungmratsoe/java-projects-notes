# Lesson: Build Complete Exception Handling System for Café POS using Java MVC Architecture

ဒီ Lesson မှာ **Real-world Java Swing + MySQL Café POS System** မှာ အသုံးပြုမယ့် **Professional Exception Handling Architecture** ကို တည်ဆောက်ပါမယ်။

ကျွန်တော်တို့ ရည်ရွယ်ချက်က:

- User ကို technical error မပြခြင်း
    
- Developer အတွက် error detail မပျောက်စေခြင်း
    
- Database error, Business error, Validation error ခွဲခြားခြင်း
    
- MVC Architecture နဲ့ clean ဖြစ်အောင် design လုပ်ခြင်း
    

---

# 1. Café POS Application Architecture Review

Professional POS System:

```
                 USER
                  |
                  |
              Swing View
                  |
                  |
             Controller
                  |
                  |
             Service Layer
                  |
                  |
                DAO
                  |
                  |
              MySQL Database
```

Exception Flow:

```
Database Error
      |
      v
 DAO catches SQLException
      |
      v
 throws DatabaseException
      |
      v
 Service catches/throws BusinessException
      |
      v
 Controller handles
      |
      v
 JOptionPane shows message
```

---

# 2. Project Package Structure

Professional structure:

```
src
 |
 |-- model
 |     |
 |     Product.java
 |     User.java
 |
 |-- view
 |     |
 |     LoginFrame.java
 |     ProductFrame.java
 |
 |-- controller
 |     |
 |     ProductController.java
 |
 |-- service
 |     |
 |     ProductService.java
 |
 |-- dao
 |     |
 |     ProductDAO.java
 |
 |-- exception
 |     |
 |     AppException.java
 |     DatabaseException.java
 |     ValidationException.java
 |     StockException.java
 |
 |-- util
       |
       ExceptionHandler.java
       LoggerUtil.java
```

---

# 3. Create Base Exception

အရင်ဆုံး Application မှာရှိမယ့် Exception အားလုံးရဲ့ Parent Class ဖန်တီးပါမယ်။

## AppException.java

```java
package exception;


public class AppException extends RuntimeException {


    private String errorCode;


    public AppException(
            String errorCode,
            String message
    ){

        super(message);
        this.errorCode = errorCode;

    }


    public AppException(
            String errorCode,
            String message,
            Throwable cause
    ){

        super(message,cause);
        this.errorCode = errorCode;

    }


    public String getErrorCode(){

        return errorCode;

    }

}
```

---

ဒီ Class က:

```
AppException
     |
     |
-------------------
|        |         |
DB     Stock    Validation
```

အတွက် Base ဖြစ်ပါတယ်။

---

# 4. Database Exception

Database Layer အတွက်:

## DatabaseException.java

```java
package exception;


public class DatabaseException 
extends AppException {


    public DatabaseException(
            String message,
            Throwable cause
    ){

        super(
            "DB_001",
            message,
            cause
        );

    }

}
```

---

Example:

Database connection fail:

```
DB_001
Database connection failed
```

---

# 5. Validation Exception

User Input Error:

## ValidationException.java

```java
package exception;


public class ValidationException 
extends AppException {


    public ValidationException(
            String message
    ){

        super(
            "VALID_001",
            message
        );

    }

}
```

---

Example:

```
VALID_001

Product name cannot be empty
```

---

# 6. Stock Exception

Inventory error:

## StockException.java

```java
package exception;


public class StockException 
extends AppException {


    public StockException(
            String message
    ){

        super(
            "STOCK_001",
            message
        );

    }

}
```

---

Example:

```
STOCK_001

Not enough coffee beans
```

---

# 7. DAO Layer Exception Handling

DAO က Database နဲ့ တိုက်ရိုက်ဆက်သွယ်ပါတယ်။

Example:

ProductDAO.java

```java
public class ProductDAO {


public void save(Product product){


Connection con=null;


try{


con = DBConnection.getConnection();


PreparedStatement ps =
con.prepareStatement(
"INSERT INTO products VALUES(?)"
);


ps.executeUpdate();



}
catch(SQLException e){


throw new DatabaseException(
"Unable to save product",
e
);


}


}


}
```

---

ဒီမှာ:

Original:

```
SQLException
```

ကို

ပြောင်းလိုက်ပါတယ်:

```
DatabaseException
```

အဖြစ်။

---

# 8. Service Layer Business Exception

Service မှာ Business Rule တွေရှိပါတယ်။

Example:

ProductService.java

```java
public class ProductService {


private ProductDAO dao =
new ProductDAO();



public void sellProduct(
Product product,
int qty
){


if(qty <=0){

throw new ValidationException(
"Quantity must be greater than zero"
);

}



if(product.getStock() < qty){


throw new StockException(
"Not enough stock"
);


}



dao.save(product);


}


}
```

---

Service က:

- Input validate
    
- Business rule check
    
- DAO call
    

လုပ်ပါတယ်။

---

# 9. Controller Exception Handling

Controller က View နဲ့ Service ကြားမှာရှိပါတယ်။

Example:

ProductController.java

```java
public class ProductController {



private ProductService service =
new ProductService();



public void sell(Product product,int qty){


try{


service.sellProduct(product,qty);



JOptionPane.showMessageDialog(
null,
"Sale Successful"
);



}
catch(AppException e){


ExceptionHandler.handle(e);


}


}


}
```

---

Controller မှာ:

```java
catch(AppException e)
```

တစ်ခုတည်းနဲ့

အားလုံး handle လုပ်နိုင်ပါတယ်။

---

# 10. Global Exception Handler

ExceptionHandler.java

```java
package util;


import exception.AppException;

import javax.swing.JOptionPane;



public class ExceptionHandler {



public static void handle(
AppException e
){


String message =
e.getErrorCode()
+
"\n"
+
e.getMessage();



JOptionPane.showMessageDialog(
null,
message,
"Error",
JOptionPane.ERROR_MESSAGE
);



}


}
```

---

User မြင်ရမယ့် output:

```
STOCK_001

Not enough stock
```

---

# 11. Exception Logging

Production System မှာ error သိမ်းရပါတယ်။

Example:

```
logs

 |
 |-- application.log

```

---

LoggerUtil.java

```java
import java.util.logging.*;


public class LoggerUtil {


private static Logger logger =
Logger.getLogger("POS");



public static void log(Exception e){


logger.severe(
e.getMessage()
);


}


}
```

---

Update ExceptionHandler:

```java
public static void handle(AppException e){


LoggerUtil.log(e);



JOptionPane.showMessageDialog(
null,
e.getMessage()
);


}
```

---

အခု flow:

```
Exception

   |
   |
Logger

   |
   |
User Message

```

---

# 12. Exception Chaining Example

Database:

```java
catch(SQLException e){


throw new DatabaseException(
"Cannot connect database",
e
);


}
```

---

ဒါဆို:

```
DatabaseException

    |
    |
SQLException

    |
    |
Connection refused

```

မပျောက်ပါဘူး။

Debug လုပ်လို့ရပါတယ်။

---

# 13. Swing Button Example

Login Button:

```java
loginButton.addActionListener(e->{


try{


controller.login(
username,
password
);



}
catch(AppException ex){


ExceptionHandler.handle(ex);


}


});
```

---

User:

```
username:
admin

password:
wrong
```

Output:

```
AUTH_001

Invalid username or password
```

---

# 14. Real POS Exception Map

## Authentication

```
AuthenticationException

AUTH_001
Invalid username/password
```

## Product

```
ProductNotFoundException

PRODUCT_001

Product does not exist
```

## Inventory

```
StockException

STOCK_001

Insufficient stock
```

## Payment

```
PaymentException

PAY_001

Payment failed
```

## Database

```
DatabaseException

DB_001

Database unavailable
```

---

# 15. What Senior Java Developers Do

Beginner:

```java
try{

}
catch(Exception e){

System.out.println(e);

}
```

Senior:

```java
try{

service.processOrder();


}
catch(BusinessException e){

showUserMessage(e);


}
catch(DatabaseException e){

logError(e);


}
```

---

# 16. Final Architecture

```
             Swing UI
                |
                |
          Controller
                |
        catch(AppException)
                |
                |
          Service Layer
                |
      Business Exceptions
                |
                |
            DAO Layer
                |
       Database Exceptions
                |
                |
             MySQL


ExceptionHandler
        |
        |
   Logger + Message
```

---

# Practice Task (Recommended)

Café POS မှာ ဒီ Exceptions တွေ create လုပ်ပါ:

```
exception

AppException

AuthenticationException

ProductNotFoundException

StockException

PaymentException

DatabaseException

ValidationException
```

ပြီးရင်:

Flow တစ်ခုရေးပါ:

```
Click Sell Button

↓

Validate Product

↓

Check Stock

↓

Save Sale

↓

Update Inventory

↓

Show Success

```

Error ဖြစ်ရင်:

```
StockException
DatabaseException
ValidationException
```

ကို handle လုပ်ပါ။

---

ဒီ Lesson ပြီးရင် သင်က **Professional Swing Application တွေမှာ Exception Architecture ဘယ်လို design လုပ်ရမလဲ** ဆိုတာ နားလည်သွားပါပြီ။

နောက် Lesson (Part 2 မစခင် Recommended):

**"Java Exception Handling Advanced Topics: Global Handler, Custom Error Code, Logging, Debugging, Best Practices"**

ပြီးရင်:

**Part 2 → Collections Framework Deep Dive** ကို စပါမယ်။