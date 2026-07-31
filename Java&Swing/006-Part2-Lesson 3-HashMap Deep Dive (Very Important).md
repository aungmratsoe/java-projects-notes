# Part 2: Advanced Java Knowledge for Swing

# Lesson 3: HashMap Deep Dive (Very Important)

ဒီ Lesson က **Java Developer Interview + Real-world Application Development** အတွက် အလွန်အရေးကြီးပါတယ်။

Café POS System မှာ:

- Product ID နဲ့ Product ရှာခြင်း
    
- User ID နဲ့ User ရှာခြင်း
    
- Permission Mapping
    
- Configuration Data
    
- Cache System
    

တွေမှာ `HashMap` ကို အများကြီးအသုံးပြုပါတယ်။

---

# 1. What is HashMap?

`HashMap` ဆိုတာ:

> Data ကို Key - Value pair အဖြစ် သိမ်းတဲ့ Collection ဖြစ်ပါတယ်။

Structure:

```id="9hmj8m"
Key        Value

1001  ---> Coffee
1002  ---> Cake
1003  ---> Tea

```

---

Example:

```java
HashMap<Integer,String> products =
new HashMap<>();


products.put(1001,"Coffee");
products.put(1002,"Cake");
products.put(1003,"Tea");
```

---

Data:

```
1001 = Coffee
1002 = Cake
1003 = Tea
```

---

# 2. Why HashMap?

ArrayList မှာ:

```java
products.get(500);
```

ဆိုရင် index နဲ့ရှာပါတယ်။

ဒါပေမယ့် Real Application မှာ:

"Product ID 1005 ကိုရှာ"

ဆိုတာမျိုးများပါတယ်။

---

ArrayList:

```java
for(Product p : products){

    if(p.getId()==1005){

    }

}
```

အကြိမ်များရင် နှေးနိုင်ပါတယ်။

---

HashMap:

```java
products.get(1005);
```

တိုက်ရိုက်ရှာနိုင်ပါတယ်။

---

# 3. HashMap Structure

Internally:

```
HashMap

      |
      |
  Array of Buckets


Bucket 0
Bucket 1
Bucket 2
Bucket 3

```

---

Example:

```
Bucket Array


0  -> null

1  -> (1001,Coffee)

2  -> (1002,Cake)

3  -> null

```

---

# 4. How HashMap Works Internally?

ဒီအပိုင်းက Interview မှာ အများဆုံးမေးပါတယ်။

When:

```java
map.put(1001,"Coffee");
```

ဖြစ်တဲ့အချိန်:

## Step 1: Hash Function

Key:

```
1001
```

ကို hash value ပြောင်းပါတယ်။

Example:

```
1001.hashCode()

= 123456
```

---

## Step 2: Bucket Calculation

Hash value ကို bucket index ပြောင်းပါတယ်။

Example:

```
123456 % 16

= 5

```

ဒါဆို:

```
Bucket 5

     |
     |
1001 -> Coffee

```

ထည့်ပါတယ်။

---

# 5. HashMap Internal Structure Java 8+

Java 8 နောက်ပိုင်း:

```
Bucket

    |
    |
 Linked List

       OR

 Red Black Tree

```

---

Before:

```
Bucket

A
 |
B
 |
C
 |
D

```

Collision များရင် Tree ပြောင်းနိုင်ပါတယ်။

---

# 6. Hash Collision

Question:

"Different keys have same hash value ဖြစ်ရင်?"

ဒါကို Collision လို့ခေါ်ပါတယ်။

Example:

```
Key A

hash = 10


Key B

hash = 10

```

တူနေပါတယ်။

---

HashMap:

```
Bucket 10


A -> Value A

B -> Value B

```

လို LinkedList နဲ့ သိမ်းပါတယ်။

---

# 7. equals() and hashCode()

HashMap ရဲ့ Heart က:

```
hashCode()

+
equals()

```

ဖြစ်ပါတယ်။

---

Example:

```java
String a="Coffee";

String b="Coffee";


System.out.println(
a.equals(b)
);

```

Output:

```
true
```

---

HashMap မှာ:

First:

```
hashCode()
```

စစ်ပါတယ်။

ပြီးရင်:

```
equals()
```

စစ်ပါတယ်။

---

# 8. Important Rule

Java Rule:

> If two objects are equal, their hashCode must be equal.

Example:

Correct:

```java
a.equals(b)==true

a.hashCode()==b.hashCode()

```

---

Wrong:

```java
equals true

hashCode different

```

HashMap problem ဖြစ်နိုင်ပါတယ်။

---

# 9. Custom Object as HashMap Key

Real application:

Example:

User:

```java
class User{


int id;

String name;


}
```

---

HashMap:

```java
HashMap<User,String> map;

```

သုံးမယ်ဆိုရင်:

`hashCode()` နဲ့ `equals()` override လုပ်ရပါတယ်။

---

Example:

```java
public class User {


private int id;


@Override
public int hashCode(){

return id;

}


@Override
public boolean equals(Object obj){

User u=(User)obj;

return this.id==u.id;

}


}
```

---

# 10. HashMap Basic Methods

## put()

Add data:

