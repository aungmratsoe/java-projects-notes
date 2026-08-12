CardLayout ကို NetBeans Swing GUI Builder မှာ Drag & Drop နဲ့ အသုံးပြုပုံကို ရှင်းပြပါမယ်။

## 1. CardLayout ဆိုတာဘာလဲ

CardLayout က panel တစ်ခုတည်းအတွင်းမှာ "ကတ်များ" (cards) အများကြီးကို layer ချထားပြီး တစ်ကြိမ်ကို တစ်ကတ်ပဲ ပြပေးတဲ့ layout manager ဖြစ်ပါတယ်။ Wizard step တွေ၊ Login/Register screen switch၊ tab-like navigation စတာတွေအတွက် အသုံးများပါတယ်။

## 2. NetBeans GUI Builder မှာ Drag & Drop နည်း

**Step 1** — Container တစ်ခု (ဥပမာ `JPanel`) ကို Form ပေါ်ကို drag & drop လုပ်ပါ။ ဒီ panel ကပဲ CardLayout ရဲ့ "master container" ဖြစ်ပါလိမ့်မယ်။

**Step 2** — ထည့်ထားတဲ့ panel ကို click ရွေးပြီး Properties window ထဲက **Layout** dropdown ကနေ **CardLayout** ကို ရွေးပါ။

**Step 3** — အခု panel ရဲ့ layout က CardLayout ဖြစ်သွားပြီ ဖြစ်လို့၊ ဒီ panel အထဲကို card တစ်ခုချင်းစီအတွက် sub-panel (JPanel) တွေကို ထပ်ဆွဲထည့်ပါ (ဥပမာ - `panelLogin`, `panelRegister`, `panelDashboard`)။

**Step 4** — Sub-panel တစ်ခုချင်းစီကို right-click ပြီး **"Card Name"** ကို သတ်မှတ်ပေးရပါမယ် (default က "Card1", "Card2"... စသဖြင့် auto assign ဖြစ်ပါတယ်၊ ပြင်ချင်ရင် edit လုပ်နိုင်ပါတယ်)။ ဒီ name ကိုပဲ code ထဲမှာ card ပြောင်းဖို့ သုံးရမှာပါ။

## 3. Auto-Generated Code

NetBeans က အောက်ပါလို code ကို auto-generate လုပ်ပေးပါလိမ့်မယ်:

```java
private javax.swing.JPanel mainPanel;
private java.awt.CardLayout cardLayout1;

// initComponents() ထဲမှာ
mainPanel.setLayout(new java.awt.CardLayout());
mainPanel.add(panelLogin, "cardLogin");
mainPanel.add(panelRegister, "cardRegister");
```

## 4. Code ကနေ Card ပြောင်းနည်း

Button တစ်ခု click လိုက်တာနဲ့ card ပြောင်းချင်ရင်:

```java
CardLayout cl = (CardLayout) mainPanel.getLayout();
cl.show(mainPanel, "cardRegister");
```

- `cl.show(container, "cardName")` — ဖော်ပြထားတဲ့ name အတိုင်း card ကို ပြောင်းပြပေးတယ်
- `cl.next(mainPanel)` — နောက်ကတ်ကို ပြောင်း
- `cl.previous(mainPanel)` — ရှေ့ကတ်ကို ပြန်ပြောင်း
- `cl.first(mainPanel)` / `cl.last(mainPanel)` — ပထမ/နောက်ဆုံးကတ်ကို ပြောင်း

## 5. လက်တွေ့ ဥပမာ (Login → Dashboard Switch)

```java
private void btnLoginActionPerformed(java.awt.event.ActionEvent evt) {
    if (validateLogin()) {
        CardLayout cl = (CardLayout) mainPanel.getLayout();
        cl.show(mainPanel, "cardDashboard");
    }
}
```

## 6. NetBeans GUI Builder သုံးရင် သတိထားရမယ့်အချက်များ

- Card name တွေကို **spelling အတိအကျ** မှတ်ထားပါ — `cl.show()` မှာ typo ရှိရင် exception မတက်ဘဲ silently ဘာမှ မဖြစ်ဘူးလို့ ခံစားရနိုင်ပါတယ် (card ပြောင်းမသွားဘူး)
- Design view ထဲမှာ card multiple ရှိနေရင် တစ်ချိန်ကို တစ်ခုပဲ visible ပြပါလိမ့်မယ် — အောက်ခြေက dropdown (သို့) right-click "Switch Card" ကနေ Design view ထဲမှာလည်း preview ပြောင်းကြည့်လို့ ရပါတယ်
- Card sub-panel တစ်ခုချင်းစီအတွင်းမှာ layout ကို သီးခြား (ဥပမာ GroupLayout, BorderLayout) သတ်မှတ်နိုင်ပါတယ် — CardLayout က parent container ပေါ်မှာပဲ သက်ရောက်ပါတယ်

---

Card name string တွေကို hardcode မလုပ်ချင်ရင် `constant` (`public static final String CARD_LOGIN = "cardLogin";`) အနေနဲ့ ထားပြီး သုံးရင် typo error လျှော့နိုင်ပါတယ်။ ဆက်လက်လေ့လာချင်တဲ့ Swing topic ရှိရင် ပြောပြပါ။