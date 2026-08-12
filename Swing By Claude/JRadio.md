JRadioButton (Radio Selector) ကို NetBeans Swing GUI Builder မှာ Drag & Drop နဲ့ အသုံးပြုပုံနဲ့ Group လုပ်ပုံကို ရှင်းပြပေးပါမယ်။

## 1. JRadioButton ဆိုတာဘာလဲ

JRadioButton ဆိုတာ user ကို options အများကြီးထဲက **တစ်ခုတည်း** ရွေးခိုင်းချင်တဲ့အခါ သုံးတဲ့ component ဖြစ်ပါတယ်။ Checkbox နဲ့ မတူတာက Checkbox က တစ်ခုထက်ပိုရွေးလို့ရပေမယ့် Radio Button က group ထဲမှာ တစ်ခုပဲ ရွေးလို့ရပါတယ်။

## 2. NetBeans GUI Builder မှာ Drag & Drop နည်း

**Step 1** — NetBeans ဘေးက Palette panel မှာ "Swing Controls" category ကို ဖွင့်ပြီး **Radio Button** ကို ရှာပါ။

**Step 2** — Radio Button icon ကို mouse နဲ့ ဆွဲပြီး Form Designer canvas ပေါ်ကို drop လုပ်ပါ။ ဒီလိုမျိုး ချင်တဲ့ radio button အရေအတွက်အလိုက် ထပ်ခါထပ်ခါ drag လုပ်ပါ (ဥပမာ - Male, Female ဆိုရင် ၂ ခု)။

**Step 3** — Radio button တစ်ခုချင်းစီကို click ပြီး Properties window (ညာဘက်အောက်ခြေ) မှာ:

- `text` — button ပေါ်မှာ ပြမယ့် စာသား (ဥပမာ "Male")
- `variableName` (Code tab) — code ထဲမှာ ခေါ်မယ့် variable name (ဥပမာ `rbMale`)

ဒါတွေကို ပြင်ပါ။

## 3. Group လုပ်ခြင်း (ButtonGroup)

**ဒီအဆင့်က အရေးအကြီးဆုံး** — Radio button တွေကို Group မလုပ်ရင် တစ်ခုချင်းစီက သီးခြားစီ ရွေးလို့ရနေမှာဖြစ်ပြီး "တစ်ခုတည်းရွေးရမယ်" ဆိုတဲ့ သဘောသဘာဝ ပျက်သွားပါလိမ့်မယ်။

NetBeans GUI Builder မှာ group လုပ်ဖို့ လွယ်ကူပါတယ်:

**နည်းလမ်း (1) — Right-click Method:**

1. Group ချင်တဲ့ radio button အားလုံးကို **Ctrl** နှိပ်ထားပြီး click ရွေးပါ (multiple select)
2. ရွေးထားတဲ့ radio button တွေပေါ်မှာ right-click လုပ်ပါ
3. **"Add to Button Group" → "New Button Group..."** ကို ရွေးပါ
4. ButtonGroup name တစ်ခု ပေးပါ (ဥပမာ `genderGroup`)

ဒါဆိုရင် NetBeans က code အောက်ပါအတိုင်း auto-generate လုပ်ပေးပါလိမ့်မယ်:

```java
private javax.swing.ButtonGroup genderGroup;

// initComponents() ထဲမှာ:
genderGroup = new javax.swing.ButtonGroup();
genderGroup.add(rbMale);
genderGroup.add(rbFemale);
```

**နည်းလမ်း (2) — Code ကို ကိုယ်တိုင် ရေးချင်ရင်:**

```java
ButtonGroup genderGroup = new ButtonGroup();
genderGroup.add(rbMale);
genderGroup.add(rbFemale);
```

`ButtonGroup` က UI component မဟုတ်ဘူး — logical grouping object ပဲဖြစ်တယ်။ ဒါကြောင့် screen ပေါ်မှာ မမြင်ရပါဘူး၊ ပေါင်းထည့်ထားတဲ့ button တွေကို "အချင်းချင်း exclusive" (တစ်ခုတည်းပဲ ရွေးလို့ရအောင်) ချိတ်ပေးတဲ့ logic ကို handle လုပ်ပေးတာပါ။

## 4. ရွေးထားတဲ့ Value ကို ဘယ်လို စစ်မလဲ

```java
if (rbMale.isSelected()) {
    gender = "Male";
} else if (rbFemale.isSelected()) {
    gender = "Female";
}
```

သို့မဟုတ် ButtonGroup ကနေ တိုက်ရိုက် စစ်ချင်ရင်:

```java
ButtonModel selected = genderGroup.getSelection();
String selectedValue = selected.getActionCommand();
```

(ဒီနည်းသုံးမယ်ဆိုရင် radio button တစ်ခုချင်းစီရဲ့ `actionCommand` property ကို Properties window မှာ သတ်မှတ်ပေးထားရပါမယ်)

## 5. Default Selection သတ်မှတ်ချင်ရင်

Form ပွင့်တာနဲ့ radio button တစ်ခုကို default ရွေးထားချင်ရင် Properties window မှာ `selected` property ကို `true` လုပ်ပေးပါ (တစ်ခုတည်းပဲ true ဖြစ်သင့်တယ်, group ထဲက တခြားတွေက false ဖြစ်ရမယ်)။

---

**မှတ်ချက်** — Group တစ်ခုအတွင်း radio button တွေက ချင်းချင်းလှမ်းနေရင်တောင် (ဥပမာ panel မတူတာတောင်) ButtonGroup နဲ့ ချိတ်ထားရင် logical group အနေနဲ့ ပေါင်းထားနိုင်ပါတယ်။ Layout position နဲ့ Group logic က သီးခြားစီပါ။

Group အများကြီး (ဥပမာ Gender group, Payment Method group) သုံးမယ်ဆိုရင် ButtonGroup object ကို group အလိုက် သီးခြား ဖန်တီးရပါမယ်လို့ မှတ်ထားပါ။