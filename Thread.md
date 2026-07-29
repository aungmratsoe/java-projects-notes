# Java Thread အကြောင်း အပြည့်အစုံ (မြန်မာလို)

Java မှာ **Thread** ဆိုတာ **Program တစ်ခုအတွင်းမှာ အလုပ်တစ်ခုကို သီးသန့်လုပ်ဆောင်နေတဲ့ execution path** တစ်ခု ဖြစ်ပါတယ်။

လွယ်လွယ်ပြောရရင်...

> **Process = အိမ်တစ်လုံး**
> 
> **Thread = အိမ်ထဲမှာ အလုပ်လုပ်နေတဲ့ လူတစ်ယောက်**

ဥပမာ Browser တစ်ခုကို ဖွင့်ထားတယ်ဆိုပါစို့။

Browser Process တစ်ခုအတွင်းမှာ

- UI Thread
    
- Download Thread
    
- Audio Thread
    
- Video Thread
    
- Network Thread
    

စသဖြင့် Thread အများကြီး အတူတူ အလုပ်လုပ်နေပါတယ်။

---

# Thread မရှိရင် ဘာဖြစ်မလဲ?

ဥပမာ

```java
public class Main {
    public static void main(String[] args) {

        downloadMovie();   // 20 sec

        System.out.println("Program Finished");
    }
}
```

Movie download က 20 seconds ကြာရင်

Program က

```
Downloading...

(20 seconds)

Program Finished
```

ဒီအချိန်မှာ Program က တခြားဘာမှ မလုပ်နိုင်ဘူး။

ဒါကို **Single Thread** လို့ခေါ်တယ်။

---

# Thread ရှိရင်

Download ကို Thread အသစ်မှာလုပ်မယ်။

Main Thread ကတော့

```
Downloading...

Program Finished
```

ဆိုပြီး ချက်ချင်း Continue လုပ်နိုင်တယ်။

Download ကတော့ နောက်ကနေ ဆက်လုပ်နေတယ်။

ဒါကို

**Multithreading**

လို့ခေါ်ပါတယ်။

---

# Thread Diagram

```
Process
│
├── Main Thread
│
├── Thread-1
│
├── Thread-2
│
└── Thread-3
```

Process တစ်ခုထဲမှာ Thread အများကြီး ရှိနိုင်ပါတယ်။

---

# Java Program စဖွင့်လိုက်တာနဲ့ Thread ဘယ်နှစ်ခုရှိလဲ?

Program စဖွင့်တာနဲ့

Java က

```
Main Thread
```

ကို အလိုအလျောက် ဖန်တီးပေးထားပြီးသား ဖြစ်ပါတယ်။

ဒီ

```java
public static void main(String[] args)
```

ဟာ Main Thread မှာ Run နေတာ ဖြစ်ပါတယ်။

---

# Thread Lifecycle

Thread တစ်ခုရဲ့ ဘဝကို ကြည့်ရအောင်။

```
NEW
 ↓
RUNNABLE
 ↓
RUNNING
 ↓
WAITING
 ↓
TIMED WAITING
 ↓
TERMINATED
```

---

## 1. NEW

Thread Object ကို Create လုပ်ပြီး

start() မခေါ်သေးတဲ့အချိန်။

```java
Thread t = new Thread();
```

State

```
NEW
```

---

## 2. RUNNABLE

```
t.start();
```

ခေါ်လိုက်တာနဲ့

Java Scheduler က

CPU ရတဲ့အချိန် Run ဖို့ စောင့်နေတယ်။

```
RUNNABLE
```

---

## 3. RUNNING

CPU ရသွားပြီ။

Thread အလုပ်လုပ်နေပြီ။

```
RUNNING
```

---

## 4. WAITING

Thread က

တခြား Thread ကိုစောင့်နေတယ်။

ဥပမာ

```
join()
```

---

## 5. TIMED_WAITING

အချိန်တစ်ခုစောင့်နေတယ်။

```
sleep(3000)
```

3 seconds

---

## 6. TERMINATED

အလုပ်ပြီးသွားပြီ။

```
Dead
```

---

# Thread ကို Create လုပ်နည်း (၂ မျိုး)

## Method (1)

Thread Class ကို Extend လုပ်ခြင်း

```java
class MyThread extends Thread {

    public void run() {
        System.out.println("Hello Thread");
    }

}
```

Main

```java
public class Main {

    public static void main(String[] args) {

        MyThread t = new MyThread();

        t.start();

    }

}
```

Output

```
Hello Thread
```

---

## Method (2)

Runnable Interface

ဒီနည်းက Professional Project တွေမှာ ပိုသုံးပါတယ်။

```java
class MyTask implements Runnable {

    public void run() {

        System.out.println("Running");

    }

}
```

Main

```java
public class Main {

    public static void main(String[] args) {

        MyTask task = new MyTask();

        Thread t = new Thread(task);

        t.start();

    }

}
```

Output

```
Running
```

---

# run() နဲ့ start() ကွာခြားချက်

