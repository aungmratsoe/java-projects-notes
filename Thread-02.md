ဟုတ်ပါတယ်။ Professional Project တွေမှာ **`Runnable` ကို `implements` လုပ်ပြီး `Thread` နဲ့တွဲသုံးတဲ့နည်း** ကို ပိုပြီးအသုံးများပါတယ်။

---

# Example 1: Runnable + Thread (Basic Example)

```java
class MyTask implements Runnable {

    @Override
    public void run() {

        for (int i = 1; i <= 5; i++) {

            System.out.println("Child Thread : " + i);

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

    }

}

public class Main {

    public static void main(String[] args) {

        // Create Runnable object
        MyTask task = new MyTask();

        // Pass Runnable object to Thread
        Thread thread = new Thread(task);

        // Start new Thread
        thread.start();

        // Main Thread
        for (int i = 1; i <= 5; i++) {

            System.out.println("Main Thread : " + i);

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

    }

}
```

---

# Output (Example)

```text
Main Thread : 1
Child Thread : 1
Main Thread : 2
Child Thread : 2
Main Thread : 3
Child Thread : 3
Main Thread : 4
Child Thread : 4
Main Thread : 5
Child Thread : 5
```

သို့မဟုတ်

```text
Child Thread : 1
Main Thread : 1
Child Thread : 2
Main Thread : 2
...
```

Output အစဉ်က CPU Scheduler ပေါ်မူတည်လို့ အမြဲတူမနေပါဘူး။

---

# Step-by-Step Explanation

## Step 1: Runnable ကို implements လုပ်ခြင်း

```java
class MyTask implements Runnable {
```

`Runnable` ဟာ **Interface** ဖြစ်ပါတယ်။

Interface ကို `implements` လုပ်ရင်

```java
public void run()
```

ကို မဖြစ်မနေ ရေးရပါတယ်။

---

## Step 2: run() Method

```java
@Override
public void run() {
    System.out.println("Child Thread");
}
```

ဒီ `run()` ထဲက Code တွေက **Child Thread** မှာ အလုပ်လုပ်မယ့် Code တွေပါ။

---

## Step 3: Runnable Object Create

```java
MyTask task = new MyTask();
```

ဒီအချိန်မှာ

```text
task
```

ဟာ Runnable Object ပဲ ဖြစ်သေးတယ်။

**Thread မဟုတ်သေးပါဘူး။**

---

## Step 4: Thread Object Create

```java
Thread thread = new Thread(task);
```

ဒီလိုရေးလိုက်တော့

```text
Thread
   │
   ▼
Runnable (task)
```

ဆိုပြီး Thread က Runnable Object ကို အသုံးပြုဖို့ ပြင်ထားတာ ဖြစ်ပါတယ်။

---

## Step 5: Start Thread

```java
thread.start();
```

ဒီတစ်ကြောင်းက

Thread အသစ်ကို Create လုပ်ပြီး

```java
task.run();
```

ကို **အလိုအလျောက်** ခေါ်ပေးပါတယ်။

---

# Diagram

```text
                MyTask
          implements Runnable
                  │
                  ▼
             run() Method
                  ▲
                  │
Thread thread = new Thread(task);
                  │
                  ▼
            thread.start()
                  │
                  ▼
         JVM calls run()
```

---

# Execution Flow

```text
Program Start
      │
      ▼
Create MyTask Object
      │
      ▼
Create Thread Object
      │
      ▼
thread.start()
      │
      ├────────────► Child Thread
      │                  │
      │                  ▼
      │              run()
      │
      ▼
Main Thread continues
```

---

# Real Life Example

Restaurant ကို ဥပမာယူကြည့်ရအောင်။

```text
Chef = Runnable
Kitchen = Thread
```

Chef က

```text
"ငါ ဘယ်လိုချက်မလဲ"
```

ဆိုတဲ့ Skill (`run()`) ကိုပဲ သိတယ်။

Kitchen က

```text
"Chef ကို အလုပ်လုပ်ခွင့်ပေးမယ်"
```

ဆိုတဲ့ Thread ဖြစ်တယ်။

```text
Chef (Runnable)
        │
        ▼
Thread (Kitchen)
        │
        ▼
start()
        │
        ▼
Chef Starts Cooking
```

Chef တစ်ယောက်တည်းက Kitchen အမျိုးမျိုးမှာ အလုပ်လုပ်နိုင်သလို၊ `Runnable` တစ်ခုကို `Thread` နဲ့တွဲပြီး အသုံးပြုတာဖြစ်ပါတယ်။

---

# Example 2: Lambda Expression (Java 8+)

Professional Project တွေမှာ `Runnable` ကို Class မရေးဘဲ Lambda နဲ့ ရေးတာလည်း အရမ်းများပါတယ်။

```java
public class Main {

    public static void main(String[] args) {

        Runnable task = () -> {

            for (int i = 1; i <= 5; i++) {

                System.out.println("Child Thread : " + i);

                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }

        };

        Thread thread = new Thread(task);
        thread.start();

        for (int i = 1; i <= 5; i++) {
            System.out.println("Main Thread : " + i);
        }
    }
}
```

---

# `extends Thread` vs `implements Runnable`

|`extends Thread`|`implements Runnable`|
|---|---|
|`class MyThread extends Thread`|`class MyTask implements Runnable`|
|Thread ကို တိုက်ရိုက် Inheritance လုပ်တယ်|Runnable Interface ကို Implement လုပ်တယ်|
|Class တစ်ခုက `Thread` ကို Extend လုပ်ပြီးရင် တခြား Class ကို မထပ် Extend နိုင်တော့ဘူး|Class က တခြား Class ကိုလည်း Extend လုပ်နိုင်သေးတယ်|
|အသေးစား Example တွေအတွက် လွယ်တယ်|Professional Project တွေမှာ ပိုအသုံးများတယ်|
|Thread နဲ့ Task တွဲနေတယ်|**Task (Runnable)** နဲ့ **Thread** ကို ခွဲထားတဲ့ Design ဖြစ်လို့ ပိုကောင်းတယ်|

## Interview Tip

Interview မှာ **"ဘယ်နည်းကို ပိုသုံးသင့်လဲ?"** လို့ မေးရင်

**အဖြေ**

> Professional Java Application တွေမှာ `Runnable` ကို `implements` လုပ်တဲ့နည်းကို ပိုအသုံးများပါတယ်။ ဘာကြောင့်လဲဆိုတော့ **Task** နဲ့ **Thread** ကို ခွဲထားနိုင်ပြီး၊ Class Inheritance ကိုလည်း ဆက်လက်အသုံးပြုနိုင်တဲ့အတွက် ပိုပြီး Flexible ဖြစ်ပါတယ်။