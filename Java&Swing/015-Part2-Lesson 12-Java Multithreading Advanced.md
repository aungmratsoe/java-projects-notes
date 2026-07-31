# Part 2: Advanced Java Knowledge for Swing

# Lesson 12: Java Multithreading Advanced

## Building Background Processing for Swing Café POS System

ဒီ Lesson က **Java Desktop Application (Swing)** အတွက် အလွန်အရေးကြီးပါတယ်။

ဘာကြောင့်လဲ?

Swing Application မှာ UI ကို Hang မဖြစ်အောင်:

- Database Query
    
- Report Generation
    
- Receipt Printing
    
- File Export
    
- Network Request
    
- Large Data Processing
    

တွေကို Background Thread မှာ run လုပ်ရပါတယ်။

---

# 1. What is Thread?

Thread ဆိုတာ:

> Program တစ်ခုအတွင်းမှာ အလုပ်လုပ်နေတဲ့ independent execution path တစ်ခု ဖြစ်ပါတယ်။

Example:

Café POS Application:

```
Main Application

        |
        |
 -------------------------
 |           |            |
UI Thread   DB Thread    Print Thread

```

---

Single Thread:

```
User Click

    |
    v

Save Order

    |
    v

Database

    |
    v

UI Update

```

Problem:

Database ကြာရင် UI Freeze ဖြစ်မယ်။

---

# 2. Swing UI Thread (EDT)

Swing မှာ အရေးကြီးဆုံး Concept:

## Event Dispatch Thread (EDT)

Swing UI အားလုံး run နေတဲ့ Thread:

```
EDT

 |
 |
Button Click
Table Update
Label Change

```

---

Rule:

> Swing Component တွေကို EDT မှာပဲ update လုပ်ရမယ်။

Example:

```java
button.setText("Saved");
```

EDT မှာလုပ်ရပါတယ်။

---

# 3. The Problem Without Multithreading

Example:

```java
saveButton.addActionListener(e -> {


    saveToDatabase();


    showMessage();


});

```

Flow:

```
Button Click

      |
      v

EDT

      |
      v

Database 10 seconds


      |
      v

UI Frozen

```

User experience မကောင်းပါ။

---

# 4. Creating Thread

## Method 1: Extending Thread

Example:

```java
class MyThread extends Thread {


@Override
public void run(){


System.out.println(
"Running Thread"
);


}

}

```

Run:

```java
MyThread t =
new MyThread();


t.start();

```

---

Important:

မသုံးသင့်တာ:

```java
t.run();

```

ဘာကြောင့်လဲ?

`run()` ကို တိုက်ရိုက်ခေါ်ရင်:

Normal Method ပဲဖြစ်ပါတယ်။

---

Correct:

```java
start();

```

Java က Thread အသစ် create လုပ်ပေးပါတယ်။

---

# 5. Method 2: Implement Runnable

Professional ပိုသုံးပါတယ်။

```java
class Task implements Runnable{


@Override
public void run(){


System.out.println(
"Task Running"
);


}

}

```

Run:

```java
Thread t =
new Thread(
new Task()
);


t.start();

```

---

# 6. Thread Lifecycle

Thread States:

```
NEW

 |

RUNNABLE

 |

RUNNING

 |

WAITING

 |

TERMINATED

```

---

Example:

Create:

```
NEW
```

start():

```
RUNNABLE
```

CPU ရရင်:

```
RUNNING
```

finish:

```
TERMINATED
```

---

# 7. Thread Methods

## sleep()

Thread ခဏနား:

```java
Thread.sleep(1000);

```

1000 ms = 1 second

Example:

```java
for(int i=0;i<5;i++){


System.out.println(i);


Thread.sleep(1000);


}

```

---

## join()

Thread ပြီးမှ ဆက်လုပ်:

```java
thread.start();

thread.join();

System.out.println(
"Finished"
);

```

---

# 8. Runnable in Café POS

Example:

Receipt Printing:

```java
class ReceiptTask
implements Runnable{


private Order order;


public ReceiptTask(Order order){

this.order=order;

}


public void run(){


printReceipt(order);


}

}

```

---

Use:

```java
new Thread(
new ReceiptTask(order)
)
.start();

```

---

Flow:

```
Customer Payment

       |

Main Thread

       |

Receipt Thread

       |

Print

```

---

# 9. Problem with Many Threads

Suppose:

1000 orders

Create:

```
Thread 1
Thread 2
Thread 3
...
Thread 1000

```

Problem:

- Memory usage
    
- Slow
    
- Difficult management
    

Solution:

```
Thread Pool

```

---

# 10. ExecutorService

ExecutorService ဆိုတာ:

> Thread တွေကို manage လုပ်ပေးတဲ့ high-level API ဖြစ်ပါတယ်။

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

Meaning:

```
Maximum 5 Threads

```

---

Submit Task:

```java
executor.submit(
() -> {

saveOrder();

}
);

```

---

Flow:

```
Task

 |

ExecutorService

 |

Thread Pool

 |

Worker Thread

```

---

# 11. Thread Pool Concept

Without:

```
Task
 |
Create Thread
 |
Destroy Thread

```

Every time.

With Pool:

```
Thread Pool

[Thread]
[Thread]
[Thread]


Reuse

```

---

# 12. shutdown()

Executor ပြီးရင်:

```java
executor.shutdown();

```

Meaning:

```
No new tasks

Existing tasks finish

```

---

# 13. Callable vs Runnable

Runnable:

```java
void run()

```

Return မရှိ:

```java
Task
 |
 |
No Result

```

---

Callable:

```java
V call()

```

Result ပြန်ပေးနိုင်:

```java
Task

 |

Result

```

---

Example:

Runnable:

```java
Runnable r =
() -> save();

```

