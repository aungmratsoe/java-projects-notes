# Part 2: Advanced Java Knowledge for Swing

# Lesson 20: Advanced Multithreading for Café POS

## Java 25 Concurrency Architecture

### (ExecutorService + Thread Pool + CompletableFuture + Virtual Threads + Atomic Classes)

ဒီ Lesson မှာ Java concurrency ကို **Professional Java Developer Level** နဲ့ လေ့လာပါမယ်။

Café POS System မှာ Thread မသုံးရင် ဖြစ်နိုင်တဲ့ပြဿနာများ:

- UI Freeze
    
- Slow Database Operation
    
- Payment Waiting
    
- Printer Blocking
    
- Multiple Cashier Conflict
    
- Inventory Race Condition
    

---

# 1. What is Concurrency?

Concurrency ဆိုတာ:

> Task အများကြီးကို တစ်ချိန်တည်းမှာ manage လုပ်နိုင်ခြင်း

Example Café POS:

```text
Cashier 1
    |
    |-- Create Order


Cashier 2
    |
    |-- Payment


Kitchen Printer
    |
    |-- Print Receipt


Inventory
    |
    |-- Update Stock

```

အားလုံးကို application က handle လုပ်ရပါတယ်။

---

# 2. Process vs Thread

## Process

Application တစ်ခု:

```
Café POS Application

Memory
CPU
Resources

```

---

## Thread

Process အတွင်းမှာရှိတဲ့ execution unit:

```
POS Application

 |
 +-- UI Thread
 |
 +-- Database Thread
 |
 +-- Printing Thread

```

---

# 3. Swing and Thread

Swing မှာ အရေးကြီးတာ:

```
EDT
(Event Dispatch Thread)

```

Swing UI ကို ဒီ Thread က control လုပ်ပါတယ်။

---

Wrong:

```java
button.addActionListener(e -> {


loadProductsFromDatabase();


});

```

Problem:

```
Click

 |

Database Query 10 seconds

 |

UI Freeze

```

---

Correct:

```
EDT

 |
 |
Worker Thread

 |
 |
Database

```

---

# 4. Creating Thread

Old way:

```java
Thread thread =
new Thread(() -> {


processOrder();


});


thread.start();

```

---

Problem:

Every task creates new thread.

Example:

```
10000 Orders

10000 Threads

```

Memory issue.

---

Solution:

Thread Pool.

---

# 5. Thread Pool Concept

Thread Pool ဆိုတာ:

> Reusable threads collection

Diagram:

```
Tasks

 |
 |
 v

+----------------+

 Thread Pool

+----------------+

 Thread 1

 Thread 2

 Thread 3

 Thread 4


```

---

Task ပြီးရင် Thread ကို ပြန်သုံးပါတယ်။

---

# 6. ExecutorService

ExecutorService:

> Thread Pool ကို manage လုပ်တဲ့ Java API

Package:

```java
java.util.concurrent

```

---

Create:

```java
ExecutorService executor =
Executors.newFixedThreadPool(5);

```

---

Meaning:

```
Maximum 5 worker threads

```

---

# 7. Café POS Executor Example

```java
ExecutorService executor =
Executors.newFixedThreadPool(10);


executor.submit(() -> {


processOrder();


});

```

---

Flow:

```
Button Click

     |

ExecutorService

     |

Worker Thread

     |

Database

```

---

# 8. execute() vs submit()

## execute()

Runnable only:

```java
executor.execute(() -> {


printReceipt();


});

```

Return:

```
Nothing

```

---

## submit()

Runnable / Callable:

```java
Future<Product> future =
executor.submit(() -> {


return loadProduct();


});

```

Return:

```
Future

```

---

# 9. Runnable

Runnable:

> Task without return value

Example:

```java
Runnable task = () -> {


System.out.println(
"Printing"
);


};

```

---

No result.

---

# 10. Callable

Callable:

> Task with return value

Runnable:

