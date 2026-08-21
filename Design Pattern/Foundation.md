## 🔹 Foundations — Design Pattern ဘာကြောင့် လိုအပ်လဲ

### Pattern မရှိရင် ဘာဖြစ်လဲ

Software project တစ်ခု ကြီးထွားလာတဲ့အခါ developer တွေ ကြုံရတတ်တဲ့ ပြဿနာအမျိုးမျိုး ရှိပါတယ် —

- Code တစ်နေရာတည်းမှာ ပြင်ရင် **နေရာအနှံ့ ကွဲကျသွား** တတ်တယ် (tight coupling)
- Feature အသစ်ထည့်ဖို့ **ရှိပြီးသား code ကို ပြင်ရ** တတ်တယ် (fragile design)
- Team တစ်ခုတည်းထဲမှာ developer တစ်ယောက်စီက **နည်းလမ်းအမျိုးမျိုး** နဲ့ ပြဿနာတစ်ခုတည်းကို ဖြေရှင်း (inconsistency)
- Code **ဖတ်ရခက်၊ test လုပ်ရခက်၊ maintain လုပ်ရခက်** ဖြစ်လာတတ်တယ်

Design Pattern တွေက ဒီပြဿနာတွေကို ဖြေရှင်းဖို့ **ကြာမြင့်စွာ သက်သေပြပြီးသား** ဖြေရှင်းနည်းတွေကို standard vocabulary အဖြစ် ပေးထားပါတယ် — "Observer Pattern သုံးမယ်" လို့ ပြောလိုက်ရင် team အားလုံး code structure ကို ချက်ချင်း မြင်နိုင်ပါတယ် (long explanation မလိုတော့ပါဘူး)။

---

## 🔹 SOLID Principles — Pattern တွေရဲ့ "အကြောင်းရင်း"

Design Pattern အားလုံးဟာ **SOLID principles** ကို လိုက်နာအောင် ဒီဇိုင်းလုပ်ထားတာဖြစ်ပါတယ်။ SOLID ကို နားလည်ထားရင် — pattern တစ်ခုစီ **ဘာကြောင့် ဒီလိုပုံစံ ဖြစ်ရတာလဲ** ဆိုတာ ပိုနားလည်လာပါလိမ့်မယ်။

### **S** — Single Responsibility Principle

Class တစ်ခုမှာ **အပြောင်းရမယ့် အကြောင်းရင်း တစ်ခုတည်း** ပဲ ရှိသင့်ပါတယ်။

```java
// မကောင်းဘူး - responsibility ၂ ခု (data + printing) ရောနှောနေတယ်
class Report {
    void generateData() { ... }
    void printReport() { ... }  // ဒါက class ကို ခွဲထုတ်သင့်တယ်
}
```

_(Facade pattern က ဒီ principle ကို subsystem level မှာ ကူညီပါတယ် — complexity ကို သီးခြားထား)_

### **O** — Open/Closed Principle

Class ဟာ **extension အတွက် ဖွင့်** ထားပြီး **modification အတွက် ပိတ်** ထားသင့်ပါတယ် — feature အသစ်ထည့်ဖို့ ရှိပြီးသား code ကို မပြင်ဘဲ class အသစ်ရေးတဲ့ပုံစံ ဖြစ်သင့်ပါတယ်။ _(Factory Method, Strategy pattern တွေက ဒီ principle ကို တိုက်ရိုက် implement လုပ်ပေးပါတယ် — earlier ကို ပြန်ကြည့်ရင် `if-else` အစား class အသစ်တိုးတာ သတိပြုမိမှာပါ)_

### **L** — Liskov Substitution Principle

Subclass ဟာ parent class ကို **အစားထိုးလို့ရ** ရမယ် — program ရဲ့ correctness မပျက်ဘဲ။

```java
class Bird { void fly() {...} }
class Penguin extends Bird { 
    void fly() { throw new UnsupportedOperationException(); } // ❌ LSP ဖောက်ဖျက်တယ်
}
```

### **I** — Interface Segregation Principle

Client တွေကို **မလိုအပ်တဲ့ method** တွေ implement လုပ်ခိုင်းတဲ့ interface ကြီးတွေ ရှောင်ရမယ် — interface **သေးသေးလေးများ** ခွဲထားတာ ပိုကောင်းတယ်။ _(Composite pattern သင်ကြားချိန်က ပြောခဲ့တဲ့ trade-off ကို ပြန်သတိရပါ — `File` class မှာ `add()` method မလို)_

