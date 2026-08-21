ကောင်းပါပြီ၊ **Singleton Pattern** ကနေ အစပြုကြရအောင်။

---

## 🔹 Singleton Pattern

### ပြဿနာက ဘာလဲ

တစ်ခါတစ်လေ application တစ်ခုမှာ class တစ်ခုရဲ့ **object တစ်ခုတည်းသာ** ရှိသင့်တယ်။ ဥပမာ -

- Database connection manager
- Application configuration/settings
- Logger (log ရေးတဲ့ class)

ဒီလို case တွေမှာ object တစ်ခုထက်ပိုပြီး ဖန်တီးမိရင် — resource ကို ထပ်ခါထပ်ခါ သုံးမိတာ၊ setting တွေ mismatch ဖြစ်တာ၊ memory အလကား ကုန်တာစတဲ့ ပြဿနာတွေ ဖြစ်နိုင်ပါတယ်။

### Singleton က ဘယ်လို ဖြေရှင်းလဲ

Singleton Pattern က class တစ်ခုအနေနဲ့ **object တစ်ခုတည်းသာ** ဖန်တီးခွင့်ပြုပြီး၊ ဘယ်နေရာကမဆို ဒီ object တစ်ခုတည်းကိုပဲ ပြန်ရအောင် globally access point တစ်ခု ပေးထားပါတယ်။

ဒါကို လုပ်ဖို့ အဓိက နည်းလမ်း ၃ ခု —

1. `constructor` ကို `private` လုပ်ထား — class အပြင်ကနေ `new` နဲ့ တိုက်ရိုက်ဖန်တီးလို့ မရအောင်
2. class ထဲမှာ object ကို `static` variable အနေနဲ့ သိမ်းထား
3. `public static` method (ပုံမှန် `getInstance()`) ပေးထား — object ကို ပြန်ယူနိုင်ဖို့

### Java Code — အခြေခံ (Eager Initialization)

```java
public class Logger {
    // class load ဖြစ်တာနဲ့ object ကို တစ်ခါတည်း ဖန်တီးလိုက်တယ်
    private static final Logger instance = new Logger();

    // constructor ကို private လုပ်ထား - class ပြင်ကနေ new မလုပ်နိုင်တော့ဘူး
    private Logger() {
        System.out.println("Logger object created");
    }

    public static Logger getInstance() {
        return instance;
    }

    public void log(String message) {
        System.out.println("LOG: " + message);
    }
}
```

သုံးပုံ -

```java
Logger logger1 = Logger.getInstance();
Logger logger2 = Logger.getInstance();

System.out.println(logger1 == logger2); // true - object တစ်ခုတည်းပဲ
```

### Java Code — Lazy Initialization (Thread-safe)

Real project တွေမှာ thread multiple ခု တစ်ပြိုင်နက်တည်း `getInstance()` ခေါ်နိုင်တဲ့အတွက် thread-safety ကို ဂရုစိုက်ရပါတယ်။

```java
public class Config {
    private static volatile Config instance;
    private Config() {}

    public static Config getInstance() {
        if (instance == null) {
            synchronized (Config.class) {
                if (instance == null) {
                    instance = new Config();
                }
            }
        }
        return instance;
    }
}
```

ဒါကို **"Double-Checked Locking"** လို့ခေါ်ပါတယ် — `synchronized` block ထဲကို တစ်ခါဖန်တီးပြီးရင် ထပ်ဝင်စရာမလိုအောင် performance ကောင်းအောင် စစ်ဆေးမှု ၂ ကြိမ်လုပ်တာဖြစ်ပါတယ်။

### Real-world Java ဥပမာများ

- `Runtime.getRuntime()`
- `java.awt.Desktop.getDesktop()`

### သတိထားရမယ့်အချက် (Trade-offs)

- Unit testing မှာ Singleton ကို mock လုပ်ဖို့ ခက်တတ်ပါတယ်
- Global state ဖြစ်နေတာကြောင့် code coupling မြင့်လာနိုင်ပါတယ်
- Overuse လုပ်ရင် "anti-pattern" လို့တောင် ယူဆကြပါတယ်

---

နားလည်ပြီလား။ Practice အနေနဲ့ **သင့်ဘာသာ Singleton class တစ်ခု** (ဥပမာ- `AppSettings`) ရေးကြည့်ချင်ပါသလား၊ ဒါမှမဟုတ် နောက် pattern **Factory Method** ကို ဆက်သင်ချင်ပါသလား။