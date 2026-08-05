NetBeans GUI Builder (drag-and-drop) နဲ့ **responsive/dynamic layout** ရေးလို့ ရပါတယ် — ဒါပေမယ့် web development ရဲ့ "responsive" (screen size အလိုက် layout ပြောင်းတာ) နဲ့ concept အနည်းငယ် ကွာပါတယ်။ GroupLayout (NetBeans default layout manager) ရဲ့ resize behavior ကို ရှင်းပြပေးပါမယ်။

## GroupLayout ဆိုတာ ဘာလဲ (Drag-and-Drop ရဲ့ Engine)

NetBeans GUI Builder ကနေ component drag ဆွဲထည့်တိုင်း background မှာ **`GroupLayout`** class ကို auto-generate code ရေးပေးနေတာပါ — window resize လုပ်ရင် component တွေ ဘယ်လို behave လုပ်မလဲ ဆိုတာကို GroupLayout ရဲ့ "anchor + gap" constraint တွေက ဆုံးဖြတ်ပေးပါတယ်။

---

## Design View ကနေ Responsive Behavior ချိန်ညှိနည်း (Code မရေးဘဲ)

### 1. Component ကို Edge ဆီ "Anchor" ချထားခြင်း (Snap Guide)

Component ကို drag ဆွဲထားတဲ့အခါ NetBeans က **guide line** (pink/blue dashed line) auto ပြပေးပါတယ် — component ကို form ရဲ့ **edge** (border) ဆီ snap ထားလိုက်ရင်၊ window resize လုပ်တဲ့အခါ ၎င်း component က edge ကနေ **fixed distance** ကို ထိန်းထားမယ်။

```
┌─────────────────────────┐
│ [Label]                  │  ← left edge ကို snap ထားထားရင်
│                          │     window ကျယ်လာရင်တောင် left ကနေ fixed distance ထိန်းမယ်
└─────────────────────────┘
```

### 2. Component ကို "Stretch/Resize" ဖြစ်အောင် Set လုပ်ခြင်း

Component (ဥပမာ JTable, JTextArea) ကို **width/height stretch** ဖြစ်အောင် လုပ်ချင်ရင်:

1. Component ကို ရွေးပါ
2. Right-edge (ဒါမှမဟုတ် bottom-edge) ကို mouse ကနေ **drag** ဆွဲပြီး form ရဲ့ opposite edge ထိ ဆွဲသွားပါ
3. Resize arrow (↔ ဒါမှမဟုတ် ↕) ပေါ်လာရင် drop လုပ်ပါ — NetBeans က ဒါကို "resizable" gap အနေနဲ့ mark လုပ်ပေးပါလိမ့်မယ်

**Result**: Window ချဲ့လိုက်ရင် JTable/JTextArea ရဲ့ width/height **auto stretch** ဖြစ်သွားမယ် (ဒီ pattern ကို ရှေ့မှာ JDesktopPane full-stretch လုပ်ခဲ့တုန်းက အသုံးပြုခဲ့တာပါ)။

### 3. Multiple Component ကို "Same Row/Column" Align ချိန်ညှိခြင်း

Component ၂ ခု ကို horizontal/vertical တန်းတူ ချထားချင်ရင် — drag ဆွဲတဲ့အခါ **alignment guide line** (yellow dashed) ပေါ်လာမယ် — ၎င်း line ပေါ် snap ထားရင် auto-align ဖြစ်ပါလိမ့်မယ်။

---

## Practical Example — Responsive Form (Student Edit Dialog)

```
┌───────────────────────────────────┐
│ Name:  [___________________]       │ ← TextField width stretch (right edge ကို drag)
│                                     │
│ Class: [ComboBox▼]                 │ ← fixed width (stretch မလို)
│                                     │
│ ┌─────────────────────────────┐   │
│ │  (Photo Preview)              │   │ ← both width+height stretch
│ └─────────────────────────────┘   │
│                                     │
│                    [Save] [Cancel] │ ← bottom-right corner anchor (fixed position)
└───────────────────────────────────┘
```

**Setup steps**:

1. `nameField` — right edge ကို form right-edge ထိ drag ဆွဲ (width stretch)
2. `classCombo` — resize မလုပ်ဘဲ fixed width ထား (dropdown ဖြစ်လို့ stretch မလို)
3. `photoPreviewLabel` — right edge **နှင့်** bottom edge နှစ်ခုစလုံး drag ဆွဲ (width+height stretch)
4. `saveBtn`/`cancelBtn` — bottom-right corner ကို snap (window resize လုပ်ရင်တောင် corner ကနေ fixed distance ထိန်းမယ်)