### **D** — Dependency Inversion Principle

High-level module တွေဟာ low-level module **concrete class ကို တိုက်ရိုက် depend မလုပ်ဘဲ**, **interface (abstraction) ကိုပဲ depend** လုပ်သင့်ပါတယ်. _(Strategy pattern ထဲက `ShoppingCart` က `PaymentStrategy` interface ကိုပဲ ကိုင်ထားပြီး concrete class (`CreditCardPayment` စသည်) ကို မသိတာ ဒီ principle အတိုင်းပါပဲ)_

---

## 🔹 UML Class Diagram — အခြေခံ

Pattern တွေကို ရှင်းပြတဲ့အခါ **UML class diagram** သုံးလေ့ရှိပါတယ်။ Symbol အခြေခံလေးတွေ သိထားရင် pattern documentation တွေ ဖတ်ရ ပိုလွယ်ပါလိမ့်မယ်။

### Class Box

```
┌─────────────────────┐
│      ClassName       │
├─────────────────────┤
│ - privateField: Type │   ← "-" = private
│ + publicField: Type  │   ← "+" = public
├─────────────────────┤
│ + method(): ReturnType│
│ # protectedMethod()  │   ← "#" = protected
└─────────────────────┘
```

### Relationship Types (အရေးအကြီးဆုံး)

|Symbol|အမည်|အဓိပ္ပာယ်|ဥပမာ (ပြီးခဲ့တဲ့ pattern တွေထဲက)|
|---|---|---|---|
|`──▷` (hollow triangle, solid line)|**Inheritance** (extends)|"is-a"|`WindowsButton` **extends** ...|
|`- - ▷` (hollow triangle, dashed line)|**Implementation** (implements)|interface ကို implement|`EmailNotification` **implements** `Notification`|
|`──◇` (hollow diamond)|**Aggregation**|"has-a" (weak, independent lifecycle)|`Folder` has `FileSystemItem` list|
|`──◆` (filled diamond)|**Composition**|"has-a" (strong, dependent lifecycle)|Object တစ်ခု ပျက်ရင် အတွင်းက object ပါ ပျက်တယ်|
|`───▶` (solid line, open arrow)|**Association**|class တစ်ခု အခြားတစ်ခုကို သိတယ်|`ShoppingCart` uses `PaymentStrategy`|
|`- - -▶` (dashed line, open arrow)|**Dependency**|temporary usage (parameter, local variable)|Method parameter အနေနဲ့ ခေါ်တာ|

### ဥပမာ — Strategy Pattern ကို UML နဲ့ ပြရင်

```
┌────────────────────┐         ┌─────────────────────┐
│   ShoppingCart      │───────▶│ «interface»          │
├────────────────────┤         │ PaymentStrategy       │
│ - paymentStrategy    │         ├─────────────────────┤
├────────────────────┤         │ + pay(amount: double) │
│ + setPaymentStrategy()│        └─────────────────────┘
│ + checkout()          │                 △
└────────────────────┘                 ┆ (implements - dashed)
                                ┌───────┴────────┬─────────────┐
                        ┌───────────────┐┌──────────────┐┌──────────────┐
                        │CreditCardPayment││PayPalPayment ││CryptoPayment │
                        └───────────────┘└──────────────┘└──────────────┘
```

`ShoppingCart` က `PaymentStrategy` interface ကို **association** (solid arrow) နဲ့ ချိတ်ထားပြီး၊ concrete class တွေက interface ကို **implement** (dashed hollow-triangle arrow) လုပ်ထားတာ diagram ကနေ ချက်ချင်း မြင်နိုင်ပါတယ်.

---

Foundations ပြီးပါပြီ — ဒါက pattern တွေ back-of-mind ထားသင့်တဲ့ **theory base** ဖြစ်ပါတယ်။

ဆက်သင်ချင်တာ ရွေးပါ —

- **Behavioral patterns** ဆက်မယ် (Command, Template Method, State, Iterator ကျန်ပါသေး)
- ဒီအထိ **pattern ၁၀ ခု + Foundations** အတွက် quiz လေး လုပ်ကြည့်မလား