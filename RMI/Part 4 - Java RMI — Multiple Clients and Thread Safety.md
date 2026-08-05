# Java RMI — Multiple Clients & Thread Safety — Part 4

## RMI ရဲ့ Default Behavior ကို အရင်နားလည်ရမယ်

**RMI Server ဟာ Client တစ်ယောက်ချင်းစီအတွက် သီးခြား Thread တစ်ခုစီ အလိုအလျောက် ဖန်တီးပေးပါတယ်။** ဒါက အားသာချက်ဖြစ်ပေမယ့် (client များစွာ တစ်ပြိုင်နက် request ပို့လို့ရလို့) — **Server ဘက်က shared data (ဥပမာ - list, map, counter) ကို client အများကြီးက တစ်ပြိုင်နက် ပြင်ဆင်ရင် data corruption ဖြစ်နိုင်** ပါတယ်။

```
Client A ──┐
Client B ──┼──►  Server (thread 3 ခု တစ်ပြိုင်နက်)
Client C ──┘         │
                      ▼
                shared inventory list
                (protection မထားရင် 
                 data ပျက်စီးနိုင်)
```

## Race Condition ဥပမာ (မှားနေတဲ့ Code)

Inventory system တစ်ခုမှာ stock လျှော့တာကို branch (client) များစွာက တစ်ပြိုင်နက် လုပ်ချင်တဲ့အခါ:

```java
// ❌ Thread-unsafe — မှားနေပါတယ်
public class InventoryImpl extends UnicastRemoteObject implements InventoryInterface {

    private int stockCount = 100;

    protected InventoryImpl() throws RemoteException { super(); }

    @Override
    public boolean reduceStock(int quantity) throws RemoteException {
        if (stockCount >= quantity) {
            // ⚠️ ဒီနေရာမှာ thread နှစ်ခု တစ်ပြိုင်နက် ဝင်လာနိုင်တယ်
            // Thread A: check stockCount >= quantity (true)
            // Thread B: check stockCount >= quantity (true) ← A ရေးမှမြင်ခင်
            // A: stockCount -= quantity
            // B: stockCount -= quantity ← Stock negative ဖြစ်နိုင်တယ်!
            stockCount -= quantity;
            return true;
        }
        return false;
    }

    @Override
    public int getStock() throws RemoteException {
        return stockCount;
    }
}
```

ဒီ code မှာ Branch A နဲ့ Branch B က Stock 100 ကို တစ်ပြိုင်နက် `reduceStock(80)` ခေါ်လိုက်ရင် **နှစ်ဖက်စလုံး success ဖြစ်ပြီး stock -60 ဖြစ်သွားနိုင်** ပါတယ် (real-world ဆိုရင် ဖြစ်လို့မရသင့်တဲ့ ကိစ္စ)။

## Fix 1 — `synchronized` Keyword (အလွယ်ဆုံးနည်း)

```java
// ✅ Thread-safe
public class InventoryImpl extends UnicastRemoteObject implements InventoryInterface {

    private int stockCount = 100;

    protected InventoryImpl() throws RemoteException { super(); }

    // synchronized ထည့်လိုက်ရင် တစ်ကြိမ်မှာ Thread တစ်ခုပဲ ဒီ method ကို 
    // ဝင်ခွင့်ရမယ် (တခြား thread တွေက ဆိုင်းငံ့စောင့်ရမယ်)
    @Override
    public synchronized boolean reduceStock(int quantity) throws RemoteException {
        if (stockCount >= quantity) {
            stockCount -= quantity;
            return true;
        }
        return false;
    }

    @Override
    public synchronized int getStock() throws RemoteException {
        return stockCount;
    }
}
```

**`synchronized`** က object တစ်ခုလုံးကို lock ချသလိုမျိုးပါ — Thread A က `reduceStock()` ထဲမှာ run နေတုန်း Thread B က `reduceStock()` ဒါမှမဟုတ် `getStock()` ကို ဝင်ချင်ရင် A ပြီးမှ ဝင်ခွင့်ရမှာပါ။

## Fix 2 — `ReentrantLock` (ပိုချုပ်ကိုင်ချင်ရင်)

```java
import java.util.concurrent.locks.ReentrantLock;

public class InventoryImpl extends UnicastRemoteObject implements InventoryInterface {

    private int stockCount = 100;
    private final ReentrantLock lock = new ReentrantLock();

    protected InventoryImpl() throws RemoteException { super(); }

    @Override
    public boolean reduceStock(int quantity) throws RemoteException {
        lock.lock();
        try {
            if (stockCount >= quantity) {
                stockCount -= quantity;
                return true;
            }
            return false;
        } finally {
            lock.unlock(); // exception ဖြစ်ရင်တောင် unlock ဖြစ်ရမယ်
        }
    }
}
```

