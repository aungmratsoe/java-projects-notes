Swing **Glue** (Horizontal Glue / Vertical Glue) ကို ဘယ်အခြေအနေမျိုးမှာ သုံးသင့်လဲ ရှင်းပြပေးပါမယ်။

## အခြေခံ သဘောတရား ပြန်သတိရစေချင်တာ

**Glue** = "flexible/elastic" space ဖြစ်ပြီး — window resize လုပ်ရင် **stretch/shrink** အလိုအလျောက် ဖြစ်တယ် (Strut/Rigid Area က fixed size ဖြစ်တာနဲ့ ဆန့်ကျင်ဘက်)။ Glue ရဲ့ အဓိက အလုပ်က component တွေကို **"တွန်းထုတ်"** ပေးတာပါ — component တွေကို ဘေးတစ်ဖက်ဖက်ကို ကပ်စေချင်တဲ့အခါ သုံးတယ်။

---

## Glue သုံးသင့်တဲ့ Concrete Condition များ

### 1. Button တွေကို Left/Right သီးခြား ကပ်စေချင်တဲ့အခါ

**ဥပမာ**: Dialog အောက်ခြေမှာ "Delete" button ကို left ဘက်၊ "Save"/"Cancel" ကို right ဘက် ကပ်ချင်တဲ့အခါ

```java
JPanel bottomPanel = new JPanel();
bottomPanel.setLayout(new BoxLayout(bottomPanel, BoxLayout.X_AXIS));

bottomPanel.add(deleteBtn);
bottomPanel.add(Box.createHorizontalGlue()); // ကြားက space ကို "တွန်း" ချဲ့ထုတ်
bottomPanel.add(saveBtn);
bottomPanel.add(Box.createHorizontalStrut(10));
bottomPanel.add(cancelBtn);
```

**Window resize လုပ်လိုက်ရင်**: Delete က left ဘက်၊ Save/Cancel က right ဘက် အမြဲ ကပ်နေမယ် — glue က middle space ကို auto adjust လုပ်ပေးနေလို့ပါ။

### 2. Toolbar/Header မှာ Left Section vs Right Section ခွဲချင်တဲ့အခါ

**ဥပမာ**: Toolbar left ဘက်မှာ "New", "Open" button တွေ၊ right ဘက်မှာ "Search" field ကပ်ချင်တဲ့အခါ

```java
toolbar.add(newBtn);
toolbar.add(openBtn);
toolbar.add(Box.createHorizontalGlue()); // left group နဲ့ right group ခွဲ
toolbar.add(searchField);
```

### 3. Content ကို Center Align လုပ်ချင်တဲ့အခါ (Glue ၂ ခု ဘေးနှစ်ဖက်ထားခြင်း)

Component တစ်ခုကို panel ရဲ့ **အလယ်** မှာ ကပ်စေချင်ရင် — glue ကို ဘေးနှစ်ဖက်လုံးမှာ ထည့်ရတယ်:

```java
panel.add(Box.createHorizontalGlue());
panel.add(centerLabel); // ဒီ component က middle မှာ ရောက်နေမယ်
panel.add(Box.createHorizontalGlue());
```

### 4. Form ရဲ့ Label/Field ကို Top ဆီ ကပ်ပြီး Button ကို Bottom ဆီ ကပ်ချင်တဲ့အခါ

**Vertical Glue** ဥပမာ — Panel ရဲ့ height ကြီးလာရင်တောင် button တွေက အောက်ဆုံးမှာ အမြဲကပ်နေမယ်:

```java
panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));

panel.add(titleLabel);        // အပေါ်ဆုံး ကပ်နေမယ်
panel.add(Box.createVerticalGlue()); // ကြားထဲက space ကို ဆွဲဖြန့်
panel.add(okBtn);             // အောက်ဆုံးကို တွန်းချသွားမယ်
```

### 5. Status Bar Pattern (Left status text, Right version/info)

```java
statusPanel.add(statusLabel);        // "Connected" (left)
statusPanel.add(Box.createHorizontalGlue());
statusPanel.add(versionLabel);       // "v1.0" (right)
```

---

## Glue **မသုံးသင့်**တဲ့ အခြေအနေများ

**1. Fixed gap လိုအပ်တဲ့အခါ** — Glue က resize အလိုက် ပြောင်းနေမှာမို့ **fixed spacing** လိုအပ်ရင် (ဥပမာ button ၂ ခုကြား 10px ချောင်းပေးချင်ရင်) — **Strut** ကိုသာ သုံးရမယ်

**2. GroupLayout (NetBeans default) သုံးနေတဲ့အခါ** — GroupLayout က gap/spacing ကို GUI ကနေ constraint-based auto handle လုပ်ပေးလို့ Glue manual ထည့်ဖို့ မလိုအပ်ပါဘူး (BoxLayout ကို တမင်သုံးမှသာ Glue သက်ဆိုင်တယ်)

**3. GroupLayout မှာ "push to edge" effect လိုချင်ရင်** — Glue အစား GroupLayout ရဲ့ **`addGap(GroupLayout.PREFERRED_SIZE, GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)`** ကို သုံးရမယ် (concept တူပေမယ့် syntax ကွာတယ်)

---

## Summary Table

|Condition|Glue Type|
|---|---|
|Button/component ဘေးချင်း left-right ကွဲစေချင်ရင်|Horizontal Glue|
|Component အပေါ်-အောက် ကွဲစေချင်ရင်|Vertical Glue|
|Content ကို center align လုပ်ချင်ရင်|Glue ၂ ခု ဘေးနှစ်ဖက်|
|Fixed/constant gap လိုအပ်ရင်|❌ Glue မဟုတ်ဘဲ **Strut** သုံး|
|NetBeans GroupLayout (default GUI Builder mode) သုံးနေရင်|❌ Glue manual မလို|

---

**QRCode Project အတွက် Practical Point**: NetBeans GUI Builder (drag-and-drop) မှာ default က **GroupLayout** ကို auto သုံးထားလို့ Glue ကို manual ထည့်ဖို့ များသောအားဖြင့် **မလိုအပ်ပါဘူး** — GroupLayout ရဲ့ "resizable gap" constraint ကို Design view ကနေ auto handle လုပ်ပေးနေလို့ပါ။ Glue ကတော့ **BoxLayout ကို code ထဲမှာ manual ရေး** မှသာ relevant ဖြစ်ပါတယ် (ဥပမာ - dialog button panel ကို programmatically ရေးမယ်ဆိုရင်)။