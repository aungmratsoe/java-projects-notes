ကောင်းပြီ၊ Apache POI ကို Excel export လုပ်ဖို့ step-by-step သင်ပေးပါမယ်။ Cafe POS App ရဲ့ **daily sales report** ကို Excel ထုတ်တဲ့ဥပမာနဲ့ သင်မယ်။

## 📘 Lesson 1: Apache POI ဆိုတာ ဘာလဲ?

Apache POI က Java ကနေ Microsoft Office file တွေ (Excel, Word) ကို ဖန်တီး/ဖတ်/ပြင်ဖို့ library ဖြစ်တယ်။ Excel အတွက်ဆိုရင် class hierarchy ကို အခြေခံနားလည်ထားရမယ်:

|POI Class|ဆိုလိုသည်မှာ|
|---|---|
|`Workbook`|Excel file တစ်ခုလုံး (.xlsx)|
|`Sheet`|Workbook ထဲက sheet တစ်ခု (tab)|
|`Row`|Sheet ထဲက row (horizontal line)|
|`Cell`|Row ထဲက cell (data ထည့်ရာနေရာ)|

**Hierarchy:** Workbook → Sheet → Row → Cell (အပေါ်ကနေအောက်ကို drill down လုပ်ရတယ်)

---

## Step 1: pom.xml မှာ dependency ထည့်ပါ

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.3.0</version>
</dependency>
```

NetBeans မှာ pom.xml ကို edit ပြီးရင် project ကို right-click → "Clean and Build" လုပ်ပြီး dependency ကို download ဖြစ်အောင်လုပ်ပါ။

---

### 🎯 Practice လုပ်ကြည့်ရအောင်

pom.xml ထဲ dependency ထည့်ပြီးပြီလား? "Clean and Build" ပြီးရင် dependency error တစ်ခုမှမရှိလား စစ်ကြည့်ပါ။ ပြီးရင် "ready" လို့ပြောပါ — Step 2 (Workbook ဖန်တီးပြီး Sheet ထဲ data ထည့်နည်း) ကို ဆက်သင်ပေးပါမယ်။