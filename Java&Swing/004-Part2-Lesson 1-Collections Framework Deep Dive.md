# Part 2: Advanced Java Knowledge for Swing

# Lesson 1: Collections Framework Deep Dive

အခုကစပြီး **Java Swing + Real-world Application Development** အတွက် အရေးကြီးတဲ့ **Java Collections Framework** ကို စပါမယ်။

Swing Application (ဥပမာ Café POS System) မှာ Database ထဲက data တွေကိုယူပြီး:

- JTable ထဲပြခြင်း
    
- Search လုပ်ခြင်း
    
- Filter လုပ်ခြင်း
    
- Sorting လုပ်ခြင်း
    
- Temporary data သိမ်းခြင်း
    

အတွက် Collections ကို မဖြစ်မနေသုံးရပါတယ်။

---

# 1. What is Java Collection Framework?

Java Collection Framework ဆိုတာ:

> Objects အစုအဝေး (group of objects) တွေကို သိမ်းဆည်း၊ ရှာဖွေ၊ ပြင်ဆင်၊ စီမံရန် Java မှာပေးထားတဲ့ Data Structure Library ဖြစ်ပါတယ်။

ဥပမာ:

Café POS မှာ Product 100 ခုရှိတယ်။

Database:

```
products table

id | name     | price
---------------------
1  | Coffee   | 2000
2  | Cake     | 3000
3  | Tea      | 1500
```

Java မှာ:

```java
List<Product> products;
```

နဲ့ သိမ်းနိုင်ပါတယ်။

---

# 2. Why Do We Need Collections?

Old Java:

```java
Product p1;
Product p2;
Product p3;
Product p4;
```

Problem:

Product အရေအတွက် မသိနိုင်ပါဘူး။

---

Solution:

```java
ArrayList<Product> products =
new ArrayList<>();
```

Now:

```
Product List

Coffee
Cake
Tea
Juice
Burger
```

အလွယ်တကူ manage လုပ်နိုင်ပါတယ်။

---

# 3. Collection Framework Architecture

Java Collections Structure:

```
                 Iterable
                    |
                Collection
                    |
        -------------------------
        |           |           |
       List        Set        Queue


                    Map

```

အဓိက 4 ခု:

1. List
    
2. Set
    
3. Queue
    
4. Map
    

---

# 4. List Interface

## What is List?

List ဆိုတာ:

- Ordered collection
    
- Duplicate allowed
    
- Index ရှိ
    

ဖြစ်ပါတယ်။

Example:

```
Index

0       1       2
------------------
Coffee Cake    Tea

```

---

Common List Classes:

```
List

 |
 |-- ArrayList
 |
 |-- LinkedList
 |
 |-- Vector
 |
 |-- Stack

```

---

# 5. ArrayList (Most Important)

Swing Application မှာ အများဆုံးသုံးတာ ArrayList ဖြစ်ပါတယ်။

Example:

```java
import java.util.ArrayList;


ArrayList<String> products =
new ArrayList<>();


products.add("Coffee");
products.add("Cake");
products.add("Tea");
```

Memory:

```
ArrayList

[Coffee][Cake][Tea][ ]
```

---

# 6. ArrayList Basic Methods

## add()

Data ထည့်ခြင်း

```java
products.add("Coffee");
```

Result:

```
Coffee
```

---

## get()

Index နဲ့ယူခြင်း

```java
String item =
products.get(0);
```

Output:

```
Coffee
```

---

## remove()

ဖျက်ခြင်း

```java
products.remove(1);
```

Before:

```
0 Coffee
1 Cake
2 Tea
```

After:

```
0 Coffee
1 Tea
```

---

## size()

```java
int count =
products.size();
```

Output:

```
3
```

---

# 7. ArrayList with Custom Object

Real-world မှာ String မသုံးပါဘူး။

Object သုံးပါတယ်။

Product Model:

```java
public class Product {


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


}
```

---

Create List:

```java
ArrayList<Product> products =
new ArrayList<>();
```

Add:

```java
products.add(
new Product(
1,
"Coffee",
2000
)
);
```

---

Get:

```java
Product p =
products.get(0);


System.out.println(
p.getName()
);
```

Output:

```
Coffee
```

