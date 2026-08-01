Noted — FlatLaf ကို သီးခြား topic တစ်ခုအနေနဲ့ နောက်မှာ ဆက်သင်ပေးပါမယ် (Layout Managers လေ့လာပြီးရင် FlatLaf ကို default look and feel အဖြစ် ဘယ်လိုထည့်မလဲဆိုတာ ပြပေးနိုင်ပါတယ်)။

## Part 2: Layout Managers

Layout Manager ဆိုတာ container (JPanel, JFrame) ထဲက components တွေကို **ဘယ်လိုစီစဉ်၊ ဘယ်နေရာမှာထားမလဲ** ဆိုတာကို ကိုင်တွယ်ပေးတဲ့ algorithm ပါ။ Layout manager မရှိရင် components တွေရဲ့ position/size ကို pixel by pixel manual တွက်ရမှာဖြစ်လို့ ဒါက Swing UI ဆောက်ရာမှာ **အရေးအကြီးဆုံး concept** တွေထဲက တစ်ခုပါ။

### 1. FlowLayout (Default layout of JPanel)

Components တွေကို **left-to-right** row အလိုက် စီပေးပါတယ်။ Row ပြည့်ရင် နောက် row ကို auto-ချသွားပါတယ်။

```java
JPanel panel = new JPanel();  // default layout = FlowLayout
panel.setLayout(new FlowLayout(FlowLayout.CENTER, 10, 10));  // alignment, hgap, vgap

panel.add(new JButton("Button 1"));
panel.add(new JButton("Button 2"));
panel.add(new JButton("Button 3"));
```

- `FlowLayout.LEFT`, `FlowLayout.CENTER`, `FlowLayout.RIGHT` - alignment
- `hgap`, `vgap` - components ကြားက horizontal/vertical space

**ဘယ်အခါသုံးလဲ:** simple form တွေ၊ button group တွေအတွက် ကောင်းပါတယ်။ ဒါပေမယ့် window resize လုပ်ရင် components position က shift သွားနိုင်လို့ complex UI အတွက် မသင့်လျော်ပါဘူး။

### 2. BorderLayout (Default layout of JFrame)

Container ကို **5 regions** ခွဲထားပါတယ်: North, South, East, West, Center

```java
JFrame frame = new JFrame();
frame.setLayout(new BorderLayout());

frame.add(new JButton("North"), BorderLayout.NORTH);
frame.add(new JButton("South"), BorderLayout.SOUTH);
frame.add(new JButton("East"), BorderLayout.EAST);
frame.add(new JButton("West"), BorderLayout.WEST);
frame.add(new JButton("Center"), BorderLayout.CENTER);
```

```
+------------------NORTH------------------+
|                                          |
| WEST |          CENTER          | EAST  |
|                                          |
+------------------SOUTH------------------+
```

**Rules:**

- North/South က full width ယူပြီး height ကို fix လုပ်ပါတယ်
- East/West က full height ယူပြီး width ကို fix လုပ်ပါတယ်
- Center က ကျန်တဲ့ space အားလုံးကို ယူပါတယ် (window resize လုပ်ရင် stretch ဖြစ်ပါတယ်)
- Region တစ်ခုစီမှာ component **တစ်ခုသာ** ထည့်လို့ရပါတယ် (အများကြီးထည့်ချင်ရင် JPanel နဲ့ wrap လုပ်ရပါမယ်)

**ဘယ်အခါသုံးလဲ:** application ရဲ့ main window layout - toolbar (North), status bar (South), sidebar (West), main content (Center) စသဖြင့် standard app structure တွေအတွက် ကောင်းပါတယ်။

### 3. GridLayout

Components တွေကို **equal-size grid** (rows x columns) ပုံစံနဲ့ စီပါတယ်။

```java
JPanel panel = new JPanel();
panel.setLayout(new GridLayout(3, 2, 5, 5));  // 3 rows, 2 columns, hgap=5, vgap=5

for (int i = 1; i <= 6; i++) {
    panel.add(new JButton("Button " + i));
}
```

```
+--------+--------+
| Btn 1  | Btn 2  |
+--------+--------+
| Btn 3  | Btn 4  |
+--------+--------+
| Btn 5  | Btn 6  |
+--------+--------+
```

Component တစ်ခုစီရဲ့ size က **တူညီ** ဖြစ်ပါတယ် (equal width & height)။

**ဘယ်အခါသုံးလဲ:** calculator button တွေ၊ grid-based menu တွေအတွက် ကောင်းပါတယ်။

### 4. GridBagLayout (အင်အားအကြီးဆုံး၊ ရှုပ်ထွေးဆုံး)