ဒီ Interview မှာ အရမ်းမေးပါတယ်။

## run()

```java
t.run();
```

ဒီလိုခေါ်ရင်

Thread အသစ်မဖန်တီးဘူး။

Normal Method Call ပဲ ဖြစ်ပါတယ်။

```
Main Thread
    |
    ---> run()
```

---

## start()

```java
t.start();
```

ဒီလိုခေါ်မှ

Java က

Thread အသစ်တစ်ခု Create လုပ်ပေးပါတယ်။

```
Main Thread

Thread-1
```

အလုပ်နှစ်ခုကို အပြိုင်လုပ်နိုင်ပါတယ်။

---

# Example

```java
class MyThread extends Thread {

    public void run() {

        System.out.println("Thread Running");

    }

}

public class Main {

    public static void main(String[] args) {

        MyThread t = new MyThread();

        t.start();

        System.out.println("Main Thread");

    }

}
```

Output

```
Main Thread

Thread Running
```

သို့မဟုတ်

```
Thread Running

Main Thread
```

**အရေးကြီးချက်:** Output အစဉ်က မတည်ငြိမ်ပါဘူး။ ဘာကြောင့်လဲဆိုတော့ CPU Scheduler က ဘယ် Thread ကို အရင်အလုပ်လုပ်ခွင့်ပေးမလဲဆိုတာ ဆုံးဖြတ်တာကြောင့် ဖြစ်ပါတယ်။

---

# sleep()

Thread ကို

ခဏရပ်စေတယ်။

```java
Thread.sleep(3000);
```

3 Seconds နားတယ်။

Example

```java
public class Main {

    public static void main(String[] args) throws Exception {

        System.out.println("1");

        Thread.sleep(3000);

        System.out.println("2");

    }

}
```

Output

```
1

(3 sec)

2
```

---

# join()

Thread တစ်ခုကို စောင့်တယ်။

```java
t.join();
```

Example

```java
class MyThread extends Thread {

    public void run() {

        for(int i=1;i<=5;i++) {

            System.out.println(i);

        }

    }

}
```

Main

```java
MyThread t = new MyThread();

t.start();

t.join();

System.out.println("Finished");
```

Output

```
1

2

3

4

5

Finished
```

`join()` မရှိရင် `"Finished"` က အရင်ထွက်နိုင်ပါတယ်။

---

# Thread Priority

Priority

```
1

↓

10
```

Default

```
5
```

```java
t.setPriority(10);
```

ဒါပေမယ့် **Priority မြင့်တာနဲ့ CPU က အမြဲအရင် Run ပေးမယ်လို့ အာမမခံပါဘူး**။ Operating System Scheduler အပေါ် မူတည်ပါတယ်။

---

# currentThread()

လက်ရှိ Run နေတဲ့ Thread ကို ကြည့်တယ်။

```java
System.out.println(Thread.currentThread().getName());
```

Output

```
main
```

---

# Thread Name

```java
t.setName("Download");
```

```java
System.out.println(t.getName());
```

Output

```
Download
```

---

# Multiple Threads

```java
class A extends Thread {

    public void run() {

        for(int i=1;i<=5;i++)
            System.out.println("A");

    }

}

class B extends Thread {

    public void run() {

        for(int i=1;i<=5;i++)
            System.out.println("B");

    }

}
```

Main

```java
A a = new A();

B b = new B();

a.start();

b.start();
```

Output

```
A

B

A

A

B

B

A

B
```

Output အစဉ်က မတူနိုင်ပါဘူး။

---

# Thread Synchronization

Thread နှစ်ခုက Data တစ်ခုတည်းကို တစ်ပြိုင်တည်း ပြင်ရင် ပြဿနာဖြစ်နိုင်ပါတယ်။

ဥပမာ -

Bank Account Balance = 1000

Thread-1

```
Withdraw 500
```

Thread-2

```
Withdraw 700
```

Synchronization မရှိရင်

Balance မှားသွားနိုင်ပါတယ်။

ဒါကို **Race Condition** လို့ခေါ်ပါတယ်။

ဖြေရှင်းဖို့

```java
synchronized
```

ကို သုံးပါတယ်။

```java
public synchronized void withdraw() {

}
```

---

# Thread-safe ဆိုတာဘာလဲ?

Thread အများကြီး တစ်ချိန်တည်း အသုံးပြုခဲ့ရင်တောင် Data မမှားအောင် ဒီဇိုင်းလုပ်ထားတဲ့ Code ကို **Thread-safe** လို့ခေါ်ပါတယ်။

ဥပမာ -

```java
public synchronized void deposit() {

    balance += amount;

}
```

---

# Daemon Thread

Daemon Thread က Background Service Thread ဖြစ်ပါတယ်။

ဥပမာ

- Garbage Collector (GC)
    
- Auto Save
    
- Background Logger
    

```java
t.setDaemon(true);
```

Main Thread တွေ အားလုံးပြီးသွားရင် Daemon Thread တွေလည်း အလိုအလျောက် ရပ်သွားနိုင်ပါတယ်။

