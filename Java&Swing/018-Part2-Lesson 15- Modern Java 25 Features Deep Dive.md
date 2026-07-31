# Part 2: Advanced Java Knowledge for Swing

# Lesson 15: Modern Java 25 Features Deep Dive

## Writing Professional Java 25 Code for Café POS System

ဒီ Lesson မှာ **Java 25 ရဲ့ Modern Features** တွေကို လေ့လာပါမယ်။

Java 25 Developer တစ်ယောက်အနေနဲ့ Java 8 style code တင်မဟုတ်ဘဲ modern style ကို သိထားရပါမယ်။

Java 25 မှာ အရေးကြီးတဲ့ Concept တွေ:

- `var`
    
- Records
    
- Sealed Classes
    
- Pattern Matching
    
- Switch Expressions
    
- Text Blocks
    
- Enhanced Collections
    
- Virtual Threads
    
- Modern Coding Practices
    

---

# 1. Java Evolution Overview

Java Version Evolution:

```
Java 8
 |
 |-- Lambda
 |-- Stream API
 |-- Optional
 |-- Date API

Java 11
 |
 |-- HTTP Client
 |-- var

Java 17
 |
 |-- Records
 |-- Sealed Classes

Java 21+
 |
 |-- Virtual Threads
 |-- Pattern Matching

Java 25
 |
 |-- Modernized Java Platform
 |-- Latest LTS Features
```

---

# 2. Local Variable Type Inference (`var`)

Before Java 10:

```java
String productName = "Coffee";

ArrayList<Product> products =
new ArrayList<Product>();

```

Type ကို နှစ်ခါရေးရပါတယ်။

---

Java 25:

```java
var productName = "Coffee";

var products =
new ArrayList<Product>();

```

Compiler က Type ကို infer လုပ်ပေးပါတယ်။

---

Important:

`var` က dynamic typing မဟုတ်ပါ။

Example:

```java
var price = 5000;

```

Compiler sees:

```java
int price = 5000;

```

---

# 3. Where var Can Be Used

Allowed:

## Local Variables

```java
public void test(){

var name = "Coffee";

}

```

---

## Loop

```java
for(var product : products){

System.out.println(
product.getName()
);

}

```

---

# 4. Where var Cannot Be Used

Field:

```java
class Product{


var name;

}

```

❌ Not allowed

---

Method Parameter:

```java
void save(var product){

}

```

❌ Not allowed

---

# 5. var Best Practice

Good:

```java
var products =
repository.findAll();

```

Because right side is clear.

---

Bad:

```java
var x = getData();

```

Unknown.

---

Rule:

> Use var when it improves readability, not hides information.

---

# 6. Records

Before Java 16:

Data class ရေးရင်:

```java
public class ProductDTO {


private final int id;

private final String name;


public ProductDTO(
int id,
String name
){

this.id=id;
this.name=name;

}


public int getId(){

return id;

}


public String getName(){

return name;

}


}

```

Code အများကြီးရေးရပါတယ်။

---

Java 25:

```java
public record ProductDTO(
int id,
String name
){

}

```

ဒါနဲ့ပြီးပါပြီ။

---

# 7. What Does Record Provide?

Automatically creates:

```java
constructor

getter methods

equals()

hashCode()

toString()

```

---

Example:

```java
var product =
new ProductDTO(
1,
"Coffee"
);


System.out.println(product.name());

```

Getter:

```java
name()

```

not:

```java
getName()

```

---

# 8. Café POS DTO Example

API Response:

```java
public record ProductResponse(

int id,

String name,

double price

){

}

```

---

JSON:

```json
{
"id":1,
"name":"Coffee",
"price":5000
}

```

---

Perfect for:

- API Response
    
- Request Object
    
- DTO
    

---

# 9. Records Are Immutable

Example:

```java
record Sale(
int id,
double amount
){

}

```

Cannot:

```java
sale.amount = 10000;

```

---

Why?

Data Transfer Objects should not change.

---

# 10. Custom Methods in Record

Record can have methods.

Example:

```java
record Product(
String name,
double price
){


public boolean isExpensive(){

return price > 10000;

}


}

```

---

Usage:

```java
if(product.isExpensive()){

}

```

---

# 11. Sealed Classes

Problem:

Normal inheritance:

```java
class Payment{

}


class CashPayment
extends Payment{


}


class CardPayment
extends Payment{


}


```

Anyone can extend Payment.

---

Sometimes we want controlled inheritance.

---

Sealed:

```java
public sealed class Payment
permits CashPayment,
CardPayment {


}

```

---

Meaning:

Only these classes can extend.

---

# 12. Café POS Payment Example

```java
sealed interface Payment
permits Cash,
Card,
MobilePay{


}


final class Cash
implements Payment{


}


final class Card
implements Payment{


}


final class MobilePay
implements Payment{


}

```

---

Architecture:

```
Payment

   |
 ----------------

Cash

Card

MobilePay

```

---

# 13. Pattern Matching instanceof

Old Java:

```java
if(payment instanceof Cash){

Cash cash =
(Cash) payment;

}

```

---

Modern Java:

```java
if(payment instanceof Cash cash){

System.out.println(
cash.amount()
);

}

```

Cleaner.

---

# 14. Pattern Matching with Conditions

Example:

```java
if(payment instanceof Card card
&& card.isValid()){


process(card);


}

```

---

Old style:

```java
Cast

then

Check

```

---

Modern:

```text
Check + Cast together
```

---

# 15. Switch Expressions

Old switch:

