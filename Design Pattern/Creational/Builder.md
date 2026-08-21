## 🔹 Builder Pattern

### ပြဿနာက ဘာလဲ

Object တစ်ခုမှာ **field/parameter အများကြီး** ရှိနေတဲ့အခါ (အချို့ optional ဖြစ်တဲ့) constructor ကို ရေးရခက်ပါတယ်။

```java
// မကောင်းတဲ့ approach - "Telescoping Constructor" ပြဿနာ
public class Pizza {
    public Pizza(String size, String crust, boolean cheese, boolean pepperoni, 
                 boolean mushroom, boolean onion, boolean extraSauce) {
        // ... parameter ၇ ခု, order မှားရင် bug!
    }
}

// Client code - ဘာလိုက်ဘာလဲ မသိနိုင်ဘူး
Pizza pizza = new Pizza("Large", "Thin", true, false, true, false, true);
```

Parameter များတဲ့အခါ:

- ဘယ် `boolean` က ဘာကို ကိုယ်စားပြုလဲ ဖတ်ရခက်ပါတယ်
- optional field တွေအတွက် constructor overload အများကြီး ရေးရနိုင်ပါတယ်
- order မှားရင် compile error မတက်ဘဲ logic bug တက်တတ်ပါတယ်

### Builder က ဘယ်လို ဖြေရှင်းလဲ

Object ဖန်တီးတဲ့ logic ကို **သီးခြား Builder class** တစ်ခုထဲ ခွဲထုတ်လိုက်ပြီး၊ field တွေကို **တစ်ခုချင်းစီ, step by step, readable** အနေနဲ့ သတ်မှတ်ပေးနိုင်အောင် လုပ်ပါတယ် (**method chaining** သုံးလေ့ရှိပါတယ်)။

### Java Code

```java
public class Pizza {
    // final fields - ဖန်တီးပြီးရင် ပြောင်းလို့မရ (immutable)
    private final String size;
    private final String crust;
    private final boolean cheese;
    private final boolean pepperoni;
    private final boolean extraSauce;

    // constructor ကို private လုပ်ထား - Builder ကနေပဲ ဖန်တီးခွင့်ပြု
    private Pizza(Builder builder) {
        this.size = builder.size;
        this.crust = builder.crust;
        this.cheese = builder.cheese;
        this.pepperoni = builder.pepperoni;
        this.extraSauce = builder.extraSauce;
    }

    public String toString() {
        return size + " " + crust + " pizza [cheese=" + cheese +
               ", pepperoni=" + pepperoni + ", extraSauce=" + extraSauce + "]";
    }

    // Static nested Builder class
    public static class Builder {
        private String size;          // required
        private String crust = "Regular"; // default value
        private boolean cheese = false;
        private boolean pepperoni = false;
        private boolean extraSauce = false;

        public Builder(String size) {
            this.size = size; // required field ကို constructor ထဲမှာ ယူ
        }

        public Builder crust(String crust) {
            this.crust = crust;
            return this; // ⭐ this ကို return လုပ်လို့ method chain လုပ်လို့ရတယ်
        }

        public Builder cheese(boolean value) {
            this.cheese = value;
            return this;
        }

        public Builder pepperoni(boolean value) {
            this.pepperoni = value;
            return this;
        }

        public Builder extraSauce(boolean value) {
            this.extraSauce = value;
            return this;
        }

        public Pizza build() {
            return new Pizza(this); // နောက်ဆုံး Pizza object ကို ဖန်တီး
        }
    }
}
```

သုံးပုံ -

```java
Pizza pizza = new Pizza.Builder("Large")
                    .crust("Thin")
                    .cheese(true)
                    .extraSauce(true)
                    .build();

System.out.println(pizza);
// Output: Large Thin pizza [cheese=true, pepperoni=false, extraSauce=true]
```

**ဖတ်ရလွယ်တယ်** — `pepperoni(false)` ကို ချန်ထားလို့ရတယ် (default က false ဖြစ်နေတာကြောင့်)၊ ဘယ် value က ဘာဖြစ်တယ်ဆိုတာလည်း method name ကနေ ချက်ချင်း သိနိုင်ပါတယ်။

### Real-world Java ဥပမာများ

- `StringBuilder` — `new StringBuilder().append("a").append("b")`
- `StringBuffer`
- `java.time.LocalDate` build-style APIs

### Abstract Factory နဲ့ ဘာကွာသလဲ

||Abstract Factory|Builder|
|---|---|---|
|Focus|related object **family** ကို ချက်ချင်း ဖန်တီး|object **တစ်ခုတည်း**ကို **step-by-step** ဖွဲ့စည်း|
|ဘယ်အခါသုံးလဲ|product family အမျိုးမျိုးကို switch လုပ်ချင်တဲ့အခါ|object ဟာ parameter/config အများကြီးရှိတဲ့အခါ|

### သတိထားရမယ့်အချက် (Trade-offs)

- Class ရေအတွက်ထပ်များလာတယ် (Builder inner class ထပ်ရေးရလို့)
- Simple object (field ၂-၃ ခုပဲရှိရင်) အတွက် Builder က overkill ဖြစ်နိုင်ပါတယ် — ရိုးရိုး constructor ပဲ လုံလောက်ပါတယ်

---

ဒါဆိုရင် **Creational Patterns** ၄ ခု ပြီးသွားပြီ — Singleton, Factory Method, Abstract Factory, Builder။ နောက်ဆုံးကျန်တဲ့ **Prototype** ကို ဆက်သင်မလား၊ ဒါမှမဟုတ် **Structural Patterns** (Adapter) ဘက် ကူးမလား။