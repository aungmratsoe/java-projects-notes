# Part 2: Advanced Java Knowledge for Swing

# Lesson 5: Queue & PriorityQueue Deep Dive

## Building Real-world Order Processing System

ဒီ Lesson မှာ **Queue Data Structure** ကို လေ့လာပါမယ်။

Café POS System အတွက် Queue က အလွန်အရေးကြီးပါတယ်။

Real-world examples:

- Kitchen Order Queue
    
- Printer Queue
    
- Customer Waiting Line
    
- Payment Processing Queue
    
- Background Task Queue
    

---

# 1. What is Queue?

Queue ဆိုတာ:

> First In First Out (FIFO) Principle နဲ့ အလုပ်လုပ်တဲ့ Data Structure ဖြစ်ပါတယ်။

FIFO:

**First In → First Out**

ဥပမာ:

Customer Queue:

```id="f3j2ad"
Customer A
Customer B
Customer C
```

ရောက်လာတဲ့အတိုင်း:

```
First customer = First served
```

---

Real life:

Bus Queue:

```
Entrance

 ↓

A → B → C → D

 ↓

Bus

```

A က အရင်ဝင်ပါတယ်။

---

# 2. Queue Structure

Queue:

```id="v4j8zq"
Front                         Rear

 |                            |
 v                            v

[ A ][ B ][ C ][ D ]

```

Operations:

- Add → Rear
    
- Remove → Front
    

---

# 3. Queue Interface

Java Collection:

```
Collection

     |
     |
   Queue

     |
     |
----------------
|              |
LinkedList  PriorityQueue

```

---

# 4. Queue Methods

Queue မှာ အဓိက 6 ခုရှိပါတယ်။

|Method|Purpose|
|---|---|
|offer()|Add element|
|poll()|Remove element|
|peek()|View first element|
|add()|Add element|
|remove()|Remove|
|element()|View|

---

# 5. Creating Queue with LinkedList

Example:

```java
import java.util.Queue;
import java.util.LinkedList;


Queue<String> orders =
new LinkedList<>();


orders.offer("Coffee");
orders.offer("Cake");
orders.offer("Tea");
```

Queue:

```
Front

Coffee
Cake
Tea

Rear

```

---

# 6. offer() Method

Add data:

```java
orders.offer("Coffee");
```

Result:

```
Coffee

```

---

Add more:

```java
orders.offer("Cake");
orders.offer("Tea");
```

Result:

```
Coffee → Cake → Tea
```

---

# 7. poll() Method

Remove first element:

```java
String order =
orders.poll();
```

Return:

```
Coffee
```

Queue becomes:

```
Cake → Tea
```

---

# 8. peek() Method

Look only:

```java
String first =
orders.peek();
```

Output:

```
Coffee
```

Queue unchanged:

```
Coffee → Cake → Tea
```

---

# 9. Queue Example: Café Kitchen

Customer Orders:

```
10:00 Coffee

10:01 Cake

10:02 Tea

```

Queue:

```
Front

Coffee
Cake
Tea

Rear

```

Kitchen:

```java
prepareNextOrder();
```

First:

```
Coffee
```

Then:

```
Cake
```

---

# 10. Build Simple Kitchen Queue

Order Class:

```java
public class Order {


private int id;

private String item;


public Order(
int id,
String item
){

this.id=id;
this.item=item;

}


public String getItem(){

return item;

}


}
```

---

Queue:

```java
Queue<Order> kitchenQueue =
new LinkedList<>();
```

---

Add Orders:

```java
kitchenQueue.offer(
new Order(1,"Coffee")
);


kitchenQueue.offer(
new Order(2,"Cake")
);


kitchenQueue.offer(
new Order(3,"Tea")
);

```

---

Process:

```java
Order order =
kitchenQueue.poll();


System.out.println(
order.getItem()
);
```

Output:

```
Coffee
```

---

# 11. Queue vs Stack

Important Interview Topic.

## Queue

FIFO:

```
A B C

remove A

```

---

## Stack

LIFO:

Last In First Out

```
A
B
C

remove C

```

Example:

Browser Back Button.

---

# 12. PriorityQueue

Now advanced part.

Normal Queue:

```
Order Time

A
B
C

```

PriorityQueue:

```
Priority

Emergency
VIP
Normal

```

---

Definition:

> PriorityQueue processes elements according to priority instead of insertion order.

---

# 13. Creating PriorityQueue

Example:

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>();