```java
void process()

```

Callable:

```java
Product process()
```

---

Example:

```java
Callable<Double> task =
() -> {


return calculateTotal();


};

```

---

# 11. Future

Future:

> Represents a result that will be available later

Example:

```java
Future<Double> result =
executor.submit(task);

```

---

Get result:

```java
Double total =
result.get();

```

---

Problem:

`get()` blocks.

```
Wait until finished

```

---

# 12. CompletableFuture

Modern Java approach.

Instead of:

```java
Future.get()

```

Use:

```java
CompletableFuture

```

---

Example:

```java
CompletableFuture
.supplyAsync(() -> {


return loadProducts();


})

.thenAccept(products -> {


updateTable(products);


});

```

---

Flow:

```
Load Products

      |

When finished

      |

Update UI

```

---

# 13. Swing + CompletableFuture

Example:

```java
button.addActionListener(e -> {


CompletableFuture
.supplyAsync(() -> {


return productService.findAll();


})

.thenAccept(products -> {


SwingUtilities.invokeLater(() -> {


table.update(products);


});


});


});

```

---

Result:

```
UI Never Freezes

```

---

# 14. Virtual Threads (Java 25)

Java 21+ major feature.

Traditional:

```
10000 Requests

10000 Platform Threads


Memory High

```

---

Virtual Thread:

```
10000 Requests

10000 Virtual Threads


Low Memory

```

---

Create:

```java
Thread.startVirtualThread(() -> {


processOrder();


});

```

---

# 15. Platform Thread vs Virtual Thread

||Platform|Virtual|
|---|---|---|
|Memory|High|Low|
|Creation|Slow|Fast|
|Best For|CPU tasks|I/O tasks|
|Database|OK|Excellent|
|HTTP Calls|OK|Excellent|

---

# 16. Café POS Virtual Thread Example

Multiple branches:

```
Yangon Branch

Mandalay Branch

Bago Branch


        |

Virtual Threads


        |

Central Server

```

---

Code:

```java
Thread.startVirtualThread(() -> {


syncBranchOrders();


});

```

---

# 17. ExecutorService vs Virtual Thread

Traditional:

```java
ExecutorService pool =
Executors.newFixedThreadPool(20);

```

Good for:

- Controlled resources
    
- CPU tasks
    

---

Virtual:

```java
Executors.newVirtualThreadPerTaskExecutor();

```

Good for:

- Many blocking tasks
    
- API calls
    
- Database
    

---

# 18. Atomic Classes

Problem:

Multiple threads changing same data.

Example:

```
Stock = 10


Thread A sells 1

Thread B sells 1


```

---

Race condition:

```
Both read 10

Both write 9


Wrong!

```

---

# 19. AtomicInteger

Instead of:

```java
int stock;

```

Use:

```java
AtomicInteger stock =
new AtomicInteger(10);

```

---

Decrease:

```java
stock.decrementAndGet();

```

---

Thread safe.

---

# 20. AtomicBoolean

Very common.

Example:

Application running state:

```java
AtomicBoolean running =
new AtomicBoolean(true);

```

---

Check:

```java
if(running.get()){


process();


}

```

---

Stop:

```java
running.set(false);

```

---

# 21. Café POS AtomicBoolean Example

Printer service:

```java
class PrinterService {


private AtomicBoolean running =
new AtomicBoolean(true);



void stop(){

running.set(false);

}


}

```

---

Thread safe shutdown.

---

# 22. Synchronization

Keyword:

```java
synchronized

```

---

Example:

```java
public synchronized void updateStock(){


stock--;


}

```

---

Meaning:

Only one thread enters.

---

# 23. Concurrent Collections

Normal:

```java
HashMap

```

Not thread safe.

---

Use:

```java
ConcurrentHashMap

```

---

Example:

```java
Map<Integer,Product> cache =
new ConcurrentHashMap<>();

```

---

Multiple threads can access safely.

---

