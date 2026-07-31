# Part 2: Advanced Java Knowledge for Swing

# Lesson 2: LinkedList vs ArrayList Deep Dive

ဒီ Lesson မှာ **ArrayList နဲ့ LinkedList ဘာကွာသလဲ**, **ဘယ်အချိန်မှာ ဘယ်ဟာသုံးသင့်သလဲ** ကို လေ့လာပါမယ်။

Java Swing + Café POS System မှာ data management လုပ်တဲ့အခါ Collection ကို မှန်ကန်စွာရွေးချယ်နိုင်ဖို့ ဒီ Concept က အရေးကြီးပါတယ်။

---

# 1. Review: ArrayList

ပြီးခဲ့တဲ့ Lesson မှာ ArrayList ကို လေ့လာခဲ့ပါတယ်။

ArrayList ရဲ့ Internal Structure:

```
ArrayList

Object[]

+-------+-------+-------+-------+
|Coffee | Cake  | Tea   | Juice |
+-------+-------+-------+-------+

Index:
   0       1       2       3
```

ArrayList က **Dynamic Array** ဖြစ်ပါတယ်။

---

Example:

```java
List<Product> products = new ArrayList<>();

products.add(
    new Product(1,"Coffee",2000)
);
```

---

# 2. What is LinkedList?

LinkedList ဆိုတာ:

> Data တွေကို Node (Element + Link) အနေနဲ့ သိမ်းတဲ့ Data Structure ဖြစ်ပါတယ်။

ArrayList က:

```
[ A ][ B ][ C ][ D ]
```

LinkedList က:

```
Node        Node        Node

[A|next] -> [B|next] -> [C|next] -> [D|null]

```

---

Node တစ်ခုမှာ:

```
+-------------+
| Data        |
| Next Node   |
+-------------+
```

ပါပါတယ်။

---

# 3. LinkedList Structure

Example:

```java
LinkedList<String> list =
new LinkedList<>();

list.add("Coffee");
list.add("Cake");
list.add("Tea");
```

Memory:

```
Head

 |
 v

+---------+       +---------+       +---------+
| Coffee  | ----> | Cake    | ----> | Tea     |
+---------+       +---------+       +---------+

```

---

# 4. Creating LinkedList

```java
import java.util.LinkedList;


public class Test {


public static void main(String[] args){


LinkedList<String> menu =
new LinkedList<>();


menu.add("Coffee");
menu.add("Cake");
menu.add("Tea");


System.out.println(menu);


}

}
```

Output:

```
[Coffee, Cake, Tea]
```

---

# 5. LinkedList Methods

## add()

End မှာထည့်ခြင်း

```java
menu.add("Juice");
```

Result:

```
Coffee
Cake
Tea
Juice
```

---

## addFirst()

အစမှာထည့်ခြင်း

```java
menu.addFirst("Water");
```

Result:

```
Water
Coffee
Cake
Tea
```

---

## addLast()

နောက်ဆုံးမှာထည့်ခြင်း

```java
menu.addLast("Burger");
```

---

## removeFirst()

```java
menu.removeFirst();
```

---

## removeLast()

```java
menu.removeLast();
```

---

# 6. ArrayList vs LinkedList Internal Difference

## ArrayList

```
Memory:

[0][1][2][3][4]

```

Continuous memory ဖြစ်ပါတယ်။

---

## LinkedList

```
Memory:

Node1       Node2       Node3

Coffee ---> Cake -----> Tea

```

Memory နေရာမတူနိုင်ပါဘူး။

---

# 7. Performance Comparison

ဒီအပိုင်းက Interview မှာ အရေးကြီးပါတယ်။

## Add Operation

### ArrayList

End မှာ add:

```
O(1)
```

Fast

Middle မှာ add:

```
O(n)
```

because shifting လုပ်ရပါတယ်။

Example:

Before:

```
Coffee Cake Tea
```

Insert Milk:

```
Coffee Milk Cake Tea
```

Cake, Tea ကို shift လုပ်ရပါတယ်။

---

### LinkedList

Insert:

```
O(1)
```

Node link ပြောင်းရုံပါ။

Before:

```
Coffee -> Cake -> Tea
```

After:

```
Coffee -> Milk -> Cake -> Tea
```

---

# 8. Remove Operation

## ArrayList

Remove middle:

Before:

```
A B C D

remove B

A C D

```

C,D shift လုပ်ရပါတယ်။

Performance:

```
O(n)
```

---

## LinkedList

Remove:

```
A -> B -> C

remove B


A ------> C

```

Link ပြောင်းရုံပါ။

Performance:

