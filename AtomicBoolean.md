# Java `AtomicBoolean` ဆိုတာဘာလဲ? (မြန်မာလို အသေးစိတ်ရှင်းပြချက်)

`AtomicBoolean` ဆိုတာ Java ရဲ့ **`java.util.concurrent.atomic` package** ထဲမှာရှိတဲ့ Class တစ်ခုဖြစ်ပြီး **Thread များစွာက တစ်ချိန်တည်းမှာ Boolean (`true` / `false`) တန်ဖိုးကို လုံခြုံစွာ (Thread-safe) ဖတ်/ပြင်နိုင်အောင်** ပြုလုပ်ပေးပါတယ်။

ပုံမှန် `boolean` ကို Thread အများကြီးက တစ်ပြိုင်နက် အသုံးပြုရင် **Race Condition** ဖြစ်နိုင်ပါတယ်။ `AtomicBoolean` ကတော့ Lock (`synchronized`) မသုံးဘဲ CPU ရဲ့ Atomic Operation ကို အသုံးပြုပြီး Thread-safe ဖြစ်အောင် လုပ်ပေးပါတယ်။

---

# Package

```java
import java.util.concurrent.atomic.AtomicBoolean;
```

---

# Atomic ဆိုတာဘာလဲ?

**Atomic** ဆိုတာ

> "Operation တစ်ခုကို အလယ်မှာ Thread တစ်ခုက ဝင်မနှောင့်ယှက်နိုင်ဘဲ အကုန်ပြီးမှ ပြီးသွားတဲ့ Operation"

ကို ဆိုလိုပါတယ်။

ဥပမာ

```
Thread A
    |
    |---- Update Boolean ----|
                              Finished
```

Operation လုပ်နေတုန်း Thread B က ကြားထဲမဝင်နိုင်ပါ။

---

# ဘာကြောင့် AtomicBoolean လိုတာလဲ?

ဥပမာ

```java
boolean running = false;
```

Thread နှစ်ခုရှိတယ်။

### Thread A

```
running = true;
```

### Thread B

```
if(running){
    ...
}
```

Thread နှစ်ခု တစ်ပြိုင်နက် လုပ်နေတဲ့အချိန်

- Memory Visibility Problem
    
- Race Condition
    

ဖြစ်နိုင်ပါတယ်။

---

# AtomicBoolean အသုံးပြုခြင်း

```java
AtomicBoolean running = new AtomicBoolean(false);
```

ဒီလိုရေးလိုက်ရင်

Thread ဘယ်နှစ်ခုသုံးသုံး Safe ဖြစ်သွားပါတယ်။

---

# Constructor

### Default

```java
AtomicBoolean flag = new AtomicBoolean();
```

Default Value

```
false
```

---

### Initial Value

```java
AtomicBoolean flag = new AtomicBoolean(true);
```

ဒါဆို

```
true
```

နဲ့ စပါမယ်။

---

# get()

Boolean Value ကိုယူချင်ရင်

```java
AtomicBoolean flag = new AtomicBoolean(true);

System.out.println(flag.get());
```

Output

```
true
```

---

# set()

Value ပြောင်းချင်ရင်

```java
flag.set(false);
```

ဥပမာ

```java
AtomicBoolean flag = new AtomicBoolean(true);

flag.set(false);

System.out.println(flag.get());
```

Output

```
false
```

---

# compareAndSet()

ဒီ Method က `AtomicBoolean` ရဲ့ အရေးကြီးဆုံး Method ဖြစ်ပါတယ်။

Syntax

```java
compareAndSet(expectedValue, newValue)
```

အဓိပ္ပါယ်

```
Expected Value နဲ့တူရင်

↓

New Value ပြောင်း

↓

မတူရင်

↓

မပြောင်း
```

---

## Example 1

```java
AtomicBoolean flag = new AtomicBoolean(false);

boolean result = flag.compareAndSet(false, true);

System.out.println(result);
System.out.println(flag.get());
```

Output

```
true
true
```

ဘာဖြစ်တာလဲ?

```
false ရှိလား?

YES

↓

true ပြောင်း

↓

Return true
```

---

## Example 2

```java
AtomicBoolean flag = new AtomicBoolean(true);

boolean result = flag.compareAndSet(false, true);

System.out.println(result);
```

