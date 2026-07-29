ဒီ Example က **Java Thread Interview** မှာ အမေးအများဆုံး Example ဖြစ်ပါတယ်။

ဒီ Example ကို Run ကြည့်ရင် **Race Condition** ဆိုတာကို ကိုယ်တိုင်မြင်နိုင်ပါတယ်။

---

# Example 1: `synchronized` မရှိ (Race Condition ဖြစ်နိုင်)

## Bank.java

```java
class BankAccount {

    private int balance = 1000;

    public void withdraw(int amount) {

        if (balance >= amount) {

            System.out.println(Thread.currentThread().getName()
                    + " is withdrawing...");

            // Simulate processing time
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }

            balance = balance - amount;

            System.out.println(Thread.currentThread().getName()
                    + " Remaining Balance = " + balance);

        } else {

            System.out.println(Thread.currentThread().getName()
                    + " Not enough balance.");

        }

    }

}
```

---

## Customer.java

```java
class Customer extends Thread {

    private BankAccount account;

    public Customer(BankAccount account, String name) {

        this.account = account;
        setName(name);

    }

    @Override
    public void run() {

        account.withdraw(700);

    }

}
```

---

## Main.java

```java
public class Main {

    public static void main(String[] args) {

        BankAccount account = new BankAccount();

        Customer c1 = new Customer(account, "Alice");
        Customer c2 = new Customer(account, "Bob");

        c1.start();
        c2.start();

    }

}
```

---

# Output (တစ်မျိုး)

```text
Alice is withdrawing...
Bob is withdrawing...

Alice Remaining Balance = 300
Bob Remaining Balance = -400
```

ဒါမှမဟုတ်

```text
Bob Remaining Balance = 300
Alice Remaining Balance = -400
```

---

# ဘာကြောင့် `-400` ဖြစ်သွားတာလဲ?

အစမှာ

```text
Balance = 1000
```

Alice

```text
Balance ကိုဖတ်တယ် = 1000
```

Thread က

```java
Thread.sleep(1000);
```

ရောက်သွားတယ်။

---

ဒီအချိန်

Bob လည်း

```text
Balance = 1000
```

ကို ဖတ်လိုက်တယ်။

---

ဒါဆို

```text
Alice → 1000 - 700 = 300

Bob   → 1000 - 700 = 300
```

နှစ်ယောက်လုံးက **တူညီတဲ့ Balance ကို ဖတ်ထားတာ** ဖြစ်တယ်။

တစ်ယောက်က Update လုပ်ပြီး နောက်တစ်ယောက်ကလည်း အဟောင်းတန်ဖိုးကို အခြေခံပြီး Update လုပ်တဲ့အတွက် Data မှားသွားနိုင်ပါတယ်။

ဒါကို

> **Race Condition**

လို့ခေါ်ပါတယ်။

---

# Diagram

```text
Balance = 1000

        │
        │
────────┼──────────
        │
     Alice
        │
 Read = 1000
        │
 sleep()
        │
────────┼──────────
        │
      Bob
        │
 Read = 1000
        │
 sleep()

Alice writes 300

Bob writes 300 (or invalid update)
```

Thread နှစ်ခုက Data တစ်ခုတည်းကို **တစ်ချိန်တည်း ဝင်သုံးနေတာ** ဖြစ်ပါတယ်။

---

# Example 2: `synchronized` သုံးပြီး ပြင်မယ်

```java
class BankAccount {

    private int balance = 1000;

    public synchronized void withdraw(int amount) {

        if (balance >= amount) {

            System.out.println(Thread.currentThread().getName()
                    + " is withdrawing...");

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
            }

            balance = balance - amount;

            System.out.println(Thread.currentThread().getName()
                    + " Remaining Balance = " + balance);

        } else {

            System.out.println(Thread.currentThread().getName()
                    + " Not enough balance.");

        }

    }

}
```

**ပြောင်းတာ တစ်ကြောင်းတည်းပါ။**

```java
public synchronized void withdraw(int amount)
```

---

# Output

