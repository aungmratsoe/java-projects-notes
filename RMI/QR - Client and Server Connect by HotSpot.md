# Mobile Hotspot ကနေတဆင့် RMI Test လုပ်ခြင်း — အလုပ်ဖြစ်ပါတယ်

Mobile hotspot ဟာ **WiFi router တစ်ခုအလားတူ** အလုပ်လုပ်ပါတယ် — Computer နှစ်လုံးကို hotspot ချိတ်လိုက်ရင် **same local network (subnet)** ပေါ်ရောက်သွားမှာမို့ RMI test လုပ်ဖို့ **လုံးဝ အဆင်ပြေ**ပါတယ်။ ဒါပေမယ့် သတိထားရမယ့်အချက် အနည်းငယ် ရှိပါတယ်.

## ✅ ဒီနည်းလမ်းရဲ့ အားသာချက်

- Office/Home WiFi router လိုမျိုး **"AP/Client Isolation" default disable** ဖြစ်နေတတ်ပါတယ် (Mobile hotspot တွေဟာ device-to-device communication ကို ပုံမှန် ခွင့်ပြုထားလေ့ရှိပါတယ်) — ယခင် message က ပြောခဲ့တဲ့ "Client Isolation" ပြဿနာ ဖြစ်ဖို့ ရှားပါတယ်
- Setup လွယ် — Router config panel ထဲ ဝင်စရာမလို

## ⚠️ သတိထားရမယ့် အချက်များ

### ၁. Windows Firewall — Network Profile "Public" ဖြစ်နေတတ်ခြင်း

Mobile hotspot ကို Windows ကနေ ချိတ်တဲ့အခါ **default အနေနဲ့ "Public network"** အဖြစ် classify လုပ်တတ်ပါတယ် — Public profile မှာ Windows Firewall က inbound connection အများစုကို **default ပိတ်ထား**ပါတယ်။ Port 1099 ကို firewall rule ထဲ ဖွင့်ပေးထားလည်း **"Public" profile အတွက် rule ထဲ ထည့်မထားရင်** fail ဖြစ်နိုင်ပါတယ်.

**ဖြေရှင်းနည်း**:

```
Settings → Network & Internet → WiFi → (hotspot ရဲ့ network name) → 
Network profile type → "Private" ပြောင်း
```

ဒါမှမဟုတ် Firewall rule ဖန်တီးတဲ့အခါ **"Public" profile ကိုပါ** apply ဖြစ်အောင် checkbox ၃ ခု (Domain, Private, Public) အားလုံး tick ပါ.

### ၂. Server ရဲ့ Hotspot IP ကို ရှာနည်း

```bash
# Server computer ကနေ (Windows)
ipconfig
# "Wireless LAN adapter Wi-Fi" ဆိုတဲ့ section အောက်က IPv4 Address ကို ကြည့်ပါ
# ဥပမာ - Android hotspot ဆိုရင် 192.168.43.x range ဖြစ်ချေများပါတယ်
```

`java.rmi.server.hostname` ကို ဒီ IP နဲ့ **update** လုပ်ရပါမယ် (Home WiFi IP `192.168.1.10` အဟောင်းကို ဖယ်ပြီး hotspot IP အသစ်နဲ့ ပြောင်း):

```java
System.setProperty("java.rmi.server.hostname", "192.168.43.15"); // Hotspot IP အသစ်
```

### ၃. Hotspot IP က ချိတ်တိုင်း ပြောင်းနိုင်ခြင်း (DHCP)

Mobile hotspot ရဲ့ DHCP ဟာ device ကို connect လုပ်တိုင်း **တူညီတဲ့ IP ပြန်ပေးချင်ပေးမယ်, တစ်ခါတစ်ခါ ပြောင်းနိုင်** ပါတယ် — ဒါကြောင့် test လုပ်တိုင်း `ipconfig` နဲ့ IP ကို ပြန်စစ်ပြီး hardcode IP ကို update လုပ်ရပါလိမ့်မယ် (ယခင် message က ပြောခဲ့တဲ့ **Config file (Option 1)** သုံးထားရင် code ပြန် compile မလိုဘဲ file ပဲ ပြင်ရမှာမို့ ဒီအခါမှာ အဆင်ပြေပါတယ်).

### ၄. Data Usage (Mobile Data Cost)

RMI traffic ကို **Wi-Fi hotspot ကနေတဆင့်ပဲ** (Local network) ဆက်သွယ်တာမို့ **mobile data (internet)** ကို လုံးဝ သုံးစရာမလိုပါ — Computer နှစ်လုံးက local network ပေါ်မှာပဲ တိုက်ရိုက် ပြောနေတာဖြစ်လို့ Data cost ပူစရာမလိုပါ (Phone ကို internet ချိတ်မထားလည်း hotspot ကို on ထားရင် local RMI communication အလုပ်လုပ်ပါလိမ့်မယ်).

### ၅. Phone ရဲ့ Battery/Sleep Mode

Test လုပ်နေတဲ့ အချိန်ကြာကြာ phone screen ပိတ်သွားရင် hotspot connection ပြတ်တောက်တတ်တဲ့ phone အချို့ ရှိပါတယ် (Power saving mode) — Testing လုပ်နေစဉ် phone screen on ထားခြင်း (သို့) hotspot power-saving setting ကို disable ထားခြင်း လုပ်ထားသင့်ပါတယ်.

## Test Checklist

```
1. Phone hotspot ဖွင့် (WiFi hotspot, Bluetooth hotspot မဟုတ်ဘဲ)
2. Computer ၂ လုံးစလုံးကို hotspot ချိတ်
3. Network profile ကို "Private" ပြောင်း (Windows Firewall issue ရှောင်ဖို့)
4. Server computer ကနေ ipconfig နဲ့ hotspot IP ရှာ
5. java.rmi.server.hostname ကို ဒီ IP နဲ့ update
6. Server run
7. Client computer ကနေ ping <server-hotspot-ip> နဲ့ reachability စစ်
8. Client run ပြီး Registry connect စမ်း
```

---

Test လုပ်ရာမှာ error တက်ရင် (stack trace) ပေးရင် ချက်ချင်း debug လုပ်ပေးနိုင်ပါတယ်။ ဒါမှမဟုတ် Config file (Option 1) ကို hotspot testing scenario အတွက် IP hardcode မလိုအောင် ပြီးပြည့်စုံစွာ setup ပေးရမလား?