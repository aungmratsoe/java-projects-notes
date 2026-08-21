## 🔹 Facade Pattern

### ပြဿနာက ဘာလဲ

Complex subsystem တစ်ခုကို သုံးဖို့ **class များစွာ**, ဆက်နွယ်နေတဲ့ **method များစွာ** ကို order အလိုက် ခေါ်ရတတ်ပါတယ်။ Client code ရဲ့ တစ်နေရာစီမှာ ဒီ complex logic တွေကို ထပ်ခါထပ်ခါ ရေးရရင် — code ဟာ subsystem ရဲ့ internal detail တွေနဲ့ **tightly coupled** ဖြစ်သွားပါတယ်။

ဥပမာ — video conversion library တစ်ခု သုံးမယ်ဆိုပါစို့: `VideoFile`, `Codec`, `CodecFactory`, `BitrateReader`, `AudioMixer` — class ၅ ခုကို order မှန်အောင် ခေါ်မှ video တစ်ခု convert လုပ်ပြီးမှာပါ။ Client code တိုင်းက ဒီ sequence တစ်ခုလုံးကို သိထားရမယ်ဆိုရင် error-prone ဖြစ်ပြီး maintain လုပ်ရခက်ပါတယ်။

### Facade က ဘယ်လို ဖြေရှင်းလဲ

Complex subsystem ရှေ့မှာ **ရိုးရှင်းတဲ့ interface** တစ်ခု (Facade class) ထားပေးပါတယ် — client က Facade ရဲ့ method ရိုးရိုးလေးကိုပဲ ခေါ်ရုံနဲ့ အလုပ်ပြီးသွားပါတယ်၊ အတွင်းက complexity ကို client က **သိစရာမလိုတော့** ပါဘူး။

### Java Code

```java
// Complex subsystem classes
class CPU {
    public void freeze() { System.out.println("CPU: freeze"); }
    public void jump(long position) { System.out.println("CPU: jump to " + position); }
    public void execute() { System.out.println("CPU: execute"); }
}

class Memory {
    public void load(long position, byte[] data) {
        System.out.println("Memory: loading data at " + position);
    }
}

class HardDrive {
    public byte[] read(long lba, int size) {
        System.out.println("HardDrive: reading sector " + lba);
        return new byte[size];
    }
}
```

```java
// ⭐ Facade - subsystem class တွေကို ခေါ်ပြီး ရိုးရှင်းတဲ့ method တစ်ခု ပေး
class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;

    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }

    // client အတွက် method တစ်ခုတည်း - အတွင်းက steps အများကြီးကို ဖျောက်ထား
    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}
```

သုံးပုံ -

```java
// Facade မရှိရင် client က CPU, Memory, HardDrive အားလုံးကို သိပြီး order မှန်အောင် ခေါ်ရမှာ
// Facade ရှိရင်တော့...
ComputerFacade computer = new ComputerFacade();
computer.start(); // steps အားလုံးကို client မသိတော့ဘူး
```

### Real-world Java ဥပမာများ

- `javax.faces.context.FacesContext` — JSF framework ရဲ့ complex object များစွာကို ဖုံးထား
- Spring Framework ရဲ့ `JdbcTemplate` — JDBC ရဲ့ boilerplate code (connection, statement, resultset handling) တွေကို ဖုံးပြီး `query()` method ရိုးရိုးလေးပဲ ပေးထား

### Adapter နဲ့ ဘာကွာသလဲ

||Adapter|Facade|
|---|---|---|
|Object အရေအတွက်|object **တစ်ခု** ကို interface ကူးတို|object/subsystem **များစွာ** ကို ရိုးရှင်းအောင် wrap|
|ရည်ရွယ်ချက်|interface မတူတာကို **ကိုက်အောင်** လုပ်|complexity ကို **ဖုံးအုပ်**|

Facade က subsystem class တွေကို **restrict မထား** ပါဘူး — client လိုအပ်ရင် subsystem class တွေကို တိုက်ရိုက် access လုပ်ချင်ရင်လည်း လုပ်နိုင်ပါသေးတယ်။ Facade က **convenience layer** တစ်ခုပဲ ဖြစ်ပါတယ်။

### သတိထားရမယ့်အချက် (Trade-offs)

- Facade ဟာ application ထဲက class **အားလုံး** နဲ့ coupled ဖြစ်လာနိုင်တဲ့အတွက် **"god object"** ဖြစ်မသွားအောင် ဂရုစိုက်ရပါတယ်
- Subsystem ကို flexibility မလိုချင်ဘဲ **simple access** ပဲ လိုချင်ရင် အသုံးဝင်ဆုံးပါတယ်

---

**Structural Patterns** ၃ ခု ပြီးပါပြီ (Adapter, Decorator, Facade) — ကျန်တာက Composite, Proxy ရှိပါသေးတယ်။ ဆက်သင်ချင်ပါသလား၊ ဒါမှမဟုတ် ဒီအထိ pattern ၈ ခုအတွက် quiz လေး အရင်လုပ်ကြည့်ချင်ပါသလား။