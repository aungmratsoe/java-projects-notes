`ExecutorService`, `Thread Pool`, `Runnable`, `Callable`, `Future`, `shutdown()` နဲ့ `submit()`

# Java Concurrency အရေးကြီး Concepts များ (မြန်မာလို အသေးစိတ်ရှင်းပြချက်)

Java Interview တွေမှာ အမေးများဆုံးဖြစ်တဲ့

- ExecutorService
    
- Thread Pool
    
- Runnable
    
- Callable
    
- Future
    
- submit()
    
- shutdown()
    

တို့ကို နားလည်ထားရင် Java Concurrency အခြေခံကို နားလည်ပြီးသားလို့ ပြောလို့ရပါတယ်။

---

# အရင်ဆုံး Relationship ကိုနားလည်ရအောင်

```text
                    Task
               (Runnable/Callable)
                      │
                      ▼
                 submit()
                      │
                      ▼
             ExecutorService
                      │
                      ▼
                Thread Pool
                      │
                      ▼
                 Worker Threads
                      │
                      ▼
                Execute Task
                      │
                      ▼
             Future (Result)
```

ဆိုလိုတာက

```
Runnable / Callable
        ↓
submit()
        ↓
ExecutorService
        ↓
Thread Pool
        ↓
Thread
        ↓
Future
```

---

# 1. Thread ဆိုတာဘာလဲ?

Thread ဆိုတာ

**Program တစ်ခုအတွင်း အလုပ်လုပ်နေတဲ့ လမ်းကြောင်း (Execution Path)** ဖြစ်ပါတယ်။

ဥပမာ

```java
new Thread(() -> {
    System.out.println("Hello");
}).start();
```

ဒီမှာ

```
Thread အသစ်တစ်ခု
↓

Hello
```

Run သွားပါတယ်။

---

# ပြဿနာ

Task 1000 ခုရှိရင်

```java
for(int i=0;i<1000;i++){

    new Thread(...).start();

}
```

ဖြစ်လာမယ်

ဒီလိုဆို

```
Thread 1000 ခု
```

ဖန်တီးရမယ်။

Memory များတယ်။

CPU Scheduling ခက်တယ်။

Performance ကျတယ်။

ဒါကြောင့် Thread Pool ပေါ်လာပါတယ်။

---

# 2. Thread Pool ဆိုတာဘာလဲ?

Thread Pool ဆိုတာ

> Thread တွေကို ကြိုတင်ဖန်တီးထားတဲ့ Pool (အုပ်စု)

ဖြစ်ပါတယ်။

ဥပမာ

```text
Pool

Thread-1
Thread-2
Thread-3
Thread-4
```

Task တွေရောက်လာရင်

```text
Task1 → Thread1

Task2 → Thread2

Task3 → Thread3

Task4 → Thread4

Task5 → Waiting
```

Thread အသစ်မဖန်တီးတော့ဘူး။

ပြီးသွားရင်

Thread ကို

Reuse ပြန်လုပ်တယ်။

---

## Thread Pool အားသာချက်

✅ Thread Reuse

✅ Memory သက်သာ

✅ CPU Utilization ကောင်း

✅ Faster

---

# 3. ExecutorService ဆိုတာဘာလဲ?

Thread Pool ကို Control လုပ်ပေးတဲ့ Interface ဖြစ်ပါတယ်။

Diagram

```text
Task

↓

ExecutorService

↓

Thread Pool

↓

Worker Threads
```

ဖန်တီးပုံ

```java
ExecutorService executor =
Executors.newFixedThreadPool(3);
```

ဒီမှာ

```
Thread 3 ခု
```

ကိုပဲ အသုံးပြုမယ်။

---

## ExecutorService က ဘာတွေလုပ်ပေးလဲ?

- Task လက်ခံတယ်။
    
- Queue ထဲထည့်တယ်။
    
- Thread အလွတ်ရရင် Execute လုပ်တယ်။
    
- Thread တွေကို Reuse လုပ်တယ်။
    
- Shutdown လုပ်နိုင်တယ်။
    

---

# 4. Runnable ဆိုတာဘာလဲ?

Runnable ဆိုတာ

> Return Value မလိုတဲ့ Task

ဖြစ်ပါတယ်။

Syntax

```java
Runnable r = () -> {

    System.out.println("Working");

};
```

Run

```java
executor.submit(r);
```

---

Runnable Interface

```java
public interface Runnable{

    void run();

}
```

သတိထားပါ

```
void run()
```

