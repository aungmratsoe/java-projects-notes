# Java Swing Layout များ ရှင်းပြချက်

Swing မှာ Layout Manager တွေက component တွေကို ဘယ်လို arrange လုပ်မလဲဆိုတာ ထိန်းချုပ်ပေးပါတယ်။ မေးထားတဲ့ ၄ခုကို တစ်ခုချင်း ရှင်းပြပါမယ်။

## 1. Null Layout (Absolute Positioning)

Layout Manager လုံးဝ **မသုံးဘဲ** component တစ်ခုချင်းစီရဲ့ တည်နေရာ (x, y) နဲ့ အရွယ်အစား (width, height) ကို ကိုယ်တိုင် pixel အနေနဲ့ သတ်မှတ်ပေးတဲ့ နည်းလမ်းပါ။

```java
panel.setLayout(null); // Layout manager ကို ပိတ်ထားတယ်
JButton btn = new JButton("Click");
btn.setBounds(50, 100, 100, 30); // x=50, y=100, width=100, height=30
panel.add(btn);
```

- `setBounds(x, y, width, height)` သုံးရပါတယ်
- window ရဲ့ အရွယ်အစား ပြောင်းလဲရင် (resize) component တွေ ကိုယ်တိုင် adjust မဖြစ်ပါဘူး၊ နေရာအတိအကျမှာသာ ရှိနေမှာပါ
- Screen resolution အမျိုးမျိုးမှာ layout ပျက်နိုင်လို့ **practical app တွေမှာ အကြံပြုချက်မပေးပါ** — ဒါပေမယ့် GUI builder tool တွေ (drag & drop design) နောက်ကွယ်မှာ ဒီနည်းလမ်းသုံးလေ့ရှိပါတယ်

> **Absolute Layout** ဆိုတာလည်း Null Layout နဲ့ concept တူပါတယ် — pixel အတိအကျ position သတ်မှတ်တဲ့ approach ကို ခေါ်တဲ့ နာမည်ပါ။ NetBeans GUI Builder ရဲ့ `AbsoluteLayout` custom class ကတော့ null layout ကို wrap လုပ်ပြီး ပိုအဆင်ပြေအောင် လုပ်ပေးထားတာပါ။

## 2. CardLayout

Panel တစ်ခုထဲမှာ "ကတ်" တွေလိုပုံစံ layer အနေနဲ့ component (ပုံမှန်အားဖြင့် JPanel) များစွာကို ထည့်ထားပြီး **တစ်ကြိမ်မှာ တစ်ခုသာ** ပြသတဲ့ layout ဖြစ်ပါတယ်။ Wizard steps, Login/Register screen switch, Tab-like navigation စတာတွေအတွက် အသုံးများပါတယ်။

```java
CardLayout cl = new CardLayout();
JPanel mainPanel = new JPanel(cl);

mainPanel.add(loginPanel, "LOGIN");
mainPanel.add(registerPanel, "REGISTER");

cl.show(mainPanel, "REGISTER"); // "REGISTER" ဆိုတဲ့ card ကို ပြသမယ်
```

- Panel တစ်ခုစီကို name (String) နဲ့ register လုပ်ရပါတယ်
- `next()`, `previous()`, `first()`, `last()`, `show()` methods သုံးပြီး ပြောင်းနိုင်ပါတယ်

## 3. OverlayLayout

Component တွေကို **layer ချင်း ထပ်တင်** ပြသချင်တဲ့အခါ သုံးတဲ့ layout ဖြစ်ပါတယ် (transparent effect, watermark, badge overlay စတာတွေအတွက်)။ CardLayout နဲ့ မတူတာက CardLayout က တစ်ခါတည်း တစ်ခုချင်းပဲ ပြသပေမယ့် OverlayLayout က component **အားလုံးကို တပြိုင်နက် ထပ်တင်ပြသ**ပေးတာပါ။

```java
JPanel panel = new JPanel();
panel.setLayout(new OverlayLayout(panel));

JLabel background = new JLabel(new ImageIcon("bg.png"));
JLabel textLabel = new JLabel("Overlay Text");
textLabel.setAlignmentX(0.5f);
textLabel.setAlignmentY(0.5f);
background.setAlignmentX(0.5f);
background.setAlignmentY(0.5f);

panel.add(textLabel);   // အပေါ်ဆုံး layer
panel.add(background);  // အောက်ဆုံး layer
```

- `setAlignmentX()` / `setAlignmentY()` သုံးပြီး layer တွေရဲ့ alignment ကို ထိန်းချုပ်ရပါတယ်
- Component တွေ ထည့်တဲ့ order က layering order ကို သက်ရောက်ပါတယ် (ပထမဆုံး ထည့်ထားတာက ပေါ်နေမယ်)

---

**နှိုင်းယှဉ်ချက် အနှစ်ချုပ်:**

|Layout|ဘာသုံးလဲ|Multiple component တစ်ချိန်တည်း မြင်ရလား|
|---|---|---|
|Null/Absolute|Pixel အတိအကျ position|✅ (fixed position)|
|CardLayout|Screen/panel switching|❌ (တစ်ခုချင်းပဲ)|
|OverlayLayout|Layer ထပ်တင်ပြသ|✅ (ထပ်ပြီး ပြသ)|

FlatLaf နဲ့ တွဲသုံးမယ်ဆိုရင် Null Layout ကတော့ theme scale (HiDPI) ပြောင်းရင် ပြဿနာ ဖြစ်နိုင်လို့ FlatLaf official doc တွေမှာလည်း GridBagLayout/MigLayout လို responsive layout တွေကို ပိုအကြံပြုလေ့ရှိပါတယ်။

Next lesson ဆက်လက်လေ့လာချင်ရင် ဒီ ၄ခုထဲက ဘယ်တစ်ခုကို code example ပိုပြီး လက်တွေ့ practice လုပ်ချင်လဲ?