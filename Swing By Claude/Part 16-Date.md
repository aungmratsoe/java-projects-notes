# Java ရဲ့ Date/Time Classes တွေအကြောင်း (မြန်မာလို ရှင်းပြပါမယ်)

Java ထဲမှာ Date/Time classes တွေက အုပ်စု ၂ခု ရှိပါတယ် — **Legacy (အဟောင်း)** နဲ့ **Modern (java.time — အသစ်)**။ အကုန်လုံးကို တစ်ခါတည်း သင်ယူဖို့ ကြိုးစားရင် ရှုပ်ထွေးနေမှာပါ၊ ဒီတော့ အုပ်စုချပြီး ခွဲရှင်းပေးမယ်။

## 1. Legacy Classes (java.util package)

**`java.util.Date`**

- Date + Time နှစ်ခုစလုံးကို ကိုင်တွယ်ပေမယ့် design ကောင်းမွန်မှု မရှိဘူး (mutable, timezone handling ဆိုးတယ်)
- `new Date()` လို့ ခေါ်ရင် "အခုအချိန်" ကို ရမယ်

**`java.util.Calendar`**

- `Date` ကို field-by-field (နှစ်၊ လ၊ ရက်၊ နာရီ) ပြင်ဆင်ဖို့ သုံးတဲ့ class
- `Calendar.getInstance()` နဲ့ instance ယူပြီး `cal.set(Calendar.YEAR, 2026)` စတာတွေ လုပ်လို့ရ

👉 ဒီနှစ်ခုစလုံးက **deprecated အဖြစ်** သဘောထားရပါတယ် — အသစ်တွေအတွက် မသုံးသင့်တော့ဘူး၊ ဒါပေမယ့် old code/library တွေမှာ တွေ့ရနေဆဲပါ။

## 2. Modern Classes (java.time package — Java 8+)

**`java.time.LocalDate`**

- **ရက်စွဲ** (နှစ်၊ လ၊ ရက်) ကိုပဲ ကိုင်တွယ်တယ်၊ Time မပါဘူး
- ဥပမာ: `LocalDate.now()` → 2026-08-01

**`java.time.LocalDateTime`**

- ရက်စွဲ + အချိန် နှစ်ခုစလုံး ပါတယ်၊ ဒါပေမယ့် **Time zone မပါဘူး**
- ဥပမာ: `LocalDateTime.now()` → 2026-08-01T14:30:00

👉 ဒါတွေက **immutable** (တန်ဖိုးပြောင်းရင် object အသစ် return လုပ်တယ်) ဖြစ်လို့ Thread-safe ပါတယ်။ **အခုခေတ် Java projects အားလုံးနီးပါး ဒါတွေကိုပဲ သုံးသင့်ပါတယ်။**

## 3. Database အတွက် (java.sql package)

**`java.sql.Date`**

- Database ရဲ့ `DATE` column အတွက်၊ Date ပဲပါတယ် Time မပါဘူး
- `java.util.Date` ကို **inherit** လုပ်ထားတယ်

**`java.sql.Timestamp`**

- Database ရဲ့ `TIMESTAMP`/`DATETIME` column အတွက်၊ Date + Time + Nanoseconds ပါတယ်
- ဒါလည်း `java.util.Date` ကို inherit လုပ်တယ်

👉 ဒါတွေကို **JDBC (Database connection) code ထဲမှာသာ** သုံးပါ၊ business logic ထဲမှာတော့ `LocalDate`/`LocalDateTime` သုံးတာ ပိုကောင်းတယ်။

## 4. Formatting Classes (Text ⇄ Date ပြောင်းဖို့)

**`SimpleDateFormat`** → `java.util.Date` အတွက် (legacy)

```java
SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");
String text = sdf.format(new Date());       // Date → String
Date d = sdf.parse("01/08/2026");            // String → Date
```

⚠️ Thread-safe **မဟုတ်ဘူး** — Multi-thread environment မှာ သတိထားပါ။

**`DateTimeFormatter`** → `LocalDate`/`LocalDateTime` အတွက် (modern)

```java
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy");
String text = date.format(fmt);                      // LocalDate → String
LocalDate d = LocalDate.parse("01/08/2026", fmt);     // String → LocalDate
```

✅ Thread-safe ဖြစ်တယ်၊ **အသစ်လုပ်ရင် ဒါကိုပဲ သုံးပါ။**

## 5. Class တစ်ခုနဲ့ တစ်ခု ပြောင်းနည်း (Conversion — Casting မဟုတ်ဘူး)

