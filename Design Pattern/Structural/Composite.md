## 🔹 Composite Pattern

### ပြဿနာက ဘာလဲ

**Tree structure** (parent-child hierarchy) ဖြစ်နေတဲ့ object တွေကို handle လုပ်ရတဲ့အခါ ဖြစ်ပါတယ် — ဥပမာ file system (folder ထဲမှာ file တွေနဲ့ sub-folder တွေ ရှိတာ), UI component tree (panel ထဲမှာ button, panel အခြား ရှိတာ)။

ပြဿနာက — **"individual object" (leaf)** နဲ့ **"group of objects" (composite)** ကို client code က **ခွဲခြားပြီး** handle လုပ်ရင် code ထဲမှာ `if (isLeaf) ... else ...` type checking တွေ အနှံ့ ဖြစ်လာတတ်ပါတယ်။

```java
// မကောင်းတဲ့ approach
void printSize(Object item) {
    if (item instanceof File) {
        // file size ပြ
    } else if (item instanceof Folder) {
        // folder ထဲက file တွေအားလုံး loop ပြီး sum
    }
}
```

### Composite က ဘယ်လို ဖြေရှင်းလဲ

**Leaf (individual object)** နဲ့ **Composite (group)** ကို **interface တစ်ခုတည်း** အောက်မှာ ထားပြီး၊ client က နှစ်ခုလုံးကို **တူညီတဲ့ နည်းလမ်းနဲ့ပဲ** treat လုပ်နိုင်အောင် လုပ်ပါတယ် — "part-whole hierarchy" ကို tree structure အနေနဲ့ ဖွဲ့စည်းပါတယ်။

### Java Code

```java
// Component interface - Leaf နဲ့ Composite နှစ်ခုလုံး implement လုပ်မယ့် common interface
interface FileSystemItem {
    void showDetails(String indent);
    long getSize();
}
```

```java
// Leaf - individual object (child မရှိ)
class File implements FileSystemItem {
    private String name;
    private long size;

    public File(String name, long size) {
        this.name = name;
        this.size = size;
    }

    public void showDetails(String indent) {
        System.out.println(indent + "📄 " + name + " (" + size + "KB)");
    }

    public long getSize() { return size; }
}
```

```java
// Composite - group object, child (leaf သို့ composite) တွေကို list ထဲသိမ်းထား
class Folder implements FileSystemItem {
    private String name;
    private List<FileSystemItem> children = new ArrayList<>();

    public Folder(String name) { this.name = name; }

    public void add(FileSystemItem item) {
        children.add(item);
    }

    public void showDetails(String indent) {
        System.out.println(indent + "📁 " + name + "/");
        for (FileSystemItem item : children) {
            item.showDetails(indent + "  "); // ⭐ recursive - child ဟာ File ဖြစ်ဖြစ် Folder ဖြစ်ဖြစ် ခေါ်ပုံတူ
        }
    }

    public long getSize() {
        long total = 0;
        for (FileSystemItem item : children) {
            total += item.getSize(); // Folder ဆိုရင် ဒီထဲက recursive ခေါ်ဆိုမှု ထပ်ဖြစ်မယ်
        }
        return total;
    }
}
```

သုံးပုံ -

```java
File file1 = new File("resume.pdf", 200);
File file2 = new File("photo.jpg", 500);

Folder documents = new Folder("Documents");
documents.add(file1);
documents.add(file2);

Folder subFolder = new Folder("Projects");
subFolder.add(new File("code.java", 50));

Folder root = new Folder("Root");
root.add(documents);
root.add(subFolder); // Folder ကို Folder ထဲ ထည့်လို့ရတယ် - tree ဖွဲ့တယ်

root.showDetails("");
// 📁 Root/
//   📁 Documents/
//     📄 resume.pdf (200KB)
//     📄 photo.jpg (500KB)
//   📁 Projects/
//     📄 code.java (50KB)

System.out.println("Total size: " + root.getSize() + "KB"); // 750KB
```

Client (`root.showDetails()` ခေါ်တဲ့နေရာ) က **`File` ဖြစ်ဖြစ် `Folder` ဖြစ်ဖြစ်** `FileSystemItem` interface အနေနဲ့ပဲ တူညီစွာ treat လုပ်ထားတာ သတိပြုပါ — `instanceof` check လုံးဝ **မလိုအပ်** ပါဘူး။

### Real-world Java ဥပမာများ

- Java Swing — `Container` (Composite) ထဲမှာ `Component` (Leaf) တွေ သို့မဟုတ် `Container` အခြားတွေ ထည့်နိုင်တယ်
- `javax.xml` DOM tree — `Node` interface ကို `Element`, `Text` စတာတွေက implement လုပ်တယ်

### Decorator နဲ့ ဘာကွာသလဲ

နှစ်ခုစလုံး **tree-like recursive structure** သုံးပေမယ့် —

||Decorator|Composite|
|---|---|---|
|Structure|**linear chain** (wrapper တစ်ခုချင်း)|**tree** (branching, child များစွာ)|
|ရည်ရွယ်ချက်|behavior **ထပ်ဖြည့်**|part-whole **hierarchy ဖွဲ့စည်း**|

### သတိထားရမယ့်အချက် (Trade-offs)

- Interface ကို **general** ဖြစ်အောင် design လုပ်ရတာကြောင့် တစ်ခါတစ်လေ Leaf class တွေမှာ **သက်ဆိုင်မှုမရှိတဲ့ method** တွေ (ဥပမာ `add()` ကို `File` မှာ သုံးလို့မရသင့်ပါဘူး) ရှိလာနိုင်ပါတယ် — ဒါက Interface Segregation Principle နဲ့ တင်းကျပ်စွာ လိုက်နာရခက်စေတတ်ပါတယ်
- Tree ကို **အလွန်ယေဘူယျ** ဖြစ်အောင်ထားရင် type-safety ဆုံးရှုံးနိုင်ပါတယ်

---

**Structural Patterns** ၄ ခု ပြီးပါပြီ (Adapter, Decorator, Facade, Composite) — ကျန်တာက **Proxy** တစ်ခုတည်းပါပဲ။ ဆက်သင်ချင်ပါသလား၊ ဒါမှမဟုတ် ဒီအထိ pattern ၉ ခုအတွက် quiz လေး အရင်လုပ်ကြည့်ချင်ပါသလား။