# Java `ExecutorService` ဆိုတာဘာလဲ? (မြန်မာလို ရှင်းပြချက်)

`ExecutorService` ဆိုတာ **Java မှာ Thread တွေကို စီမံခန့်ခွဲပေးတဲ့ Interface** ဖြစ်ပါတယ်။

Thread တွေကို `new Thread()` နဲ့ တစ်ခါတည်းဖန်တီးပြီး Run လုပ်မယ့်အစား `ExecutorService` ကိုသုံးရင် Thread တွေကို **Reuse (ပြန်အသုံးပြု)** လုပ်နိုင်ပြီး Performance ကောင်းလာပါတယ်။

---

# ExecutorService ကို ဘာကြောင့်သုံးသလဲ?

ဥပမာ...

သင့်မှာ Task 100 ခုရှိတယ်ဆိုပါစို့။

```java
for(int i = 0; i < 100; i++){
    new Thread(() -> {
        System.out.println("Running...");
    }).start();
}
```

ဒီလိုရေးရင်

- Thread 100 ခု အသစ်ဖန်တီးရမယ်
    
- Memory ပိုသုံးတယ်
    
- CPU Scheduling ခက်တယ်
    
- Performance ကျနိုင်တယ်
    

ဒါကြောင့် Java က

**ExecutorService**

ကိုပေးထားတာပါ။

---

# ExecutorService အလုပ်လုပ်ပုံ

```
Task 1
Task 2
Task 3
Task 4
Task 5
    |
    V

+----------------+
| ExecutorService|
+----------------+
       |
       V

 Thread Pool

 Thread-1
 Thread-2
 Thread-3
```

Task တွေကို Thread Pool ထဲက Thread တွေနဲ့ Run ပေးပါတယ်။

Thread အသစ်အသစ် မဖန်တီးတော့ဘူး။

---

# ExecutorService ဖန်တီးနည်း

အများဆုံးအသုံးများတာက

```java
ExecutorService executor =
        Executors.newFixedThreadPool(3);
```

ဒီမှာ

```
3
```

ဆိုတာ

Thread 3 ခုကိုပဲ အသုံးပြုမယ်လို့ ဆိုလိုတာပါ။

---

# Example 1

```java
import java.util.concurrent.*;

public class Main {

    public static void main(String[] args) {

        ExecutorService executor =
                Executors.newFixedThreadPool(3);

        executor.submit(() -> {
            System.out.println("Task 1");
        });

        executor.submit(() -> {
            System.out.println("Task 2");
        });

        executor.submit(() -> {
            System.out.println("Task 3");
        });

        executor.shutdown();
    }
}
```

Output

```
Task 1
Task 2
Task 3
```

Output အစဉ်က Thread Scheduling ပေါ်မူတည်ပြီး ပြောင်းနိုင်ပါတယ်။

---

# submit() ဆိုတာဘာလဲ?

```
submit()
```

က

ExecutorService ထဲကို Task ထည့်ပေးတာပါ။

```
executor.submit(task);
```

Task ကို Queue ထဲထည့်လိုက်တယ်။

Thread အလွတ်ရရင် Run လုပ်မယ်။

---

# execute() နဲ့ submit() ကွာခြားချက်

### execute()

```java
executor.execute(() -> {
    System.out.println("Hello");
});
```

- Runnable ပဲလက်ခံတယ်
    
- Return Value မရှိဘူး
    

---

### submit()

```java
Future<String> future =
executor.submit(() -> {
    return "Hello";
});
```

- Runnable လည်းရ
    
- Callable လည်းရ
    
- Future ပြန်ပေးတယ်
    

---

# Future ဆိုတာဘာလဲ?

Task တစ်ခုရဲ့ Result ကို နောက်မှယူဖို့ အသုံးပြုတာ။

```java
Future<Integer> future =
executor.submit(() -> {
    return 100;
});

System.out.println(future.get());
```

Output

```
100
```

`future.get()` က Task ပြီးတဲ့အထိ စောင့်ပါတယ်။

---

# Runnable vs Callable

Runnable

```java
Runnable r = () -> {
    System.out.println("Hello");
};
```

- Return မရှိ
    
- Exception မပစ်နိုင် (Checked Exception)
    

---

Callable

```java
Callable<Integer> c = () -> {
    return 50;
};
```

- Return ရှိ
    
- Exception ပစ်နိုင်
    

---

# shutdown()

အလုပ်ပြီးရင်

```java
executor.shutdown();
```

ရေးသင့်ပါတယ်။

ဒါက

> "Task အသစ်မလက်ခံတော့ဘူး။ ရှိပြီးသား Task တွေအကုန်ပြီးမှ ပိတ်မယ်။"

လို့ အဓိပ္ပါယ်ရပါတယ်။

---

# shutdownNow()

```java
executor.shutdownNow();
```

ဒါက

- Running Task ကို Interrupt လုပ်မယ်
    
- Waiting Task တွေကို Cancel လုပ်ဖို့ ကြိုးစားမယ်
    

အမြဲတမ်း ချက်ချင်းရပ်မယ်လို့ အာမမခံနိုင်ပါဘူး၊ Task က Interrupt ကို Handle လုပ်နိုင်မှ ရပ်နိုင်ပါတယ်။