`synchronized` နဲ့ ယှဉ်ရင် `ReentrantLock` က `tryLock()` (timeout ထားပြီး စောင့်ချင်ရင်), `lockInterruptibly()` စတဲ့ advanced feature တွေ ထပ်ရပါတယ်။ Simple case တွေမှာတော့ `synchronized` ပဲ လုံလောက်ပါတယ်။

## Fix 3 — Concurrent Collections (List/Map အတွက်)

Inventory list တစ်ခုလုံးကို manage လုပ်ရင်တော့ `synchronized` block ရေးနေမယ့်အစား **thread-safe collection** တွေကို တိုက်ရိုက်သုံးတာ ပိုကောင်းပါတယ်:

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

public class InventoryImpl extends UnicastRemoteObject implements InventoryInterface {

    // Product name → stock count (thread-safe map)
    private final ConcurrentHashMap<String, AtomicInteger> stock = new ConcurrentHashMap<>();

    protected InventoryImpl() throws RemoteException {
        super();
        stock.put("Laptop", new AtomicInteger(50));
        stock.put("Mouse", new AtomicInteger(200));
    }

    @Override
    public boolean reduceStock(String product, int quantity) throws RemoteException {
        AtomicInteger current = stock.get(product);
        if (current == null) return false;

        // AtomicInteger ရဲ့ updateAndGet/compareAndSet — thread-safe operation
        int updated = current.updateAndGet(v -> v >= quantity ? v - quantity : v);
        return updated >= 0 && (updated + quantity) <= current.get() + quantity; 
        // (သို့) ရိုးရိုးရှင်းရှင်း synchronized method သုံးတာ ပိုနားလည်လွယ်ပါတယ်
    }

    @Override
    public int getStock(String product) throws RemoteException {
        AtomicInteger current = stock.get(product);
        return current == null ? 0 : current.get();
    }
}
```

> **Note**: `AtomicInteger.updateAndGet()` logic က case အနည်းငယ် ရှုပ်နိုင်လို့ လက်တွေ့မှာ `synchronized` method ကို per-product lock object နဲ့ ရေးတာက ပိုနားလည်လွယ်ပါတယ်:

```java
private final Object lockObj = new Object();

@Override
public boolean reduceStock(String product, int quantity) throws RemoteException {
    synchronized (lockObj) {
        AtomicInteger current = stock.get(product);
        if (current == null || current.get() < quantity) return false;
        current.addAndGet(-quantity);
        return true;
    }
}
```

## Multiple Clients Test လုပ်ကြည့်နည်း

Client 2-3 ခုကို computer အသီးသီးက (သို့) computer တစ်ခုတည်းက Client program 2-3 ခု run ပြီး တစ်ပြိုင်နက် `reduceStock()` ခေါ်ကြည့်ပါ — `synchronized` မထားရင် stock negative ဖြစ်တာ တွေ့ရမှာပါ၊ ထားရင်တော့ stock 0 အောက် ဘယ်တော့မှ မကျပါဘူး။

## သင့် Swing Project အတွက် လက်တွေ့ လိုက်နာသင့်တဲ့ Rule များ

|Rule|အကြောင်းအရာ|
|---|---|
|Shared mutable state (list, map, counter, balance) ရှိတဲ့ method တိုင်းကို `synchronized` ထား|Data corruption ကာကွယ်ဖို့|
|Lock ကို **တတ်နိုင်သမျှ ကျဉ်းအောင်** ထား (method တစ်ခုလုံး synchronized မလုပ်ဘဲ critical section ပဲ)|Performance ကောင်းအောင် — client တွေ မလိုအပ်ဘဲ ရေရွှေ့စောင့်နေရမှာ လျှော့ဖို့|
|Read-only method (`getStock()`) ကိုလည်း synchronized ထားရမယ်|Half-updated value ဖတ်မိမှာ ကာကွယ်ဖို့ (visibility guarantee)|
|`CopyOnWriteArrayList`/`ConcurrentHashMap` ကို callback list, cache စတာတွေမှာ သုံး|Manual synchronized ထက် ရေးရလွယ်ပြီး performance ကောင်း|
|Deadlock ရှောင်ဖို့ — Lock ထားရင်း တခြား remote method ထပ်မခေါ်ရ|Lock ထားစဉ် နောက် RMI call ခေါ်ရင် client တွေ ဆိုင်းငံ့ (block) ဖြစ်နိုင်|

---

ဒီအထိဆိုရင် RMI series ရဲ့ core concept (Basic setup → Swing integration → Callback → Thread safety) ပြီးသွားပါပြီ။ နောက်တစ်ခု ဆက်လေ့လာချင်တာ ရှိပါသလား — ဥပမာ **RMI Security (SSL/authentication)**, ဒါမှမဟုတ် **Part 1 က gRPC learning ကို ပြန်ဆက်** မလား?