---

# 8. Loop Through ArrayList

## Normal for loop

```java
for(int i=0;i<products.size();i++){

Product p =
products.get(i);

System.out.println(
p.getName()
);

}
```

---

Output:

```
Coffee
Cake
Tea
```

---

## Enhanced for loop (Recommended)

```java
for(Product p : products){

System.out.println(
p.getName()
);

}
```

---

# 9. ArrayList Internal Working

ဒီ Concept က Interview မှာ မေးတတ်ပါတယ်။

ArrayList internally uses:

```
Object[]

```

Example:

```java
ArrayList<String> list;
```

Memory:

```
Object Array

[0]
[1]
[2]
[3]

```

---

Initial capacity:

```
10
```

ဖြစ်ပါတယ်။

---

Example:

```java
list.add("A");
```

Internal:

```
[A][ ][ ][ ][ ]
```

---

Capacity ပြည့်သွားရင်:

Old:

```
[A][B][C][D]
```

New bigger array:

```
[A][B][C][D][ ][ ][ ]

```

Data copy လုပ်ပါတယ်။

---

# 10. ArrayList Performance

|Operation|Speed|
|---|---|
|get(index)|Fast O(1)|
|add(end)|Fast|
|remove middle|Slow|
|search|Slow|

---

Example:

```
products.get(50)
```

Fast ဖြစ်ပါတယ်။

ဘာကြောင့်?

Index သိလို့ပါ။

---

# 11. ArrayList Real Café POS Usage

Database:

```
SELECT * FROM products
```

Result:

```
100 products
```

Java:

```java
List<Product> productList =
productDAO.findAll();
```

---

Then JTable:

```
JTable

ID | Name | Price
-----------------
1  | Coffee|2000
2  | Cake  |3000

```

Data Source:

```
List<Product>
```

---

# 12. ArrayList vs Array

Old:

```java
Product[] products =
new Product[100];
```

Problem:

Size fixed.

---

ArrayList:

```java
ArrayList<Product> products =
new ArrayList<>();
```

Advantage:

Size dynamic.

---

Comparison:

|Array|ArrayList|
|---|---|
|Fixed size|Dynamic|
|Primitive support|Object only|
|Fast|Flexible|
|Low level|High level|

---

# 13. Important Interview Questions

## Q1: Difference between Array and ArrayList?

Answer:

Array:

- Fixed size
    
- Can store primitives
    
- Faster
    

ArrayList:

- Dynamic size
    
- Only objects
    
- More methods
    

---

## Q2: How ArrayList grows?

Answer:

When capacity is full:

- Creates larger array
    
- Copies old elements
    
- New element added
    

---

## Q3: Can ArrayList store primitive?

No.

Wrong:

```java
ArrayList<int>
```

Correct:

```java
ArrayList<Integer>
```

Because Java uses Wrapper Class.

---

# 14. Wrapper Classes

Primitive:

```
int
double
boolean
```

Wrapper:

```
Integer
Double
Boolean
```

Example:

```java
ArrayList<Integer> numbers =
new ArrayList<>();

numbers.add(10);
```

Java automatically:

```
int → Integer

(auto boxing)
```

---

# 15. Practice Exercise

Create Café POS Product Management:

Class:

```
Product

fields:

id
name
price
stock

```

Create:

```java
List<Product> products;
```

Functions:

```
addProduct()

removeProduct()

searchProduct()

displayProducts()

```

---

# Lesson 1 Summary

ဒီနေ့သိရမည့်အချက်:

✅ Collection Framework Concept  
✅ List Interface  
✅ ArrayList  
✅ ArrayList Methods  
✅ ArrayList Internal Working  
✅ ArrayList Performance  
✅ Custom Object Storage  
✅ POS Application Usage

---

## Next Lesson

# Lesson 2: LinkedList vs ArrayList Deep Dive

သင်ယူမည့်အရာ:

- LinkedList Internal Structure
    
- Node Concept
    
- ArrayList vs LinkedList Performance
    
- When to use which?
    
- Real-world POS examples
    
- Java Interview Questions
    

ပြီးရင်:

**Lesson 3 → HashMap Internals (Very Important for Real Applications)**

ကို ဆက်သွားပါမယ်။