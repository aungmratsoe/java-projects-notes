## Part 2: try-catch-finally Block

### try-catch Basic Syntax

Exception ဖြစ်နိုင်တဲ့ code ကို **try** block ထဲမှာထည့်ပြီး၊ exception ဖြစ်ရင် ဘယ်လိုကိုင်တွယ်မလဲဆိုတာကို **catch** block ထဲမှာရေးပါတယ်။

```java
try {
    // Exception ဖြစ်နိုင်တဲ့ code
    int result = 10 / 0;  // ArithmeticException ဖြစ်မယ်
} catch (ArithmeticException e) {
    // Exception ကို ဘယ်လိုကိုင်တွယ်မလဲ
    System.out.println("Error: " + e.getMessage());
}
```

**ဘယ်လိုအလုပ်လုပ်လဲ:**

1. `try` block ထဲက code ကို run ပါတယ်
2. Exception ဖြစ်ရင် `try` block က ကျန်တဲ့ code တွေကို skip လုပ်ပြီး `catch` block ကို ချက်ချင်း ကျော်သွားပါတယ်
3. `catch` block ထဲက exception type နဲ့ ကိုက်ညီရင် ဒီ block ကို run ပါတယ်
4. Exception မဖြစ်ရင် `catch` block ကို လုံးဝ run မှာမဟုတ်ပါဘူး

### Real Example

```java
public class Example {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3};
        
        try {
            System.out.println(numbers[5]);  // Index မရှိဘူး
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Array index မှားနေပါတယ်: " + e.getMessage());
        }
        
        System.out.println("Program ဆက်လက် run နေပါတယ်");
    }
}
```

Output:

```
Array index မှားနေပါတယ်: Index 5 out of bounds for length 3
Program ဆက်လက် run နေပါတယ်
```

catch block ထဲက error ကို ကိုင်တွယ်ပြီးရင် program က crash မဖြစ်ဘဲ **ဆက်လက် run** နေတာ သတိထားပါ။

### Multiple catch blocks

Exception အမျိုးမျိုးကို ခွဲပြီး handle လုပ်နိုင်ပါတယ်:

```java
try {
    int[] arr = new int[5];
    arr[10] = 50 / 0;  
} catch (ArithmeticException e) {
    System.out.println("Arithmetic error: " + e.getMessage());
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Array error: " + e.getMessage());
} catch (Exception e) {
    System.out.println("General error: " + e.getMessage());
}
```

**Rule:** catch block တွေက အထက်ကနေ အောက်ကို အစဉ်လိုက် check လုပ်ပါတယ်။ ဒါကြောင့် **specific exception** တွေကို **အပေါ်ဆုံးမှာ**၊ **general exception (Exception class)** ကို **အနောက်ဆုံးမှာ** ထားရပါမယ်။ မဟုတ်ရင် compile error တက်ပါလိမ့်မယ်။

```java
// ❌ Wrong order - compile error!
catch (Exception e) { ... }
catch (ArithmeticException e) { ... }  // unreachable code error
```

### finally Block

`finally` block ကတော့ exception ဖြစ်ဖြစ်၊ မဖြစ်ဖြစ် **အမြဲတမ်း run ပါတယ်**။ ပုံမှန်အားဖြင့် resource တွေ (file, database connection) ကို ပိတ်ဖို့ သုံးပါတယ်။

```java
try {
    int result = 10 / 0;
    System.out.println(result);
} catch (ArithmeticException e) {
    System.out.println("Error ဖြစ်နေပါတယ်: " + e.getMessage());
} finally {
    System.out.println("ဒီ block ကတော့ ဘာဖြစ်ဖြစ် run ပါတယ်");
}
```

Output:

```
Error ဖြစ်နေပါတယ်: / by zero
ဒီ block ကတော့ ဘာဖြစ်ဖြစ် run ပါတယ်
```

**finally ဘယ်အခါ run လုပ်လဲ:**

- Exception ဖြစ်ရင်လည်း run ပါတယ်
- Exception မဖြစ်ရင်လည်း run ပါတယ်
- `return` statement ရှိပေမယ့် run ပါတယ် (special case)

### try-with-resources (Modern approach)

File, database connection စတာတွေကို auto-close လုပ်ချင်ရင် Java 7 ကနေစပြီး ဒီ syntax ကို သုံးလို့ရပါတယ်:

```java
try (FileReader file = new FileReader("test.txt")) {
    // file ကို အလုပ်လုပ်ပါ
} catch (IOException e) {
    System.out.println("File error: " + e.getMessage());
}
// file က auto-close ဖြစ်သွားပါမယ် - finally မလိုအပ်တော့ပါ
```

---

ဒါက **Part 2 (try-catch-finally)** ပါ။ ဒီ part မှာ questions ရှိရင် မေးလို့ရပါတယ်။ နားလည်ပြီဆိုရင် **Part 3** မှာ `throw` နဲ့ `throws` keyword တွေရဲ့ ကွာခြားချက်၊ Custom Exception ရေးနည်းကို ဆက်သင်ပေးပါမယ်။