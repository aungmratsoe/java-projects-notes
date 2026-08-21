## 🔹 Factory Method Pattern

### ပြဿနာက ဘာလဲ

Object တစ်ခုကို ဖန်တီးတဲ့အခါ `new` keyword နဲ့ တိုက်ရိုက်ဖန်တီးလိုက်ရင် — code ဟာ specific class တစ်ခုတည်းနဲ့ **tightly coupled** ဖြစ်သွားပါတယ်။ နောက်ပိုင်း object အမျိုးအစား ထပ်တိုးချင်တဲ့အခါ (ဥပမာ - `Car` အသစ်၊ `Notification` အသစ်) code အနှံ့ ပြင်ရတတ်ပါတယ်။

ဥပမာ — Notification system တစ်ခု စဉ်းစားကြည့်ပါ:

```java
// မကောင်းတဲ့ approach
Notification notification;
if (type.equals("EMAIL")) {
    notification = new EmailNotification();
} else if (type.equals("SMS")) {
    notification = new SmsNotification();
}
// type အသစ်ထပ်တိုးရင် ဒီ if-else ကို ထပ်ပြင်ရဦးမယ်
```

ဒီလို code မျိုးက **Open/Closed Principle** ကို ဖောက်ဖျက်ပါတယ် (class အသစ်ထည့်ဖို့ ရှိပြီးသား code ကို ပြင်ရလို့).

### Factory Method က ဘယ်လို ဖြေရှင်းလဲ

Object ဖန်တီးတဲ့ logic ကို subclass တွေဆီ **လွှဲအပ်** လိုက်ပါတယ်။ Parent class (သို့) interface မှာ object ဖန်တီးဖို့ method တစ်ခု declare ထားပြီး၊ actual ဘယ် class ကို ဖန်တီးမလဲဆိုတာကို subclass တစ်ခုချင်းစီက ဆုံးဖြတ်ပါတယ်။

### Java Code

```java
// Product interface
interface Notification {
    void notifyUser();
}

// Concrete Products
class EmailNotification implements Notification {
    public void notifyUser() {
        System.out.println("Sending an EMAIL notification");
    }
}

class SmsNotification implements Notification {
    public void notifyUser() {
        System.out.println("Sending an SMS notification");
    }
}

class PushNotification implements Notification {
    public void notifyUser() {
        System.out.println("Sending a PUSH notification");
    }
}
```

```java
// Creator (abstract class) — factory method ကို declare လုပ်ထား
abstract class NotificationCreator {
    // ⭐ ဒါက Factory Method
    public abstract Notification createNotification();

    // Common logic - factory method ကို သုံးတယ်
    public void send() {
        Notification notification = createNotification();
        notification.notifyUser();
    }
}
```

```java
// Concrete Creators - object ဘယ်ဟာဖန်တီးမလဲ ဆုံးဖြတ်တယ်
class EmailNotificationCreator extends NotificationCreator {
    public Notification createNotification() {
        return new EmailNotification();
    }
}

class SmsNotificationCreator extends NotificationCreator {
    public Notification createNotification() {
        return new SmsNotification();
    }
}
```

သုံးပုံ -

```java
NotificationCreator creator = new EmailNotificationCreator();
creator.send(); // "Sending an EMAIL notification"

creator = new SmsNotificationCreator();
creator.send(); // "Sending an SMS notification"
```

**Notification** အသစ် (ဥပမာ `PushNotification`) ထပ်ထည့်ချင်ရင် — ရှိပြီးသား code ကို လုံးဝ မပြင်ဘဲ `PushNotificationCreator` class အသစ်တစ်ခု ထပ်ဖန်တီးရုံပါပဲ။

### Real-world Java ဥပမာများ

- `Calendar.getInstance()` — locale/timezone အလိုက် Calendar implementation မတူတာကို ပြန်ပေးတယ်
- `NumberFormat.getInstance()`
- Java Collections Framework ထဲက `iterator()` method (Collection interface)

### Singleton နဲ့ ဘာကွာသလဲ

||Singleton|Factory Method|
|---|---|---|
|ရည်ရွယ်ချက်|object **တစ်ခုတည်း** ရှိစေရန်|object **အမျိုးအစားများစွာ** ကို flexible ဖန်တီးရန်|
|Focus|instance count ကို ထိန်းချုပ်တယ်|object creation logic ကို subclass ဆီ လွှဲတယ်|

### သတိထားရမယ့်အချက် (Trade-offs)

- Subclass အသစ်တိုင်းအတွက် Creator class အသစ်တစ်ခု လိုအပ်တာကြောင့် **class အရေအတွက် များလာနိုင်ပါတယ်**
- Simple case အနည်းငယ်ပဲရှိရင် Factory Method က overkill ဖြစ်နိုင်ပါတယ် — situation ကို ကြည့်ပြီး ဆုံးဖြတ်ရပါမယ်

---

Practice အနေနဲ့ **`PushNotificationCreator`** class ကို သင့်ဘာသာ ရေးကြည့်ချင်ပါသလား၊ ဒါမှမဟုတ် နောက် pattern **Abstract Factory** ကို ဆက်သင်ချင်ပါသလား။