---

# Concurrency vs Parallelism

**Concurrency**

- Thread အများကြီးကို အလှည့်ကျ (interleaving) အလုပ်လုပ်စေခြင်း။
    
- CPU Core တစ်ခုတည်းပေါ်မှာလည်း ဖြစ်နိုင်ပါတယ်။
    

```
Thread A
↓

Thread B
↓

Thread A
↓

Thread C
```

**Parallelism**

- CPU Core များစွာပေါ်မှာ Thread များကို တကယ်တမ်း တစ်ချိန်တည်း အလုပ်လုပ်စေခြင်း။
    

```
Core 1 → Thread A

Core 2 → Thread B

Core 3 → Thread C
```

---

# Thread Pool (အရေးကြီး)

Thread အသစ်ကို အမြဲ Create လုပ်နေရင် Memory နဲ့ Performance ကုန်ကျပါတယ်။

ဒါကြောင့် Java မှာ **Thread Pool** ကို အသုံးပြုပါတယ်။

```java
ExecutorService service = Executors.newFixedThreadPool(4);
```

ဆိုလိုတာက

```
Thread-1

Thread-2

Thread-3

Thread-4
```

ကို တစ်ခါတည်း ဖန်တီးထားပြီး အလုပ်အသစ်တွေကို အဲဒီ Thread တွေဆီ ခွဲပေးပါတယ်။

**အားသာချက်များ**

- Thread ဖန်တီး/ဖျက်သိမ်းမှု လျော့နည်းသည်။
    
- Performance ကောင်းသည်။
    
- Resource အသုံးပြုမှုကို ထိန်းချုပ်နိုင်သည်။
    
- Server Application များတွင် အလွန်အသုံးများသည်။
    

---

# Interview မှာ အမေးများတဲ့ မေးခွန်းများ

1. **Thread ဆိုတာ ဘာလဲ?**
    
    - Process တစ်ခုအတွင်းရှိ အလုပ်လုပ်ဆောင်သည့် execution path တစ်ခုဖြစ်သည်။
        
2. **Process နဲ့ Thread ကွာခြားချက်?**
    
    - Process တစ်ခုစီမှာ ကိုယ်ပိုင် Memory Space ရှိသည်။ Thread များကတော့ Process တစ်ခုတည်း၏ Memory ကို မျှဝေ (share) အသုံးပြုကြသည်။
        
3. **Thread ကို ဘယ်လို Create လုပ်မလဲ?**
    
    - `Thread` class ကို `extends` လုပ်ခြင်း၊ `Runnable` interface ကို `implements` လုပ်ခြင်း (သို့) ခေတ်သစ် Java မှာ Lambda ဖြင့် `Runnable` အသုံးပြုခြင်း။
        
4. **`start()` နဲ့ `run()` ကွာခြားချက်?**
    
    - `start()` က Thread အသစ်ဖန်တီးပြီး `run()` ကို Thread အသစ်ထဲမှာ လုပ်ဆောင်စေသည်။ `run()` ကို တိုက်ရိုက်ခေါ်ရင် သာမန် Method Call ပဲ ဖြစ်သည်။
        
5. **`sleep()` နဲ့ `join()` က ဘာလုပ်တာလဲ?**
    
    - `sleep()` က လက်ရှိ Thread ကို ခဏရပ်စေသည်။
        
    - `join()` က အခြား Thread တစ်ခု အလုပ်ပြီးသည်အထိ စောင့်စေသည်။
        
6. **`synchronized` ကို ဘာကြောင့် သုံးတာလဲ?**
    
    - Shared Data များကို Thread အများကြီးက တစ်ပြိုင်တည်း အသုံးပြုရာမှာ Race Condition မဖြစ်စေရန်။
        
7. **Thread Pool ရဲ့ အကျိုးကျေးဇူးက ဘာလဲ?**
    
    - Thread များကို ပြန်လည်အသုံးပြုခြင်းဖြင့် Performance ကောင်းစေပြီး Resource အသုံးပြုမှုကို လျှော့ချပေးသည်။
        

---

## အကျဉ်းချုပ်

- **Thread** = Process တစ်ခုအတွင်းရှိ အလုပ်လုပ်နေသော execution path။
    
- Java Program တိုင်းမှာ **Main Thread** ရှိပြီးသားဖြစ်သည်။
    
- Thread အသစ်ဖန်တီးရန် `start()` ကို အသုံးပြုရသည်။
    
- `run()` ကို တိုက်ရိုက်ခေါ်ခြင်းသည် Thread အသစ် မဖန်တီးပေးပါ။
    
- Thread များသည် Memory ကို မျှဝေသုံးသောကြောင့် Race Condition ဖြစ်နိုင်ပြီး `synchronized` ဖြင့် ကာကွယ်နိုင်သည်။
    
- Production Application များတွင် `ExecutorService` နှင့် **Thread Pool** ကို အသုံးပြုခြင်းက ပိုမိုကောင်းမွန်သော နည်းလမ်းဖြစ်သည်။

---