Component တစ်ခုချင်းစီရဲ့ size/position ကို **precise** ထိန်းချုပ်နိုင်ပါတယ်။ `GridBagConstraints` object နဲ့ column, row, span, weight စတာတွေကို သတ်မှတ်ပါတယ်။

```java
JPanel panel = new JPanel(new GridBagLayout());
GridBagConstraints gbc = new GridBagConstraints();

gbc.insets = new Insets(5, 5, 5, 5);  // padding

// Component 1: row 0, column 0
gbc.gridx = 0;
gbc.gridy = 0;
panel.add(new JLabel("Name:"), gbc);

// Component 2: row 0, column 1
gbc.gridx = 1;
gbc.gridy = 0;
panel.add(new JTextField(15), gbc);

// Component 3: row 1, column 0, span 2 columns
gbc.gridx = 0;
gbc.gridy = 1;
gbc.gridwidth = 2;
panel.add(new JButton("Submit"), gbc);
```

**Key properties:**

- `gridx`, `gridy` - grid ထဲက column/row position
- `gridwidth`, `gridheight` - column/row ဘယ်နှစ်ခုကို span လုပ်မလဲ
- `weightx`, `weighty` - window resize လုပ်ရင် extra space ကို ဘယ်လောက် ယူမလဲ
- `fill` - component က cell ထဲမှာ ဘယ်လို fill ဖြစ်မလဲ (`HORIZONTAL`, `VERTICAL`, `BOTH`)

**ဘယ်အခါသုံးလဲ:** professional forms (login page, settings dialog) တွေမှာ component sizes မတူဘဲ precise layout လိုအပ်ရင် သုံးပါတယ်။ ရှုပ်ပေမယ့် အင်အားအကြီးဆုံးပါ။

### 5. BoxLayout

Components တွေကို **single row (horizontal)** သို့မဟုတ် **single column (vertical)** အတိုင်း စီပါတယ်။

```java
JPanel panel = new JPanel();
panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));  // vertical

panel.add(new JButton("Top"));
panel.add(Box.createVerticalStrut(10));  // spacing ထည့်ဖို့
panel.add(new JButton("Middle"));
panel.add(Box.createVerticalStrut(10));
panel.add(new JButton("Bottom"));
```

**ဘယ်အခါသုံးလဲ:** simple vertical/horizontal forms အတွက် GridBagLayout ထက် ရိုးရှင်းပြီး ကောင်းပါတယ်။

### Nested Layouts (Layout များကို ပေါင်းသုံးခြင်း)

Real application တွေမှာ layout manager **တစ်ခုတည်း** နဲ့ complex UI ဆောက်ဖို့ ခက်ပါတယ်။ ဒါကြောင့် JPanel တွေကို nest (အထပ်ထပ်) လုပ်ပြီး layout မတူတဲ့ panel တွေကို ပေါင်းသုံးလေ့ရှိပါတယ်:

```java
JFrame frame = new JFrame();
frame.setLayout(new BorderLayout());

// Top panel - FlowLayout
JPanel topPanel = new JPanel(new FlowLayout());
topPanel.add(new JButton("Save"));
topPanel.add(new JButton("Load"));
frame.add(topPanel, BorderLayout.NORTH);

// Center panel - GridLayout
JPanel centerPanel = new JPanel(new GridLayout(2, 2));
centerPanel.add(new JButton("1"));
centerPanel.add(new JButton("2"));
centerPanel.add(new JButton("3"));
centerPanel.add(new JButton("4"));
frame.add(centerPanel, BorderLayout.CENTER);
```

ဒါက real-world application design ရဲ့ **standard approach** ပါ - layout တစ်ခုတည်းနဲ့ ကြိုးစားမနေဘဲ panel တွေခွဲပြီး layout မတူတာတွေ ပေါင်းသုံးတာက ပိုလွယ်ကူပါတယ်။

### Quick Comparison Table

| Layout        | ဘယ်လိုစီလဲ                 | ဘယ်အခါသုံးလဲ                     |
| ------------- | -------------------------- | -------------------------------- |
| FlowLayout    | Left-to-right, auto-wrap   | Simple button groups             |
| BorderLayout  | 5 regions (N,S,E,W,Center) | Main app window structure        |
| GridLayout    | Equal-size grid            | Calculator, grid menus           |
| GridBagLayout | Precise control            | Professional forms               |
| BoxLayout     | Single row/column          | Simple vertical/horizontal forms |

---

ဒါက **Part 2 (Layout Managers)** ပါ။ Code တွေကို run ကြည့်ပြီးလား? နားလည်ပြီဆိုရင် **Part 3 (Event Handling)** ကို ဆက်သွားမလား၊ ဒါမှမဟုတ် FlatLaf intro ကို အရင်လုပ်ချင်ပါသလား?