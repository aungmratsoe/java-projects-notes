အကောင်းဆုံးနည်းကတော့ **Real Life Example** နဲ့ ကြည့်ရင် Thread ကို ချက်ချင်းနားလည်သွားပါလိမ့်မယ်။

---

# Example 1: Single Thread (အလုပ်တစ်ခုပြီးမှ နောက်တစ်ခု)

ဥပမာ သင်က

1. Movie Download လုပ်မယ်
    
2. Music ဖွင့်မယ်
    

Single Thread ဆိုရင်

```text
Start

↓

Download Movie (10 sec)

↓

Play Music

↓

End
```

Movie Download ပြီးမှ Music ဖွင့်နိုင်တယ်။

### Java Code

```java
public class Main {

    public static void downloadMovie() {

        for (int i = 1; i <= 5; i++) {

            System.out.println("Downloading... " + i);

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }

        }

    }

    public static void playMusic() {

        System.out.println("Playing Music");

    }

    public static void main(String[] args) {

        downloadMovie();

        playMusic();

    }

}
```

### Output

```text
Downloading... 1
Downloading... 2
Downloading... 3
Downloading... 4
Downloading... 5

Playing Music
```

Music က Download ပြီးမှ စတယ်။

---

# Example 2: Multi Thread

ဒီတစ်ခါ Download ကို Thread အသစ်မှာလုပ်မယ်။

```java
class DownloadThread extends Thread {

    public void run() {

        for (int i = 1; i <= 5; i++) {

            System.out.println("Downloading... " + i);

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }

        }

    }

}

public class Main {

    public static void main(String[] args) {

        DownloadThread t = new DownloadThread();

        t.start();

        System.out.println("Playing Music");

    }

}
```

Output

```text
Playing Music

Downloading... 1
Downloading... 2
Downloading... 3
Downloading... 4
Downloading... 5
```

ဒါက

```text
Main Thread
    |
    ---> Playing Music

Download Thread
    |
    ---> Downloading...
```

Thread နှစ်ခု အတူတူအလုပ်လုပ်နေတာ။

---

# Example 3: Main Thread နဲ့ Child Thread

```java
class Worker extends Thread {

    public void run() {

        for(int i = 1; i <= 5; i++) {

            System.out.println("Worker : " + i);

            try {
                Thread.sleep(500);
            } catch(Exception e){}

        }

    }

}

public class Main {

    public static void main(String[] args) {

        Worker w = new Worker();

        w.start();

        for(int i = 1; i <= 5; i++) {

            System.out.println("Main : " + i);

            try {
                Thread.sleep(500);
            } catch(Exception e){}

        }

    }

}
```

Output တစ်မျိုး

```text
Main : 1
Worker : 1

Main : 2
Worker : 2

Main : 3
Worker : 3

Main : 4
Worker : 4

Main : 5
Worker : 5
```

ဒါပေမယ့်

Output က

```text
Worker : 1
Worker : 2
Main : 1
Main : 2
Worker : 3
Main : 3
```

ဒီလိုလည်း ဖြစ်နိုင်တယ်။

### ဘာကြောင့်လဲ?

CPU Scheduler က

> "ဘယ် Thread ကို အရင် Run ပေးရမလဲ"

ဆိုတာ ဆုံးဖြတ်တာကြောင့်။

---

# Example 4: run() vs start()

ဒီ Example က Interview မှာ အရမ်းမေးပါတယ်။

## run()

```java
class MyThread extends Thread {

    public void run() {

        System.out.println("Inside Thread");

    }

}

public class Main {

    public static void main(String[] args) {

        MyThread t = new MyThread();

        t.run();

        System.out.println("Main End");

    }

}
```

Output

```text
Inside Thread
Main End
```

Diagram

```text
Main Thread
     |
     |
     ---> run()
```

Thread အသစ်မဖန်တီးဘူး။

---

## start()

```java
MyThread t = new MyThread();

t.start();

System.out.println("Main End");
```

Output

```text
Main End

Inside Thread
```

သို့မဟုတ်

```text
Inside Thread

Main End
```

Diagram

```text
Main Thread
      |
      ---> Main End

Thread-1
      |
      ---> Inside Thread
```

ဒီတစ်ခါ Thread အသစ် Create ဖြစ်တယ်။

---

# Example 5: Bank Account (Race Condition)

ဒါက Professional Interview မှာ အရမ်းကြိုက်တဲ့ Example ပါ။

အစမှာ

```text
Balance = $1000
```

Thread-1

```text
Withdraw $500
```

Thread-2

```text
Withdraw $700
```

နှစ်ခုလုံး တစ်ချိန်တည်း Balance ကိုဖတ်တယ်။

```text
Thread-1 Read = 1000

Thread-2 Read = 1000
```

Thread-1

```text
1000 - 500 = 500
```

Thread-2

```text
1000 - 700 = 300
```

နောက်ဆုံး

```text
Balance = 300
```

ဖြစ်သွားတယ်။

