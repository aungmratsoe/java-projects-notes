`join()` က Java Thread မှာ **အရေးကြီးဆုံး Method** တွေထဲက တစ်ခုပါ။

**အဓိပ္ပာယ်ကတော့**

> **"ဒီ Thread အလုပ်ပြီးတဲ့အထိ စောင့်ပါ"** လို့ ပြောတာပါ။

---

# Example 1: `join()` မရှိတဲ့ Code

```java
class MyThread extends Thread {

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

        MyThread thread = new MyThread();

        thread.start();

        System.out.println("Main Thread Finished");

    }

}
```

---

## Output

```text
Main Thread Finished
Child Thread : 1
Child Thread : 2
Child Thread : 3
Child Thread : 4
Child Thread : 5
```

သို့မဟုတ်

```text
Child Thread : 1
Main Thread Finished
Child Thread : 2
Child Thread : 3
...
```

ဘာကြောင့်လဲ?

`thread.start();`

ပြီးတာနဲ့

Main Thread က

```java
System.out.println("Main Thread Finished");
```

ကို ဆက်လုပ်သွားတယ်။

Child Thread ကို **မစောင့်ဘူး**။

---

# Flow Diagram (`join()` မရှိ)

```text
Main Thread
     │
     ├── thread.start()
     │
     ├── Main Thread Finished
     │
     ▼
Program End

Child Thread
     │
     ├── 1
     ├── 2
     ├── 3
     ├── 4
     └── 5
```

Main Thread က Child Thread ကို မစောင့်ပါ။

---

# Example 2: `join()` ပါတဲ့ Code

```java
class MyThread extends Thread {

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

    public static void main(String[] args) throws InterruptedException {

        MyThread thread = new MyThread();

        thread.start();

        thread.join();      // Wait for Child Thread

        System.out.println("Main Thread Finished");

    }

}
```

---

# Output

```text
Child Thread : 1
Child Thread : 2
Child Thread : 3
Child Thread : 4
Child Thread : 5
Main Thread Finished
```

ဒီတစ်ခါ

Main Thread က

```java
thread.join();
```

ရောက်တဲ့အချိန်

ရပ်သွားတယ်။

Child Thread ပြီးတဲ့အထိ

စောင့်နေတယ်။

ပြီးမှ

```java
System.out.println("Main Thread Finished");
```

ကို Run တယ်။

---

# Flow Diagram (`join()` ပါ)

```text
Main Thread
     │
     ├── thread.start()
     │
     ├── thread.join()
     │      ▲
     │      │ Waiting...
     │      │
     └──────┘
            │
            ▼
Main Thread Finished

Child Thread
     │
     ├── 1
     ├── 2
     ├── 3
     ├── 4
     └── 5
```

---

# Timeline

### `join()` မရှိ

```text
Time

0 sec
-------------------
Main Start

Child 1

Main Finished

1 sec
-------------------
Child 2

2 sec
-------------------
Child 3

3 sec
-------------------
Child 4

4 sec
-------------------
Child 5
```

Main Thread က စောင့်မနေဘူး။

---

### `join()` ပါ

```text
Time

0 sec
-------------------
Main Start

Child 1

1 sec
-------------------
Child 2

2 sec
-------------------
Child 3

3 sec
-------------------
Child 4

4 sec
-------------------
Child 5

5 sec
-------------------
Main Finished
```

Main Thread က Child Thread ပြီးတဲ့အထိ စောင့်နေတယ်။

---

# Real Life Example

Restaurant မှာ

```text
Chef = Child Thread

Waiter = Main Thread
```

### `join()` မရှိ

```text
Chef starts cooking

↓

Waiter serves food ❌

↓

Chef finishes cooking
```

Waiter က စောင့်မနေဘဲ စားပွဲတင်ဖို့ သွားတယ်။

---

### `join()` ပါ

```text
Chef starts cooking

↓

Waiter waits

↓

Chef finishes cooking

↓

Waiter serves food ✅
```

ဒါက `join()` ရဲ့ အဓိပ္ပာယ်ပါ။

---

# Example 3 (Interview Favorite)

```java
class DownloadThread extends Thread {

    @Override
    public void run() {

        System.out.println("Downloading File...");

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Download Complete");

    }

}

public class Main {

    public static void main(String[] args) throws InterruptedException {

        DownloadThread download = new DownloadThread();

        download.start();

        download.join();   // Wait until download completes

        System.out.println("Opening File...");

    }

}
```

### Output

```text
Downloading File...
(5 seconds later)
Download Complete
Opening File...
```

**ဘာကြောင့် `join()` သုံးတာလဲ?**

File မပြီးသေးခင်

```text
Opening File...
```

လုပ်လို့မရဘူး။

ဒါကြောင့်

```java
download.join();
```

ကို သုံးပြီး Download ပြီးတဲ့အထိ စောင့်ခိုင်းတာဖြစ်ပါတယ်။

---

# Interview မှတ်ထားရန်

|Method|အလုပ်လုပ်ပုံ|
|---|---|
|`start()`|Thread အသစ်တစ်ခု ဖန်တီးပြီး `run()` ကို စတင်စေတယ်|
|`sleep(1000)`|**လက်ရှိ Thread** ကို 1 second အိပ်စေတယ်|
|`join()`|**အခြား Thread** အလုပ်ပြီးတဲ့အထိ စောင့်တယ်|

### Interview Question

**Q:** `join()` ကို ဘာကြောင့် သုံးတာလဲ?

**A:** `join()` ကို Thread တစ်ခုရဲ့ အလုပ်ပြီးဆုံးတာကို စောင့်ချင်တဲ့အခါ သုံးပါတယ်။ ဥပမာ Download ပြီးမှ File ဖွင့်ချင်တာ၊ Data Load ပြီးမှ Report ထုတ်ချင်တာ၊ သို့မဟုတ် Child Thread တွေအားလုံး ပြီးမှ Main Thread က နောက်အလုပ်ဆက်လုပ်ချင်တဲ့အခါ `join()` ကို အသုံးပြုပါတယ်။