```
O(1)
```

---

# 9. Get Operation

ဒီနေရာမှာ ArrayList အနိုင်ရပါတယ်။

## ArrayList

```java
products.get(50);
```

Index တိုက်ရိုက်သွားနိုင်ပါတယ်။

```
Index 50

Direct Access

O(1)

```

---

## LinkedList

```java
products.get(50);
```

LinkedList က:

```
Head

 |
 v

1
2
3
...
50

```

တစ်ခုချင်းစီ လိုက်သွားရပါတယ်။

Performance:

```
O(n)

```

---

# 10. Complete Comparison Table

|Operation|ArrayList|LinkedList|
|---|---|---|
|Search|get(index)|Slow|
|Insert End|Fast|Fast|
|Insert Middle|Slow|Fast|
|Remove Middle|Slow|Fast|
|Memory|Less|More|
|Random Access|Excellent|Poor|

---

# 11. When to Use ArrayList?

Most applications:

```
90% use ArrayList
```

Use ArrayList when:

✅ Reading data များ  
✅ JTable display  
✅ Search များ  
✅ Database result storage

Example Café POS:

Database:

```
SELECT * FROM products
```

Result:

```java
List<Product> products =
productDAO.findAll();

```

ဒီမှာ ArrayList သုံးတာ သင့်တော်ပါတယ်။

---

# 12. When to Use LinkedList?

Use LinkedList when:

✅ Insert/Delete များ  
✅ Queue behavior  
✅ Frequent first/last operations

Example:

Kitchen Order Queue:

```
New Order

     |
     v

[Order 1]
[Order 2]
[Order 3]

```

New order add:

```
addLast()

```

Complete:

```
removeFirst()

```

LinkedList သင့်တော်ပါတယ်။

---

# 13. Real Café POS Example

## Product Management

Requirement:

```
Show 10,000 products
Search products
Sort products

```

Use:

```java
ArrayList<Product>
```

Reason:

- Reading များ
    
- JTable display
    
- Random access
    

---

## Kitchen Queue

Requirement:

```
Order received

Order prepared

Remove first

```

Use:

```java
LinkedList<Order>
```

Reason:

```
FIFO

First In First Out

```

---

# 14. Queue with LinkedList

Java:

```java
Queue<Order> queue =
new LinkedList<>();


queue.offer(order);


Order next =
queue.poll();

```

---

Example:

```
Customer 1
Customer 2
Customer 3


Queue:

Front

Customer 1
Customer 2
Customer 3


poll()

Remove Customer 1

```

---

# 15. Important Java Interview Questions

## Q1: Why ArrayList is faster than LinkedList for searching?

Answer:

ArrayList supports:

```
Random Access

O(1)

```

because it uses index.

LinkedList requires traversal:

```
O(n)

```

---

## Q2: Why LinkedList uses more memory?

Because every node stores:

```
Data

Previous Link

Next Link

```

---

## Q3: Is LinkedList always faster?

No.

Only for:

- insertion
    
- deletion
    

in certain situations.

For searching:

ArrayList is faster.

---

# 16. Practical Exercise

Create Café Kitchen Queue.

Class:

```java
class Order {

int id;

String customer;

String item;

}
```

Create:

```java
Queue<Order> orders =
new LinkedList<>();
```

Functions:

```java
addOrder()

processNextOrder()

showPendingOrders()

```

Example:

Input:

```
Order 1 Coffee
Order 2 Cake
Order 3 Tea

```

Process:

```
Processing Coffee

Remaining:

Cake
Tea

```

---

# 17. Professional Recommendation

For your Café POS Project:

## Use ArrayList:

```
Product List

Customer List

Employee List

Sales History

Report Data

```

---

## Use LinkedList:

```
Kitchen Queue

Printer Queue

Notification Queue

Task Queue

```

---

# Lesson 2 Summary

Today you learned:

✅ LinkedList concept  
✅ Node structure  
✅ ArrayList vs LinkedList internal difference  
✅ Performance Big-O  
✅ When to use each  
✅ Queue with LinkedList  
✅ Real POS usage

---

## Next Lesson

# Lesson 3: HashMap Deep Dive (Very Important)

We will learn:

- HashMap Internal Working
    
- Hash Function
    
- Bucket
    
- Collision Handling
    
- equals() and hashCode()
    
- HashMap vs ArrayList
    
- Real-world POS usage
    

Example:

```
Product ID → Product Object

1001 → Coffee
1002 → Cake

```

HashMap က Enterprise Java မှာ အလွန်အသုံးများတဲ့ Collection ဖြစ်ပါတယ်။