အမှန်ဆို

```text
1000 - 500 - 700 = -200
```

ဖြစ်ရမှာ။

ဒီလို Data မှားတာကို

**Race Condition**

လို့ခေါ်ပါတယ်။

---

## synchronized ဖြင့် ဖြေရှင်းခြင်း

```java
class Bank {

    private int balance = 1000;

    public synchronized void withdraw(int amount) {

        if(balance >= amount) {

            balance -= amount;

            System.out.println(Thread.currentThread().getName()
                    + " withdrew " + amount);

            System.out.println("Balance = " + balance);

        } else {

            System.out.println("Not enough money");

        }

    }

}
```

`synchronized` ကြောင့်

```text
Thread-1

↓

Finish

↓

Thread-2
```

အလှည့်ကျ ဝင်ရတာဖြစ်လို့ Balance မမှားတော့ဘူး။

---

# Example 6: Restaurant (အလွယ်ဆုံး ဥပမာ)

စားသောက်ဆိုင်တစ်ဆိုင်ကို စဉ်းစားကြည့်ပါ။

```text
Restaurant (Process)
```

Chef (Thread-1)

```text
Cook Fried Rice
```

Chef (Thread-2)

```text
Cook Noodle
```

Waiter (Thread-3)

```text
Serve Customer
```

Cashier (Thread-4)

```text
Receive Payment
```

Diagram

```text
Restaurant (Process)
        |
 -------------------------
 |       |       |       |
Chef1   Chef2  Waiter  Cashier
```

Chef တစ်ယောက်တည်းဆိုရင်

```text
Fried Rice

↓

Noodle

↓

Soup

↓

Dessert
```

အချိန်ကြာတယ်။

Chef ၄ ယောက်ရှိရင်

```text
Fried Rice

Noodle

Soup

Dessert
```

တစ်ပြိုင်တည်းချက်နိုင်တယ်။

ဒါက **Multithreading** ရဲ့ အဓိက အားသာချက်ပါ။

---

# Example 7: YouTube (နေ့စဉ်သုံး App)

သင် YouTube ကြည့်နေတယ်ဆိုပါစို့။

```text
YouTube App (Process)
```

အထဲမှာ

```text
Video Thread
```

Video ပြတယ်။

```text
Audio Thread
```

အသံထွက်တယ်။

```text
Comment Thread
```

Comment တွေ Load လုပ်တယ်။

```text
Network Thread
```

Internet က Data ယူတယ်။

```text
Subtitle Thread
```

Subtitle ပြတယ်။

အားလုံးက အတူတူ အလုပ်လုပ်နေကြတာပါ။

---

## Thread ကို မှတ်မိလွယ်အောင်

|Concept|ဥပမာ|
|---|---|
|Process|Restaurant|
|Thread|Chef, Waiter, Cashier|
|Main Thread|Restaurant Manager|
|Child Thread|Chef 1, Chef 2|
|`start()`|Employee အသစ်ကို အလုပ်စခိုင်းခြင်း|
|`run()`|Manager ကိုယ်တိုင် အလုပ်ဝင်လုပ်ခြင်း (လူအသစ်မခန့်)|
|`sleep()`|ဝန်ထမ်း ခဏနားခြင်း|
|`join()`|"Chef အလုပ်ပြီးမှ Waiter စားပွဲတင်မယ်"|
|`synchronized`|မီးဖိုတစ်လုံးကို Chef တစ်ယောက်ချင်းစီ အလှည့်ကျ အသုံးပြုခြင်း|
|Race Condition|Chef နှစ်ယောက် တစ်မီးဖိုတည်းကို တစ်ပြိုင်တည်း လုသုံးပြီး အလုပ်ရှုပ်သွားခြင်း|

### အကြံပြုချက်

Thread ကို စတင်လေ့လာနေတယ်ဆိုရင် အောက်ပါအစီအစဉ်နဲ့ လက်တွေ့ရေးကြည့်ပါ။

1. `Thread` ကို `extends` လုပ်ပြီး `start()` သုံးပါ။
    
2. `Runnable` ကို `implements` လုပ်ပြီး `Thread` နဲ့တွဲသုံးပါ။
    
3. `sleep()` နဲ့ Thread တွေ အလှည့်ကျအလုပ်လုပ်တာကို ကြည့်ပါ။
    
4. `join()` ကို ထည့်ပြီး Output ဘယ်လိုပြောင်းလဲလဲ စမ်းကြည့်ပါ။
    
5. `synchronized` မရှိ/ရှိ Bank Account Example ကို Run ကြည့်ပြီး Race Condition ကို ကိုယ်တိုင်မြင်အောင် လေ့ကျင့်ပါ။
    

ဒီလို လက်တွေ့ Run ကြည့်ရင် Thread ရဲ့အလုပ်လုပ်ပုံကို ပိုပြီးရှင်းရှင်းလင်းလင်း နားလည်လာပါလိမ့်မယ်။