```java
map.put(
1,
"Coffee"
);

```

---

## get()

```java
String name =
map.get(1);

```

Output:

```
Coffee
```

---

## remove()

```java
map.remove(1);

```

---

## containsKey()

```java
if(map.containsKey(1001)){

}
```

---

## containsValue()

```java
map.containsValue("Coffee");

```

---

## size()

```java
map.size();

```

---

# 11. Iterating HashMap

Example:

```java
HashMap<Integer,String> products =
new HashMap<>();


products.put(1,"Coffee");
products.put(2,"Cake");

```

---

## keySet()

```java
for(Integer id : products.keySet()){


System.out.println(id);


}

```

Output:

```
1
2
```

---

## values()

```java
for(String name : products.values()){


System.out.println(name);


}

```

---

## entrySet() (Recommended)

```java
for(Map.Entry<Integer,String> entry 
: products.entrySet()){


System.out.println(
entry.getKey()
+
" "
+
entry.getValue()
);


}

```

Output:

```
1 Coffee

2 Cake

```

---

# 12. HashMap Performance

|Operation|Performance|
|---|---|
|put()|O(1) average|
|get()|O(1) average|
|remove()|O(1) average|
|search|O(1)|

ဒါကြောင့် အလွန်မြန်ပါတယ်။

---

# 13. HashMap vs ArrayList

## ArrayList

Data:

```
Index

0 Coffee
1 Cake
2 Tea

```

Search:

```
Loop required

O(n)

```

---

## HashMap

Data:

```
ID

1001 -> Coffee

1002 -> Cake

```

Search:

```
Direct

O(1)

```

---

# 14. Café POS Real-world Usage

## Example 1: Product Cache

Database:

```
products table

ID | Name

1 Coffee
2 Cake

```

Load:

```java
HashMap<Integer,Product> cache;

```

Data:

```
1 -> Product(Coffee)

2 -> Product(Cake)

```

---

Search:

```java
Product p =
cache.get(1);

```

No database query needed။

---

# Example 2: User Permission

Role:

```
ADMIN

MANAGER

CASHIER

```

Mapping:

```java
HashMap<String,List<String>> permissions;

```

Example:

```
ADMIN

 |
 |
[ADD_PRODUCT,
DELETE_PRODUCT,
REPORT]

```

---

# Example 3: Shopping Cart

Cart:

```
Product ID

1001 -> quantity 3

1002 -> quantity 2

```

Java:

```java
HashMap<Integer,Integer> cart;

```

---

# 15. HashMap with Swing JTable

Database Result:

```java
List<Product>
```

But fast lookup:

```java
HashMap<Integer,Product>

```

Both can coexist:

```java
List<Product> productList;


HashMap<Integer,Product> productMap;

```

---

Display:

```
JTable

List

```

Search:

```
HashMap

```

---

# 16. HashMap Problems

## Problem 1: Null Key

Allowed:

```java
map.put(null,"Coffee");

```

HashMap supports one null key.

---

## Problem 2: Not Thread Safe

Wrong:

Multiple threads:

```
Thread 1
Thread 2

HashMap

```

Problem ဖြစ်နိုင်ပါတယ်။

Solution:

```java
ConcurrentHashMap

```

---

# 17. HashMap vs Hashtable

|HashMap|Hashtable|
|---|---|
|Not synchronized|Synchronized|
|Fast|Slow|
|Allows null|No null|
|Modern Java|Legacy|

---

# 18. HashMap Interview Questions

## Q1: How HashMap works internally?

Answer:

1. Key hashCode generated
    
2. Bucket calculated
    
3. Entry stored
    
4. Collision handled by LinkedList/Tree
    
5. equals() checks key equality
    

---

## Q2: Why override hashCode and equals?

Because HashMap uses them to identify keys.

---

## Q3: Can HashMap have duplicate keys?

No.

Example:

```java
map.put(1,"Coffee");

map.put(1,"Tea");

```

Result:

```
1 -> Tea

```

Old value replaced.

---

# 19. Practice Project

Create Café POS Product Cache.

Class:

```java
Product

id
name
price
stock

```

Create:

```java
HashMap<Integer,Product> productCache;

```

Methods:

```java
addProduct()

findProductById()

removeProduct()

displayProducts()

```

Example:

Add:

```
1001 Coffee

1002 Cake

```

Search:

```
findProductById(1001)

Return Coffee

```

---

# Lesson 3 Summary

Today you learned:

✅ HashMap concept  
✅ Key-Value structure  
✅ Internal bucket system  
✅ Hash function  
✅ Collision handling  
✅ hashCode() & equals()  
✅ HashMap performance  
✅ Real POS usage  
✅ Cache design

---

# Next Lesson

## Lesson 4: HashSet Deep Dive

We will learn:

- Set concept
    
- Duplicate prevention
    
- HashSet internal working
    
- HashSet vs HashMap
    
- equals/hashCode importance
    
- Real-world usage:
    
    - Unique product tags
        
    - User roles
        
    - Permission management
        

ပြီးရင်:

**Lesson 5: Queue & PriorityQueue (Order Processing System)**

ကို ဆက်သွားပါမယ်။