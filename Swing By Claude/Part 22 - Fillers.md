Swing filler (invisible spacing) component တွေကို ရှင်းပြပေးပါမယ်။ ဒါတွေအားလုံးက **`Box`** class ထဲကနေ static method တွေနဲ့ ဖန်တီးရတဲ့ **invisible component** တွေပါ - `BoxLayout` နဲ့ တွဲသုံးရင် အသုံးအများဆုံးဖြစ်ပြီး၊ component တွေကြားက space ကို control လုပ်ဖို့ ရည်ရွယ်ပါတယ်။

## အခြေခံ ကွာခြားချက် - Strut vs Glue vs Rigid Area

|Type|Space အရွယ်အစား|Stretch လုပ်နိုင်လား|
|---|---|---|
|**Strut**|fixed (တစ်ဖက်တည်း dimension)|မလုပ်နိုင်ဘူး|
|**Rigid Area**|fixed (width+height နှစ်ဖက်စလုံး)|မလုပ်နိုင်ဘူး|
|**Glue**|flexible (elastic)|window resize လုပ်ရင် stretch ဖြစ်တယ်|

---

## 1. **Horizontal Strut**

- **ဘေးချင်း (left-right)** component တွေကြားမှာ **fixed width** space ခံချင်ရင် သုံးတယ်။

```java
panel.setLayout(new BoxLayout(panel, BoxLayout.X_AXIS)); // horizontal layout

panel.add(new JButton("Save"));
panel.add(Box.createHorizontalStrut(20)); // 20px fixed gap
panel.add(new JButton("Cancel"));
```

## 2. **Vertical Strut**

- **အပေါ်-အောက် (top-bottom)** component တွေကြားမှာ **fixed height** space ခံချင်ရင် သုံးတယ်။

```java
panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS)); // vertical layout

panel.add(new JLabel("Name:"));
panel.add(Box.createVerticalStrut(10)); // 10px fixed gap
panel.add(new JTextField(20));
```

## 3. **Rigid Area**

- Width **နှင့်** height **နှစ်ခုစလုံး** fixed size ဖြစ်တဲ့ space (rectangle box) ခံချင်ရင် သုံးတယ် (Strut က တစ်ဖက်တည်းသာ fix လုပ်ပေးတယ်၊ ဒါက နှစ်ဖက်စလုံး)။

```java
panel.add(new JButton("A"));
panel.add(Box.createRigidArea(new Dimension(15, 30))); // 15px width, 30px height fixed
panel.add(new JButton("B"));
```

## 4. **Horizontal Glue**

- Component တွေကို **ဘေးချင်း** ဖယ်ရှားချင်တဲ့အခါ (push လုပ်ချင်တဲ့အခါ) သုံးတဲ့ **flexible/elastic** space ပါ - window resize လုပ်ရင် stretch/shrink လိုက်ဖြစ်တယ်။
- **ဥပမာ**: Button တစ်ခုကို left ဘက်၊ တစ်ခုကို right ဘက် ကပ်ချင်ရင်။

```java
panel.setLayout(new BoxLayout(panel, BoxLayout.X_AXIS));

panel.add(new JButton("Back"));      // left ဘက် ကပ်နေမယ်
panel.add(Box.createHorizontalGlue()); // ကြားက space ကို "တွန်း" ဆွဲထုတ်ပေးမယ်
panel.add(new JButton("Next"));      // right ဘက် ကပ်သွားမယ်
```

## 5. **Vertical Glue**

- Horizontal Glue နဲ့ concept တူတယ်၊ ဒါပေမယ့် **အပေါ်-အောက်** direction အတွက်ပါ။
- **ဥပမာ**: Label တစ်ခုကို panel ရဲ့ အပေါ်ဆုံးမှာ၊ button တစ်ခုကို အောက်ဆုံးမှာ ကပ်ချင်ရင်။

```java
panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));

panel.add(new JLabel("Title"));     // အပေါ်ဆုံး ကပ်နေမယ်
panel.add(Box.createVerticalGlue()); // ကြားထဲက space ကို ဆွဲဖြန့်ပေးမယ်
panel.add(new JButton("OK"));       // အောက်ဆုံးကို တွန်းချသွားမယ်
```

## 6. **Glue** (generic)

- `createGlue()` — direction သီးသန့် မသတ်မှတ်ဘဲ layout context ပေါ်မူတည်ပြီး auto stretch ဖြစ်တဲ့ generic glue ပါ (Horizontal/Vertical Glue ကို တိတိကျကျ သတ်မှတ်တာက ပိုအသုံးများတယ်)။

```java
panel.add(Box.createGlue());
```

---

## Full Example - Toolbar-style layout (Glue သုံးထားပုံ)

```java
JPanel bottomPanel = new JPanel();
bottomPanel.setLayout(new BoxLayout(bottomPanel, BoxLayout.X_AXIS));

JButton deleteBtn = new JButton("Delete");
JButton saveBtn = new JButton("Save");
JButton cancelBtn = new JButton("Cancel");

bottomPanel.add(deleteBtn);
bottomPanel.add(Box.createHorizontalGlue()); // Delete ကို left ကျန်ကျန် ချန်ထား
bottomPanel.add(saveBtn);
bottomPanel.add(Box.createHorizontalStrut(10)); // Save/Cancel ကြား fixed gap
bottomPanel.add(cancelBtn);
```

ရလဒ်: `[Delete] ← ← ← ← ← ← ← ← ← ← [Save] [Cancel]` — Delete က left ဘက်၊ Save/Cancel နှစ်ခုက right ဘက် ကပ်နေမယ့် form ဖြစ်ပါလိမ့်မယ်။

---

**မှတ်ချက်**: NetBeans GUI Builder (Free Design / drag-drop mode) သုံးရင် GroupLayout ကို default သုံးလို့ ဒီ filler component တွေ manual ထည့်ဖို့ မလိုတတ်ပါဘူး (GroupLayout က gap တွေကို auto handle လုပ်ပေးတယ်)။ ဒါပေမယ့် code ကို manual ရေးမယ်၊ ဒါမှမဟုတ် BoxLayout ကို တမင် သုံးမယ်ဆိုရင် ဒီ filler တွေက အရေးကြီးပါတယ်။