# JOptionPane.showMessageDialog() ရဲ့ parameter သုံးမျိုး ကွာခြားချက်

`showMessageDialog()` ရဲ့ ပထမ parameter (`parentComponent`) က dialog box ကို ဘယ်နေရာမှာ **ဗဟိုပြု (center)** ပြီးပေါ်စေမလဲဆိုတာကို သတ်မှတ်ပေးတာပါ။ Object သုံးမျိုးစလုံးက အလုပ်လုပ်ပုံ အနည်းငယ်ကွာသွားနိုင်ပါတယ်။

## 1. `frame` ကို သုံးရင်

```java
JOptionPane.showMessageDialog(frame, "Operation completed successfully!");
```

- `frame` ဆိုတာ သင် ဖန်တီးထားတဲ့ `JFrame` object (ဥပမာ - `JFrame frame = new JFrame();`) ကို ညွှန်းတာပါ။
- Dialog box ကို **ဒီ frame ရဲ့ အလယ်ဗဟို** မှာ ပေါ်စေပါတယ်။
- ဒီနည်းလမ်းက method တစ်ခု (static method ဒါမှမဟုတ် frame reference ရှိတဲ့ အခြား class) ထဲကနေခေါ်တဲ့အခါ သုံးလေ့ရှိပါတယ်။

## 2. `this` ကို သုံးရင်

```java
JOptionPane.showMessageDialog(this, "Operation completed successfully!");
```

- `this` က **လက်ရှိ class instance ကိုယ်တိုင်** ကို ညွှန်းတာပါ။
- ဒါကို JFrame ဒါမှမဟုတ် JDialog ကို extend လုပ်ထားတဲ့ class တစ်ခုရဲ့ **အတွင်းက method** တစ်ခုထဲမှာ ခေါ်တဲ့အခါ သုံးပါတယ် (ဥပမာ - button click event handler ထဲမှာ)။
- `this` ဟာ frame/component ကိုယ်တိုင် ဖြစ်နေတဲ့အတွက်၊ dialog ကို **ဒီ frame ရဲ့ အလယ်ဗဟို** မှာပဲ ပေါ်စေပါတယ် — `frame` ကိုသုံးတာနဲ့ ရလဒ်တူပါတယ်၊ ဒါပေမယ့် context ကွာတယ်။

## 3. `null` ကို သုံးရင်

```java
JOptionPane.showMessageDialog(null, "Operation completed successfully!");
```

- Parent component မရှိတဲ့အခါ၊ dialog ကို **screen ရဲ့ အလယ်ဗဟို** (screen center) မှာ ပေါ်စေပါတယ်။
- ဘယ် frame နဲ့မှ ချိတ်ဆက်မထားလို့၊ frame ကို move လုပ်ရင်တောင် dialog က screen center မှာပဲ ရှိနေမှာပါ။
- Frame reference မရှိတဲ့ static context တွေ (ဥပမာ - `main` method) မှာ သုံးလေ့ရှိပါတယ်။

## အကျဉ်းချုပ် နှိုင်းယှဉ်ချက်

|Parameter|Dialog ဘယ်မှာပေါ်မလဲ|ဘယ်အချိန်သုံးသင့်လဲ|
|---|---|---|
|`frame`|Frame ရဲ့ အလယ်ဗဟို|Frame object ကို reference variable အနေနဲ့ ရနိုင်တဲ့အခါ|
|`this`|လက်ရှိ frame/component ရဲ့ အလယ်ဗဟို|Frame class ကိုယ်တိုင်ရဲ့ inner method ထဲက ခေါ်တဲ့အခါ|
|`null`|Screen ရဲ့ အလယ်ဗဟို|Frame reference မရှိတဲ့အခါ (static method စသည်)|

**အကြံပြုချက်**: Application ရဲ့ main window ရှိပြီးသားဆိုရင်၊ `frame` (သို့) `this` ကိုသုံးတာက user experience ပိုကောင်းပါတယ် — dialog က application window နဲ့ဆက်စပ်နေလို့ user တွေအတွက် intuitive ဖြစ်ပါတယ်။ `null` ကိုတော့ frame reference လက်လှမ်းမမီတဲ့အခါမှသာ အသုံးပြုပါ။