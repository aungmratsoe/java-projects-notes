JavaBeans (Swing context) အကြောင်း ရှင်းပြပေးပါမယ်။

## Bean ဆိုတာ ဘာလဲ?

**JavaBean** ဆိုတာ Java class တစ်ခုဖြစ်ပြီး၊ အောက်ပါ **rule (convention)** တွေကို လိုက်နာထားရင် "Bean" လို့ခေါ်ပါတယ်:

1. **`public no-argument constructor`** ရှိရမယ် (parameter မပါတဲ့ constructor)
2. **Property တွေက `private`** ဖြစ်ပြီး **`getXxx()` / `setXxx()`** method တွေနဲ့ access လုပ်ရမယ် (boolean ဆိုရင် `isXxx()`)
3. **`Serializable`** interface implement လုပ်ထားသင့်တယ် (object state ကို save/restore လုပ်နိုင်ဖို့)

```java
public class Student implements Serializable {
    private String name;
    private int age;

    public Student() { } // no-arg constructor - Bean requirement

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

ဒါလေးက Bean pattern ရဲ့ **အခြေခံ** ပါ - Model class (Student, Book, Employee) တွေကို ဒီ pattern အတိုင်း ရေးလေ့ရှိပါတယ်။

## Swing components တွေဟာ Beans ပါ

Swing component (JButton, JTextField, JLabel) **အားလုံးဟာ Bean rule ကို လိုက်နာထားတဲ့ Bean တွေပါပဲ**:

```java
JButton btn = new JButton();  // no-arg constructor ✓
btn.setText("Save");          // setter ✓
String text = btn.getText();  // getter ✓
```

ဒါကြောင့် **NetBeans GUI Builder** (drag-and-drop mode) က Component ကို Palette ကနေ drag ဆွဲချလိုက်ရင် auto ဖြစ်တာပါ - GUI Builder က Bean introspection (reflection သုံးပြီး getter/setter တွေကို auto detect) လုပ်ပြီး Properties panel ထဲမှာ setting တွေ (text, color, font) ပြပေးတာဖြစ်ပါတယ်။

## Bean Properties - 3 မျိုး

**1. Simple Property**

```java
private String name;
public String getName() { return name; }
public void setName(String name) { this.name = name; }
```

**2. Boolean Property** (getter က `is` prefix)

```java
private boolean active;
public boolean isActive() { return active; }
public void setActive(boolean active) { this.active = active; }
```

**3. Bound Property** (value ပြောင်းရင် listener တွေကို auto notify)

```java
public class Student implements Serializable {
    private String name;
    private PropertyChangeSupport support = new PropertyChangeSupport(this);

    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }

    public String getName() { return name; }

    public void setName(String newName) {
        String oldName = this.name;
        this.name = newName;
        support.firePropertyChange("name", oldName, newName); // change event ပို့တယ်
    }
}
```

## Bean Persistence - `XMLEncoder`/`XMLDecoder`

Bean rule လိုက်နာထားလို့ Object state ကို XML file အနေနဲ့ save/load လုပ်နိုင်ပါတယ် (settings, preferences စတာတွေအတွက် အသုံးဝင်):

```java
// Save Bean state to XML
Student s = new Student();
s.setName("Aung Aung");
s.setAge(15);

XMLEncoder encoder = new XMLEncoder(new BufferedOutputStream(new FileOutputStream("student.xml")));
encoder.writeObject(s);
encoder.close();

// Load Bean state back from XML
XMLDecoder decoder = new XMLDecoder(new BufferedInputStream(new FileInputStream("student.xml")));
Student loaded = (Student) decoder.readObject();
decoder.close();
```

## NetBeans GUI Builder နဲ့ ဆက်စပ်ပုံ

QRCode project မှာ GUI Builder (drag-and-drop) သုံးနေတာမို့ ဒီအရာက **အလိုအလျောက် အလုပ်ဖြစ်နေပါတယ်**:

- Component (JTextField) ကို drag ဆွဲချလိုက်တဲ့အခါ, Properties window ထဲမှာ "text", "font", "background" စတာတွေ ပြတာက **Bean introspection** သုံးလို့ပါ
- `initComponents()` method ထဲမှာ auto-generate ဖြစ်တဲ့ code တွေမှာ `setText()`, `setFont()` စတာတွေ တွေ့ရမှာဖြစ်ပြီး ၎င်းတို့က Bean setter method တွေကို ခေါ်နေတာပါ

**Model class (Student, ဥပမာ) ကို Bean pattern အတိုင်း ရေးထားရင်**:

- Table (JTable) ထဲ data ချိတ်တဲ့အခါ `BeanTableModel`-style approach တွေ သုံးလို့ရတယ်
- `XMLEncoder`/`XMLDecoder` သုံးပြီး Student list ကို config/backup file အနေနဲ့ save လုပ်လို့ရတယ်

---

**Summary**: Bean ဆိုတာ "no-arg constructor + getter/setter + Serializable" ဆိုတဲ့ convention ကို လိုက်နာတဲ့ Java class ပါ။ Swing component တွေကိုယ်တိုင်က Bean ဖြစ်ပြီး၊ Model class (Student) တွေကိုလည်း ဒီ pattern အတိုင်း ရေးလေ့ရှိတာက NetBeans GUI Builder, data binding, serialization စတာတွေနဲ့ smooth ဖြစ်အောင် ဖြစ်ပါတယ်။

QRCode project ရဲ့ Student model class ကို Bean pattern reference လုပ်ပြီး review ချင်ရင် ဒါမှမဟုတ် XMLEncoder သုံးပြီး backup/export feature ထည့်ချင်ရင် ပြောပါ။