---

## ⚠️ GroupLayout ရဲ့ Limitation (Web CSS Responsive နဲ့ ကွာခြားချက်)

NetBeans GroupLayout က **"stretch/anchor"** behavior ကိုသာ ပေးနိုင်ပြီး — web development ရဲ့ **breakpoint-based responsive** (screen size ကို 3 category ခွဲပြီး layout လုံးဝ ပြောင်းတာ, mobile/tablet/desktop) ကို **native support မလုပ်ပါဘူး**။

|Feature|GroupLayout (NetBeans)|Web CSS (responsive)|
|---|---|---|
|Component stretch on resize|✅ ရ|✅ ရ|
|Edge anchor|✅ ရ|✅ ရ|
|Breakpoint-based layout change (screen size အလိုက် layout structure လုံးဝ ပြောင်း)|❌ မရ|✅ ရ|
|Component ဝှက်/ပြပြောင်း screen size အလိုက်|❌ manual code လိုအပ်|✅ ရ|

Desktop application မှာ **"Component resize + anchor"** လောက်ပဲ ပုံမှန် လိုအပ်ပြီး (Java Swing desktop app က browser လို screen size အလွန်ကျယ်ပြန့် support လုပ်စရာ မလိုတာမို့) — ဒီ level ကတော့ Drag-and-drop GUI Builder ကနေ လုံလောက်စွာ handle လုပ်နိုင်ပါတယ်။

---

## Dynamic Layout ကို Code ကနေ ထပ်ထိန်းချင်ရင် (Advanced)

GUI Builder ရဲ့ auto-anchor ထက် ပိုပြီး control လိုအပ်ရင် — component resize event ကို manual listen လုပ်လို့ရပါတယ် (`initComponents()` အပြင်ဘက်မှာ):

```java
addComponentListener(new java.awt.event.ComponentAdapter() {
    @Override
    public void componentResized(java.awt.event.ComponentEvent evt) {
        int width = getWidth();
        if (width < 600) {
            // Window သေးလွန်းရင် column အချို့ ဝှက်ချင်ရင် (JTable column hide)
            studentTable.getColumnModel().getColumn(3).setMinWidth(0);
            studentTable.getColumnModel().getColumn(3).setMaxWidth(0);
        } else {
            studentTable.getColumnModel().getColumn(3).setMinWidth(75);
            studentTable.getColumnModel().getColumn(3).setMaxWidth(Integer.MAX_VALUE);
        }
    }
});
```

ဒါက GUI Builder auto-generate code (`initComponents()`) ကို **ထိခိုက်စေခြင်းမရှိဘဲ** — ထပ်ပေါင်း behavior အနေနဲ့ ထည့်နိုင်ပါတယ်။

---

## Quick Checklist

|Goal|Method|
|---|---|
|Component width/height auto stretch|Design view: edge drag ဆွဲ → resize guide arrow ပေါ်ချိန် drop|
|Component fixed position (corner anchor)|Design view: edge ဆီ snap ချထား၊ resize မလုပ်|
|Screen size အလိုက် column ဝှက်/ပြ|Manual `ComponentListener` code (Advanced)|
|Breakpoint-based layout structure ပြောင်း|❌ GroupLayout native support မရှိ — manual logic လိုအပ်|

---

**Summary**: NetBeans drag-and-drop (GroupLayout) နဲ့ **"component stretch + anchor"** level responsive behavior ကို GUI ကနေ **code မရေးဘဲ** ရနိုင်ပါတယ် — edge ကို drag ဆွဲပြီး resize/anchor mark ရုံပါပဲ။ Web-style "breakpoint responsive" (layout structure လုံးဝ ပြောင်းတာ) ကတော့ desktop app မှာ ပုံမှန် မလိုအပ်ပါဘူး၊ လိုအပ်ရင်တော့ manual `ComponentListener` code ဖြည့်ရပါလိမ့်မယ်။

Concrete form တစ်ခု (ဥပမာ Student edit dialog) ကို responsive resize ဖြစ်အောင် edge drag ဆွဲနည်း step-by-step screenshot-level detail လိုချင်ရင် ပြောပါ။