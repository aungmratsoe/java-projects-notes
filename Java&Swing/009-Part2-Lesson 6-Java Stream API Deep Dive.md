# Part 2: Advanced Java Knowledge for Swing

# Lesson 6: Java Stream API Deep Dive

## Modern Java Data Processing for Café POS System

ဒီ Lesson က Java 8 နောက်ပိုင်းမှာ အရေးကြီးဆုံး Feature တွေထဲက တစ်ခုဖြစ်တဲ့ **Stream API** ကို လေ့လာပါမယ်။

Professional Java Developer တွေက:

- Collection data processing
    
- Database result filtering
    
- Report generation
    
- Sorting
    
- Data transformation
    

တွေမှာ Stream API ကို အသုံးများပါတယ်။

---

# 1. Why Stream API?

Traditional Java Style:

Example:

Product List ရှိတယ်။

Requirement:

> Price 5000 ထက်ကြီးတဲ့ Product တွေကို ရှာပါ။

Old Style:

```java
List<Product> result =
new ArrayList<>();


for(Product p : products){

    if(p.getPrice() > 5000){

        result.add(p);

    }

}

```

Problem:

- Code များတယ်
    
- Read ခက်တယ်
    
- Logic ရော data loop ရော ရောနေတယ်
    

---

Stream API:

```java
List<Product> result =
products.stream()
        .filter(p -> p.getPrice() > 5000)
        .collect(Collectors.toList());

```

ပိုရှင်းပါတယ်။

---

# 2. What is Stream API?

Stream ဆိုတာ:

> Collection ထဲက data တွေကို functional programming style နဲ့ process လုပ်ဖို့ အသုံးပြုတဲ့ API ဖြစ်ပါတယ်။

Important:

Stream က data ကို store မလုပ်ပါဘူး။

သူက:

```
Collection

   |
   |
 Stream

   |
   |
 Processing

   |
   |
 Result

```

---

# 3. Collection vs Stream

## Collection

Data သိမ်းထားတယ်။

Example:

```java
List<Product> products;

```

Memory:

```
Coffee
Cake
Tea
```

---

## Stream

Data ကို process လုပ်တယ်။

Example:

```
Filter

Sort

Convert

Calculate

```

---

# 4. Stream Creation

## From List

```java
List<String> menu =
List.of(
"Coffee",
"Cake",
"Tea"
);


Stream<String> stream =
menu.stream();

```

---

# 5. Stream Pipeline

Stream အလုပ်လုပ်ပုံ:

```
Source

(List)

 |
 v

Intermediate Operations

(filter, map, sorted)

 |
 v

Terminal Operation

(collect, forEach)

```

Example:

```java
products.stream()
.filter(...)
.map(...)
.collect(...);

```

---

# 6. First Stream Example

Product Class:

```java
class Product {


private int id;

private String name;

private double price;


public Product(
int id,
String name,
double price
){

this.id=id;
this.name=name;
this.price=price;

}


public String getName(){

return name;

}


public double getPrice(){

return price;

}

}

```

---

Create Data:

```java
List<Product> products =
new ArrayList<>();


products.add(
new Product(1,"Coffee",2000)
);


products.add(
new Product(2,"Cake",7000)
);


products.add(
new Product(3,"Burger",9000)
);

```

---

# 7. forEach()

Stream မှာ loop လုပ်ခြင်း:

Old:

```java
for(Product p: products){

System.out.println(
p.getName()
);

}

```

---

Stream:

```java
products.stream()
.forEach(
p -> System.out.println(
p.getName()
)
);

```

Output:

```
Coffee
Cake
Burger
```

---

# 8. filter()

Most Important.

Meaning:

> Condition နဲ့ data ရွေးခြင်း

Example:

Price > 5000

```java
products.stream()
.filter(
p -> p.getPrice() > 5000
)
.forEach(
p -> System.out.println(
p.getName()
)
);

```

Output:

```
Cake
Burger
```

---

# 9. filter Flow

Data:

```
Coffee 2000
Cake   7000
Burger 9000

```

filter:

```
price > 5000

```

Result:

```
Cake
Burger

```

---

# 10. map()

Meaning:

> Data ကို ပြောင်းခြင်း

Example:

Product Object → Product Name

```java
products.stream()
.map(
p -> p.getName()
)
.forEach(
System.out::println
);

```

Output:

```
Coffee
Cake
Burger

```

---

Before:

```
Product

{id=1,name=Coffee}

```

After map:

```
Coffee

```

---

# 11. sorted()

Sorting လုပ်ရန်:

Example:

Price အနည်းဆုံးကနေ အများဆုံး:

```java
products.stream()
.sorted(
(a,b) ->
Double.compare(
a.getPrice(),
b.getPrice()
)
)
.forEach(
p -> System.out.println(
p.getName()
)
);

```

Output:

```
Coffee
Cake
Burger

```

---

# 12. collect()

Stream result ကို Collection ပြန်ယူရန်:

Example:

