## Part 3: throw vs throws, Custom Exception

### throw Keyword

`throw` ကို exception တစ်ခုကို **manually (ကိုယ်တိုင်) ဖြစ်ပေါ်စေဖို့** သုံးပါတယ်။ Exception object တစ်ခုကို create လုပ်ပြီး "throw" ချလိုက်တာပါ။

```java
public class Example {
    public static void checkAge(int age) {
        if (age < 18) {
            throw new IllegalArgumentException("အသက် ၁၈ အောက်ဖြစ်လို့ မရပါ");
        }
        System.out.println("Age valid ပါတယ်");
    }
    
    public static void main(String[] args) {
        checkAge(15);  // ArithmeticException throw ဖြစ်မယ်
    }
}
```

Output:

```
Exception in thread "main" java.lang.IllegalArgumentException: အသက် ၁၈ အောက်ဖြစ်လို့ မရပါ
```

**Note:** `throw` statement တစ်ခုချင်းစီနောက်မှာ code တစ်ခုပဲ ရေးလို့ရပါတယ် (တစ်ကြိမ်ချင်း exception တစ်ခုပဲ throw လုပ်လို့ရလို့ပါ)။

### throws Keyword

`throws` ကတော့ method declaration မှာ သုံးပါတယ်။ ဒီ method ထဲမှာ **exception ဖြစ်နိုင်ချေရှိတယ်**ဆိုတာကို caller ကို သတိပေးတာပါ (checked exceptions အတွက် အဓိက သုံးပါတယ်)။

```java
import java.io.*;

public class Example {
    // ဒီ method က IOException throw လုပ်နိုင်ကြောင်း declare လုပ်ထားတယ်
    public static void readFile() throws IOException {
        FileReader file = new FileReader("test.txt");
    }
    
    public static void main(String[] args) {
        try {
            readFile();
        } catch (IOException e) {
            System.out.println("File ဖတ်လို့ မရပါ: " + e.getMessage());
        }
    }
}
```

### throw vs throws ခြားနားချက်

||throw|throws|
|---|---|---|
|**သုံးနေရာ**|Method body ထဲမှာ|Method signature မှာ|
|**ရည်ရွယ်ချက်**|Exception ကို actually ဖြစ်ပေါ်စေဖို့|Exception ဖြစ်နိုင်ကြောင်း declare ဖို့|
|**အရေအတွက်**|တစ်ကြိမ်ကို တစ်ခုပဲ|Multiple exceptions ကို comma ခံပြီး ရေးလို့ရ|
|**ဥပမာ**|`throw new Exception("msg")`|`void method() throws IOException, SQLException`|

```java
// Multiple exceptions ကို throws နဲ့ declare လုပ်ခြင်း
public void process() throws IOException, SQLException {
    // code
}
```

### Custom Exception ဖန်တီးနည်း

တစ်ခါတစ်ရံ built-in exception တွေ (ArithmeticException, IOException) က သင့် application logic အတွက် မလုံလောက်ပါဘူး။ ဒါကြောင့် **Custom Exception** ကို ကိုယ်တိုင် create လုပ်နိုင်ပါတယ်။

**Custom Exception ဖန်တီးဖို့ steps:**

1. `Exception` (checked) သို့မဟုတ် `RuntimeException` (unchecked) class ကို **extend** လုပ်ပါ
2. Constructor ရေးပြီး `super(message)` ကို ခေါ်ပါ

```java
// Custom Checked Exception
class InsufficientBalanceException extends Exception {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}

// Bank account class ထဲမှာ သုံးခြင်း
class BankAccount {
    private double balance;
    
    public BankAccount(double balance) {
        this.balance = balance;
    }
    
    public void withdraw(double amount) throws InsufficientBalanceException {
        if (amount > balance) {
            throw new InsufficientBalanceException(
                "ငွေလက်ကျန်မလုံလောက်ပါ! လက်ကျန်: " + balance + ", တောင်းခံငွေ: " + amount
            );
        }
        balance -= amount;
        System.out.println("ငွေထုတ်ယူမှု အောင်မြင်ပါတယ်။ လက်ကျန်: " + balance);
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount(1000);
        
        try {
            account.withdraw(1500);
        } catch (InsufficientBalanceException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

Output:

```
Error: ငွေလက်ကျန်မလုံလောက်ပါ! လက်ကျန်: 1000.0, တောင်းခံငွေ: 1500.0
```

### Custom Unchecked Exception

```java
class InvalidAgeException extends RuntimeException {
    public InvalidAgeException(String message) {
        super(message);
    }
}

public class Main {
    public static void setAge(int age) {
        if (age < 0) {
            throw new InvalidAgeException("Age negative ဖြစ်လို့ မရပါ: " + age);
        }
    }
    
    public static void main(String[] args) {
        setAge(-5);  // Unchecked ဖြစ်လို့ try-catch မလိုအပ်ဘူး (ဒါပေမယ့် handle လုပ်သင့်တယ်)
    }
}
```

**Checked vs Unchecked ကို ဘယ်အခါသုံးမလဲ:**

- **Checked Exception** (Exception extend) - caller က **မဖြစ်မနေ handle လုပ်ရမယ့်** critical situations (ဥပမာ - file not found, network error)
- **Unchecked Exception** (RuntimeException extend) - programming logic error (ဥပမာ - invalid input, business rule violation)

---

ဒါက **Part 3 (throw, throws, Custom Exception)** ပါ။ နားလည်ပြီလား? **Part 4** မှာ Best Practices တွေနဲ့ Common Mistakes တွေကို ဆက်သင်ပေးပါမယ် (ဒါက နောက်ဆုံး part ဖြစ်နိုင်ပါတယ်)။