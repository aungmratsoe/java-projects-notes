## 🔹 Prototype Pattern

### ပြဿနာက ဘာလဲ

တစ်ခါတစ်လေ object တစ်ခုကို **အစကနေ ပြန်ဖန်တီးရတာ (ဥပမာ `new` သုံးပြီး field တွေ ပြန်ထည့်ရတာ) costly** ဖြစ်နိုင်ပါတယ် — ဥပမာ:

- Database ကနေ data ဆွဲယူပြီးမှ object ဖန်တီးရတာ
- Network call တစ်ခုခု လုပ်ရတာ
- Complex calculation တွေ ပြန်လုပ်ရတာ

ရှိပြီးသား object တစ်ခုနဲ့ **ဆင်တူတဲ့ object အသစ်** တစ်ခု (value အနည်းငယ်ပဲ ကွာတဲ့) လိုချင်ရင်၊ အစကနေ ပြန်ဖန်တီးနေစရာ မလိုသင့်ပါဘူး။

Class ကိုလည်း တစ်ခါတစ်လေ **မသိချင်ဘူး** (concrete class ကို import လုပ်ချင်မှသာ `new` သုံးလို့ရမှာ) — object interface ကိုပဲ မှီပြီး copy လုပ်ချင်တာမျိုးလည်း ရှိပါတယ်။

### Prototype က ဘယ်လို ဖြေရှင်းလဲ

Object ကိုယ်တိုင်ကို **clone (ကူးယူ)** နိုင်တဲ့ method တစ်ခု ပေးထားပါတယ် — class ကို သိစရာမလိုဘဲ၊ existing object ကနေ တိုက်ရိုက် copy အသစ်တစ်ခု ရနိုင်ပါတယ်။ Java မှာ `Cloneable` interface နဲ့ `clone()` method ကို သုံးပါတယ်။

### Java Code

```java
class Sheep implements Cloneable {
    private String name;
    private int age;

    public Sheep(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // ⭐ ဒါက Prototype method
    @Override
    public Sheep clone() {
        try {
            return (Sheep) super.clone(); // Object class ရဲ့ built-in shallow copy
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    public void setName(String name) { this.name = name; }
    public String toString() {
        return "Sheep{name='" + name + "', age=" + age + "}";
    }
}
```

သုံးပုံ -

```java
Sheep original = new Sheep("Dolly", 2);

Sheep clone1 = original.clone();
clone1.setName("Dolly-2");

System.out.println(original); // Sheep{name='Dolly', age=2}
System.out.println(clone1);   // Sheep{name='Dolly-2', age=2}
```

### ⚠️ Shallow Copy vs Deep Copy — အရေးကြီးတဲ့ အချက်

`super.clone()` က **shallow copy** ပဲ လုပ်ပါတယ် — object ထဲက reference type field (array, list, object အခြား) တွေကို **reference ကိုပဲ copy** လုပ်ပါတယ်၊ inner object ကို အသစ် ပြန်မဖန်တီးပါဘူး။

```java
class Flock implements Cloneable {
    private List<String> sheepNames = new ArrayList<>();

    @Override
    public Flock clone() {
        try {
            Flock cloned = (Flock) super.clone();
            // ⭐ Deep copy - list ကို အသစ်ပြန်ဖန်တီးပေးရမယ်
            cloned.sheepNames = new ArrayList<>(this.sheepNames);
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    public List<String> getSheepNames() { return sheepNames; }
}
```

Deep copy မလုပ်ဘဲ list ကို ဒီအတိုင်း share ထားရင် — clone object ရဲ့ list ကို ပြင်လိုက်ရင် original ရဲ့ list ပါ ပြောင်းသွားနိုင်ပါတယ် (bug ဖြစ်စရာ)။

### Real-world Java ဥပမာများ

- `Object.clone()` (Java built-in mechanism ကိုယ်တိုင်ပဲ Prototype pattern ပါ)
- `ArrayList.clone()`, `HashMap.clone()`

### သတိထားရမယ့်အချက် (Trade-offs)

- Java ရဲ့ `Cloneable` interface က design အနေနဲ့ **ချို့ယွင်းချက်များ** လို့ လူသိများပါတယ် (marker interface ဖြစ်ပြီး `clone()` method ကို override မလုပ်ရင် `CloneNotSupportedException` တက်တတ်တယ်)
- Deep copy ဖြစ်၊ မဖြစ် **သတိထားရမယ်** — bug အများဆုံး ဖြစ်ရာအကြောင်းရင်းတွေထဲက တစ်ခုပါ
- Real project အများစုမှာ copy constructor (ဥပမာ `new Sheep(original)`) ကို ပိုသုံးကြပါတယ် — ပိုရှင်းလင်းလို့ပါ

---

🎉 **Creational Patterns ငါးခုလုံး ပြီးပါပြီ**: Singleton → Factory Method → Abstract Factory → Builder → Prototype

နောက်တစ်ဆင့် **Structural Patterns** ဘက် ကူးကြရအောင် — **Adapter Pattern** ကနေ စမလား၊ ဒါမှမဟုတ် ဒီ Creational patterns ၅ ခုအတွက် quiz လေး အရင်လုပ်ကြည့်ချင်ပါသလား။