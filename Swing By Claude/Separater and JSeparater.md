NetBeans GUI Builder (drag-and-drop) မှာ **JSeparator** ကို context အလိုက် ၂ မျိုးခွဲပြီး ရှင်းပြပေးပါမယ် — (1) Menu/Popup ထဲက Separator, (2) Panel/Toolbar ထဲက standalone JSeparator component.

## 1. Menu ဒါမှမဟုတ် PopupMenu ထဲမှာ Separator ထည့်ခြင်း

Menu item တွေကို logical group ခွဲပြချင်တဲ့အခါ သုံးတယ် (ဥပမာ: File > New/Open/Save တစ်အုပ်စု, line ခြားပြီး Exit တစ်ခုတည်း)။

### Steps:

1. `jMenu1` ဒါမှမဟုတ် `jPopupMenu1` ကို **double-click** လုပ်ပြီး dropdown edit mode ဝင်ပါ
2. Palette ထဲက **"Swing Menus"** category ကနေ **"Separator"** ကို ရှာပါ
3. Menu item ၂ ခုကြားက **အကွက်လွတ်** နေရာကို drag ဆွဲထည့်ပါ (NetBeans က insertion point ကို visual line နဲ့ auto ပြပေးမယ်)

```
File Menu
  ├── New
  ├── Open
  ├── ─────────── (Separator drag ချထားတာ)
  └── Exit
```

Drag ဆွဲထည့်ရုံသက်သက်ပါ — code manual ရေးစရာ **လုံးဝ မလိုပါဘူး** (NetBeans က `fileMenu.addSeparator();` ကို auto-generate `initComponents()` ထဲမှာ ရေးပေးမယ်)။

### Result code (auto-generated, ကိုယ်တိုင် ရေးစရာမလို — reference အတွက်ပဲ):

```java
fileMenu.add(newMenuItem);
fileMenu.add(openMenuItem);
fileMenu.add(jSeparator1);  // auto-added
fileMenu.add(exitMenuItem);
```

---

## 2. Panel/Form ထဲမှာ Standalone JSeparator ထည့်ခြင်း

Toolbar ဒါမှမဟုတ် form section တွေကို visual line ခွဲပြချင်တဲ့အခါ သုံးတယ် (ဥပမာ: "Student Info" section နဲ့ "Contact Info" section ကြား line ခွဲတာမျိုး)။

### Steps:

1. Palette ထဲက **"Swing Controls"** category ကနေ **"Separator"** ကို ရှာပါ
2. Form design view ပေါ် **တိုက်ရိုက် drag** ဆွဲချပါ (visible component မို့ position/size သတ်မှတ်ရမယ်)
3. **GroupLayout** default ဆိုရင် — separator ကို drag ဆွဲချတဲ့အခါ NetBeans က auto snap guide line ပြပေးမယ်, form width အပြည့် stretch ချင်ရင် edge ၂ ဖက်ကို guide line ထိအောင် drag ဆွဲရမယ်

### Properties (Design view ကနေ ချိန်ညှိလို့ရတဲ့ setting):

- `orientation` → `HORIZONTAL` (default) ဒါမှမဟုတ် `VERTICAL`

```
┌─────────────────────────┐
│  Student Info Section    │
│  [Name] [ID]             │
├───────────────────────── │ ← standalone JSeparator (horizontal)
│  Contact Info Section    │
│  [Phone] [Email]         │
└─────────────────────────┘
```

---

## Toolbar ထဲမှာ Separator ထည့်ချင်ရင်

JToolBar ထဲက button တွေကြား gap ခွဲချင်ရင်လည်း Palette ကနေ **"Separator"** ကို toolbar ထဲ တိုက်ရိုက် drag ဆွဲထည့်လို့ရပါတယ် (Menu ထဲထည့်သလိုပဲ — direct drag, code manual မလို):

```
[Save] [Open] | [Cut] [Copy] [Paste]
              ↑ toolbar separator (drag ထည့်ထားတာ)
```

---

## Quick Checklist

|Context|Palette Category|Drag ချရမည့်နေရာ|
|---|---|---|
|Menu/PopupMenu ထဲ|Swing Menus → Separator|Menu item ၂ ခုကြား (dropdown edit mode ထဲမှာ)|
|Panel/Form ထဲ|Swing Controls → Separator|Form design view ပေါ် တိုက်ရိုက်|
|Toolbar ထဲ|Swing Menus → Separator|Toolbar ထဲက button ကြား|

---

**Key point**: Separator က logic မပါတဲ့ visual-only component မို့ — drag ဆွဲထည့်ရင်ပဲ ပြီးသွားတယ်၊ event handler တွေ code ရေးစရာ **လုံးဝ မလိုပါဘူး** (ဒါက ဒီ session မှာ ရှင်းခဲ့တဲ့ component တွေထဲက အလွယ်ဆုံး တစ်ခုပါ)။

QRCode project ရဲ့ form layout မှာ Section ၂ ခုကြား Separator ထည့်ချင်တဲ့ concrete design ရှိရင် ပြောပါ။