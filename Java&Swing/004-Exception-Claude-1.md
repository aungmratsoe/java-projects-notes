ကောင်းပါတယ်! Java Exception Handling ကို အပိုင်းလိုက် သင်ပေးပါမယ်။ ဒါဟာ topic အနည်းငယ် ကျယ်ဝန်းလို့ **Part အလိုက်** ခွဲပြီး သင်ပေးမှာဖြစ်ပါတယ်။

## Part 1: Exception ဆိုတာဘာလဲ (Introduction)

### Exception ဆိုတာဘာလဲ

**Exception** ဆိုတာ program run နေတဲ့အချိန်မှာ ဖြစ်ပေါ်လာတဲ့ **unexpected event (မမျှော်လင့်ထားတဲ့ ဖြစ်ရပ်)** တစ်ခုပါ။ ဒီဖြစ်ရပ်က program ရဲ့ normal flow ကို ဖျက်ဆီးလိုက်ပါတယ်။

ဥပမာအားဖြင့်:

- ဂဏန်းကို ၀ နဲ့စား (Divide by zero)
- Array ရဲ့ index ကို မှားခေါ်တာ (ArrayIndexOutOfBounds)
- File တစ်ခုကို ရှာမတွေ့တာ (FileNotFoundException)
- null value ကို သုံးလို့ရအောင် ကြိုးစားတာ (NullPointerException)

### ဘာကြောင့် Exception Handling လိုအပ်လဲ

Exception ကို မကိုင်တွယ်ဘူးဆိုရင် program က ရုတ်တရက် **crash** ဖြစ်သွားပါလိမ့်မယ်။ Exception handling လုပ်ထားရင်:

- Program crash မဖြစ်ဘဲ ဆက်လက် run နိုင်ပါတယ်
- User ကို error message ကောင်းကောင်း ပြပေးနိုင်ပါတယ်
- Resource တွေ (file, database connection) ကို properly ပိတ်နိုင်ပါတယ်

### Exception Hierarchy (အခြေခံ Structure)

```
                Throwable
               /          \
          Exception        Error
         /          \
  Checked Exception   Unchecked Exception
   (Compile-time)      (Runtime - RuntimeException)
```

- **Throwable** - ဒါက root class ဖြစ်ပြီး Exception နဲ့ Error နှစ်ခုလုံးရဲ့ parent ပါ
- **Error** - JVM level errors (OutOfMemoryError, StackOverflowError) - ပုံမှန် code ကနေ catch လုပ်ဖို့ မကြိုးစားသင့်ပါ
- **Exception** - programmer ကိုင်တွယ်နိုင်တဲ့ error တွေ

### Exception ရဲ့ အမျိုးအစားနှစ်မျိုး

1. **Checked Exception** - compile time မှာ compiler က handle လုပ်ဖို့ မဖြစ်မနေ တောင်းဆိုတဲ့ exception (ဥပမာ - IOException, SQLException)
2. **Unchecked Exception (RuntimeException)** - compile time မှာ check မလုပ်ဘဲ runtime မှာပဲ ဖြစ်တတ်တဲ့ exception (ဥပမာ - NullPointerException, ArithmeticException)

---

ဒါက **Part 1 (Introduction)** ပါ။ နားလည်ပြီလား? နားလည်ရင် **Part 2** မှာ try-catch-finally block ရေးနည်းကို ဆက်သင်ပေးပါမယ်။