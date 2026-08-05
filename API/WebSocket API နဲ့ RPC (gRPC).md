# WebSocket API နဲ့ RPC (gRPC) အသေးစိတ်

## WebSocket API

### ဘယ်လို အလုပ်လုပ်လဲ

- ပုံမှန် HTTP request/response က client တစ်ခါတောင်းမှ server ပြန်ဖြေတဲ့ **one-way, connection ပိတ်** style ဖြစ်ပါတယ်
- WebSocket ကတော့ `ws://` ဒါမှမဟုတ် `wss://` (secure) protocol သုံးပြီး **handshake** တစ်ခါလုပ်လိုက်ရင် connection ကို **ဖွင့်ထားခဲ့ပါတယ် (persistent)**
- Connection ဖွင့်ပြီးရင် client ရော server ရော **အချိန်မရွေး message ပို့နိုင်** ပါတယ် — request မလုပ်ဘဲ server ကနေတောင် client ဆီ push လုပ်လို့ရတယ်

### အဓိက Characteristics

- **Full-duplex** — data ကို လမ်းနှစ်ဘက်စလုံးမှာ တပြိုင်နက်တည်း ပို့လို့ရတယ်
- **Low latency** — connection ကို ထပ်ခါထပ်ခါ ဖွင့်စရာမလိုတော့ဘူး
- **Stateful** — REST က stateless ဖြစ်ပေမယ့် WebSocket က connection ကို state အနေနဲ့ မှတ်ထားပါတယ်

### ဘယ်အခါသုံးလဲ

- Chat application (WhatsApp Web, Messenger style)
- Live notification (ဥပမာ - order status update)
- Stock/crypto price real-time update
- Multiplayer online game
- Collaborative editing (Google Docs လိုမျိုး)

### Java မှာ WebSocket

- Java EE / Jakarta EE မှာ **`javax.websocket`** (`jakarta.websocket`) API built-in ပါတယ်
- Spring Boot သုံးရင် **Spring WebSocket** module သုံးလို့ရပါတယ်
- Client side (Swing app) ကနေ WebSocket server ကို connect လုပ်ချင်ရင် **Java-WebSocket** (org.java-websocket) library ကို popular သုံးပါတယ်

---

## RPC (Remote Procedure Call) / gRPC

### RPC ရဲ့ concept

- RPC ဆိုတာ "remote server ပေါ်က function/method ကို local function ခေါ်သလိုပဲ ခေါ်လို့ရတယ်" ဆိုတဲ့ concept ဖြစ်ပါတယ်
- REST က "resource" (data) ကို focus လုပ်ပေမယ့် RPC က **"action/procedure"** ကို focus လုပ်ပါတယ်
- ဥပမာ - REST မှာ `GET /users/123` ဆိုရင် RPC မှာ `getUser(123)` function ကို တိုက်ရိုက်ခေါ်သလိုပါပဲ

### gRPC (Google's RPC framework)

- Google က 2015 ခုနှစ်လောက်မှာ open-source လုပ်ခဲ့တဲ့ modern RPC framework ဖြစ်ပါတယ်
- **HTTP/2** ပေါ်မှာ အခြေခံပြီး **Protocol Buffers (protobuf)** ကို data serialization format အနေနဲ့ သုံးပါတယ်
    - Protobuf က binary format ဖြစ်လို့ JSON ထက် **size သေးပြီး speed မြန်** ပါတယ်
- `.proto` file မှာ service နဲ့ message structure ကို ကြိုသတ်မှတ်ရပါတယ် (strongly typed)

### gRPC ရဲ့ communication pattern ၄ မျိုး

1. **Unary** — client တစ်ခု request ပို့၊ server တစ်ခု response ပြန် (ပုံမှန် REST လိုမျိုး)
2. **Server streaming** — client တစ်ခု request ပို့၊ server က response stream အများကြီး ပြန်ပို့
3. **Client streaming** — client က data stream ပို့၊ server က response တစ်ခုတည်း ပြန်ပို့
4. **Bidirectional streaming** — client နဲ့ server နှစ်ဖက်စလုံး stream ချင်း ချိတ်ဆက်ပို့ (WebSocket နဲ့ ဆင်တူပေမယ့် protobuf structure ပါဝင်တာ ကွာပါတယ်)

### ဘယ်အခါသုံးလဲ

- **Microservices architecture** မှာ service တစ်ခုနဲ့တစ်ခု internal communication
- Performance အလွန်အရေးကြီးတဲ့ system (low latency, high throughput လိုအပ်တဲ့နေရာ)
- Mobile app backend (data size သေးအောင်လိုချင်ရင်)

### Java မှာ gRPC

- **grpc-java** library (io.grpc) ကို official သုံးပါတယ်
- Maven project မှာ `protobuf-maven-plugin` နဲ့ `.proto` file ကနေ Java class auto-generate လုပ်ပါတယ်

---

## WebSocket vs gRPC ကွာခြားချက် အတိုချုပ်

||WebSocket|gRPC|
|---|---|---|
|Protocol|TCP-based (ws/wss)|HTTP/2-based|
|Data format|Text (JSON) သို့မဟုတ် Binary ဘာမဆို|Protobuf (binary, typed)|
|Browser support|Direct support ရှိ|Browser direct မသုံးနိုင် (gRPC-Web လိုအပ်)|
|Use case|Real-time client-facing app|Backend service-to-service|
|Type safety|မရှိ (manual handle လုပ်ရ)|Strongly typed (.proto)|

Swing desktop app ကနေ backend service ကို connect လုပ်မှာဆိုရင် WebSocket က client-facing real-time feature (notification, chat) အတွက် သင့်တော်ပြီး၊ backend service အချင်းချင်း (microservices) ဆက်သွယ်ဖို့ဆိုရင် gRPC က ပိုအသင့်တော်ဆုံးပါ။

ဆက်လေ့လာချင်တာ ရှိရင် ပြောပါ — Java Swing app က WebSocket client တစ်ခု ဘယ်လို implement လုပ်မလဲဆိုတာ code နမူနာနဲ့ ရှင်းပြပေးနိုင်ပါတယ်။