⚠️ အရေးကြီးတာက — ဒီ classes တွေကြားမှာ **`(LocalDate) someDate`** လိုမျိုး တိုက်ရိုက် **cast လုပ်လို့ မရဘူး**။ Method သီးသန့်တွေနဲ့ **convert** လုပ်ရမှာပါ။

**`java.util.Date` → `LocalDate`/`LocalDateTime`**

```java
Date utilDate = new Date();
LocalDate localDate = utilDate.toInstant()
        .atZone(ZoneId.systemDefault())
        .toLocalDate();

LocalDateTime localDateTime = utilDate.toInstant()
        .atZone(ZoneId.systemDefault())
        .toLocalDateTime();
```

**`LocalDate`/`LocalDateTime` → `java.util.Date`**

```java
Date utilDate = Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
// LocalDate ဆိုရင် atStartOfDay() ခေါ်ပြီးမှ atZone လုပ်ရမယ်
Date fromLocalDate = Date.from(localDate.atStartOfDay(ZoneId.systemDefault()).toInstant());
```

**`LocalDate` → `java.sql.Date`** (DB ထဲသိမ်းမယ်ဆိုရင်)

```java
java.sql.Date sqlDate = java.sql.Date.valueOf(localDate);   // LocalDate → sql.Date
LocalDate backToLocal = sqlDate.toLocalDate();                // sql.Date → LocalDate
```

**`LocalDateTime` → `java.sql.Timestamp`**

```java
Timestamp ts = Timestamp.valueOf(localDateTime);   // LocalDateTime → Timestamp
LocalDateTime backToLDT = ts.toLocalDateTime();      // Timestamp → LocalDateTime
```

**`java.util.Date` → `java.sql.Date`** (inheritance ရှိလို့ တိုက်ရိုက် constructor သုံးလို့ရ)

```java
java.sql.Date sqlDate = new java.sql.Date(utilDate.getTime());
```

## 6. JDateChooser / JCalendar (Swing UI Component)

သင် Swing သင်နေတာမို့ ဒါကို အထူးရှင်းပြပါမယ် —

- `JDateChooser` (JCalendar library ကနေလာတယ်) က Swing form ထဲမှာ **ရက်စွဲ ရွေးချယ်ဖို့ dropdown calendar widget** ပါ
- ဒါက **`java.util.Date`** ကိုပဲ return ပေးတယ် (`getDate()` method သုံးရမယ်) — modern `LocalDate` တိုက်ရိုက် မပေးဘူး

```java
JDateChooser dateChooser = new JDateChooser();
Date selectedDate = dateChooser.getDate();   // java.util.Date ရမယ်

// LocalDate လိုချင်ရင် convert လုပ်ရမယ်
LocalDate localDate = selectedDate.toInstant()
        .atZone(ZoneId.systemDefault())
        .toLocalDate();
```

## အနှစ်ချုပ် (Cheat Sheet)

| အသုံးပြုရမည့်နေရာ                     | Class                                  |
| ------------------------------------- | -------------------------------------- |
| Business logic, calculation           | `LocalDate` / `LocalDateTime`          |
| String ⇄ Date format ပြောင်း (modern) | `DateTimeFormatter`                    |
| Database column (DATE)                | `java.sql.Date`                        |
| Database column (TIMESTAMP)           | `java.sql.Timestamp`                   |
| JDateChooser ကနေရလာတဲ့ result         | `java.util.Date` (ပြီးရင် convert)     |
| Legacy code ထိန်းသိမ်းရေး             | `Date`, `Calendar`, `SimpleDateFormat` |

**အကြံပြုချက်**: Swing project အသစ်လုပ်ရင် `JDateChooser` ကနေ `getDate()` နဲ့ `Date` ရလာရင်တောင် **အမြန်ဆုံး `LocalDate` ကို convert လုပ်ပြီး** app logic တစ်ခုလုံးမှာ `LocalDate`/`LocalDateTime` ကိုပဲ တသမတ်တည်း သုံးပါ။ Database ထဲထည့်ချိန်မှသာ `java.sql.Date`/`Timestamp` ကို convert ပြန်လုပ်ပါ။

ဒီအပိုင်းက ရှင်းရဲ့လား၊ ဘယ် conversion ကို code ဥပမာ ပိုပြီး လက်တွေ့ကျကျ ရှင်းပြပေးရမလဲ (ဥပမာ- Swing form ထဲမှာ JDateChooser နဲ့ Database ချိတ်ဆက်တာမျိုး)?