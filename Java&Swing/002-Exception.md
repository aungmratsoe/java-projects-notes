ကောင်းပါတယ်။ Java Swing / Enterprise Application (ဥပမာ သင်လုပ်မယ့် **Café POS System**) တည်ဆောက်မယ်ဆိုရင် **Exception Handling** ကို နားလည်ထားရမယ်။ Real-world application တွေမှာ Error ဖြစ်တာကို မဖျောက်ဘဲ **handle လုပ်ပြီး user ကို မှန်ကန်တဲ့ message ပေးခြင်း၊ log သိမ်းခြင်း၊ system မပျက်အောင် ထိန်းခြင်း** အတွက် အသုံးပြုပါတယ်။

ဒီ Lesson မှာ:

1. Exception ဆိုတာဘာလဲ
    
2. Exception Types
    
3. try-catch-finally
    
4. throws / throw
    
5. Custom Exception ဖန်တီးခြင်း
    
6. Real-world Application Design
    
7. Café POS Example
    

ပြီးရင် **Part 2 (Advanced Java for Swing)** ကို ဆက်သွားပါမယ်။

---

# Lesson: Java Exception Handling (Real-world Level)

# 1. Exception ဆိုတာဘာလဲ?

Exception ဆိုတာ **Program run နေစဉ် ဖြစ်ပေါ်လာတဲ့ unexpected problem** ဖြစ်ပါတယ်။

ဥပမာ:

```java
int result = 10 / 0;
```

Output:

```
Exception in thread "main"
java.lang.ArithmeticException: / by zero
```

Program က ရပ်သွားပါတယ်။

---

Real Application မှာ:

ဥပမာ Café POS:

User က quantity ထည့်တယ်:

```
Quantity:
abc
```

ဒါပေမယ့် program က:

```java
int qty = Integer.parseInt(input);
```

ဆိုရင် error ဖြစ်ပါတယ်။

ဒါကို handle မလုပ်ရင် application crash ဖြစ်နိုင်ပါတယ်။

---

# 2. Exception Hierarchy

Java Exception Structure:

```
                 Throwable
                    |
        ------------------------
        |                      |
      Error              Exception
                              |
                 ---------------------
                 |                   |
        RuntimeException        IOException
                 |
        ----------------
        |
   NullPointerException
   ArithmeticException
   NumberFormatException
```

---

# 3. Error vs Exception

## Error

System level problem

Example:

```
OutOfMemoryError
StackOverflowError
```

Developer များသောအားဖြင့် မhandle ပါ။

---

## Exception

Application problem

Example:

```
Database connection fail
Invalid input
File not found
User permission denied
```

ဒါတွေကို handle လုပ်ရပါတယ်။

---

# 4. Checked vs Unchecked Exception

## Checked Exception

Compile time မှာ check လုပ်ပါတယ်။

Example:

```java
FileReader file =
new FileReader("data.txt");
```

Compiler ကပြောမယ်:

```
Unhandled exception FileNotFoundException
```

Handle လုပ်ရမယ်။

---

## Unchecked Exception

Runtime မှာဖြစ်ပါတယ်။

Example:

```java
int x = 10/0;
```

```
ArithmeticException
```

---

# 5. try-catch

Basic Syntax:

```java
try{

    // risky code

}
catch(Exception e){

    // handle error

}
```

Example:

```java
public class Test {

public static void main(String[] args){

    try{

        int result = 10/0;

        System.out.println(result);

    }
    catch(Exception e){

        System.out.println(
            "Something went wrong"
        );

    }

}

}
```

Output:

```
Something went wrong
```

---

# 6. Multiple Catch

Real application မှာ error အမျိုးမျိုးရှိပါတယ်။

Example:

```java
try{

    int number =
    Integer.parseInt("abc");


}
catch(NumberFormatException e){

    System.out.println(
    "Invalid Number"
    );

}
catch(Exception e){

    System.out.println(
    "System Error"
    );

}
```

---

# 7. finally

`finally` က မဖြစ်မနေ execute ဖြစ်ပါတယ်။

အသုံးများတဲ့နေရာ:

- Database Connection ပိတ်ခြင်း
    
- File Close
    

Example:

```java
Connection con=null;


try{

    con = DBConnection.getConnection();


}
catch(Exception e){

}
finally{

    con.close();

}
```

---

# 8. throw Keyword

ကိုယ်တိုင် Exception ပစ်ချင်ရင်သုံးပါတယ်။

Example:

```java
public void withdraw(double amount){


    if(amount <=0){

        throw new IllegalArgumentException(
            "Invalid amount"
        );

    }

}
```

---

# 9. throws Keyword

Method က Exception ဖြစ်နိုင်ကြောင်း ကြိုပြောခြင်း။

