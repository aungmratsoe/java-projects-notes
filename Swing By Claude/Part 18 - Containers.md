JavaSwing container တွေရဲ့ အသုံးဝင်ပုံ (purpose) တွေကို တစ်ခုချင်းစီ ရှင်းပြပေးပါမယ်။

## 1. **JTabbedPane** (Tabbed Pane)

- Screen တစ်ခုထဲမှာ tab (စာမျက်နှာအလွှာ) များစွာကို ထည့်ပြီး user ကို tab ခေါင်းစဉ်ကို click နှိပ်ပြီး ပြောင်းကြည့်ခိုင်းတဲ့ container ပါ။
- **ဥပမာ**: Settings dialog တစ်ခုမှာ "General", "Appearance", "Advanced" ဆိုပြီး tab သုံးခု ခွဲထားတာမျိုး။
- Screen space ကို စုစည်းသုံးချင်တဲ့အခါ (form တွေ၊ data category တွေ များနေရင်) အသုံးများပါတယ်။

## 2. **JSplitPane** (Split Pane)

- Panel နှစ်ခုကို ဘေးချင်းချင်း (horizontal) ဒါမှမဟုတ် အပေါ်-အောက် (vertical) ခွဲပြီး၊ user ကိုယ်တိုင် divider ကို drag ဆွဲပြီး အရွယ်အစား ချိန်ခွင်လျှာ ချိန်နိုင်အောင် ပေးထားတဲ့ container ပါ။
- **ဥပမာ**: File explorer တစ်ခုမှာ ဘယ်ဘက် folder tree, ညာဘက် file list ခွဲပြထားတာမျိုး (NetBeans IDE ကိုယ်တိုင်ကလည်း ဒီ pattern သုံးထားတယ်)။

## 3. **JToolBar** (Tool Bar)

- Button တွေ၊ icon တွေကို တန်းစီပြီး frequently used action တွေကို quick access ပေးဖို့ container ပါ။
- Default အနေနဲ့ drag လုပ်ပြီး frame ဘေးတစ်ဖက်ဖက်ကို ရွှေ့နိုင်တယ် (floatable feature)။
- **ဥပမာ**: Word processor ရဲ့ Save, Print, Cut, Copy, Paste icon တန်းတွေမျိုး။

## 4. **JDesktopPane** (Desktop Pane)

- MDI (Multiple Document Interface) application တွေအတွက် "desktop background" အဖြစ် သုံးတဲ့ container ဖြစ်ပြီး၊ **JInternalFrame** တွေကို host လုပ်ပေးတဲ့ base container ပါ။
- **ဥပမာ**: Application တစ်ခုထဲမှာ window သေးသေးလေးများစွာကို (child windows) တစ်ပြိုင်နက်တည်း ဖွင့်ထားနိုင်တဲ့ Photoshop/Office-style app တွေ။

## 5. **JInternalFrame** (Internal Frame)

- JDesktopPane ထဲမှာ ပေါ်တဲ့ "window ထဲက window" လေးတွေပါ - ပုံမှန် JFrame လိုပဲ title bar, minimize/maximize/close button ရှိတယ်၊ ဒါပေမယ့် main application frame ထဲမှာပဲ ရွေ့လျား၊ resize လုပ်နိုင်တယ်။
- JDesktopPane နဲ့ တွဲသုံးရတယ်။

## 6. **JLayeredPane** (Layered Pane)

- Component တွေကို **layer (Z-axis) အလိုက်** အထပ်ထပ် ထားနိုင်တဲ့ container ပါ - ဘယ် component က ဘယ် component ရဲ့ အပေါ်/အောက် ပြပေးမလဲဆိုတာ control လုပ်နိုင်တယ်။
- JDesktopPane ကိုယ်တိုင်က JLayeredPane ကို base အနေနဲ့ အသုံးပြုထားတာပါ (internal frame windowing effect ရအောင်)။
- **ဥပမာ**: Tooltip တစ်ခု ဒါမှမဟုတ် popup menu တစ်ခုကို component တခြားတွေရဲ့ အပေါ်ကနေ ပေါ်စေချင်တဲ့အခါ layer နဲ့ control လုပ်တာမျိုး။

---

**အနှစ်ချုပ်**: JTabbedPane/JSplitPane/JToolBar က ပုံမှန် business application UI တွေအတွက် (form၊ data entry) အသုံးများပြီး၊ JDesktopPane + JInternalFrame + JLayeredPane သုံးခုက အတူတူဆက်နွယ်နေပြီး MDI-style (window ထဲ window) application တွေ ရေးတဲ့အခါမှသာ တွဲသုံးရတာ ဖြစ်ပါတယ်။ QRCode project အတွက်ဆိုရင် JTabbedPane နဲ့ JToolBar က ပိုမိုသင့်တော်ပါလိမ့်မယ်။

နောက်ထပ် code example တွေနဲ့ အသေးစိတ် လေ့လာချင်တဲ့ container ရှိရင် ပြောပါ။