```java
String type;


switch(payment){

case CASH:

type="Cash";

break;


case CARD:

type="Card";

break;

}

```

---

Modern:

```java
String type =
switch(payment){

case CASH ->
"Cash";


case CARD ->
"Card";


};

```

---

Cleaner.

---

# 16. Switch with Multiple Values

Example:

```java
String category =
switch(productType){


case COFFEE, TEA ->
"Drink";


case CAKE, BREAD ->
"Food";


};

```

---

# 17. Switch + Pattern Matching

Example:

```java
String process(Payment payment){


return switch(payment){


case Cash c ->
"Cash Payment";


case Card c ->
"Card Payment";


case MobilePay m ->
"Mobile Payment";


};


}

```

---

# 18. Text Blocks

Before:

```java
String json =
"{\n"+
"\"name\":\"Coffee\"\n"+
"}";

```

Hard to read.

---

Java:

```java
String json =
"""
{
"name":"Coffee",
"price":5000
}
""";

```

---

Used for:

- JSON
    
- SQL
    
- HTML
    

---

# 19. Café POS SQL Example

Before:

```java
String sql =
"SELECT * FROM products "
+
"WHERE price > ?";

```

---

Modern:

```java
String sql =
"""
SELECT *
FROM products
WHERE price > ?
""";

```

---

# 20. Enhanced Collections

Java 25 supports modern collection usage.

Example:

```java
List<String> drinks =
List.of(
"Coffee",
"Tea",
"Juice"
);

```

Immutable list.

---

# 21. Stream Improvements

Example:

```java
var total =
sales.stream()
.mapToDouble(
Sale::amount
)
.sum();

```

---

Record makes this clean:

```java
record Sale(
double amount
){}

```

---

# 22. Virtual Threads

Java 21+ major feature.

Traditional:

```
1000 Requests

1000 Platform Threads

Heavy Memory

```

---

Virtual Threads:

```
1000 Requests

Lightweight Threads

Low Memory

```

---

Create:

```java
Thread.startVirtualThread(
() -> {


processOrder();


}
);

```

---

# 23. Café POS Virtual Thread Example

Many orders:

```java
for(Order order : orders){


Thread.startVirtualThread(
() -> process(order)
);


}

```

---

Useful for:

- API calls
    
- Database waiting
    
- File operations
    

---

# 24. CompletableFuture + Virtual Thread

Example:

```java
CompletableFuture
.supplyAsync(
() -> loadProducts()
)
.thenAccept(
products -> updateUI(products)
);

```

---

Flow:

```
Swing UI

 |
 |
Async Task

 |
 |
Database/API

 |
 |
Update UI

```

---

# 25. Modern Exception Design

Java 25 style:

```java
public sealed class AppException
extends RuntimeException
permits
DatabaseException,
NetworkException{


}

```

---

Then:

```java
final class DatabaseException
extends AppException{


}

```

---

Controlled error hierarchy.

---

# 26. Modern Café POS Model

Traditional:

```java
class Product{


private int id;

private String name;

private double price;


}

```

---

Java 25:

```java
public record Product(

int id,

String name,

double price

){

}

```

---

Payment:

```java
sealed interface Payment
permits Cash,Card{


}

```

---

API:

```java
record ProductResponse(
int id,
String name
){

}

```

---

Clean architecture.

---

# 27. Java 25 Coding Style

Professional:

Use:

✅ Records for DTO

✅ Sealed classes for controlled hierarchy

✅ Pattern matching

✅ Switch expressions

✅ Text blocks

✅ Virtual Threads for I/O

✅ Optional

✅ Immutable objects

Avoid:

❌ Mutable DTO

❌ Deep inheritance

❌ Old Date API

❌ Thread creation everywhere

---

# 28. Interview Questions

## Q1: Is var dynamically typed?

No.

Java remains statically typed.

---

## Q2: Why use records?

To create immutable data carriers with less code.

---

## Q3: Why sealed classes?

To control inheritance.

---

## Q4: What are virtual threads?

Lightweight threads designed for high concurrency.

---

## Q5: Difference between class and record?

Class:

- Flexible
    
- Mutable possible
    

Record:

- Immutable data carrier
    
- Less boilerplate
    

---

# Practice Project

Modernize Café POS Models:

Convert:

```
Product
Customer
Order
Payment
DTO

```

to Java 25 style.

Implement:

### Product

```java
record Product(
int id,
String name,
double price
){}
```

---

### Payment

```java
sealed interface Payment
permits Cash,Card
{}
```

---

### API Response

```java
record ApiResponse<T>(
boolean success,
T data,
String message
){}
```

---

# Lesson 15 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Java 25 Modern Style  
✅ var  
✅ Records  
✅ Immutable Data  
✅ Sealed Classes  
✅ Pattern Matching  
✅ Switch Expressions  
✅ Text Blocks  
✅ Modern Collections  
✅ Virtual Threads  
✅ Modern Exception Design  
✅ Café POS Modern Architecture

---

# Next Lesson

# Lesson 16: Java Design Patterns for Professional Swing Applications

## Building Enterprise Café POS Architecture

Topics:

- Singleton Pattern
    
- Factory Pattern
    
- Builder Pattern
    
- Strategy Pattern
    
- Observer Pattern
    
- MVC Pattern
    
- Repository Pattern
    
- Service Layer Pattern
    
- Dependency Injection Concept
    

ပြီးရင် Café POS System ကို Enterprise Level Architecture အဖြစ် Design လုပ်ပါမယ်။