Return မရှိဘူး။

---

ဥပမာ

```java
Runnable r = () -> {

    System.out.println("Save File");

};
```

Save File လုပ်တယ်။

Result မလိုဘူး။

Runnable သုံးလို့ရတယ်။

---

# Runnable Diagram

```text
Runnable

↓

run()

↓

No Return
```

---

# 5. Callable ဆိုတာဘာလဲ?

Runnable နဲ့ဆင်တူတယ်။

ဒါပေမယ့်

Result ပြန်ပေးနိုင်တယ်။

```java
Callable<Integer> c = () -> {

    return 100;

};
```

Interface

```java
public interface Callable<V>{

    V call();

}
```

Runnable

```
run()

↓

void
```

Callable

```
call()

↓

Return Value
```

---

ဥပမာ

Calculator

```java
Callable<Integer> add = () -> {

    return 10+20;

};
```

Result

```
30
```

---

# Runnable vs Callable

|Runnable|Callable|
|---|---|
|`run()`|`call()`|
|Return မရှိ|Return ရှိ|
|Checked Exception မပစ်နိုင်|Checked Exception ပစ်နိုင်|
|Simple Task|Result လိုတဲ့ Task|

---

# 6. submit() ဆိုတာဘာလဲ?

ExecutorService ထဲကို

Task ထည့်တဲ့ Method ဖြစ်ပါတယ်။

```java
executor.submit(task);
```

Flow

```text
Task

↓

submit()

↓

Executor Queue

↓

Worker Thread
```

---

Runnable Submit

```java
executor.submit(() -> {

    System.out.println("Hello");

});
```

---

Callable Submit

```java
Future<Integer> future =
executor.submit(() -> {

    return 500;

});
```

---

# execute() vs submit()

execute()

```java
executor.execute(task);
```

Return

```
Nothing
```

submit()

```java
Future<Integer> future =
executor.submit(task);
```

Return

```
Future
```

---

# 7. Future ဆိုတာဘာလဲ?

Future ဆိုတာ

> "အနာဂတ်မှာ Result ရမယ့် Object"

ဖြစ်ပါတယ်။

Diagram

```text
Callable

↓

Running

↓

Future

↓

Result
```

ဥပမာ

```java
Future<Integer> future =
executor.submit(() -> {

    return 99;

});

System.out.println(future.get());
```

Output

```
99
```

---

`future.get()`

ဆိုတာ

Task မပြီးသေးရင်

```
စောင့်တယ်
```

ပြီးမှ

Result ပြန်ပေးတယ်။

---

Future Methods

```java
future.get();
```

Result ယူတယ်။

---

```java
future.isDone();
```

ပြီးပြီလား?

---

```java
future.cancel(true);
```

Cancel လုပ်တယ်။

---

# Future Diagram

```text
submit()

↓

Future

↓

Running...

↓

Finished

↓

Result
```

---

# 8. shutdown() ဆိုတာဘာလဲ?

ExecutorService ပိတ်ဖို့။

```java
executor.shutdown();
```

ဒါက

```
Task အသစ်

×

လက်မခံတော့
```

ဒါပေမယ့်

ရှိပြီးသား Task တွေ

```
ပြီးအောင် Run
```

လုပ်တယ်။

Diagram

```text
Task1 ✔

Task2 ✔

Task3 ✔

↓

shutdown()

↓

No New Tasks
```

---

# shutdownNow()

```java
executor.shutdownNow();
```

ဒါက

```
Running Task

Interrupt
```

လုပ်ဖို့ ကြိုးစားတယ်။

Waiting Task တွေကိုလည်း Cancel လုပ်ဖို့ ကြိုးစားတယ်။

---

# Real World Example

Online Shop

Customer

```
A

B

C

D
```

Orders

↓

ExecutorService

↓

Thread Pool

```
Cashier1

Cashier2

Cashier3
```

Customer 100 ယောက်လာရင်

Cashier အသစ် 100 ယောက် မခန့်ဘူး။

Cashier 3 ယောက်ပဲ

Order အလှည့်ကျ လုပ်ပေးတယ်။

ဒါက

```
Thread Pool
```

အလုပ်လုပ်ပုံနဲ့ တူပါတယ်။

---

# Complete Flow Example

```java
import java.util.concurrent.*;

public class Main {

    public static void main(String[] args) throws Exception {

        ExecutorService executor =
                Executors.newFixedThreadPool(2);

        Callable<Integer> task = () -> {

            Thread.sleep(2000);

            return 500;

        };

        Future<Integer> future =
                executor.submit(task);

        System.out.println("Waiting...");

        System.out.println(future.get());

        executor.shutdown();

    }

}
```