Output

```
false
```

ဘာကြောင့်?

```
Expected = false

Actual = true

↓

မတူ

↓

မပြောင်း
```

---

# Compare And Set Diagram

```
Current Value

↓

false

↓

compareAndSet(false,true)

↓

Success

↓

true
```

---

# Thread Example

```java
import java.util.concurrent.atomic.AtomicBoolean;

public class Main {

    static AtomicBoolean started =
            new AtomicBoolean(false);

    public static void main(String[] args) {

        Runnable task = () -> {

            if(started.compareAndSet(false, true)) {

                System.out.println(
                        Thread.currentThread().getName()
                        + " Started");

            } else {

                System.out.println(
                        Thread.currentThread().getName()
                        + " Already Running");
            }

        };

        new Thread(task).start();
        new Thread(task).start();
    }
}
```

Output (ဥပမာ)

```
Thread-0 Started
Thread-1 Already Running
```

Thread နှစ်ခုလုံး တစ်ပြိုင်နက် Run သော်လည်း

**တစ်ခုတည်းက `false → true` ပြောင်းနိုင်ပါတယ်။**

---

# ဘာကြောင့် ဒီလိုဖြစ်တာလဲ?

CAS (Compare And Swap)

```
Current = false

↓

Thread-1 Compare

↓

Success

↓

Current = true

↓

Thread-2 Compare

↓

false မဟုတ်တော့

↓

Fail
```

ဒီ Technique ကို

**CAS (Compare-And-Swap)** လို့ခေါ်ပါတယ်။

---

# Lock မလိုဘူး

AtomicBoolean က

```
synchronized
```

မသုံးပါဘူး။

CPU Level CAS Instruction ကို သုံးပါတယ်။

အကျိုးကျေးဇူး

- Faster
    
- Less Blocking
    
- High Performance
    

---

# LazySet()

```java
flag.lazySet(true);
```

`set()` နဲ့ ဆင်တူပေမယ့် Memory Update ကို ချက်ချင်းမဟုတ်ဘဲ အနည်းငယ် နောက်ကျပြီး Publish လုပ်နိုင်ပါတယ်။

Performance အတွက် အသုံးဝင်နိုင်ပေမယ့် Beginner အတွက် `set()` ကိုပဲ အရင်အသုံးပြုတာ ပိုကောင်းပါတယ်။

---

# getAndSet()

တန်ဖိုးဟောင်းကို ပြန်ပေးပြီး အသစ်ပြောင်းပေးတယ်။

```java
AtomicBoolean flag =
        new AtomicBoolean(false);

boolean old = flag.getAndSet(true);

System.out.println(old);
System.out.println(flag.get());
```

Output

```
false
true
```

---

# AtomicBoolean vs boolean

|boolean|AtomicBoolean|
|---|---|
|Thread-safe မဟုတ်|Thread-safe ဖြစ်|
|Race Condition ဖြစ်နိုင်|Race Condition ကို လျှော့ချပေး|
|Multi-thread အတွက် မသင့်|Multi-thread အတွက် သင့်တော်|
|Simple Variable|Atomic Operation ပါဝင်|

---

# synchronized vs AtomicBoolean

### synchronized

```java
synchronized(lock){
    if(!running){
        running = true;
    }
}
```

Advantages

- Logic ကြီးကြီးရေးနိုင်
    
- Critical Section အများကြီးထည့်နိုင်
    

Disadvantages

- Lock ဖြစ်တယ်
    
- Performance ကျနိုင်တယ်
    

---

### AtomicBoolean

```java
running.compareAndSet(false,true);
```

Advantages

- Lock မလို
    
- Faster
    
- Simple
    

Disadvantages

- Boolean တစ်ခုလို Simple State ပြောင်းတာအတွက်ပဲ အကောင်းဆုံး
    

---

# Real World Example (API Server)

Server တစ်ခုမှာ Scheduler ကို **တစ်ကြိမ်ပဲ Start လုပ်ချင်တယ်** ဆိုပါစို့။

```java
AtomicBoolean started =
        new AtomicBoolean(false);

public void start(){

    if(started.compareAndSet(false,true)){

        System.out.println("Start Scheduler");

    }else{

        System.out.println("Already Started");
    }

}
```