pq.offer(30);
pq.offer(10);
pq.offer(20);
```

Output order:

```
10
20
30
```

Because default:

Min Heap

---

# 14. Internal Working of PriorityQueue

PriorityQueue uses:

```
Binary Heap

        10

      /    \

    20      30

```

Smallest element always at root.

---

# 15. PriorityQueue Example

Restaurant Orders:

Priority:

```
1 = Emergency

2 = VIP

3 = Normal

```

Order:

```java
class Order {


int priority;

String name;


}
```

---

Create Comparator:

```java
PriorityQueue<Order> queue =
new PriorityQueue<>(
(a,b)-> a.priority-b.priority
);

```

---

Add:

```java
queue.offer(
new Order(3,"Normal")
);


queue.offer(
new Order(1,"Emergency")
);


queue.offer(
new Order(2,"VIP")
);

```

---

Process:

```
Emergency

VIP

Normal

```

---

# 16. Comparator Concept

Comparator ဆိုတာ:

> Object တွေကို ဘယ်လို compare လုပ်မလဲ သတ်မှတ်ပေးတဲ့ interface ဖြစ်ပါတယ်။

Example:

Sort Product Price:

```java
Comparator<Product> priceComparator =
(a,b)->
Double.compare(
a.price,
b.price
);

```

---

# 17. Real Café POS Usage

## Normal Kitchen Queue

Use:

```
LinkedList Queue

```

Example:

```
Order received time

10:01 Coffee

10:02 Cake

```

---

## Priority Kitchen Queue

Use:

```
PriorityQueue

```

Example:

```
VIP Customer

Food Allergy

Emergency Order

```

---

# 18. Queue in Swing Application

Example:

Order Button:

```
Click Sell

        |

Create Order

        |

Add Queue

        |

Kitchen Screen

```

---

Code:

```java
sellButton.addActionListener(e->{


Order order =
createOrder();


kitchenQueue.offer(order);


});
```

---

Kitchen Thread:

```java
while(true){


Order order =
kitchenQueue.poll();


prepare(order);


}

```

---

# 19. Queue + Multithreading Preview

Later we will learn:

```
Swing UI Thread

        |

ExecutorService

        |

Worker Thread

        |

Queue

```

Real applications use:

- BlockingQueue
    
- ExecutorService
    
- Thread Pool
    

---

# 20. BlockingQueue (Advanced Preview)

Normal Queue:

```java
Queue<Order>
```

Problem:

No order available?

Need checking.

BlockingQueue:

```java
BlockingQueue<Order> queue =
new LinkedBlockingQueue<>();
```

Features:

- Thread safe
    
- Wait automatically
    
- Producer / Consumer pattern
    

Example:

```
Cashier Thread

      |
      v

 Order Queue

      |
      v

Kitchen Thread

```

---

# 21. Queue Interview Questions

## Q1: What is Queue?

Answer:

A FIFO data structure where insertion happens at rear and removal happens at front.

---

## Q2: Difference between Queue and PriorityQueue?

Queue:

```
Insertion order

```

PriorityQueue:

```
Priority order

```

---

## Q3: Difference between poll() and remove()?

poll():

```
Empty queue → returns null

```

remove():

```
Empty queue → throws exception

```

---

## Q4: Difference between peek() and element()?

peek():

```
Empty → null

```

element():

```
Empty → exception

```

---

# 22. Café POS Collection Architecture

Now our POS data structure looks like:

```
Products

    |
    |
ArrayList<Product>


Product Search

    |
    |
HashMap<Integer,Product>


Permissions

    |
    |
HashSet<String>


Kitchen Orders

    |
    |
Queue<Order>


VIP Orders

    |
    |
PriorityQueue<Order>

```

---

# Lesson 5 Summary

Today you learned:

✅ Queue concept  
✅ FIFO principle  
✅ Queue methods  
✅ LinkedList as Queue  
✅ Kitchen Order System  
✅ PriorityQueue  
✅ Comparator  
✅ Heap concept  
✅ BlockingQueue introduction  
✅ Real POS usage

---

# Next Lesson

## Lesson 6: Java Stream API Deep Dive

We will learn:

- Why Stream API?
    
- Lambda Expression connection
    
- filter()
    
- map()
    
- sorted()
    
- collect()
    
- reduce()
    
- Real Café POS data processing
    

Example:

```java
products.stream()
.filter(p -> p.getPrice()>5000)
.collect(Collectors.toList());

```

This is the modern Java style used in professional applications.