# NetBeans Drag & Drop နဲ့ Responsive Layout ပြုလုပ်နည်း

NetBeans ရဲ့ GUI Builder (Matisse) ကို drag & drop နဲ့ component တွေ ချထားတဲ့အခါ **default အနေနဲ့ GroupLayout ကို auto-generate** လုပ်ပေးပါတယ် (null/absolute layout မဟုတ်ပါဘူး)။ ဒါပေမယ့် drag & drop ရုံပဲ လုပ်ပြီး ဘာမှ configure မလုပ်ရင် window resize လုပ်တဲ့အခါ component တွေ ကောင်းကောင်း မ adjust ဖြစ်တတ်ပါဘူး — ဒါက GroupLayout ရဲ့ **resizing behavior** ကို designer ထဲမှာ သတ်မှတ်ပေးရလို့ပါ။

## Root Cause: Anchor/Spring Handles

NetBeans Design view မှာ component တစ်ခုကို click လုပ်လိုက်ရင် ဘေးလေးဘက်မှာ **spring/anchor icon** လေးတွေ ပေါ်ပါတယ်:

- **Solid arrow (anchor)** = ဒီဘက်ကို parent container ရဲ့ edge နဲ့ fixed distance ထားမယ်
- **Spring icon (wavy line)** = ဒီဘက်ကို resize လုပ်ရင် **stretch/flexible** ဖြစ်စေမယ်

Component ကို resize-friendly ဖြစ်စေချင်ရင် အောက်ပါအတိုင်း လုပ်ပါ:

### 1. Text field / table ကို horizontally stretch ဖြစ်စေချင်ရင်

Component ကို select လုပ်ပြီး **ဘယ်ဘက်+ညာဘက် edge နှစ်ဖက်စလုံးကို** parent container edge နဲ့ anchor (solid line) ချိတ်ပါ။ NetBeans က အလိုအလျောက် အလယ်မှာ spring ထည့်ပေးပြီး window ကျယ်လာရင် field ကလည်း ကျယ်လာမှာပါ။

**လုပ်နည်း:** Component ကို right-click → "Edit" mode မှာ ဘေးက anchor line လေးကို click လုပ်ပြီး container edge ဆီကို connect လုပ်ပါ (visual guideline blue line ပေါ်ပါလိမ့်မယ်)။

### 2. Component list/table ကို vertically ပါ stretch လုပ်ချင်ရင်

အပေါ်+အောက် edge နှစ်ဖက်လုံးကို parent container edge နဲ့ anchor ချိတ်ပါ (JScrollPane/JTable တွေအတွက် အသုံးများပါတယ်)။

### 3. Button တွေလို fixed-size ထားချင်တာမျိုးကတော့

ညာဘက် (သို့) အောက်ခြေကို anchor မချိတ်ဘဲ ကျန်ခဲ့ပါ — ဒါဆို window ကျယ်လာလည်း button က original size အတိုင်းပဲ ရှိနေမယ်၊ position ကပဲ edge နဲ့ relative ပြောင်းမယ်။

## Properties Panel ကနေ တိကျစွာ Set လုပ်နည်း

Component ကို select ပြီး **Properties window** ထဲက **Code** tab ကို ကြည့်ရင် `horizontalGroup` / `verticalGroup` ဆိုတဲ့ generated code ကို တွေ့ရပါလိမ့်မယ်။ ဒါက GroupLayout ရဲ့ actual structure ဖြစ်ပါတယ်:

```java
layout.setHorizontalGroup(
    layout.createParallelGroup(GroupLayout.Alignment.LEADING)
        .addGroup(layout.createSequentialGroup()
            .addContainerGap()
            .addComponent(jLabel1)
            .addPreferredGap(LayoutStyle.ComponentPlacement.RELATED)
            .addComponent(jTextField1, GroupLayout.DEFAULT_SIZE, 
                          GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE) // <- ဒီ Short.MAX_VALUE က key
            .addContainerGap())
);
```

**`Short.MAX_VALUE`** ပါနေတဲ့ component ဟာ resize လုပ်ရင် stretch ဖြစ်မယ့် component ပါ။ Designer ထဲက anchor/spring icon နဲ့ ချိတ်ပေးလိုက်ရင် NetBeans က ဒီ code ကို auto-generate လုပ်ပေးတာပါ — manual ရေးဖို့ မလိုပါဘူး၊ visually ချိတ်ရုံပါပဲ။

## Practical Checklist

|လိုချင်တဲ့ ရလဒ်|ဘာလုပ်ရမလဲ|
|---|---|
|Text field / textarea ကို ကျယ်စေချင်တယ်|ဘယ်+ညာ edge ကို parent နဲ့ anchor|
|JTable/JScrollPane ကို resize အပြည့် ဖြစ်စေချင်တယ်|၄ ဖက်စလုံး (top/bottom/left/right) parent နဲ့ anchor|
|Button ကို size မပြောင်းစေချင်ဘူး|edge တစ်ခုပဲ (ဥပမာ ညာဘက်) anchor ချိတ်၊ ကျန်တာ leave|
|Label + field တွေ align ညီညီ ထားချင်တယ်|Multi-select လုပ်ပြီး "Align" toolbar button (top/left align icons) သုံး|
|Space equally ခွဲချင်တယ်|Component အများကြီးကို select လုပ်ပြီး right-click → "Same Size"|

## Quick Test

Design လုပ်ပြီးရင် **Preview** (F11 or Run) နှိပ်ပြီး frame ရဲ့ corner ကို drag ဆွဲကြည့်ပါ — component တွေ ဘယ်လို resize ဖြစ်လဲ ချက်ချင်း မြင်ရပါလိမ့်မယ်။ မကျေနပ်ရင် Design view ပြန်သွားပြီး anchor line တွေကို ပြန်ချိတ်ကြည့်ပါ (trial-and-error ပုံစံ လုပ်ရတတ်ပါတယ်၊ practice လိုက်ရင် intuitive ဖြစ်လာပါလိမ့်မယ်)။

---

**Note:** အကယ်၍ Design view ထဲက Palette မှာ layout ကို manual "Free Design" (GroupLayout) မဟုတ်ဘဲ **"Absolute Layout"** ကို ရွေးထားခဲ့ရင် (NetBeans Design toolbar ရဲ့ top-left dropdown) — ဒါက null layout အတိအကျပါ၊ resize လုပ်ရင် **လုံးဝ adjust မဖြစ်ပါဘူး**။ Responsive လိုချင်ရင် dropdown ကနေ **"Free Design"** ကို ပြန်ရွေးထားဖို့ လိုအပ်ပါတယ်။

Free Design mode နဲ့ ဆက်ပြီး struggle ဖြစ်နေရင် screenshot တင်ပြီး ပြပေးလို့ရရင် ပိုတိကျအောင် ကူညီပေးနိုင်ပါတယ်။