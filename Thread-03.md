ဒီ Example က `Thread.sleep()` ကိုသုံးပြီး **Main Thread** နဲ့ **Child Thread** က အလှည့်ကျ အလုပ်လုပ်တာကို မြင်နိုင်တဲ့ အကောင်းဆုံး Example ပါ။

---

# Example: sleep() နဲ့ Thread နှစ်ခု အလုပ်လုပ်ပုံ

```java
class MyThread extends Thread {

    @Override
    public void run() {

        for (int i = 1; i <= 5; i++) {

            System.out.println("Child Thread : " + i);

            try {
                Thread.sleep(1000);   // Sleep for 1 second
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

    }

}

public class Main {

    public static void main(String[] args) {

        MyThread thread = new MyThread();

        thread.start();

        for (int i = 1; i <= 5; i++) {

            System.out.println("Main Thread : " + i);

            try {
                Thread.sleep(1000);   // Sleep for 1 second
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

    }

}
```

---

# Output (တစ်မျိုး)

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

ဒါပေမယ့် Output က

```text
Child Thread : 1
Main Thread : 1
Main Thread : 2
Child Thread : 2
Child Thread : 3
Main Thread : 3
```

ဒီလိုလည်း ဖြစ်နိုင်ပါတယ်။

---

# Step-by-Step ဘာဖြစ်နေလဲ?

## Step 1

Program စတယ်။

```java
MyThread thread = new MyThread();
```

ဒီအချိန်မှာ Thread Object ပဲရှိသေးတယ်။

```text
State = NEW
```

---

## Step 2

```java
thread.start();
```

JVM က

```text
Main Thread

+

Child Thread
```

ဆိုပြီး Thread နှစ်ခု ဖြစ်လာတယ်။

---

## Step 3

Main Thread က

```java
System.out.println("Main Thread : 1");
```

ကို ထုတ်တယ်။

---

## Step 4

ပြီးတာနဲ့

```java
Thread.sleep(1000);
```

ရောက်သွားတယ်။

Main Thread က

```text
Sleeping...
```

ဆိုပြီး **၁ စက္ကန့် နားနေတယ်**။

---

## Step 5

Main Thread အိပ်နေတဲ့အချိန်မှာ

CPU က

```text
Child Thread
```

ကို အလုပ်လုပ်ခွင့်ပေးတယ်။

Child Thread က

```java
System.out.println("Child Thread : 1");
```

ကို ထုတ်တယ်။

---

## Step 6

Child Thread လည်း

```java
Thread.sleep(1000);
```

ရောက်သွားတယ်။

အခု

```text
Main Thread = Sleeping

Child Thread = Sleeping
```

---

## Step 7

၁ စက္ကန့်ပြည့်သွားရင်

Scheduler က

```text
Main Thread
```

ကို နိုးပေးနိုင်တယ်။

ဒါမှမဟုတ်

```text
Child Thread
```

ကို အရင်နိုးပေးနိုင်တယ်။

ဒါကြောင့် Output က အမြဲတူမနေဘူး။

---

# Timeline

```
Time

0 sec
-------------------------
Main  : 1
Child : 1

1 sec
-------------------------
Main  : 2
Child : 2

2 sec
-------------------------
Main  : 3
Child : 3

3 sec
-------------------------
Main  : 4
Child : 4

4 sec
-------------------------
Main  : 5
Child : 5
```

---

# CPU က ဘာလုပ်နေလဲ?

```
CPU
 │
 ├────────► Main Thread
 │
 ├────────► sleep()
 │
 ├────────► Child Thread
 │
 ├────────► sleep()
 │
 ├────────► Main Thread
 │
 ├────────► Child Thread
 │
 └────────► ...
```

**အရေးကြီးတာက `sleep()` က CPU ကို Lock မလုပ်ထားပါဘူး။**

Main Thread က အိပ်နေတဲ့အချိန်မှာ CPU က Child Thread ကို အလုပ်လုပ်ခွင့်ပေးပါတယ်။

---

# `sleep()` ကို မသုံးရင် ဘာဖြစ်မလဲ?

ဒီလိုရေးကြည့်ပါ။

```java
class MyThread extends Thread {

    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println("Child Thread : " + i);
        }
    }
}

public class Main {

    public static void main(String[] args) {

        MyThread thread = new MyThread();
        thread.start();

        for (int i = 1; i <= 5; i++) {
            System.out.println("Main Thread : " + i);
        }
    }
}
```

Output က

```text
Main Thread : 1
Main Thread : 2
Main Thread : 3
Main Thread : 4
Main Thread : 5
Child Thread : 1
Child Thread : 2
Child Thread : 3
Child Thread : 4
Child Thread : 5
```

ဒါမှမဟုတ်

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

ဖြစ်နိုင်ပါတယ်။

**ဘာကြောင့်လဲ?**

Thread တစ်ခုက `sleep()` မရှိဘဲ Loop ကို အရမ်းမြန်မြန်ပြီးသွားလို့ CPU Scheduler က အခြား Thread ကို ပြောင်းပေးဖို့ အခွင့်အရေးနည်းသွားပါတယ်။

---

## Interview Tip

**Q: `Thread.sleep(1000)` က Thread နှစ်ခုကို တစ်ပြိုင်တည်း အလုပ်လုပ်စေသလား?**

**Answer:**  
မဟုတ်ပါဘူး။

- `Thread.sleep(1000)` က **လက်ရှိ Thread ကိုသာ** ၁ စက္ကန့် ရပ်စေပါတယ်။
    
- အဲဒီအချိန်အတွင်း CPU က **အခြား Runnable Thread** တွေကို အလုပ်လုပ်ခွင့်ပေးနိုင်ပါတယ်။
    
- ဒါကြောင့် Thread တွေက အလှည့်ကျ (interleaving) အလုပ်လုပ်နေသလို မြင်ရတာဖြစ်ပါတယ်။