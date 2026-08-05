Network access ကို ဒီ container ထဲမှာ ပိတ်ထားလို့ အသက်ဝင်တဲ့ (live) end-to-end test ကို ဒီနေရာမှာ တိုက်ရိုက် run ပြပေးလို့ မရနိုင်ပါဘူး။ ဒါပေမယ့် **run-ready project နှစ်ခု** (Spring Boot server + Swing client) ကို ပြည့်စုံအောင် ဖန်တီးပေးပြီး၊ သင့် NetBeans/IntelliJ မှာ တိုက်ရိုက် run နိုင်အောင် download link ပေးပါမယ်။Server ဘက် ပြီးပါပြီ။ အခု Swing client project ကို ဆက်ဖန်တီးပါမယ်။**ဒီ project ကို run လုပ်နည်း (README.md ထဲမှာလည်း အသေးစိတ် ပါပါတယ်):**

1. Zip ကို extract လုပ်ပါ
2. Terminal ၁ — `cd server && mvn spring-boot:run` (server start)
3. Terminal ၂ — `cd swing-client && mvn clean package && java -jar target/swing-ws-client-1.0.0.jar`
4. Swing window ပွင့်ရင် **Connect** နှိပ်ပြီး live AAPL price update (3 sec တစ်ခါ) ကို ကြည့်နိုင်ပါတယ်

**Code structure:**

- `server/` — Spring Boot, `WebSocketConfig` + `StockPriceHandler` (session management, broadcast) + `StockPriceBroadcaster` (`@Scheduled` fake price generator)
- `swing-client/` — `StockPriceClient` (Java-WebSocket callbacks, `invokeLater` နဲ့ EDT safety) + `StockTickerFrame` (UI, connect/disconnect button)

Network ကို ဒီ sandbox မှာ ပိတ်ထားလို့ ကိုယ်တိုင် run ပြီး ကိုယ်တိုင်စမ်းရမှာဖြစ်ပါတယ် — run လုပ်ရင်း error/warning ရှိရင် ဒီမှာ ဆက်ပြောပါ၊ debug လုပ်ပေးနိုင်ပါတယ်။