# 24. Race Condition Example

Bad:

```java
if(stock > 0){

stock--;

}

```

---

Two threads:

```
Thread A check 1

Thread B check 1


Both decrease


```

---

Solution:

```java
synchronized

or

AtomicInteger

```

---

# 25. Deadlock

Two threads waiting forever.

Example:

```
Thread A

Lock Product

wait Order


Thread B

Lock Order

wait Product


```

---

Solution:

Always lock same order.

```
Product

then

Order

```

---

# 26. Shutdown ExecutorService

Important.

Wrong:

```java
executor.shutdown();

```

immediately after submit.

---

Correct:

```java
executor.shutdown();


executor.awaitTermination(
10,
TimeUnit.SECONDS
);

```

---

# 27. Exception Handling in Threads

Problem:

Exception disappears.

Example:

```java
executor.submit(() -> {


throw new RuntimeException();


});

```

---

Solution:

```java
CompletableFuture
.exceptionally(e -> {


log.error(
"Failed",
e
);


return null;


});

```

---

# 28. Complete Café POS Async Architecture

```
                 Swing UI

                    |

                 Controller

                    |

          CompletableFuture

                    |

          Virtual Thread Pool

                    |

        ---------------------

        |                   |

    Database             Printer


        |

     MySQL

```

---

# 29. Real Example: Order Processing

User clicks:

```
Place Order

```

---

Thread Flow:

```
EDT

 |

Create Task

 |

Virtual Thread

 |

Validate Order

 |

Save Database

 |

Update Inventory

 |

Print Receipt

 |

Notify UI

```

---

# 30. Java 25 Best Practices

Use:

✅ CompletableFuture

✅ Virtual Threads for I/O

✅ ExecutorService

✅ Atomic Classes

✅ Concurrent Collections

✅ Proper shutdown

Avoid:

❌ Creating Thread everywhere

❌ Blocking Swing EDT

❌ Shared mutable state

❌ synchronized everywhere

---

# 31. Interview Questions

## Q1: Difference between Thread and ExecutorService?

Thread:

- Creates individual thread
    

ExecutorService:

- Manages thread pool
    

---

## Q2: Runnable vs Callable?

Runnable:

```
No return

```

Callable:

```
Returns value

```

---

## Q3: Future vs CompletableFuture?

Future:

- Blocking
    

CompletableFuture:

- Async chaining
    

---

## Q4: Why AtomicInteger?

Thread-safe operations without locking.

---

## Q5: Virtual Thread purpose?

Handle massive concurrent I/O tasks efficiently.

---

# Practice Project

Build Async Café POS System:

Create:

```
OrderProcessor

PrinterService

InventoryService

NotificationService

```

Implement:

1. ExecutorService
    
2. CompletableFuture
    
3. Virtual Thread
    
4. Atomic Stock Control
    
5. Concurrent Product Cache
    
6. Graceful Shutdown
    

---

# Lesson 20 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Thread Concept  
✅ Swing EDT  
✅ ExecutorService  
✅ Thread Pool  
✅ Runnable  
✅ Callable  
✅ Future  
✅ CompletableFuture  
✅ Virtual Threads Java 25  
✅ AtomicBoolean  
✅ AtomicInteger  
✅ Synchronization  
✅ Concurrent Collections  
✅ Race Condition  
✅ Deadlock Prevention  
✅ Async Café POS Architecture

---

# Next Lesson

# Lesson 21: Enterprise Application Architecture

## Building Complete Café POS System Structure

Topics:

- Clean Architecture
    
- Layered Architecture
    
- MVC + Service + Repository
    
- Dependency Injection
    
- Module Design
    
- Package Structure
    
- Configuration Management
    
- Application Lifecycle
    
- Production Deployment Strategy
    

ဒီ Lesson ပြီးရင် Café POS Project ကို **Senior Java Developer Level Architecture** နဲ့ စတင် Build လုပ်နိုင်ပါမယ်။