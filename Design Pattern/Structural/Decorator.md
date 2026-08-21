## 🔹 Decorator Pattern

### ပြဿနာက ဘာလဲ

Object တစ်ခုကို **runtime မှာ** feature/behavior အသစ် ထပ်ထည့်ချင်တဲ့အခါ ဖြစ်ပါတယ် — ဒါပေမယ့် **inheritance (subclass) သုံးရင် class အရေအတွက် ပေါက်ကွဲသွားနိုင်** ပါတယ်။

ဥပမာ — Coffee shop application တစ်ခု စဉ်းစားကြည့်ပါ: `Coffee`, `CoffeeWithMilk`, `CoffeeWithSugar`, `CoffeeWithMilkAndSugar`, `CoffeeWithMilkAndSugarAndWhip`... — combination တစ်ခုစီအတွက် subclass အသစ်ရေးရင် **exponentially** class တွေ ပွားလာပါတယ်။

```java
// မကောင်းတဲ့ approach - class explosion
class Coffee {}
class CoffeeWithMilk extends Coffee {}
class CoffeeWithSugar extends Coffee {}
class CoffeeWithMilkAndSugar extends Coffee {}
// topping ၅ မျိုးရှိရင် combination 2^5 = 32 ခု!
```

### Decorator က ဘယ်လို ဖြေရှင်းလဲ

Object ကို **wrapper object** တွေနဲ့ **အလွှာလိုက် ဝိုင်းထားပြီး** behavior အသစ် ထပ်ထည့်ပါတယ် — inheritance အစား **composition** ကို သုံးပါတယ်။ Decorator တစ်ခုစီဟာ base interface တစ်ခုတည်းကို implement လုပ်ပြီး၊ object ရှိပြီးသားကို wrap ထားကာ behavior အသစ် ထပ်ဖြည့်ပါတယ်။

### Java Code

```java
// Component interface
interface Coffee {
    String getDescription();
    double getCost();
}
```

```java
// Concrete Component - base object
class SimpleCoffee implements Coffee {
    public String getDescription() { return "Coffee"; }
    public double getCost() { return 2.0; }
}
```

```java
// Base Decorator - Coffee ကို implement လုပ်ပြီး, Coffee object တစ်ခုကို ကိုင်ထား
abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee; // ⭐ wrap လုပ်ထားတဲ့ object

    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }

    public String getDescription() { return decoratedCoffee.getDescription(); }
    public double getCost() { return decoratedCoffee.getCost(); }
}
```

```java
// Concrete Decorators - behavior တစ်ခုချင်းစီ ထပ်ဖြည့်
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) { super(coffee); }

    public String getDescription() {
        return decoratedCoffee.getDescription() + " + Milk";
    }
    public double getCost() {
        return decoratedCoffee.getCost() + 0.5;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) { super(coffee); }

    public String getDescription() {
        return decoratedCoffee.getDescription() + " + Sugar";
    }
    public double getCost() {
        return decoratedCoffee.getCost() + 0.2;
    }
}

class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(Coffee coffee) { super(coffee); }

    public String getDescription() {
        return decoratedCoffee.getDescription() + " + Whip";
    }
    public double getCost() {
        return decoratedCoffee.getCost() + 0.7;
    }
}
```

သုံးပုံ -

```java
Coffee coffee = new SimpleCoffee();
System.out.println(coffee.getDescription() + " = $" + coffee.getCost());
// Coffee = $2.0

// ⭐ Decorator တွေကို လိုသလို layer တင်နိုင်တယ် (runtime မှာ)
Coffee myCoffee = new WhipDecorator(new MilkDecorator(new SugarDecorator(new SimpleCoffee())));
System.out.println(myCoffee.getDescription() + " = $" + myCoffee.getCost());
// Coffee + Sugar + Milk + Whip = $3.4
```

Topping ၅ မျိုးရှိရင်တောင် — subclass ၃၂ ခု **မလိုအပ်ဘဲ**, Decorator class **၅ ခုတည်း** နဲ့ combination **အားလုံး** ကို runtime မှာ freely ပေါင်းစပ်လို့ရပါတယ်။

### Real-world Java ဥပမာများ

- `java.io` package တစ်ခုလုံးဟာ Decorator pattern အပေါ်မှာ တည်ဆောက်ထားတာပါ:

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("file.txt")));
// FileInputStream ကို InputStreamReader က wrap, ပြီးတော့ BufferedReader က ထပ် wrap
```

- `Collections.synchronizedList()`, `Collections.unmodifiableList()`

### Adapter နဲ့ ဘာကွာသလဲ

||Adapter|Decorator|
|---|---|---|
|ရည်ရွယ်ချက်|interface **မတူတာကို ကိုက်အောင်** ချိတ်|interface **တူတဲ့ object ကို behavior ထပ်ဖြည့်**|
|Interface|အသစ် interface ကို implement|**interface တူတူ** ပဲ ဆက်ထား|

### သတိထားရမယ့်အချက် (Trade-offs)

- Object ကို layer အများကြီး wrap ရင် **debug လုပ်ရခက်** တတ်ပါတယ် (stack trace ရှည်လာလို့)
- Small wrapper class များစွာ ဖန်တီးရတာကြောင့် code ထဲမှာ class အရေအတွက် တိုးလာနိုင်ပါတယ်

---

**Structural Patterns** ၂ ခု ပြီးပါပြီ (Adapter, Decorator)။ ဆက်ပြီး **Facade Pattern** ကို သင်ချင်ပါသလား၊ ဒါမှမဟုတ် ဒီအထိ pattern ၇ ခုအတွက် quiz လေး လုပ်ကြည့်ချင်ပါသလား။