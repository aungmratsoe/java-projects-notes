အောက်က Example က **`Thread` class ကို `extends` လုပ်ပြီး `start()` ကို အသုံးပြုတဲ့ အခြေခံအကျဆုံး Example** ဖြစ်ပါတယ်။

---

# Example 1: Thread ကို `extends` လုပ်ပြီး `start()`

```java
class MyThread extends Thread {

    @Override
    public void run() {

        for (int i = 1; i <= 5; i++) {

            System.out.println("Child Thread : " + i);

            try {
                Thread.sleep(1000); // 1 second
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

    }
}

public class Main {

    public static void main(String[] args) {

        MyThread thread = new MyThread();

        thread.start();   // Create a new thread

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

Output က အမြဲတူမနေပါဘူး။

ဥပမာ

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

ဒါမှမဟုတ်

```text
Child Thread : 1
Main Thread : 1
Child Thread : 2
Main Thread : 2
Main Thread : 3
Child Thread : 3
Main Thread : 4
Child Thread : 4
Child Thread : 5
Main Thread : 5
```

နှစ်မျိုးလုံး မှန်ပါတယ်။

---

# Line by Line Explanation

## Step 1

```java
class MyThread extends Thread
```

`MyThread` ဆိုတဲ့ Class ကို `Thread` Class ကနေ **Inheritance** လုပ်ထားပါတယ်။

```text
Thread
   ↑
MyThread
```

ဒါကြောင့် `MyThread` ဟာ `Thread` ရဲ့ Method တွေ (`start()`, `sleep()`, `getName()`...) ကို အသုံးပြုနိုင်ပါတယ်။

---

## Step 2

```java
@Override
public void run()
```

`run()` Method က **Thread စတင်အလုပ်လုပ်တဲ့နေရာ** ဖြစ်ပါတယ်။

```java
public void run() {
    System.out.println("Hello");
}
```

ဆိုရင် Thread စရင် `"Hello"` ကို ထုတ်ပေးမယ်။

---

## Step 3

```java
MyThread thread = new MyThread();
```

ဒီအချိန်မှာ **Thread ကိုပဲ Create လုပ်ထားတာ** ဖြစ်ပြီး **မစသေးပါဘူး**။

```text
Thread State

NEW
```

---

## Step 4

```java
thread.start();
```

ဒီတစ်ကြောင်းက အရေးအကြီးဆုံးပါ။

Java က

```text
Main Thread
```

အပြင်

```text
Child Thread
```

အသစ်တစ်ခုကို Create လုပ်ပြီး

```java
run()
```

ကို အလိုအလျောက် ခေါ်ပေးပါတယ်။

Diagram

```text
Main Thread
      |
      |
      +----------------------+
                             |
                       Child Thread
                             |
                           run()
```

---

## Step 5

Main Thread က

```java
for(int i=1; i<=5; i++)
```

ကို ဆက်လုပ်နေတယ်။

Child Thread ကလည်း

```java
run()
```

ထဲက Loop ကို လုပ်နေတယ်။

ဒါကြောင့်

နှစ်ခုလုံး အတူတူ Run နေတာပါ။

---

# Thread Flow

```text
Program Start
      │
      ▼
Main Thread Starts
      │
      ▼
thread.start()
      │
      ├──────────────► Child Thread Created
      │                     │
      │                     ▼
      │                run() executes
      │
      ▼
Main Thread continues
```

---

# `start()` ကို `run()` နဲ့ အစားထိုးရင်?

ဒီလိုရေးရင်

```java
thread.run();
```

Output

```text
Child Thread : 1
Child Thread : 2
Child Thread : 3
Child Thread : 4
Child Thread : 5
Main Thread : 1
Main Thread : 2
Main Thread : 3
Main Thread : 4
Main Thread : 5
```

ဘာကြောင့်လဲ?

`run()` ကို တိုက်ရိုက်ခေါ်တာက **ပုံမှန် Method Call** ပဲဖြစ်လို့ Thread အသစ် **မဖန်တီးပါဘူး**။

Diagram

```text
Main Thread
     │
     ▼
run()
     │
     ▼
Loop Finished
     │
     ▼
Main Loop Starts
```

---

# `start()` သုံးရင်

```java
thread.start();
```

Diagram

```text
                Main Thread
               /           \
              /             \
             ▼               ▼
      Main Loop         Child Loop
```

Thread နှစ်ခုလုံး တစ်ပြိုင်တည်း အလုပ်လုပ်နိုင်ပါတယ်။

---

## Interview မှာ အမေးများတဲ့ မေးခွန်း

**Q: `start()` က `run()` ကို ဘယ်သူခေါ်တာလဲ?**

**Answer:**  
`start()` ကို သင်ခေါ်လိုက်တဲ့အခါ JVM က Thread အသစ်တစ်ခု ဖန်တီးပြီး အဲဒီ Thread ထဲမှာ `run()` method ကို **အလိုအလျောက်** ခေါ်ပေးပါတယ်။ `run()` ကို သင်ကိုယ်တိုင် မခေါ်သင့်ပါဘူး။

ဒီအချက်ကို နားလည်ထားရင် `start()` နဲ့ `run()` ရဲ့ ကွာခြားချက်ကို ရှင်းရှင်းလင်းလင်း သဘောပေါက်လာပါလိမ့်မယ်။