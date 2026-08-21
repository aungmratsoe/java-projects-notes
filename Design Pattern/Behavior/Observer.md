## 🔹 Observer Pattern

### ပြဿနာက ဘာလဲ

Object တစ်ခု (**Subject**) ရဲ့ state ပြောင်းတဲ့အခါ၊ **ဆက်စပ်နေတဲ့ object များစွာ (Observers)** ကို အသိပေးချင်တဲ့ အခြေအနေတွေ ရှိပါတယ် — ဒါပေမယ့် Subject က Observer တွေကို **hard-code** ထားရင် (ဥပမာ Observer object တွေကို တိုက်ရိုက် reference ကိုင်ထားပြီး method ခေါ်ရင်):

- Observer အသစ် ထပ်ထည့်ရင် Subject ရဲ့ code ကို ပြင်ရမယ်
- Subject က Observer type အားလုံးကို **သိနေရမယ်** (tightly coupled)
- Runtime မှာ Observer တွေကို dynamically add/remove လုပ်ဖို့ ခက်ခဲမယ်

ဥပမာ — YouTube channel တစ်ခု video အသစ် upload လိုက်တဲ့အခါ subscriber **အားလုံး** ကို notify ပေးချင်တယ် ဆိုပါစို့ — subscriber အရေအတွက်ကတော့ runtime မှာပဲ ပြောင်းလဲနေမှာပါ။

### Observer က ဘယ်လို ဖြေရှင်းလဲ

Subject ထဲမှာ Observer **list** တစ်ခု ထားပြီး၊ Observer တွေကို **register/unregister** လုပ်နိုင်တဲ့ method ပေးထားပါတယ်။ State ပြောင်းတိုင်း Subject က list ထဲက Observer **အားလုံးကို loop ပြီး notify** ပါတယ် — Subject က Observer တွေကို **interface** (concrete type မဟုတ်) ကတဆင့်ပဲ သိထားပါတယ်။ ဒါကို **Publisher-Subscriber** pattern လို့လည်း ခေါ်ကြပါတယ်။

### Java Code

```java
// Observer interface
interface Subscriber {
    void update(String videoTitle);
}
```

```java
// Subject interface
interface Channel {
    void subscribe(Subscriber sub);
    void unsubscribe(Subscriber sub);
    void notifySubscribers(String videoTitle);
}
```

```java
// Concrete Subject
class YouTubeChannel implements Channel {
    private String channelName;
    private List<Subscriber> subscribers = new ArrayList<>();

    public YouTubeChannel(String channelName) {
        this.channelName = channelName;
    }

    public void subscribe(Subscriber sub) {
        subscribers.add(sub);
    }

    public void unsubscribe(Subscriber sub) {
        subscribers.remove(sub);
    }

    public void notifySubscribers(String videoTitle) {
        for (Subscriber sub : subscribers) { // ⭐ Observer အားလုံးကို loop notify
            sub.update(videoTitle);
        }
    }

    // Business logic - video အသစ် upload
    public void uploadVideo(String title) {
        System.out.println(channelName + " uploaded: " + title);
        notifySubscribers(title); // state ပြောင်းတာနဲ့ notify ချက်ချင်း
    }
}
```

```java
// Concrete Observers
class User implements Subscriber {
    private String name;

    public User(String name) { this.name = name; }

    public void update(String videoTitle) {
        System.out.println(name + " received notification: New video - " + videoTitle);
    }
}
```

သုံးပုံ -

```java
YouTubeChannel channel = new YouTubeChannel("Java Tutorials");

Subscriber alice = new User("Alice");
Subscriber bob = new User("Bob");

channel.subscribe(alice);
channel.subscribe(bob);

channel.uploadVideo("Observer Pattern Explained");
// Java Tutorials uploaded: Observer Pattern Explained
// Alice received notification: New video - Observer Pattern Explained
// Bob received notification: New video - Observer Pattern Explained

channel.unsubscribe(bob); // Bob subscribe ဖျက်လိုက်

channel.uploadVideo("Strategy Pattern Next");
// Java Tutorials uploaded: Strategy Pattern Next
// Alice received notification: New video - Strategy Pattern Next
// (Bob ဆီ notification မရောက်တော့ဘူး)
```

`YouTubeChannel` (Subject) က `User` class ကို **တိုက်ရိုက် မသိပါဘူး** — `Subscriber` interface ကိုပဲ သိထားလို့ Observer အသစ်ဘယ်လောက် ထပ်တိုးထိုး Subject code **တစ်ကြောင်းမှ ပြင်စရာမလို** ပါဘူး။

### Real-world Java ဥပမာများ

- Java Swing `ActionListener`, `MouseListener` — button click ဖြစ်တာနဲ့ listener တွေ notify ခံရတယ်
- `java.util.Observer`/`Observable` (Java 9 က deprecated လုပ်ပြီးသား — အသစ်ရေးရင် `PropertyChangeListener` သို့ custom interface သုံးရန် အကြံပြုပါတယ်)
- Spring's `ApplicationEvent` / `ApplicationListener`

### သတိထားရမယ့်အချက် (Trade-offs)

- Observer အရေအတွက် များလာရင် notification order ကို **control လုပ်ရခက်** တတ်ပါတယ်
- Observer ထဲက `update()` method က slow/error ဖြစ်ရင် Subject ရဲ့ notify loop တစ်ခုလုံး ထိခိုက်နိုင်ပါတယ် (async handling စဉ်းစားထားသင့်)
- Memory leak ဖြစ်တတ်ပါတယ် — Observer တွေကို **unsubscribe မလုပ်ဘဲ** ထားခဲ့ရင် (ဥပမာ Android/Swing UI listener) Subject က Observer ကို reference ကိုင်ထားနေမှာမို့ garbage collect မဖြစ်နိုင်တော့ပါ

---

Behavioral pattern ပထမဆုံး Observer ပြီးပါပြီ။ ဆက်ပြီး **Strategy Pattern** ကို သင်ချင်ပါသလား၊ ဒါမှမဟုတ် ဒီအထိ pattern ၁၀ ခုအတွက် quiz လေး အရင်လုပ်ကြည့်ချင်ပါသလား။