---

# ExecutorService အမျိုးအစားများ

## 1. Fixed Thread Pool

```java
ExecutorService executor =
Executors.newFixedThreadPool(4);
```

- Thread 4 ခု အမြဲရှိ
    
- အများဆုံးအသုံးများ
    

ဥပမာ

```
Task 1 -> Thread1
Task 2 -> Thread2
Task 3 -> Thread3
Task 4 -> Thread4
Task 5 -> Waiting
```

---

## 2. Single Thread Executor

```java
ExecutorService executor =
Executors.newSingleThreadExecutor();
```

Thread တစ်ခုတည်း

```
Task1
↓

Task2
↓

Task3
```

Task တွေကို အစဉ်လိုက် (Sequential) Run လုပ်ပါတယ်။

---

## 3. Cached Thread Pool

```java
ExecutorService executor =
Executors.newCachedThreadPool();
```

- လိုသလောက် Thread အသစ်ဖန်တီးတယ်
    
- မလိုတော့ရင် Thread တွေကို ပြန်ဖျက်တယ်
    
- Short-lived Task များတဲ့အခါ အသုံးဝင်တယ်
    

---

## 4. Scheduled Thread Pool

```java
ScheduledExecutorService executor =
Executors.newScheduledThreadPool(2);
```

Delay နဲ့ Run လုပ်ချင်ရင်

```java
executor.schedule(() -> {
    System.out.println("Hello");
}, 5, TimeUnit.SECONDS);
```

5 စက္ကန့်ကြာမှ Run ပါမယ်။

---

# ExecutorService Lifecycle

```
Create
   |
   V
Submit Tasks
   |
   V
Running
   |
   V
shutdown()
   |
   V
Finished
```

---

# Real-World Example

ဥပမာ Web Server တစ်ခုမှာ User 100 ယောက် တစ်ပြိုင်နက် Request ပို့လာတယ်ဆိုပါစို့။

```
User1
User2
User3
...
User100
```

Server က

```
ExecutorService
```

ထဲကို Request တွေထည့်ပေးတယ်။

Thread Pool က

```
Thread1
Thread2
Thread3
...
Thread20
```

နဲ့ အလှည့်ကျ Request တွေကို Handle လုပ်ပေးပါတယ်။

ဒီလိုလုပ်ခြင်းအားဖြင့်

- Thread အသစ် 100 ခု မဖန်တီးရ
    
- Memory သက်သာ
    
- CPU အသုံးချမှုကောင်း
    
- Performance မြင့်
    
- Scalability ကောင်း
    

---

# Interview မှာ မေးလေ့ရှိတဲ့ မေးခွန်းများ

### 1. ExecutorService ဆိုတာဘာလဲ?

> Thread Pool ကိုအသုံးပြုပြီး Task တွေကို Execute လုပ်ပေးတဲ့ Java Concurrency Interface ဖြစ်ပါတယ်။

### 2. `execute()` နဲ့ `submit()` ဘာကွာလဲ?

- `execute()` → Return Value မရှိ (`Runnable` သာ)
    
- `submit()` → `Future` Return ပြန်ပေးတယ် (`Runnable`/`Callable` နှစ်မျိုးလုံး)
    

### 3. `shutdown()` နဲ့ `shutdownNow()` ကွာခြားချက်?

- `shutdown()` → Task အသစ်မလက်ခံတော့ဘဲ ရှိပြီးသား Task တွေပြီးမှ ပိတ်တယ်။
    
- `shutdownNow()` → Running Task တွေကို Interrupt လုပ်ပြီး Waiting Task တွေကို Cancel လုပ်ဖို့ ကြိုးစားတယ်။
    

### 4. Thread Pool သုံးခြင်းရဲ့ အကျိုးကျေးဇူး?

- Thread Reuse
    
- Memory သက်သာ
    
- CPU Scheduling ကောင်း
    
- Performance မြင့်
    
- Thread Management လွယ်ကူ
    

---

## မှတ်ထားသင့်တဲ့ အချက်

- `ExecutorService` = **Task တွေကို Thread Pool နဲ့ စီမံခန့်ခွဲပေးတဲ့ Interface**
    
- `Executors.newFixedThreadPool(n)` = Thread အရေအတွက် သတ်မှတ်ထားတဲ့ Pool
    
- `submit()` = Task တင်ပြီး `Future` ရနိုင်
    
- `execute()` = Task တင်ရုံ
    
- `shutdown()` = အလုပ်ပြီးမှ ပိတ်
    
- `Future.get()` = Task ရဲ့ Result ကို ရယူပြီး၊ မပြီးသေးရင် စောင့်ပေးမယ်
    
- `Callable` = Return Value ရှိ၊ `Runnable` = Return Value မရှိ
    

Interview အတွက်တော့ `ExecutorService`, `Thread Pool`, `Runnable`, `Callable`, `Future`, `shutdown()` နဲ့ `submit()` တို့ရဲ့ ကွာခြားချက်တွေကို နားလည်ထားရင် Java Concurrency အခြေခံမေးခွန်းအများစုကို ဖြေနိုင်ပါတယ်။