```java
List<Product> expensiveProducts =
products.stream()
.filter(
p -> p.getPrice()>5000
)
.collect(
Collectors.toList()
);

```

Result:

```
Cake
Burger

```

---

# 13. Counting Data

Requirement:

> Product ဘယ်နှစ်ခုရှိလဲ?

Old:

```java
int count=0;

for(Product p:products){

count++;

}

```

---

Stream:

```java
long count =
products.stream()
.count();

```

Output:

```
3
```

---

# 14. max() and min()

Highest price:

```java
Product expensive =
products.stream()
.max(
Comparator.comparing(
Product::getPrice
)
)
.get();

```

Result:

```
Burger

```

---

Lowest:

```java
Product cheapest =
products.stream()
.min(
Comparator.comparing(
Product::getPrice
)
)
.get();

```

Result:

```
Coffee

```

---

# 15. reduce()

Advanced Concept.

Used for:

- Sum
    
- Total
    
- Calculation
    

Example:

Total Sales:

```java
double total =
prices.stream()
.reduce(
0.0,
(a,b)->a+b
);

```

---

Example:

```
2000
3000
5000

=
10000

```

---

# 16. Café POS Real-world Examples

## Example 1: Search Product

User types:

```
coffee
```

Stream:

```java
products.stream()
.filter(
p ->
p.getName()
.equalsIgnoreCase("coffee")
)
.findFirst();

```

---

# Example 2: Generate Report

Requirement:

> Today sales total

Sale List:

```
1000
2000
3000

```

Code:

```java
double total =
sales.stream()
.mapToDouble(
Sale::getAmount
)
.sum();

```

Result:

```
6000

```

---

# Example 3: Low Stock Report

Requirement:

Stock < 10

```java
products.stream()
.filter(
p -> p.getStock()<10
)
.collect(
Collectors.toList()
);

```

Result:

```
Coffee stock 5

```

---

# 17. Method Reference (::)

Lambda:

```java
p -> p.getName()

```

Method Reference:

```java
Product::getName

```

Example:

```java
products.stream()
.map(Product::getName)
.forEach(System.out::println);

```

---

# 18. Lambda Connection

Stream API uses Lambda heavily.

Example:

```java
p -> p.getPrice()>5000

```

Meaning:

```
Input:

Product p


Return:

true/false

```

---

# 19. Stream vs Loop

|Loop|Stream|
|---|---|
|Imperative|Functional|
|More code|Less code|
|Manual iteration|Pipeline|
|Simple logic|Complex processing|

---

# 20. Important Stream Rules

## Rule 1:

Stream cannot reuse.

Wrong:

```java
Stream<Product> s =
products.stream();


s.count();


s.forEach(...);

```

Second operation error ဖြစ်ပါတယ်။

---

## Rule 2:

Original Collection မပြောင်းပါဘူး။

Example:

```java
products.stream()
.filter(...);

```

Original:

```
products

```

မပြောင်းပါဘူး။

---

# 21. Parallel Stream

Advanced:

```java
products.parallelStream()

```

Uses multiple threads.

Useful:

- Big data
    
- Heavy calculation
    

Example:

```
1 million records

Thread 1
Thread 2
Thread 3

```

---

But:

Small POS data အတွက် မလိုအပ်ပါ။

---

# 22. Stream Interview Questions

## Q1: Difference between Collection and Stream?

Collection:

- Stores data
    

Stream:

- Processes data
    

---

## Q2: Does Stream modify original collection?

No.

---

## Q3: What are intermediate operations?

Operations that return Stream:

Examples:

```
filter()
map()
sorted()

```

---

## Q4: Terminal operations?

Operations that produce result:

Examples:

```
collect()
count()
forEach()
reduce()

```

---

# 23. Café POS Data Processing Architecture

Now our POS processing:

```
Database

   |
   v

DAO

   |
   v

List<Product>

   |
   v

Stream API

   |
   |
-----------------

Filter

Sort

Report

Calculate

-----------------

   |
   v

Swing JTable

```

---

# Practice Project

Create Product Report Service:

Class:

```java
ProductReportService

```

Methods:

```java
List<Product> 
findExpensiveProducts()


List<Product>
findLowStockProducts()


double
calculateTotalValue()


Product
findMostExpensiveProduct()

```

Use:

```
ArrayList
Stream API
Lambda
Collectors

```

---

# Lesson 6 Summary

Today you learned:

✅ Stream API purpose  
✅ Stream pipeline  
✅ filter()  
✅ map()  
✅ sorted()  
✅ collect()  
✅ reduce()  
✅ max/min  
✅ Method Reference  
✅ Lambda connection  
✅ Real Café POS reporting examples

---

## Next Lesson

# Lesson 7: Lambda Expression & Functional Interface Deep Dive

We will learn:

- How Lambda works internally
    
- Functional Interface
    
- Predicate
    
- Function
    
- Consumer
    
- Supplier
    
- Custom Functional Interface
    
- Stream + Lambda Advanced Usage
    

ပြီးရင် Java Modern Features အပိုင်းကို ဆက်သွားပါမယ်။