Flow

```text
Create ExecutorService
        │
        ▼
Create Thread Pool (2 Threads)
        │
        ▼
Create Callable Task
        │
        ▼
submit(task)
        │
        ▼
Task enters Queue
        │
        ▼
Worker Thread executes Task
        │
        ▼
Future created
        │
        ▼
future.get()
        │
        ▼
Receive Result (500)
        │
        ▼
shutdown()
        │
        ▼
Executor stops accepting new tasks
```

Output

```
Waiting...

500
```

---

# Comparison Table

|Concept|အဓိပ္ပါယ်|Return Value|အဓိကအသုံးပြုမှု|
|---|---|---|---|
|**Thread**|အလုပ်လုပ်တဲ့ Execution Path|-|Task ကို တကယ် Run လုပ်ပေးတယ်|
|**Thread Pool**|Thread အုပ်စု|-|Thread တွေကို Reuse လုပ်တယ်|
|**ExecutorService**|Thread Pool ကို စီမံတဲ့ Interface|-|Task Submit၊ Shutdown စတဲ့ Management|
|**Runnable**|Result မလိုတဲ့ Task|❌|Logging, File Save, Email ပို့ခြင်း|
|**Callable**|Result လိုတဲ့ Task|✅|Database Query, Calculation, API Call|
|**submit()**|Task ကို ExecutorService ထဲထည့်တယ်|`Future`|Runnable/Callable ကို Execute လုပ်ဖို့|
|**Future**|Task ရဲ့ အနာဂတ် Result ကို ကိုယ်စားပြုတဲ့ Object|Task Result|`get()`, `isDone()`, `cancel()`|
|**shutdown()**|ExecutorService ကို စနစ်တကျပိတ်တယ်|-|Task အသစ်မလက်ခံတော့ဘဲ ရှိပြီးသား Task တွေပြီးအောင်စောင့်တယ်|

---

# Interview မှာ အမေးများတဲ့ Questions

### Q1. `Runnable` နဲ့ `Callable` ဘာကွာလဲ?

**အဖြေ**

- `Runnable` → `run()` Method၊ Return Value မရှိ၊ Checked Exception မပစ်နိုင်။
    
- `Callable` → `call()` Method၊ Return Value ရှိ၊ Checked Exception ပစ်နိုင်။
    

---

### Q2. `execute()` နဲ့ `submit()` ဘာကွာလဲ?

**အဖြေ**

- `execute()` → `Runnable` ကိုသာ လက်ခံပြီး Return Value မရှိ။
    
- `submit()` → `Runnable` နဲ့ `Callable` နှစ်မျိုးလုံး လက်ခံနိုင်ပြီး `Future` Return ပြန်ပေးတယ်။
    

---

### Q3. `Future.get()` က ဘာလုပ်သလဲ?

**အဖြေ**

Task မပြီးသေးရင် စောင့်ပြီး (blocking)၊ ပြီးသွားတဲ့အခါ Result ကို ပြန်ပေးတယ်။

---

### Q4. `shutdown()` နဲ့ `shutdownNow()` ဘာကွာလဲ?

**အဖြေ**

- `shutdown()` → Task အသစ်မလက်ခံတော့ဘဲ ရှိပြီးသား Task တွေကို ပြီးအောင် Run ခွင့်ပေးတယ်။
    
- `shutdownNow()` → Running Task တွေကို Interrupt လုပ်ပြီး Waiting Task တွေကို Cancel လုပ်ဖို့ ကြိုးစားတယ်။
    

---

# အလွယ်မှတ်ရန်

```text
Runnable
    │
    ├── Result မလို
    ▼
submit()
    │
    ▼
ExecutorService
    │
    ▼
Thread Pool
    │
    ▼
Worker Thread
    │
    ▼
Callable (Result လို)
    │
    ▼
Future
    │
    ▼
future.get() → Result
    │
    ▼
shutdown() → ExecutorService ကို စနစ်တကျပိတ်
```

ဒီ Flow ကို နားလည်ထားရင် Java Concurrency ရဲ့ အခြေခံအလုပ်လုပ်ပုံကို ရှင်းရှင်းလင်းလင်း နားလည်နိုင်ပြီး Interview မေးခွန်းအများစုကိုလည်း ယုံကြည်စိတ်ချစွာ ဖြေနိုင်ပါတယ်။