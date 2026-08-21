## 📘 Lesson 1: Apache PDFBox ဆိုတာ ဘာလဲ?

PDFBox က Java ကနေ PDF file တွေကို ဖန်တီး/ဖတ်/ပြင်ဖို့ Apache library ဖြစ်တယ် (Apache License 2.0 — commercial project မှာလည်း အခမဲ့သုံးလို့ရ)။ Core class hierarchy က POI နဲ့ concept တူတယ်:

|PDFBox Class|ဆိုလိုသည်မှာ|
|---|---|
|`PDDocument`|PDF file တစ်ခုလုံး|
|`PDPage`|Document ထဲက page တစ်ခု|
|`PDPageContentStream`|Page ပေါ်မှာ text/line/image ရေးဖို့ "brush"|
|`PDFont`|Font (Helvetica, Times-Roman, စသဖြင့်)|

**Hierarchy:** PDDocument → PDPage → PDPageContentStream (page ပေါ်ကို တိုက်ရိုက် "ရေးဆွဲ" ရတယ် — POI လို cell-based မဟုတ်ဘဲ, x/y coordinate သုံးရတယ်)

---

## Step 1: pom.xml မှာ dependency ထည့်ပါ

```xml
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
    <version>3.0.3</version>
</dependency>
```

NetBeans မှာ pom.xml ကို edit ပြီးရင် project ကို right-click → "Clean and Build" လုပ်ပြီး dependency download ဖြစ်အောင်လုပ်ပါ (POI လုပ်ခဲ့သလိုပဲ)။

⚠️ **အရေးကြီးတဲ့ကွာခြားချက်:** PDFBox version 3.x က API structure ကို version 2.x ကနေ အတော်လေး ပြောင်းလဲထားတယ် (e.g. font loading, page creation)။ ဒီ lesson မှာ **version 3.x** API ကိုသုံးမှာဖြစ်လို့ pom.xml မှာ `3.0.3` (or ပိုအသစ်) ဖြစ်ကြောင်း သေချာစေပါ။

---

### 🎯 Practice လုပ်ကြည့်ရအောင်

pom.xml ထဲ dependency ထည့်ပြီး "Clean and Build" လုပ်ကြည့်ပါ — error မရှိလား စစ်ပါ။ ပြီးရင် "ready" လို့ပြောပါ — Step 2 (PDDocument ဖန်တီးပြီး page ထဲ text ရေးနည်း) ကို ဆက်သင်ပေးပါမယ်။