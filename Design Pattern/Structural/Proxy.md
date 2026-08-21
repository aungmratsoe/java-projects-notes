## 🔹 Proxy Pattern

### ပြဿနာက ဘာလဲ

Object တစ်ခုကို **တိုက်ရိုက် access** လုပ်တာ သင့်တော်မှု မရှိတဲ့ အခြေအနေတွေ ရှိပါတယ် — ဥပမာ:

- Object ဖန်တီးဖို့ **costly** (heavy resource, network call) ဖြစ်နေတာကြောင့် **တကယ်လိုမှသာ** ဖန်တီးချင်တာ (lazy loading)
- Object ကို **access control** ချင်တာ (permission check)
- Method ခေါ်တိုင်း **log/cache/monitor** ချင်တာ

ဒီလို logic တွေကို actual object ရဲ့ code ထဲမှာ တိုက်ရိုက် ထည့်ရင် — object ရဲ့ **core responsibility** နဲ့ ဒီ concern တွေ ရောနှောသွားပါတယ် (Single Responsibility Principle ကို ဖောက်ဖျက်တယ်)။

### Proxy က ဘယ်လို ဖြေရှင်းလဲ

Real object ရှေ့မှာ **substitute/placeholder object** တစ်ခု ထားပါတယ် — Proxy ဟာ real object နဲ့ **interface တူတူ** implement လုပ်ထားပြီး၊ client က Proxy ကို real object လို့ပဲ ထင်ပြီး သုံးပါတယ်။ Proxy ကတဆင့် access ကို ထိန်းချုပ် (control) လုပ်ပါတယ်။

### Java Code — Lazy Loading Proxy (Virtual Proxy)

```java
// Common interface
interface Image {
    void display();
}
```

```java
// Real object - ဖန်တီးဖို့ costly (ဥပမာ disk ကနေ load ရတယ်)
class RealImage implements Image {
    private String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk(); // ⭐ ဒါက costly operation
    }

    private void loadFromDisk() {
        System.out.println("Loading " + filename + " from disk...");
    }

    public void display() {
        System.out.println("Displaying " + filename);
    }
}
```

```java
// ⭐ Proxy - RealImage ကို "လိုမှသာ" ဖန်တီးမယ်
class ImageProxy implements Image {
    private RealImage realImage; // ဒီအထိ null ပဲရှိမယ်
    private String filename;

    public ImageProxy(String filename) {
        this.filename = filename; // constructor မှာ RealImage မဖန်တီးသေးဘူး
    }

    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename); // ⭐ display() ခေါ်မှသာ ဖန်တီးမယ်
        }
        realImage.display();
    }
}
```

သုံးပုံ -

```java
Image image = new ImageProxy("photo.jpg");
System.out.println("Proxy object created, image not loaded yet.");

// display() ကို ခေါ်မှသာ RealImage ဖန်တီးမယ်, "Loading..." ပြမယ်
image.display();
// Loading photo.jpg from disk...
// Displaying photo.jpg

image.display(); // ဒုတိယအကြိမ် - RealImage ရှိပြီးသားဖြစ်လို့ ပြန်မ load တော့ဘူး
// Displaying photo.jpg
```

### Proxy အမျိုးအစားများ

|Proxy အမျိုးအစား|ရည်ရွယ်ချက်|
|---|---|
|**Virtual Proxy**|Object ဖန်တီးမှုကို lazy-load လုပ်ဖို့ (အပေါ်က ဥပမာ)|
|**Protection Proxy**|Access permission စစ်ဆေးဖို့|
|**Remote Proxy**|Network/remote object ကို local object ပုံဆောင်ပေးဖို့ (Java RMI)|
|**Logging/Caching Proxy**|Method call တိုင်း log ရေးဖို့ (သို့) result cache ဖို့|

### Real-world Java ဥပမာများ

- `java.lang.reflect.Proxy` — Java built-in dynamic proxy mechanism
- Hibernate ရဲ့ lazy-loading entities (`@OneToMany(fetch = FetchType.LAZY)`)
- Spring AOP — method interception (logging, transaction) အတွက် proxy object တွေ auto-generate လုပ်တယ်

### Decorator နဲ့ ဘာကွာသလဲ

နှစ်ခုစလုံး interface တူတဲ့ object ကို wrap ပေမယ့် —

||Decorator|Proxy|
|---|---|---|
|ရည်ရွယ်ချက်|behavior **ထပ်ဖြည့်**|access ကို **ထိန်းချုပ်**|
|Object ဖန်တီးချိန်|wrap လုပ်တဲ့ object ကို **အစကတည်းက** ရထားပြီးသား|Proxy ကိုယ်တိုင်က real object ကို **ဘယ်အချိန် ဖန်တီးမလဲ ဆုံးဖြတ်** နိုင်တယ် (lazy)|

### သတိထားရမယ့်အချက် (Trade-offs)

- Layer တစ်ခု ထပ်တိုးလာလို့ response time အနည်းငယ် ကြာနိုင်ပါတယ် (proxy ကတဆင့် ဖြတ်ရလို့)
- Code complexity တိုးလာနိုင်ပါတယ် — class အသစ်တစ်ခု ထပ်ရေးရလို့

---

🎉 **Structural Patterns ၅ ခုလုံး ပြီးပါပြီ**: Adapter → Decorator → Facade → Composite → Proxyဆက်ပြီး **Behavioral Patterns** ဘက် ကူးကြရအောင်။ ဒီအုပ်စုမှာ Observer, Strategy, Command, Template Method, Iterator, State, စတာတွေ ပါပါတယ် (အသုံးများဆုံးက **Observer** နဲ့ **Strategy** ပါ)။

ဘယ်ဟာကနေ ဆက်စချင်ပါသလဲ —

- **Observer Pattern** (event-driven system တွေမှာ အသုံးများဆုံး)
- **Strategy Pattern**
- ဒါမှမဟုတ် ဒီအထိ pattern ၉ ခုအတွက် quiz လေး အရင်လုပ်ကြည့်ချင်ပါသလား