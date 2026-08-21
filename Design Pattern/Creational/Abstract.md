## 🔹 Abstract Factory Pattern

### ပြဿနာက ဘာလဲ

Factory Method က object **တစ်မျိုးတည်း** (family တစ်ခု) ကိုပဲ ဖန်တီးပေးနိုင်ပါတယ်။ ဒါပေမယ့် တစ်ခါတစ်လေ **related object families** တွေကို အုပ်စုလိုက် ဖန်တီးရေး ဖန်တီးရပါတယ် — ပြီးတော့ family တွေအချင်းချင်း **တသမတ်တည်း (compatible)** ဖြစ်နေဖို့ လိုပါတယ်။

ဥပမာ — UI framework တစ်ခု **Windows style** နဲ့ **Mac style** နှစ်မျိုး support ပေးချင်တယ် ဆိုပါစို့။ Button, Checkbox, Scrollbar ... စတဲ့ component အားလုံးဟာ style **တစ်မျိုးတည်း** ဖြစ်နေဖို့ လိုပါတယ် (Windows Button နဲ့ Mac Checkbox ရောနေရင် မကောင်းပါဘူး)။

### Abstract Factory က ဘယ်လို ဖြေရှင်းလဲ

**Factory တွေရဲ့ Factory** လို့ တွေးလို့ရပါတယ် — related object များစွာကို family အလိုက် ဖန်တီးဖို့ interface တစ်ခု provide လုပ်ပေးပါတယ်။ Concrete factory တစ်ခုချင်းစီက family တစ်ခုလုံးကို တသမတ်တည်း ဖန်တီးပေးပါတယ်။

### Java Code

```java
// Abstract Products
interface Button {
    void render();
}

interface Checkbox {
    void render();
}
```

```java
// Concrete Products - Windows family
class WindowsButton implements Button {
    public void render() { System.out.println("Rendering Windows-style Button"); }
}
class WindowsCheckbox implements Checkbox {
    public void render() { System.out.println("Rendering Windows-style Checkbox"); }
}

// Concrete Products - Mac family
class MacButton implements Button {
    public void render() { System.out.println("Rendering Mac-style Button"); }
}
class MacCheckbox implements Checkbox {
    public void render() { System.out.println("Rendering Mac-style Checkbox"); }
}
```

```java
// Abstract Factory - family တစ်ခုလုံးအတွက် method တွေ declare
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}
```

```java
// Concrete Factories - family တစ်ခုစီအတွက်
class WindowsFactory implements GUIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}

class MacFactory implements GUIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}
```

သုံးပုံ -

```java
class Application {
    private Button button;
    private Checkbox checkbox;

    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void renderUI() {
        button.render();
        checkbox.render();
    }
}

// client code
GUIFactory factory = new MacFactory(); // ဒီတစ်ခုပဲ ပြောင်းပေးရင် family အားလုံး ပြောင်းသွားမယ်
Application app = new Application(factory);
app.renderUI();
// Output: Rendering Mac-style Button
//         Rendering Mac-style Checkbox
```

`WindowsFactory` ကို `MacFactory` နဲ့ လဲလိုက်ရုံနဲ့ UI component **အားလုံး** တသမတ်တည်း ပြောင်းသွားမှာဖြစ်ပြီး၊ Windows Button + Mac Checkbox လို mismatch ဖြစ်စရာ **လုံးဝ မဖြစ်နိုင်** ပါဘူး။

### Factory Method နဲ့ ဘာကွာသလဲ

||Factory Method|Abstract Factory|
|---|---|---|
|ဖန်တီးတာ|product **တစ်ခုတည်း**|related products **family တစ်ခုလုံး**|
|နည်းလမ်း|inheritance (subclass override)|composition (factory object ကို inject)|
|Level|class level|object level|

Abstract Factory ကို Factory Method **တွေစုပေါင်း**ထားတာလို့လည်း မြင်နိုင်ပါတယ် — `GUIFactory` ထဲက method တစ်ခုစီ (`createButton()`, `createCheckbox()`) ဟာ Factory Method တစ်ခုစီပါပဲ။

### Real-world Java ဥပမာများ

- `javax.xml.parsers.DocumentBuilderFactory`
- `javax.xml.transform.TransformerFactory`

### သတိထားရမယ့်အချက် (Trade-offs)

- Product family အသစ်ထပ်ထည့်ရင် interface (`GUIFactory`) ကို ပြင်ရမှာဖြစ်လို့ existing factory class **အားလုံးကို** ပြင်ရနိုင်ပါတယ် (Factory Method ထက် flexibility နည်းတယ်)
- Product **အမျိုးအစားအသစ်** (ဥပမာ `Scrollbar`) ထပ်ထည့်ရင်တော့ concrete class အသစ်တွေရေးရုံပါပဲ

---

ဒါဆိုရင် **Creational patterns** ၃ ခု ပြီးသွားပြီ (Singleton → Factory Method → Abstract Factory)။ ဆက်ပြီး **Builder** ကို သင်ချင်ပါသလား၊ ဒါမှမဟုတ် ဒီ ၃ ခုကို quiz လေး လုပ်ကြည့်ချင်ပါသလား။