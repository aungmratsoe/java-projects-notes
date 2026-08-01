## Part 4: Best Practices နဲ့ Common Mistakes

### Best Practice 1: Specific Exception ကို သုံးပါ

Generic `Exception` class ထက် specific exception type ကို catch လုပ်တာက ပိုကောင်းပါတယ်။ ဘာကြောင့်လဲဆိုတော့ debug လုပ်ရလွယ်ပြီး error ရဲ့ actual cause ကို ပိုနားလည်နိုင်လို့ပါ။

```java
// ❌ Bad practice
try {
    int result = 10 / 0;
} catch (Exception e) {  // too generic
    System.out.println("Error ဖြစ်နေပါတယ်");
}

// ✅ Good practice
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {  // specific
    System.out.println("Division by zero error: " + e.getMessage());
}
```

### Best Practice 2: Empty catch block မလုပ်ပါနဲ့

Exception ကို catch လုပ်ပြီး ဘာမှ မလုပ်ဘဲ ချန်ထားတာဟာ **worst practice** တစ်ခုပါ။ Error ဖြစ်နေတာကို ဖျောက်ထားလိုက်တာနဲ့ တူပါတယ်၊ debug လုပ်ရခက်သွားစေပါတယ်။

```java
// ❌ Bad practice - "Exception Swallowing"
try {
    riskyOperation();
} catch (Exception e) {
    // ဘာမှမလုပ်ဘူး - error ကို ဖျောက်ထားလိုက်တာ
}

// ✅ Good practice - အနည်းဆုံး log ချ၊ ဒါမှမဟုတ် သင့်လျော်တဲ့ action လုပ်ပါ
try {
    riskyOperation();
} catch (Exception e) {
    System.err.println("Error occurred: " + e.getMessage());
    e.printStackTrace();  // debugging အတွက် အသုံးဝင်ပါတယ်
}
```

### Best Practice 3: finally မှာ resource တွေကို ပိတ်ပါ (ဒါမှမဟုတ် try-with-resources သုံးပါ)

```java
// ✅ Good - try-with-resources (Java 7+) - Recommended
try (FileReader reader = new FileReader("file.txt")) {
    // file အလုပ်လုပ်ခြင်း
} catch (IOException e) {
    System.out.println("Error: " + e.getMessage());
}
// resource က auto-close ဖြစ်သွားမယ်

// Old way - manual close (necessary situation တွေမှာသာ)
FileReader reader = null;
try {
    reader = new FileReader("file.txt");
} catch (IOException e) {
    System.out.println("Error: " + e.getMessage());
} finally {
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Best Practice 4: Exception message တွေကို clear ဖြစ်အောင်ရေးပါ

```java
// ❌ Bad - message မရှင်းဘူး
throw new IllegalArgumentException("Invalid input");

// ✅ Good - specific message ပေးထားတယ်
throw new IllegalArgumentException(
    "Age must be between 0 and 150, but got: " + age
);
```

### Best Practice 5: Exception ကို ignore မလုပ်ဘဲ properly handle လုပ်ပါ

```java
// ❌ Bad - exception ကို catch လုပ်ပြီး ignore လုပ်တာ
public void processData() {
    try {
        // some code
    } catch (Exception e) {
        // ignore လုပ်တာ dangerous ပါတယ်
    }
}

// ✅ Good - either handle properly or re-throw
public void processData() throws DataProcessingException {
    try {
        // some code
    } catch (SQLException e) {
        throw new DataProcessingException("Data processing failed", e);
    }
}
```

### Common Mistakes

**Mistake 1: catch block order မှားတာ**

```java
// ❌ Compile error - general exception ကို အရင်ထားလို့
try {
    // code
} catch (Exception e) { }
catch (ArithmeticException e) { }  // unreachable!
```

**Mistake 2: finally block မှာ return သုံးတာ**

```java
public static int test() {
    try {
        return 1;
    } finally {
        return 2;  // ❌ ဒါက try ရဲ့ return value ကို override လုပ်လိုက်မယ်
    }
}
// result: 2 (မလိုချင်တဲ့ behavior ဖြစ်နိုင်ပါတယ်)
```

**Mistake 3: Exception object ကို cause မထည့်ဘဲ wrap လုပ်တာ**

```java
// ❌ Bad - original error information ပျောက်သွားမယ်
try {
    // code
} catch (SQLException e) {
    throw new RuntimeException("Database error");  // original 'e' ပါမလာဘူး
}

// ✅ Good - original exception ကို cause အနေနဲ့ ထည့်ပါ
try {
    // code
} catch (SQLException e) {
    throw new RuntimeException("Database error", e);  // 'e' ကို cause အနေနဲ့ ထည့်တယ်
}
```

### Quick Summary (Full Topic)

|Concept|ရှင်းလင်းချက်|
|---|---|
|**try**|Exception ဖြစ်နိုင်တဲ့ code ကို ထည့်ရာ|
|**catch**|Exception ကို ကိုင်တွယ်ရာ|
|**finally**|ဖြစ်ဖြစ်၊ မဖြစ်ဖြစ် အမြဲ run|
|**throw**|Exception ကို manually ဖြစ်ပေါ်စေခြင်း|
|**throws**|Method ကနေ exception ဖြစ်နိုင်ကြောင်း declare|
|**Custom Exception**|Application-specific error အတွက် class အသစ်ဖန်တီးခြင်း|

---

ဒါက Java Exception Handling topic **အပိုင်းလေးပိုင်းလုံး** ပြီးပါပြီ။ 🎉

Practice လုပ်ချင်ရင် exercise တစ်ခု ပေးပေးရမလား၊ ဒါမှမဟုတ် ဘယ် part မှာမဆို confusion ရှိရင် ပြန်မေးနိုင်ပါတယ်။