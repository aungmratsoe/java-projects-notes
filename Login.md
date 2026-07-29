ဒီ `Login` class ဟာ အလွန်လှပတဲ့ Animation တွေနဲ့ အဆင့်မြင့် Biometric Login UI တစ်ခုကို ဖန်တီးထားတာပါ။ သူ့ရဲ့ အဓိက အစိတ်အပိုင်းတွေကို အောက်ပါအတိုင်း အပိုင်းလိုက် ရှင်းပြပေးပါမယ်။

### ၁။ Animation စနစ် (Animation System)

ဒီ UI ရဲ့ အဓိက ပုံစံက Fingerprint ပုံလေးမှာ တောက်ပတဲ့ အလင်း (Glow) နဲ့ စကင်ဖတ်နေတဲ့ လေဆာ (Laser sweep) ပါဝင်ပါတယ်။

- **`Timer`**: Swing ရဲ့ `Timer` ကိုသုံးပြီး Animation တွေကို စဉ်ဆက်မပြတ် လည်ပတ်စေပါတယ်။
    
    - `pulseTimer`: Fingerprint ပုံလေးရဲ့ ပတ်လည်မှာ အလင်းတန်းလေးတွေ တောက်ပနေအောင် (`pulseAlpha`) လုပ်ပေးပါတယ်။
        
    - `laserTimer`: အပေါ်ကနေ အောက်ကို ရွေ့လျားနေတဲ့ လေဆာရောင်ခြည် (Laser line) အတွက် `laserYFraction` ကို အသုံးပြုထားပါတယ်။
        
- **`paintComponent`**: ဒီနေရာမှာ Java2D (`Graphics2D`) ကို အသုံးပြုပြီး ပုံတွေကို ဆွဲပါတယ်။ `isError` ဆိုတဲ့ Boolean variable ကိုသုံးပြီး၊ မှန်ကန်ရင် အစိမ်းရောင်၊ မှားယွင်းရင် အနီရောင် (Red) အရောင်ကို ပြောင်းလဲပေးပါတယ်။
    

### ၂။ Passcode ထည့်သွင်းခြင်း (Passcode Dialog)

- **`promptPasscode()`**: ဒီ Method က `JOptionPane` ကိုသုံးပြီး Password ရိုက်ထည့်ဖို့ Box တစ်ခုကို ပေါ်လာစေပါတယ်။
    
- **Auto-focus**: `AncestorListener` ကို သုံးထားတာက Dialog ပေါ်လာတာနဲ့ Password ရိုက်ရမယ့်နေရာ (`pf`) ကို အလိုအလျောက် Focus ရောက်သွားအောင် လုပ်ပေးတာပါ။ ဒါကြောင့် User အနေနဲ့ Dialog ပေါ်လာတာနဲ့ Password စရိုက်လို့ ရပါတယ်။
    

### ၃။ အဆင့်ဆင့် လုပ်ဆောင်ချက် (Logic Flow)

1. **UI စတင်ခြင်း (`initCustomDesign`)**: Window ပွင့်တာနဲ့ `startCoolScanAnimation()` ကို ခေါ်ပြီး Animation တွေကို စတင်ပါတယ်။
    
2. **Password စစ်ဆေးခြင်း**: User ရိုက်လိုက်တဲ့ Password က `1234` ဒါမှမဟုတ် `admin` ဖြစ်ရင် -
    
    - `isError = false` ဆိုပြီး သတ်မှတ်တယ်။
        
    - `updateStatus` ကိုသုံးပြီး အောင်မြင်ကြောင်း ပြသတယ်။
        
    - `Timer` တစ်ခုကို သုံးပြီး ခဏစောင့်ကာ `Home` screen ကို ပြောင်းပေးတယ်။
        
3. **မှားယွင်းပါက**: `isError = true` လို့ သတ်မှတ်ပြီး, Animation ကို အနီရောင်နဲ့ ပြန်စပါတယ်။
    

### ၄။ UI ဒီဇိုင်း (Custom Panel)

- **`CustomPurplePanel`**: ဒါက `JPanel` ကို Extension လုပ်ထားတာပါ။ `GradientPaint` ကို သုံးပြီး အောက်ခြေမှာ ခရမ်းရောင် အနုအရင့် (Gradient background) ကို ဖန်တီးထားတာပါ။
    
- **FlatLaf**: `com.formdev.flatlaf` library ကိုသုံးထားလို့ UI ပုံစံက ခေတ်မီပြီး သပ်ရပ်နေတာပါ။
    

### အနှစ်ချုပ် အချက်များ

- **`paintComponent` ကို override လုပ်ထားခြင်း**: ဒါက ဒီ UI ရဲ့ အသက်ပါ။ အပေါ်ယံ ပုံသေ Icon တစ်ခုတည်းကို မသုံးဘဲ ကိုယ်ပိုင် Graphic တွေ ဆွဲထားလို့ ပိုလှတာပါ။
    
- **SwingUtilities.invokeLater**: ဒါကို `main` method မှာ သုံးထားတာက သင်အရင်က မေးထားတဲ့အတိုင်း GUI ကို Thread-safe ဖြစ်အောင် (EDT ထဲမှာ) အလုပ်လုပ်စေဖို့ ဖြစ်ပါတယ်။
    

**သင်ကိုယ်တိုင် ပြင်ဆင်ကြည့်လိုပါက:** ဒီ code ထဲမှာ `1234` ဆိုတဲ့ Password နေရာကို သင်ပြောင်းချင်တဲ့ Password နဲ့ အစားထိုးကြည့်နိုင်ပါတယ်။ နောက်ပြီး Animation ရဲ့ အမြန်နှုန်းကို ပြောင်းချင်ရင် `Timer(50, ...)` ထဲက `50` ဆိုတဲ့ ဂဏန်းကို လျှော့တာ၊ တိုးတာမျိုး စမ်းသပ်ကြည့်နိုင်ပါတယ်။