```text
Alice is withdrawing...

Alice Remaining Balance = 300

Bob Not enough balance.
```

ဒါမှမဟုတ်

```text
Bob is withdrawing...

Bob Remaining Balance = 300

Alice Not enough balance.
```

---

# ဘာကြောင့် အခုမှန်သွားတာလဲ?

`synchronized` က

```text
Door Lock
```

တစ်ခုလို အလုပ်လုပ်ပါတယ်။

Alice ဝင်နေတုန်း

```text
LOCKED
```

Bob ဝင်လို့ မရဘူး။

---

Diagram

```text
BankAccount

      LOCK
       │
 ┌─────┴─────┐
 │           │
Alice      Bob
 │           │
 │<--using-->│ Waiting
 │           │
 Unlock
             │
          Bob enters
```

---

# Real Life Example

ATM တစ်လုံးရှိတယ်။

```text
ATM
```

Customer နှစ်ယောက်

```text
Alice

Bob
```

ATM ထဲကို

တစ်ချိန်တည်း

ဝင်လို့ရလား?

မရဘူး။

တစ်ယောက်ပြီးမှ

နောက်တစ်ယောက် ဝင်ရတယ်။

ATM က

```text
synchronized
```

လိုမျိုး Lock လုပ်ထားတာပါ။

---

# Thread Flow

### synchronized မရှိ

```text
Alice
   │
Read Balance =1000
   │
sleep()

Bob
   │
Read Balance =1000
   │
sleep()

Alice Update

Bob Update
```

နှစ်ယောက်လုံး တစ်ပြိုင်တည်း ဝင်နိုင်တယ်။

---

### synchronized ရှိ

```text
Alice
   │
Lock
   │
Read
   │
Update
   │
Unlock

Bob
   │
Lock
   │
Read
   │
Update
```

အလှည့်ကျပဲ ဝင်နိုင်တယ်။

---

# Professional Example

Bank System

```text
Transfer Money
```

Inventory System

```text
Update Stock
```

Ticket Booking

```text
Reserve Seat
```

Online Shopping

```text
Buy Product
```

ဒီလို System တွေအားလုံးမှာ Shared Data ရှိတဲ့အတွက် `synchronized` (သို့မဟုတ် `Lock`, `ReentrantLock`, Database Transaction စတဲ့ Concurrency Control နည်းလမ်းတွေ) ကို အသုံးပြုကြပါတယ်။

---

# Interview မှတ်ထားရန်

**Race Condition ဆိုတာ ဘာလဲ?**

> Thread နှစ်ခု (သို့) ပိုများတဲ့ Thread တွေက Shared Data တစ်ခုကို တစ်ပြိုင်တည်း ဖတ်/ရေး (read/write) လုပ်တဲ့အတွက် Program Result က မတည်ငြိမ်ဘဲ မှားယွင်းသွားနိုင်တဲ့ အခြေအနေကို **Race Condition** လို့ခေါ်ပါတယ်။

**Race Condition ကို ဘယ်လိုကာကွယ်မလဲ?**

- `synchronized`
    
- `Lock` (`ReentrantLock`)
    
- Atomic Classes (`AtomicInteger`)
    
- Concurrent Collections
    
- Database Transactions (လိုအပ်သလို)
    

---

## သင်ကိုယ်တိုင် စမ်းကြည့်ရန်

`Thread.sleep(1000);` ကို

```java
Thread.sleep(5000);
```

ပြောင်းပြီး Run ကြည့်ပါ။

ဒီလိုလုပ်လိုက်ရင် Thread နှစ်ခု အတူတူ `withdraw()` ထဲရောက်နိုင်တဲ့ အချိန်ပိုများလာတာကြောင့် Race Condition ကို ပိုပြီးမြင်လွယ်ပါလိမ့်မယ်။

**Tip:** Output က Run တိုင်း တူချင်မှတူမယ်။ Race Condition ရဲ့ သဘောတရားကိုယ်တိုင်က Timing ပေါ်မူတည်တဲ့ Non-deterministic Behavior ဖြစ်တာကြောင့်ပါ။