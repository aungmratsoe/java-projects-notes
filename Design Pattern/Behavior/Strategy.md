## 🔹 Strategy Pattern

### ပြဿနာက ဘာလဲ

Task တစ်ခုကို **algorithm/နည်းလမ်း အမျိုးမျိုး** နဲ့ လုပ်နိုင်တဲ့အခါ ဖြစ်ပါတယ် — ဒါပေမယ့် ဘယ် algorithm သုံးမလဲဆိုတာကို `if-else` သို့ `switch` statement နဲ့ hard-code ထားရင် —

- Algorithm အသစ် ထပ်ထည့်ရင် ရှိပြီးသား class ကို ပြင်ရမယ် (Open/Closed Principle ဖောက်ဖျက်)
- Method တစ်ခုတည်းထဲမှာ algorithm အမျိုးမျိုး စုပြီး ရှုပ်ထွေးလာမယ်
- Runtime မှာ algorithm ကို **dynamically ပြောင်း** ဖို့ ခက်ခဲမယ်

ဥပမာ — Shopping cart တစ်ခုမှာ payment method (Credit Card, PayPal, Crypto) အမျိုးမျိုးနဲ့ ငွေချေချင်တယ် ဆိုပါစို့။

```java
// မကောင်းတဲ့ approach
void pay(String method, double amount) {
    if (method.equals("CREDIT_CARD")) {
        // credit card logic
    } else if (method.equals("PAYPAL")) {
        // paypal logic
    } else if (method.equals("CRYPTO")) {
        // crypto logic
    }
    // Payment method အသစ်ထည့်တိုင်း ဒီ method ကို ပြန်ပြင်ရဦးမယ်
}
```

### Strategy က ဘယ်လို ဖြေရှင်းလဲ

Algorithm တစ်ခုချင်းစီကို **သီးခြား class** (Strategy) အဖြစ် ခွဲထုတ်ပြီး၊ interface တစ်ခုတည်းအောက်မှာ ထားပါတယ်။ Client (Context) က Strategy object ကို **runtime မှာ inject** လုပ်ပြီး၊ algorithm ဘယ်ဟာသုံးမလဲဆိုတာကို Strategy object ကို **အစားထိုး (swap)** လိုက်ရုံနဲ့ ပြောင်းလို့ရပါတယ်.

### Java Code

```java
// Strategy interface - algorithm family တစ်ခုလုံးရဲ့ common interface
interface PaymentStrategy {
    void pay(double amount);
}
```

```java
// Concrete Strategies - algorithm တစ်ခုချင်းစီ
class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;

    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }

    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using Credit Card ending in " 
                            + cardNumber.substring(cardNumber.length() - 4));
    }
}

class PayPalPayment implements PaymentStrategy {
    private String email;

    public PayPalPayment(String email) { this.email = email; }

    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using PayPal account " + email);
    }
}

class CryptoPayment implements PaymentStrategy {
    private String walletAddress;

    public CryptoPayment(String walletAddress) { this.walletAddress = walletAddress; }

    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using Crypto wallet " + walletAddress);
    }
}
```

```java
// Context - Strategy object ကို ကိုင်ထားပြီး delegate လုပ်တယ်
class ShoppingCart {
    private PaymentStrategy paymentStrategy; // ⭐ interface ကိုပဲ ကိုင်ထား, concrete class မသိ

    // Strategy ကို runtime မှာ inject
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }

    public void checkout(double amount) {
        if (paymentStrategy == null) {
            throw new IllegalStateException("Payment strategy not set");
        }
        paymentStrategy.pay(amount); // ⭐ actual algorithm ကို delegate လုပ်တယ်
    }
}
```

သုံးပုံ -

```java
ShoppingCart cart = new ShoppingCart();

cart.setPaymentStrategy(new CreditCardPayment("1234567890123456"));
cart.checkout(150.0);
// Paid $150.0 using Credit Card ending in 3456

// ⭐ Runtime မှာ Strategy ကို ပြောင်းလိုက်ရုံ - ShoppingCart class ကို လုံးဝ မပြင်ရဘူး
cart.setPaymentStrategy(new PayPalPayment("alice@email.com"));
cart.checkout(75.0);
// Paid $75.0 using PayPal account alice@email.com
```

Payment method အသစ် (`ApplePayPayment`) ထပ်ထည့်ချင်ရင် — `ShoppingCart` class ကို **တစ်ကြောင်းမှ မပြင်ဘဲ**, class အသစ်တစ်ခု implement လုပ်ရုံပါပဲ။

### Real-world Java ဥပမာများ

- `Comparator` interface — `Collections.sort(list, comparator)` မှာ sorting algorithm ကို strategy အနေနဲ့ inject လုပ်တာ
- `java.util.concurrent` ရဲ့ `ThreadPoolExecutor` — `RejectedExecutionHandler` strategy
- Spring Security ရဲ့ authentication strategies

### Observer နဲ့ ဘာကွာသလဲ

||Observer|Strategy|
|---|---|---|
|ရည်ရွယ်ချက်|state change ကို **object များစွာဆီ notify**|algorithm/behavior ကို **runtime မှာ swap**|
|Relationship|one-to-**many** (Subject → Observers)|one-to-**one** (Context → Strategy တစ်ခုတည်း)|

### State Pattern နဲ့ လွဲမှားတတ်ကြောင်း

Strategy နဲ့ **State Pattern** ဟာ code structure **အတူတူ** ကလိုပါပဲ (interface + concrete implementations + context) — ဒါပေမယ့် ရည်ရွယ်ချက် ကွာပါတယ်: Strategy က client က algorithm ကို **ပြင်ပကနေ ရွေးချယ်**တာ၊ State ကတော့ object ကိုယ်တိုင်က **internal state အလိုက် behavior ကို ပြောင်း**တာ ဖြစ်ပါတယ် (နောက်မှ State Pattern ရောက်ရင် ပြန်နှိုင်းပါမယ်)။

### သတိထားရမယ့်အချက် (Trade-offs)

- Client (caller) က Strategy အမျိုးမျိုးရဲ့ **ကွာခြားချက်ကို သိထားရမယ်** — ဘယ် Strategy ကို ဘယ်အခါ သုံးရမလဲ ဆုံးဖြတ်ပေးရမှာ Context ရဲ့ အလုပ် မဟုတ်ပါဘူး
- Simple algorithm ၂ ခု၊ ၃ ခုပဲရှိရင် Strategy Pattern က overkill ဖြစ်နိုင်ပါတယ် — Java 8+ မှာဆိုရင် **lambda expression** သုံးပြီး class အပြည့်အစုံ မရေးဘဲ Strategy ကို ရိုးရှင်းအောင် ရေးလို့ရပါတယ်:

```java
interface PaymentStrategy {
    void pay(double amount);
}

// Lambda ကို Strategy အနေနဲ့ တိုက်ရိုက် pass လို့ရတယ်
cart.setPaymentStrategy(amt -> System.out.println("Paid $" + amt + " via Lambda Strategy"));
```

---

Behavioral pattern ၂ ခု ပြီးပါပြီ (Observer, Strategy)။ ဆက်သင်ချင်တဲ့ pattern ရွေးပါ —

- **Command Pattern**
- **Template Method Pattern**
- **State Pattern**
- ဒါမှမဟုတ် ဒီအထိ pattern ၁၁ ခုအတွက် quiz လေး လုပ်ကြည့်မလား