# API အမျိုးအစားများ (Types of APIs)

API (Application Programming Interface) ဆိုတာ software system နှစ်ခု ဒါမှမဟုတ် ပိုပြီး ချိတ်ဆက်ပြီး ဒေတာ ဒါမှမဟုတ် function တွေကို share လုပ်ဖို့ သုံးတဲ့ interface ဖြစ်ပါတယ်။ အောက်မှာ အဓိက API အမျိုးအစားတွေကို architecture style အလိုက်၊ ရော access level အလိုက် ခွဲပြီး ရှင်းပြပေးမယ်။

## ၁။ Architecture / Communication Style အလိုက် ခွဲခြင်း

### REST API (Representational State Transfer)

- ယနေ့အသုံးအများဆုံး API style ဖြစ်ပါတယ်
- HTTP methods တွေဖြစ်တဲ့ **GET, POST, PUT, DELETE** တို့ကို သုံးပြီး resource (data) တွေကို manage လုပ်ပါတယ်
- Data ကို JSON ဒါမှမဟုတ် XML format နဲ့ ပို့ချေတာများပြီး JSON က ပိုအသုံးများပါတယ်
- **Stateless** ဖြစ်ပါတယ် — request တစ်ခုချင်းစီဟာ ကိုယ်ပိုင်လုံလောက်တဲ့ information ပါဝင်ပြီး server ဘက်က ဘာမှ session မမှတ်ထားပါ
- ဥပမာ - `GET /users/123` ဆိုရင် user id 123 ရဲ့ data ကို ရယူတာပါ

### SOAP API (Simple Object Access Protocol)

- Protocol တစ်ခုအနေနဲ့ ပိုတင်းကျပ်ပါတယ် (strict standard)
- **XML** format ကိုပဲ အသုံးပြုပါတယ်
- Security နဲ့ transaction reliability လိုအပ်တဲ့ banking, enterprise system တွေမှာ များများသုံးပါတယ်
- REST ထက် setup ခက်ပြီး data size ပိုကြီးပါတယ်

### GraphQL

- Facebook က တီထွင်ခဲ့တဲ့ query language ဖြစ်ပါတယ်
- Client ဘက်က **သူလိုချင်တဲ့ data field တွေကိုပဲ** ရွေးပြီး request လုပ်နိုင်ပါတယ် (over-fetching/under-fetching ပြဿနာကို ဖြေရှင်းတယ်)
- Endpoint တစ်ခုတည်းနဲ့ complex query တွေအားလုံးကို ကိုင်တွယ်နိုင်ပါတယ်
- REST လို endpoint အများကြီး ခွဲစရာမလိုပါ

### WebSocket API

- **Real-time**, two-way (bidirectional) communication အတွက် သုံးပါတယ်
- Connection တစ်ခုကို ဖွင့်ထားပြီး client နဲ့ server နှစ်ဖက်စလုံးက data ကို ဆက်တိုက်ပို့နိုင်ပါတယ်
- Chat app, live notification, online game, stock price update တွေမှာ သုံးပါတယ်

### gRPC (Google Remote Procedure Call)

- Google တီထွင်ခဲ့ပြီး **Protocol Buffers (protobuf)** ကို data format အနေနဲ့ သုံးပါတယ်
- JSON ထက် speed မြန်ပြီး size သေးပါတယ်
- Microservices architecture တွေမှာ service တစ်ခုနဲ့တစ်ခု communicate လုပ်ဖို့ popular ဖြစ်ပါတယ်

## ၂။ Access Level အလိုက် ခွဲခြင်း

|အမျိုးအစား|ရှင်းလင်းချက်|
|---|---|
|**Open/Public API**|Developer ဘယ်သူမဆို free ဒါမှမဟုတ် အနည်းငယ် restriction နဲ့ သုံးနိုင် (ဥပမာ - OpenWeatherMap)|
|**Internal/Private API**|Company တစ်ခုအတွင်းမှာပဲ team တွေကြားသုံးဖို့၊ public ကို မဖွင့်ပေးပါ|
|**Partner API**|Business partner တွေနဲ့ contract/agreement အောက်မှာ သုံးခွင့်ပြု (ဥပမာ - payment gateway)|
|**Composite API**|Call အများကြီးကို တစ်ခုတည်းအဖြစ် ပေါင်းပြီး efficient ဖြစ်အောင် လုပ်ထားတဲ့ API|

## ၃။ Java Developer အနေနဲ့ သိထားသင့်တဲ့ Practical Point

Swing project (NetBeans, FlatLaf) လုပ်နေတဲ့အနေနဲ့ REST API ကို Java ကနေ call လုပ်ချင်ရင် **HttpClient** (Java 11+ built-in) ဒါမှမဟုတ် **OkHttp**, **Retrofit** library တွေကို သုံးလို့ ရပါတယ်။ Response ကို JSON parse လုပ်ဖို့ **Jackson** ဒါမှမဟုတ် **Gson** library ကို သုံးလေ့ရှိပါတယ်။

---

ဒီ topic ကို part-by-part အသေးစိတ် (ဥပမာ - REST API ကို Java Swing app ထဲက ဘယ်လို call လုပ်မလဲ) ဆက်လေ့လာချင်ရင် ပြောပါ၊ ဆက်ရှင်းပြပေးပါ့မယ်။