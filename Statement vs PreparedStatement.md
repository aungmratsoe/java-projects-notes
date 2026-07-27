JDBC (Java Database Connectivity) တွင် **Statement** နှင့် **PreparedStatement** တို့သည် SQL queries များ run ရန် အသုံးပြုသည့် Object များ ဖြစ်ကြသည်။ ၎င်းတို့၏ အဓိက ကွာခြားချက်မှာ **Performance (လုပ်ဆောင်ချက် မြန်ဆန်မှု)** နှင့် **Security (လုံခြုံရေး)** တို့ ဖြစ်သည်။

အောက်တွင် ၎င်းတို့၏ အဓိက ကွာခြားချက်များကို ရှင်းလင်းစွာ နှိုင်းယှဉ်ဖော်ပြပေးထားပါသည် -

၁။ အဓိက ကွာခြားချက်များ နှိုင်းယှဉ်ချက် (Comparison Table)

|အချက်အလက် (Feature)|Statement|PreparedStatement|
|---|---|---|
|**Execution (အလုပ်လုပ်ပုံ)**|SQL Query ကို Run တိုင်း အသစ်ပြန်လည် Compile လုပ်သည်။|SQL Query ကို ကြိုတင် Compile လုပ်ထားပြီး Parametric values များကိုသာ လဲလှယ်သည်။|
|**Performance (အရှိန်အဟုန်)**|Query တစ်ခုတည်းကို ခဏခဏ Run ပါက ပို၍ နှေးသည်။|Query တစ်ခုတည်းကို values ပြောင်းပြီး ခဏခဏ Run ပါက ပို၍ မြန်သည်။|
|**Security (လုံခြုံရေး)**|**SQL Injection** Vulnerability (အားနည်းချက်) ရှိသည်။|**SQL Injection** ကို အပြည့်အဝ ကာကွယ်ပေးနိုင်သည်။|
|**Syntax (ကုဒ်ပုံစံ)**|String Concatenation (`+` လက္ခဏာ) ဖြင့် values များကို ပေါင်းစပ်ရသည်။|Placeholder (`?` လက္ခဏာ) ကို အသုံးပြုသည်။|
|**Caching (မှတ်ဉာဏ်သိမ်းဆည်းမှု)**|Database မှ Query ကို Cache မလုပ်ပါ။|Database မှ ကြိုတင် Compile လုပ်ထားသော Query ကို Cache လုပ်ထားနိုင်သည်။|

---

၂။ အသေးစိတ် ရှင်းလင်းချက် (Detailed Explanation)

Statement ဆိုတာဘာလဲ။ 

Statement ကို SQL query အသေများ (Static SQL queries) ဖြစ်သည့် `SELECT * FROM users` ကဲ့သို့သော variable မပါသည့် နေရာများတွင် သုံးရန် သင့်တော်သည်။ query ကို မောင်းနှင်သည့်အခါတိုင်း Database က SQL ကို အသစ်ထပ်မံ စစ်ဆေး (Parse) ပြီး Compile လုပ်ရသောကြောင့် တစ်ကြိမ်ထက်မက သုံးလျှင် အချိန်ပိုကြာတတ်သည်။ [[1]

- **ဥပမာ ကုဒ်စတိုင် -**

```java
String query = "SELECT * FROM users WHERE username = '" + name + "' AND password = '" + pass + "'";
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);
```

Use code with caution.

_(⚠️ သတိပြုရန် - အထက်ပါပုံစံသည် User ဘက်မှ `' OR '1'='1` ကဲ့သို့သော SQL Injection Hacker Code များ ရိုက်ထည့်ပါက Database ကျိုးပေါက်နိုင်ပါသည်။)_ 

PreparedStatement ဆိုတာဘာလဲ။ 

PreparedStatement ကို dynamic ဖြစ်သော (တစ်ခုနှင့်တစ်ခု တန်ဖိုးမတူဘဲ ပြောင်းလဲနေသည့် variable ပါသော) SQL queries များတွင် သုံးသည်။ ၎င်းသည် query template ကို ကြိုတင် Compile လုပ်ထားပြီး variable နေရာတွင် `?` (Placeholders) ကို အစားထိုးကာ သတ်မှတ်ပေးသည်။ 

- **ဥပမာ ကုဒ်စတိုင် -**

```java
String query = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement pstmt = connection.prepareStatement(query);
pstmt.setString(1, name);
pstmt.setString(2, pass);
ResultSet rs = pstmt.executeQuery();
```

Use code with caution.

_(✅ အကျိုးကျေးဇူး - `?` နေရာတွင် ဝင်လာသမျှ data တန်ဖိုးများကို text သက်သက်အဖြစ်သာ သတ်မှတ်ပြီး input ကို database က သန့်စင် (escape) လိုက်သောကြောင့် SQL Injection လုံးဝ မဖြစ်နိုင်တော့ပါ။)_ 

---

၃။ မည်သည့်အချိန်တွင် ဘာကို သုံးသင့်သလဲ။

- **PreparedStatement ကို အမြဲသုံးပါ -** Application တည်ဆောက်ရာတွင် Security နှင့် Speed သည် အရေးကြီးဆုံးဖြစ်သောကြောင့် လက်တွေ့ လုပ်ငန်းခွင် (Real-world projects) များတွင် **PreparedStatement ကိုသာ standard အဖြစ် သတ်မှတ်၍ အမြဲသုံးစွဲသင့်ပါသည်။** 
- **Statement ကို ခန့်မှန်းရလွယ်သော နေရာတွင်သာ သုံးပါ -** parameter (variable) လုံးဝ မပါဝင်ဘဲ application run ချိန်တွင် တစ်ကြိမ်သာ သုံးမည့် static query မျိုး (ဥပမာ - application စဖွင့်ချင်း table အသစ်ဆောက်သည့် `CREATE TABLE ...` မျိုး) တွင်သာ Statement ကို သုံးသင့်သည်။ [[1]

---