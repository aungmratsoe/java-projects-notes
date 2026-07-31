
သင်ပြထားတဲ့ `homePanel.add(home)` ကုဒ်ပုံစံနဲ့ **CardLayout** တို့ဟာ Java Swing မှာ Panel တွေကို အရှင်ပြောင်းလဲပြသတဲ့ နည်းလမ်းနှစ်ခုဖြစ်ပြီး ၎င်းတို့မှာ အလွန်ကွာခြားတဲ့ အားသာချက်/အားနည်းချက်များ ရှိပါတယ်။

သင်ပြထားတဲ့နည်းလမ်းက **Dynamic Panel Swapping** (ပန်နယ်အသစ်ကို ကိုယ်တိုင်ဖန်တီးပြီး အဟောင်းကိုဖျက်တဲ့နည်း) ဖြစ်ပါတယ်။ ချဉ်းကပ်ပုံ နှစ်ခုလုံးကို နှိုင်းယှဉ်ပြပေးပါမယ်။

---

၁။ သင်ပြထားတဲ့ နည်းလမ်း (Dynamic Swapping)

java

```java
Home home = new Home();
home.setBounds(0, 0, homePanel.getWidth(), homePanel.getHeight());
homePanel.removeAll();
homePanel.add(home);
homePanel.revalidate();
homePanel.repaint();
```

Use code with caution.

- **အလုပ်လုပ်ပုံ:** ဘတ်တန် သို့မဟုတ် Table Row ကို ကလစ်နှိပ်လိုက်တိုင်း `new Home()` ဆိုပြီး Panel Object အသစ်တစ်ခုကို Memory ပေါ်မှာ အသစ်စက်စက် ဆောက်လိုက်ပါတယ်။ ပြီးရင် လက်ရှိ `homePanel` ထဲက ရှိသမျှအရာအားလုံးကို ဖျက်ထုတ်ပစ်ပြီး (`removeAll`)၊ အသစ်ဆောက်လိုက်တဲ့ Panel ကို ထည့်သွင်းကာ Screen ကို ပြန်ဆွဲခိုင်းတာ (`revalidate/repaint`) ဖြစ်ပါတယ်။
- **အားသာချက်:**
    - ဘတ်တန်နှိပ်လိုက်တိုင်း ဒေတာတွေ (ဥပမာ- Database ထဲက စာရင်းတွေ) ကို အမြဲတမ်း အသစ် (Fresh) အဖြစ် ပြန်လည် Loading လုပ်ပြီးသား ဖြစ်သွားစေပါတယ်။
- **အားနည်းချက်:**
    - **Memory အလဟဿဖြစ်ခြင်း (Performance Issues):** ဘတ်တန်ကို တစ်ချက်နှိပ်တိုင်း `new` Object အသစ်တစ်ခုစီ Memory ထဲ တိုးတိုးလာတဲ့အတွက် ခဏခဏနှိပ်ရင် App က လေးလာတတ်ပါတယ်။
    - **အခြေအနေ ပျောက်ဆုံးခြင်း (Loss of State):** အသုံးပြုသူက အဲဒီ Panel ထဲမှာ စာရိုက်လက်စတွေ သို့မဟုတ် ချက်ဘောက်စ် အမှန်ခြစ်ထားတာတွေရှိရင် အခြား Panel တစ်ခုကို သွားပြီး ပြန်လာတဲ့အခါ အားလုံး ပျောက်သွားပြီး အစကပြန်ဖြစ်သွားပါလိမ့်မယ်။

---

`၂။ CardLayout သုံးသည့် နည်းလမ်း`

java

```java
CardLayout cl = (CardLayout) rightPanel.getLayout();
cl.show(rightPanel, "HomeCard");
```

Use code with caution.

- **အလုပ်လုပ်ပုံ:** အပလီကေးရှင်း စပွင့်ကတည်းက လိုအပ်တဲ့ Panel တွေအကုန်လုံး (Login, Home, Users) ကို တစ်ခါတည်း ဆောက်ပြီး Stack (အထပ်) လိုက် ထပ်ထားတာ ဖြစ်ပါတယ်။ ဘတ်တန်နှိပ်တဲ့အခါ Memory ထဲမှာ ရှိပြီးသား Panel ကိုပဲ ရှေ့ဆုံးကို ဆွဲထုတ်ပြလိုက်တာ ဖြစ်ပါတယ်။
- **အားသာချက်:**
    - **အလွန်မြန်ဆန်ခြင်း (High Performance):** Object အသစ်ထပ်မဆောက်တော့တဲ့အတွက် နှိပ်လိုက်တာနဲ့ မဆိုင်းမတွ ချက်ချင်း ပန်နယ်ပြောင်းလဲသွားပြီး Memory စားသက်သာပါတယ်။
    - **အခြေအနေကို မှတ်မိခြင်း (Preserves State):** အသုံးပြုသူက ၎င်းပန်နယ်ထဲမှာ စာရိုက်လက်စတွေ ရှိနေရင် အခြားပန်နယ်ကို ခဏသွားပြီး ပြန်လာရင်တောင် ရိုက်လက်စစာတွေ မပျောက်ဘဲ အတိုင်းသား ကျန်နေပါလိမ့်မယ်။