Example:

```java
public void readFile()
throws IOException{


}
```

Call လုပ်တဲ့နေရာမှာ handle လုပ်ရမယ်။

---

# 10. Custom Exception

Real-world application မှာ built-in exception တွေမလုံလောက်ပါဘူး။

ဥပမာ POS System:

Built-in:

```
Exception
SQLException
NullPointerException
```

ဒါပေမယ့် Business Error တွေရှိပါတယ်။

ဥပမာ:

```
InsufficientStockException

InvalidLoginException

PaymentFailedException

ProductNotFoundException
```

ဒီလို custom exception ဖန်တီးပါတယ်။

---

# Creating Custom Exception

## Step 1: Create Class

Example:

```
exception
 |
 └── InsufficientStockException.java
```

Code:

```java
package exception;


public class InsufficientStockException
extends Exception{


    public InsufficientStockException(
        String message
    ){

        super(message);

    }


}
```

---

ဒါဆို Custom Exception ရပါပြီ။

---

# Step 2: Use Custom Exception

Example:

Product:

```java
public class InventoryService{


public void sellProduct(
int quantity
)
throws InsufficientStockException{


int stock=5;


if(quantity > stock){


throw new InsufficientStockException(
"Stock is not enough"
);


}


System.out.println(
"Sale completed"
);


}


}
```

---

အသုံးပြုခြင်း:

```java
public class Main{


public static void main(String[] args){


InventoryService service =
new InventoryService();


try{


service.sellProduct(10);


}
catch(InsufficientStockException e){


System.out.println(
e.getMessage()
);


}


}

}
```

Output:

```
Stock is not enough
```

---

# Real-world Application Architecture

Professional Project:

```
Swing UI

 |
 |
Controller

 |
 |
Service Layer

 |
 |
DAO Layer

 |
 |
Database
```

Exception Flow:

```
Database Error

      |
      v

DAO Exception

      |
      v

Service Exception

      |
      v

Controller Handle

      |
      v

Show Message Dialog
```

---

# Café POS Example

User:

```
Click Sell Button
```

---

Swing:

```java
sellButton.addActionListener(e->{


try{


saleService.sell(order);


JOptionPane.showMessageDialog(
null,
"Sale Success"
);


}
catch(InsufficientStockException ex){


JOptionPane.showMessageDialog(
null,
ex.getMessage()
);


}


});
```

---

Service:

```java
public void sell(Order order)
throws InsufficientStockException{


if(stock < order.qty){


throw new InsufficientStockException(
"Not enough stock"
);


}


saleDAO.save(order);


}
```

---

# Professional Exception Package Structure

```
src

 ├── exception

 │    ├── AppException.java
 │    ├── DatabaseException.java
 │    ├── ValidationException.java
 │    ├── StockException.java
 │
 ├── service

 ├── dao

 ├── controller

 └── view
```

---

# Advanced Pattern: Base Application Exception

Large systems မှာ:

```
AppException
      |
      |
------------------------
|          |           |
Database   Business   Validation
Exception  Exception  Exception
```

Example:

```java
public class AppException
extends Exception{


public AppException(String msg){

super(msg);

}

}
```

---

Then:

```java
public class StockException
extends AppException{


public StockException(String msg){

super(msg);

}

}
```

---

# Exception Handling Best Practices

## 1. Don't do this

```java
catch(Exception e){

}
```

Empty catch မလုပ်ပါနဲ့။

---

## 2. Give meaningful message

Bad:

```
Error
```

Good:

```
Product quantity cannot exceed available stock
```

---

## 3. Don't show technical error to user

Bad:

```
SQLException: Connection refused
```

User ကို:

```
Database connection failed. Please try again.
```

Developer အတွက် log ထားပါ။

---

## 4. Create Business Exceptions

Example:

```
UserAlreadyExistsException

InvalidPasswordException

InsufficientBalanceException

OrderNotFoundException
```

---

# After mastering this Lesson, you should know:

✅ try-catch  
✅ finally  
✅ throw  
✅ throws  
✅ Checked Exception  
✅ Runtime Exception  
✅ Custom Exception  
✅ Business Exception  
✅ Exception Architecture

---

နောက် Lesson:

# Part 2: Advanced Java Knowledge for Swing

အစဉ်လိုက်:

**Lesson 1: Collections Framework Deep Dive**

- ArrayList
    
- LinkedList
    
- HashMap
    
- HashSet
    
- List vs Set vs Map
    
- Real-world POS usage
    
- JTable data handling
    

ပြီးရင်:

- Lambda Expression
    
- Stream API
    
- Multithreading
    
- ExecutorService
    
- SwingWorker
    

တွေကို ဆက်သင်ပါမယ်။