User 100 ယောက် `start()` ကို တစ်ပြိုင်နက် ခေါ်ရင်လည်း

```
Start Scheduler
```

ဆိုတာ **တစ်ကြိမ်တည်း** ထွက်ပါမယ်။

---

# Memory Visibility

`AtomicBoolean` က **Visibility** ကိုလည်း အာမခံပေးပါတယ်။

ဥပမာ

```
Thread A

flag.set(true);
```

```
Thread B

flag.get();
```

Thread B က Thread A ပြောင်းလိုက်တဲ့ Value ကို မြင်ရမှာ သေချာပါတယ်။ ဒါဟာ ပုံမှန် `boolean` မျှသာ သုံးထားတဲ့အခါ ဖြစ်နိုင်တဲ့ Memory Visibility ပြဿနာကို လျှော့ချပေးပါတယ်။

---

# Interview Questions

### 1. AtomicBoolean ဆိုတာဘာလဲ?

**ဖြေဆိုနိုင်တဲ့အဖြေ**

> `AtomicBoolean` ဟာ Thread-safe Boolean Class ဖြစ်ပြီး CAS (Compare-And-Swap) ကို အသုံးပြုကာ Lock မလိုဘဲ Boolean Value ကို Atomic Operation နဲ့ Update လုပ်ပေးပါတယ်။

---

### 2. `compareAndSet()` ဆိုတာဘာလဲ?

**ဖြေဆိုနိုင်တဲ့အဖြေ**

> Current Value က Expected Value နဲ့ တူရင်သာ New Value ကို Atomic အနေနဲ့ ပြောင်းပေးပြီး `true` ပြန်ပေးပါတယ်။ မတူရင် မပြောင်းဘဲ `false` ပြန်ပေးပါတယ်။

---

### 3. `AtomicBoolean` ကို ဘယ်အချိန်သုံးသင့်လဲ?

- Scheduler တစ်ကြိမ်ပဲ Start လုပ်ချင်တဲ့အခါ
    
- Flag တစ်ခုကို Thread အများကြီးက မျှဝေသုံးတဲ့အခါ
    
- One-time Initialization
    
- Stop/Running Flag
    
- Feature Enable/Disable Flag
    

---

# AtomicBoolean, volatile boolean, synchronized တို့ကို ဘယ်အချိန်သုံးမလဲ?

|အခြေအနေ|သင့်တော်တဲ့ရွေးချယ်မှု|
|---|---|
|Thread တစ်ခုက ရေးပြီး အခြား Thread တွေက ဖတ်ရုံ|`volatile boolean`|
|Boolean ကို Atomic အနေနဲ့ စစ်ပြီး (`false` ဖြစ်မှ `true` ပြောင်းတာလို) ပြောင်းရမယ်|`AtomicBoolean`|
|Variable တစ်ခုထက်ပိုပြီး Logic အများကြီးကို တစ်ပြိုင်နက် ကာကွယ်ရမယ်|`synchronized` သို့မဟုတ် Lock|

**အရေးကြီးတဲ့ မှတ်ချက်**  
`AtomicBoolean` ဟာ Boolean တန်ဖိုးတစ်ခုရဲ့ Atomic Operation ကိုပဲ အာမခံပေးပါတယ်။ အကယ်၍ Variable အများကြီးကို တစ်ပြိုင်နက် Consistent ဖြစ်အောင် Update လုပ်ဖို့လိုရင် `AtomicBoolean` တစ်ခုတည်းနဲ့ မလုံလောက်ဘဲ `synchronized` သို့မဟုတ် `Lock` ကို အသုံးပြုသင့်ပါတယ်။

## အကျဉ်းချုပ်

- `AtomicBoolean` = Thread-safe Boolean Class
    
- `compareAndSet()` = CAS (Compare-And-Swap) နည်းလမ်းဖြင့် Atomic Update လုပ်ပေးတယ်
    
- Lock (`synchronized`) မလိုဘဲ Performance ကောင်းတယ်
    
- `get()`, `set()`, `getAndSet()`, `compareAndSet()` တို့က အဓိက Methods တွေဖြစ်တယ်
    
- One-time initialization, Start/Stop Flag, Scheduler, Singleton Initialization စတဲ့ Multi-threaded Application များမှာ အသုံးများပါတယ်။