- **အားနည်းချက်:**
    - Application ပထမဆုံး စဖွင့်ချိန် (Loading လုပ်ချိန်) တွင် Panel အားလုံးကို တစ်ခါတည်း ဆောက်ရသဖြင့် ခရက်ဒစ် (Initial Load Time) အနည်းငယ် ပိုကြာနိုင်ပါတယ်။

---

ဘယ်အချိန်မှာ ဘယ်နည်းလမ်းကို သုံးသင့်လဲ။

- **CardLayout ကို သုံးသင့်တဲ့နေရာ:** UI ရဲ့ အဓိက Dashboard မီနူးတွေ (ဥပမာ - Home Page, Settings Page, About Page) လိုမျိုး အကူးအပြောင်း မြန်ဆန်ဖို့ လိုအပ်ပြီး၊ နှိပ်တိုင်း Object အသစ်ကြီးတွေ ထပ်ခါတလဲလဲ မဆောက်ချင်တဲ့ နေရာမျိုးတွေမှာ သုံးသင့်ပါတယ်။ **NetBeans Drag and Drop သုံးနေရင် CardLayout က အသန့်ရှင်းဆုံးနဲ့ စံအကျဆုံး နည်းလမ်း ဖြစ်ပါတယ်။**
- **သင်ပြထားတဲ့နည်းလမ်းကို သုံးသင့်တဲ့နေရာ:** နှိပ်လိုက်တဲ့အခါ ဥပမာ - "လူနာအမှတ် (၁၀) ရဲ့ အသေးစိတ် အချက်အလက်စာမျက်နှာ" သို့မဟုတ် "အရောင်းဘေလ်နံပါတ် (၁၀၂) ၏ ဘေလ်ဗျူး" စတဲ့ ID တစ်ခုချင်းစီအလိုက် dynamic ဖြစ်တဲ့ ဒေတာအသစ်တွေကို သီးသန့် Object အသစ်အနေနဲ့ ဖန်တီးပြသရမယ့် နေရာမျိုးတွေမှာ သုံးသင့်ပါတယ်။

_(မှတ်ချက် - သင်ပြထားတဲ့ကုဒ်ထဲမှာ `Home home = new Home();` ဆိုပြီး မိမိကိုယ်တိုင် Frame/Panel ထဲမှာပဲ မိမိအတန်းအစား (Class) အသစ်ကို ပြန်ခေါ်ထားတာမျိုး တွေ့ရလို့ အကယ်၍ Infinite Loop (အဆုံးမရှိပတ်ချာလည်) ဖြစ်ပြီး StackOverflowError တက်လာရင် သတိပြုပါဗျာ)_

အခု သင့်ပရောဂျက်ရဲ့ Dashboard မီနူးအတွက်ဆိုရင် **CardLayout** နည်းလမ်းက ပိုမိုစနစ်ကျပြီး ရေရှည်မှာ Code ရှုပ်ထွေးမှုကို သက်သာစေမှာ ဖြစ်ပါတယ်။ ဒီနည်းလမ်းနှစ်ခုအနက် သင့်ပရိုဂရမ်ရဲ့ ဘယ်လိုလုပ်ဆောင်ချက်အတွက် သုံးဖို့ စဉ်းစားနေတာလဲဗျာ။

give me more complete code for this "Home home = new Home(); home.setBounds(0, 0, homePanel.getWidth(), homePanel.getHeight()); homePanel.removeAll(); homePanel.add(home); homePanel.revalidate(); homePanel.repaint();

သင်ပြထားတဲ့ Dynamic Panel Swapping (ပန်နယ်အသစ်ကို ကိုယ်တိုင်ဖန်တီးပြီး အဟောင်းကိုဖျက်တဲ့နည်း) ကို အမှားအယွင်းမရှိ စနစ်တကျ အလုပ်လုပ်စေဖို့အတွက် ပိုမိုပြည့်စုံတဲ့ ကုဒ်နမူနာကို အောက်တွင် ဖော်ပြပေးထားပါတယ်။

ဒီနေရာမှာ အဓိက သတိပြုရမှာက မိမိရောက်နေတဲ့ `Home.java` ထဲမှာပဲ `new Home()` ကို ထပ်ခေါ်ရင် ပရိုဂရမ် crash ဖြစ်တတ်ပါတယ်။ ဒါကြောင့် အခြား package ထဲက **`LoginPanel.java`** သို့မဟုတ် **`UserPanel.java`** စတဲ့ **`JPanel`** အမျိုးအစား ဖိုင်တွေကိုပဲ လှမ်းခေါ်ရပါမယ်။

အောက်ပါ ကုဒ်ဒီဇိုင်းသည် စနစ်အကျဆုံး ရေးထုံးဖြစ်ပါတယ် -