---

Callable:

```java
Callable<Integer> c =
() -> calculateTotal();

```

---

# 14. Future

Callable result ကို Future နဲ့ယူပါတယ်။

Example:

```java
Future<Integer> future =
executor.submit(
callable
);

```

---

Get Result:

```java
Integer result =
future.get();

```

---

Problem:

`get()` က wait လုပ်ပါတယ်။

---

# 15. Café POS Example: Sales Report

Requirement:

Generate Monthly Report

Heavy process:

```
100,000 sales records

```

Wrong:

```java
button.addActionListener(e -> {


generateReport();


});

```

UI Freeze.

---

Correct:

```java
executor.submit(
() -> {


Report report =
generateReport();


SwingUtilities.invokeLater(
() -> updateTable(report)
);


}
);

```

---

Architecture:

```
EDT

 |
 |
ExecutorService

 |
 |
Worker Thread

 |
 |
Database

```

---

# 16. Synchronization Problem

Multiple Threads:

Example:

```
Cashier Thread

        |
        v

Stock = 1


Kitchen Thread

        |
        v

Stock = 1

```

Both update:

```
Stock--

```

Problem:

Race Condition

---

# 17. Race Condition Example

```java
class Counter{


int count=0;


void increment(){

count++;

}


}

```

Thread 1:

```
read 0
add 1
write 1

```

Thread 2:

```
read 0
add 1
write 1

```

Expected:

```
2

```

Actual:

```
1

```

---

# 18. synchronized Keyword

Solution:

```java
synchronized void increment(){


count++;


}

```

---

Meaning:

```
Only one thread at a time

```

---

# 19. Atomic Variables

Java provides:

```java
AtomicInteger

AtomicBoolean

AtomicLong

```

Example:

```java
AtomicInteger stock =
new AtomicInteger(10);


stock.decrementAndGet();

```

---

Advantages:

- Thread safe
    
- Faster than synchronized for simple operations
    

---

# 20. AtomicBoolean Example

Real POS:

Application status:

```java
AtomicBoolean running =
new AtomicBoolean(true);

```

Stop:

```java
running.set(false);

```

Check:

```java
if(running.get()){


process();


}

```

---

# 21. BlockingQueue

Advanced Producer Consumer Pattern.

Example:

```
Cashier

Producer

    |
    |
BlockingQueue

    |
    |

Kitchen

Consumer

```

---

Create:

```java
BlockingQueue<Order> queue =
new LinkedBlockingQueue<>();

```

---

Producer:

```java
queue.put(order);

```

---

Consumer:

```java
Order order =
queue.take();

```

---

Difference:

`take()` waits automatically.

---

# 22. Complete Café POS Thread Architecture

Professional Design:

```
                 Swing EDT

                    |

              User Actions


                    |

          --------------------

          ExecutorService


          |        |        |

       Order     Report   Print
       Thread    Thread   Thread


          |

       Database


          |

      BlockingQueue


          |

       Kitchen

```

---

# 23. SwingWorker

Swing မှာ built-in solution:

```java
SwingWorker<Result,Progress>

```

Example:

```java
SwingWorker<Void,Void> worker =
new SwingWorker(){


protected Void doInBackground(){


loadData();


return null;

}


protected void done(){


updateUI();


}


};

```

---

Used for:

- Long operations
    
- Background loading
    
- Progress bar
    

---

# 24. Thread Best Practices

## 1. Don't create Thread manually everywhere

Bad:

```java
new Thread()

```

many places.

Use:

```
ExecutorService

```

---

## 2. Never block EDT

Bad:

```java
Thread.sleep()

Database query

```

inside button event.

---

## 3. Use thread-safe collections

Examples:

```
ConcurrentHashMap

BlockingQueue

CopyOnWriteArrayList

```

---

# 25. Interview Questions

## Q1: Difference between Thread and ExecutorService?

Thread:

- Low level
    
- Manual management
    

ExecutorService:

- Thread pool
    
- Task management
    

---

## Q2: Difference between Runnable and Callable?

Runnable:

```
No return value

```

Callable:

```
Returns value

Can throw Exception

```

---

## Q3: What is Future?

Future represents result of asynchronous computation.

---

## Q4: synchronized vs Atomic?

synchronized:

- Lock based
    

Atomic:

- Lock-free operations
    

---

# Practice Project

Build Café POS Background System:

Create:

```
OrderProcessor

ReportGenerator

ReceiptPrinter

KitchenQueue

```

Use:

```
ExecutorService

Callable

Future

BlockingQueue

AtomicBoolean

```

Requirements:

1. Customer places order
    
2. Order goes to Queue
    
3. Kitchen processes
    
4. Receipt prints in background
    
5. UI never freezes
    

---

# Lesson 12 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Thread Concept  
✅ Swing EDT  
✅ Thread Creation  
✅ Runnable  
✅ Thread Lifecycle  
✅ ExecutorService  
✅ Thread Pool  
✅ Callable  
✅ Future  
✅ Synchronization  
✅ Atomic Variables  
✅ BlockingQueue  
✅ Producer Consumer Pattern  
✅ SwingWorker  
✅ POS Multithreading Architecture

---

# Next Lesson

# Lesson 13: Java File I/O & Serialization Deep Dive

## Building Export, Backup & Configuration System

သင်ယူမည့်အရာ:

- File Reading/Writing
    
- BufferedReader
    
- BufferedWriter
    
- Files API
    
- JSON Handling Concept
    
- Object Serialization
    
- POS Backup System
    
- Export CSV Report
    

Example:

```
Daily Sales Report

        |

sales_2026_07_31.csv

```

ပြီးရင် Café POS မှာ Data Export / Backup Feature တည်ဆောက်နိုင်ပါမယ်။