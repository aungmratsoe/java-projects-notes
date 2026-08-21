## LayoutManager ဆိုတာဘာလဲ

`LayoutManager` က Java Swing/AWT ရဲ့ **interface** တစ်ခုဖြစ်ပြီး၊ container (JPanel, JFrame) တစ်ခုအတွင်းက component တွေရဲ့ **position နဲ့ size** ကို ဘယ်လိုစီစဉ်မလဲဆိုတာ ဆုံးဖြတ်ပေးတဲ့ "စည်းကမ်း" ပါ။ MigLayout ကိုယ်တိုင်လည်း ဒီ `LayoutManager` interface ကို implement လုပ်ထားတဲ့ class တစ်ခုပါပဲ — ဒါကြောင့် "LayoutManager" ဆိုတာက category name၊ "MigLayout" ကတော့ ၄င်း category ထဲက implementation တစ်ခုပါ။

## ဘာကြောင့်လိုအပ်လဲ

Component (button, label, table) တစ်ခုကို panel ပေါ် ချထားရင် **ဘယ်နေရာမှာ ဘယ်လောက်ကြီးမလဲ** ဆိုတာကို တစုံတခုက ဆုံးဖြတ်ပေးရပါတယ်။ Manual pixel position (`setBounds()`) သုံးရင် window resize ဖြစ်ရင် layout ပျက်သွားနိုင်ပါတယ်။ `LayoutManager` ကတော့ resize/font size/OS ပြောင်းရင်တောင် **automatic ပြန်ချိန်** ပေးနိုင်ပါတယ်။

## Java built-in LayoutManager အမျိုးမျိုး

|LayoutManager|ဘယ်လိုအလုပ်လုပ်လဲ|ဘယ်အချိန် သုံးလဲ|
|---|---|---|
|`FlowLayout`|component တွေကို left-to-right တန်းစီ|Simple toolbar|
|`BorderLayout`|N/S/E/W/Center 5 ဇုန်|JFrame ရဲ့ default, simple screen|
|`GridLayout`|equal-size grid cells|Calculator button grid|
|`GridBagLayout`|flexible grid, ဒါပေမယ့် syntax ရှုပ်ထွေး|Complex form (legacy)|
|`CardLayout`|panel တွေကို layer အလိုက် switch|Wizard/multi-screen flow|
|`BoxLayout`|vertical/horizontal stack|Simple list-style arrangement|
|**`MigLayout`**|third-party, text-based constraint syntax|**modern app, flexible form/grid**|
|`null` (no layout)|manual `setBounds()`|JLayeredPane overlay တွေအတွက်ပဲ|

## ဘယ်အချိန် ဘယ် LayoutManager ကို ရွေးမလဲ

**FlowLayout** — toolbar လို simple row တစ်ခု (button 3-4 ခု horizontal စီရုံ) ဆိုရင် လုံလောက်ပါတယ်။

```java
panel.setLayout(new FlowLayout());
panel.add(new JButton("New Order"));
panel.add(new JButton("Cancel"));
```

**BorderLayout** — JFrame ရဲ့ default layout ပါ။ Header/Footer/Sidebar/Center ပုံစံ 5 ဇုန်ပဲ လိုအပ်ရင် ရိုးရှင်းစွာသုံးလို့ရပါတယ်။

```java
frame.setLayout(new BorderLayout());
frame.add(headerPanel, BorderLayout.NORTH);
frame.add(orderTable, BorderLayout.CENTER);
frame.add(totalPanel, BorderLayout.SOUTH);
```

**GridLayout** — Calculator ကီးဘုတ်၊ POS menu button grid (Cell size အားလုံး တူညီချင်ရင်) အတွက် ကောင်းပါတယ်။

```java
menuPanel.setLayout(new GridLayout(4, 3, 5, 5)); // 4 rows, 3 cols, gap 5px
for (String item : menuItems) {
    menuPanel.add(new JButton(item));
}
```

**MigLayout** — form-style layout, column/row span, grow/shrink ratio, responsive design လိုအပ်တဲ့ **modern app အများစု** အတွက် အသင့်တော်ဆုံးပါ (မင်း Cafe POS app လိုမျိုး)။

```java
panel.setLayout(new MigLayout("fill", "[grow][100]", "[][grow][]"));
```

**CardLayout** — Order → Payment → Receipt လို screen switching flow အတွက်။

**null layout (manual bounds)** — JLayeredPane ထဲက overlay/popup positioning အတွက်ပဲ။ ပုံမှန် app development မှာ **မထောက်ခံပါ** — resize ဖြစ်ရင် ချိန်ရခက်လို့ပါ။

## MigLayout ဘာကြောင့် ပိုကောင်းလဲ (built-in တွေထက်)

- `GridBagLayout` ရဲ့ power ပါ ဒါပေမယ့် syntax ပိုရိုးရှင်း (constraint string တစ်ကြောင်းတည်းနဲ့ column/row/span/grow သတ်မှတ်လို့ရ)
- `FlowLayout`/`GridLayout` လို simple layout တွေက flexible မဟုတ် — column width တွေ ကွဲပြားချင်ရင် (Item column ကျယ်၊ Qty column သေး) မလုပ်နိုင်ပါ
- MigLayout ကတော့ column-by-column size specify (`"[grow][100][80]"`) လုပ်လို့ရ

## Nested LayoutManager concept

Container တစ်ခုအတွင်း layout manager တစ်ခုတည်း **မဟုတ်ဘဲ** layer အလိုက် layout manager မတူတာလည်း ရပါတယ် — ဥပမာ- outer `BorderLayout` ရဲ့ CENTER ထဲမှာ inner panel ကို `MigLayout` သုံးထားလို့ရ။ Panel/component တစ်ခုချင်းစီမှာ layout manager သီးခြားစီ instantiate လုပ်လို့ရလို့ပါ။

```java
frame.setLayout(new BorderLayout());          // outer
JPanel formPanel = new JPanel(new MigLayout("fill")); // inner
frame.add(formPanel, BorderLayout.CENTER);
```

Card/JLayeredPane concept ကို ရှေ့မှာ လေ့လာခဲ့တဲ့အတိုင်း — layer တစ်ခုချင်းစီအတွင်း layout manager ကွဲပြားလို့ရတယ်ဆိုတာ ပြန်သတိရနိုင်ပါလိမ့်မယ်။

---

**အနှစ်ချုပ်** — LayoutManager က "category" (interface)၊ MigLayout က ၄င်းရဲ့ powerful implementation တစ်ခု။ Simple layout (toolbar, N/S/E/W) ဆိုရင် built-in (`FlowLayout`, `BorderLayout`) လုံလောက်ပေမယ့်၊ complex/responsive form (POS app) ဆိုရင် **MigLayout ကို ဆက်လက်အသုံးပြုသင့်ပါတယ်**။

built-in layout manager တွေထဲက တစ်ခုခုကို code နဲ့ practice လုပ်ကြည့်ချင်လား၊ ဒါမှမဟုတ် MigLayout ဆီပဲ ဆက်ပြီး deep dive လုပ်ချင်ပါသလား?