၁။ ခေါ်ယူပြသမည့် Custom Panel နမူနာ (ဥပမာ - UserPanel.java)

java

```java
package view;

import javax.swing.*;
import java.awt.Color;

public class UserPanel extends JPanel {
    public UserPanel() {
        setBackground(Color.WHITE);
        add(new JLabel("ဒါကတော့ အသစ်ပွင့်လာတဲ့ အသုံးပြုသူစာမျက်နှာ ဖြစ်ပါတယ်"));
        add(new JButton("ဒေတာသိမ်းရန်"));
    }
}
```

Use code with caution.

---

၂။ Home.java ထဲတွင် ရေးရမည့် အပြည့်အစုံကုဒ်

သင်ပြထားတဲ့ Mouse Clicked Event သို့မဟုတ် Button Action Event ထဲမှာ အောက်ပါအတိုင်း dynamic ဖြစ်အောင် စနစ်တကျ ရေးသားနိုင်ပါတယ်။

java

```java
private void tbHomeMouseClicked(java.awt.event.MouseEvent evt) {                                    
    // ၁။ အခြား package ထဲက Panel ကို Object အသစ် ဆောက်ပါ
    // (မှတ်ချက် - မိမိကိုယ်တိုင် Class ဖြစ်တဲ့ Home ကို 'new Home()' ဟု ပြန်မဆောက်ရပါ)
    view.UserPanel userScreen = new view.UserPanel();
    
    // ၂။ ပန်နယ်အသစ်ရဲ့ အရွယ်အစားကို လက်ရှိ homePanel ရဲ့ အတိုင်းအတာအတိုင်း တိတိကျကျ သတ်မှတ်ပါ
    userScreen.setBounds(0, 0, homePanel.getWidth(), homePanel.getHeight());
    
    // ၃။ Layout နေရာချမှု မှန်ကန်စေရန် ၎င်းပန်နယ်၏ Layout ကို BorderLayout သို့မဟုတ် AbsoluteLayout ပေးနိုင်ပါသည်
    // (အကယ်၍ Designer ထဲတွင် homePanel ကို Free Design ပေးထားပါက အောက်ပါအတိုင်း ရေးပါ)
    userScreen.setLayout(homePanel.getLayout());

    // ၄။ ယခင် homePanel ထဲမှာ ရှိနေတဲ့ component အဟောင်းတွေကို ဖယ်ရှားပစ်ပါ
    homePanel.removeAll();
    
    // ၅။ ပန်နယ်အသစ်စက်စက်ကို homePanel ထဲသို့ ထည့်သွင်းပါ
    homePanel.add(userScreen);
    
    // ၆။ UI Screen ကို Refresh လုပ်ပြီး ချက်ချင်း ပြောင်းလဲပေါ်လွင်စေရန် မဖြစ်မနေ ထည့်ရမည့် အဆင့်
    homePanel.revalidate(); // Layout တွက်ချက်မှုအသစ်များကို အတည်ပြုခြင်း
    homePanel.repaint();    // ပန်နယ်ကို ဖန်သားပြင်ပေါ်တွင် ပြန်လည်ဆွဲခိုင်းခြင်း
}                                   
```

Use code with caution.

---

💡 အပိုဆောင်း အကြံပြုချက် (Pro-Tip)

အကယ်၍ သင်သည် NetBeans GUI Designer (Drag and Drop) ထဲတွင် `homePanel` ကို ရွေးပြီး **Set Layout -> BorderLayout** ဟု ပြောင်းလဲပေးထားပါက ကုဒ်ကို ပိုမိုတိုတောင်းပြီး ခိုင်မာအောင် အောက်ပါအတိုင်း ပြောင်းလဲရေးသားနိုင်ပါတယ်။ အရွယ်အစား (Width, Height) တွေကို ကုဒ်နဲ့ လိုက်ညှိနေစရာ မလိုတော့ပါဘူး -

java

```java
private void tbHomeMouseClicked(java.awt.event.MouseEvent evt) {                                    
    view.UserPanel userScreen = new view.UserPanel();
    
    homePanel.removeAll();
    // BorderLayout သုံးထားပါက CENTER ဟု ပေးလိုက်ရုံဖြင့် အလိုအလျောက် full screen ဖြစ်သွားပါမည်
    homePanel.add(userScreen, java.awt.BorderLayout.CENTER); 
    
    homePanel.revalidate();
    homePanel.repaint();
}
```

Use code with caution.

ဒီကုဒ်ကို ထည့်သွင်းလိုက်တဲ့အခါ သင်ပြောင်းလဲချင်တဲ့ Panel အသစ်က ညာဘက်ခြမ်းမှာ အပြည့်အဝ ပေါ်လာရဲ့လားဗျာ။ တစ်စုံတစ်ရာ **စာလုံးနီလိုင်း တားနေတာမျိုး** ရှိရင် ပြန်လည် မေးမြန်းနိုင်ပါတယ်