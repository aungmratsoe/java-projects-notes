# Java Lambda Function အကြောင်း အသေးစိတ်ရှင်းပြချက်

## Lambda Function ဆိုတာ ဘာလဲ?

Lambda function ဆိုတာက Java 8 မှာ စတင်မိတ်ဆက်ခဲ့တဲ့ feature တစ်ခုဖြစ်ပြီး၊ **name မလိုအပ်တဲ့ function (anonymous function)** တစ်ခုကို short syntax နဲ့ ရေးနိုင်စေတဲ့ concept ဖြစ်ပါတယ်။ Functional interface တစ်ခုရဲ့ implementation ကို concise ပုံစံနဲ့ ရေးနိုင်အောင် ကူညီပေးပါတယ်။

Anonymous class တွေရေးရမယ့်အစား code line အနည်းငယ်ထဲမှာ logic ကို ထည့်ရေးလို့ရအောင် ရည်ရွယ်ထားတာဖြစ်ပါတယ်။

## Lambda Expression ရဲ့ Syntax

```
(parameters) -> { body }
```

**အခြေခံပုံစံ ၃ မျိုး:**

1. **Parameter မရှိတဲ့ lambda:**

```java
() -> System.out.println("Hello Lambda!");
```

2. **Parameter တစ်ခုတည်းရှိတဲ့ lambda:**

```java
(x) -> x * x
// ဒါမှမဟုတ် bracket ချန်ထားလို့လည်းရ
x -> x * x
```

3. **Parameter များစွာရှိတဲ့ lambda:**

```java
(x, y) -> x + y
```

## Functional Interface ဆိုတာ ဘာလဲ?

Lambda expression တွေက **Functional Interface** (abstract method တစ်ခုတည်းပဲပါတဲ့ interface) နဲ့သာ အလုပ်လုပ်ပါတယ်။ ဥပမာ - `Runnable`, `Comparator`, `Callable` စတာတွေဖြစ်ပါတယ်။

`@FunctionalInterface` annotation ကို သုံးပြီး ကိုယ်ပိုင် functional interface ကိုလည်း ဖန်တီးနိုင်ပါတယ်။

## အသုံးပြုပုံ ဥပမာများ

### ၁. Custom Functional Interface နှင့် အသုံးပြုခြင်း

```java
@FunctionalInterface
interface Calculator {
    int operate(int a, int b);
}

public class LambdaExample {
    public static void main(String[] args) {
        // Lambda သုံးပြီး Addition
        Calculator addition = (a, b) -> a + b;
        System.out.println("Sum: " + addition.operate(5, 3));

        // Lambda သုံးပြီး Multiplication
        Calculator multiplication = (a, b) -> a * b;
        System.out.println("Product: " + multiplication.operate(5, 3));
    }
}
```

### ၂. Runnable Interface နှင့်အတူ

```java
public class RunnableExample {
    public static void main(String[] args) {
        // Anonymous class နည်းလမ်း (Lambda မသုံးခင်)
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("Running with anonymous class");
            }
        };

        // Lambda နည်းလမ်း (ပိုတိုတိုလေး)
        Runnable r2 = () -> System.out.println("Running with lambda");

        r1.run();
        r2.run();
    }
}
```

### ၃. Collections/List များနှင့် forEach သုံးခြင်း

```java
import java.util.*;

public class ListExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Aung", "Su", "Kyaw", "Mya");

        // Lambda သုံးပြီး list ထဲက item တစ်ခုချင်းစီကို print
        names.forEach(name -> System.out.println(name));

        // Method reference နဲ့လည်း ရေးလို့ရ
        names.forEach(System.out::println);
    }
}
```

### ၄. Comparator နှင့်အတူ Sorting

```java
import java.util.*;

public class SortExample {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9));

        // Lambda သုံးပြီး sort လုပ်ခြင်း
        Collections.sort(numbers, (a, b) -> a - b);

        System.out.println(numbers);
    }
}
```

### ၅. Stream API နှင့်အတူ (Lambda အသုံးအများဆုံးနေရာ)

```java
import java.util.*;
import java.util.stream.*;

public class StreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Lambda သုံးပြီး even number တွေကိုပဲ filter လုပ်ပြီး square ထုတ်ခြင်း
        List<Integer> result = numbers.stream()
                .filter(n -> n % 2 == 0)
                .map(n -> n * n)
                .collect(Collectors.toList());

        System.out.println(result); // [4, 16, 36, 64, 100]
    }
}
```

## Lambda ရဲ့ အားသာချက်များ

- **Code ကို ပိုတိုတိုနဲ့ ရှင်းလင်းအောင် ရေးနိုင်** - boilerplate code လျှော့ချနိုင်တယ်
- **Readability ကောင်းတယ်** - logic ကို တစ်နေရာတည်းမှာ ချက်ချင်းမြင်နိုင်
- **Functional Programming style** - Stream API, Collections framework တွေနဲ့ အသင့်တော်ဆုံး
- **Parallel processing** အတွက်လည်း အသုံးဝင်တယ် (parallelStream)

## သတိထားရမည့်အချက်များ

1. Lambda expression ကို functional interface (abstract method **တစ်ခုတည်း**ရှိတဲ့ interface) မှာသာ အသုံးပြုနိုင်ပါတယ်။
2. Lambda body တစ်line ပဲဆိုရင် `{}` နဲ့ `return` keyword ချန်ထားလို့ရပါတယ်။
3. Method body များနေရင် `{}` သုံးပြီး `return` ကို ပြန်ရေးရပါမယ်:

```java
Calculator subtract = (a, b) -> {
    int result = a - b;
    return result;
};
```

Lambda function တွေက Java ကို modern language တစ်ခုအဖြစ် တိုးတက်စေတဲ့ feature တစ်ခုဖြစ်ပြီး၊ ဒီနေ့ခေတ် Java development မှာ Stream API, Collections, Multi-threading စတာတွေမှာ အလွန်ကျယ်ကျယ်ပြန့်ပြန့် သုံးနေကြပါတယ်။ ပိုမိုနက်နဲစွာ လေ့လာချင်ရင် **Method References** (`::`) အကြောင်းလည်း ဆက်လေ့လာနိုင်ပါတယ်။

---

# Java Method References (::) အကြောင်း အသေးစိတ်ရှင်းပြချက်

## Method Reference ဆိုတာ ဘာလဲ?

Method Reference ဆိုတာက **Lambda expression ကို ပိုတိုတောင်းအောင်၊ ပိုရှင်းလင်းအောင် ရေးနိုင်တဲ့ shorthand syntax** တစ်ခုဖြစ်ပါတယ်။ Lambda expression ရဲ့ body ထဲမှာ existing method တစ်ခုကိုပဲ ခေါ်တာဖြစ်ရင် အဲ့ဒီ method ကို `::` operator သုံးပြီး တိုက်ရိုက် reference လုပ်နိုင်ပါတယ်။

**အခြေခံ syntax:**

```
ClassName::methodName
```

## ဘာကြောင့် သုံးရသလဲ?

Lambda ရေးထားရင်:

```java
names.forEach(name -> System.out.println(name));
```

Method reference နဲ့ရေးရင် ပိုတိုတယ်:

```java
names.forEach(System.out::println);
```

Logic က "method ကို ခေါ်ရုံပဲ" ဆိုရင် method reference က ပို readable ဖြစ်ပါတယ်။

## Method Reference အမျိုးအစား ၄ မျိုး

### ၁. Static Method Reference

`ClassName::staticMethodName`

```java
import java.util.*;
import java.util.function.*;

public class StaticRefExample {
    public static void main(String[] args) {
        // Lambda နည်းလမ်း
        Function<String, Integer> parseLambda = s -> Integer.parseInt(s);

        // Method Reference နည်းလမ်း
        Function<String, Integer> parseRef = Integer::parseInt;

        System.out.println(parseRef.apply("123")); // 123
    }
}
```

### ၂. Instance Method Reference (Particular Object)

`objectRef::instanceMethodName`

ရှိပြီးသား object တစ်ခုရဲ့ method ကို ခေါ်တာဖြစ်ပါတယ်။

```java
import java.util.function.*;

public class InstanceRefExample {
    public static void main(String[] args) {
        String message = "Hello Myanmar";

        // Lambda နည်းလမ်း
        Supplier<String> upperLambda = () -> message.toUpperCase();

        // Method Reference နည်းလမ်း
        Supplier<String> upperRef = message::toUpperCase;

        System.out.println(upperRef.get()); // HELLO MYANMAR
    }
}
```

### ၃. Instance Method Reference (Particular Type/Class ရဲ့ Arbitrary Object)

`ClassName::instanceMethodName`

ဒီနေရာမှာတော့ object ကို parameter အနေနဲ့ runtime မှာ ပေးမယ်။

```java
import java.util.*;
import java.util.function.*;

public class ArbitraryObjectExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Aung", "su", "KYAW", "Mya");

        // Lambda နည်းလမ်း
        names.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

        // Method Reference နည်းလမ်း
        names.sort(String::compareToIgnoreCase);

        System.out.println(names);
    }
}
```

> ဒီနေရာမှာ `String::compareToIgnoreCase` ဆိုတာက list ထဲက element (`s1`) ကိုယ်တိုင်ကို method ခေါ်ဖို့ object အဖြစ် သုံးမှာပါ (`s1.compareToIgnoreCase(s2)`)။

### ၄. Constructor Reference

`ClassName::new`

```java
import java.util.function.*;
import java.util.*;

public class ConstructorRefExample {
    static class Person {
        String name;
        Person(String name) {
            this.name = name;
        }
        public String toString() {
            return "Person: " + name;
        }
    }

    public static void main(String[] args) {
        // Lambda နည်းလမ်း
        Function<String, Person> creatorLambda = name -> new Person(name);

        // Constructor Reference နည်းလမ်း
        Function<String, Person> creatorRef = Person::new;

        Person p = creatorRef.apply("Kyaw Kyaw");
        System.out.println(p); // Person: Kyaw Kyaw
    }
}
```

## Stream API နှင့်တွဲသုံးထားသော ဥပမာ

```java
import java.util.*;
import java.util.stream.*;

public class StreamMethodRefExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("aung", "su", "kyaw", "mya");

        // toUpperCase method ကို method reference နဲ့ map လုပ်ခြင်း
        List<String> upperNames = names.stream()
                .map(String::toUpperCase)
                .collect(Collectors.toList());

        System.out.println(upperNames); // [AUNG, SU, KYAW, MYA]

        // print လုပ်ရန် System.out::println
        upperNames.forEach(System.out::println);
    }
}
```

## အမျိုးအစား ၄ ခု နှိုင်းယှဉ်ဇယား

|အမျိုးအစား|Syntax|ဥပမာ|
|---|---|---|
|Static method|`Class::staticMethod`|`Integer::parseInt`|
|Particular object ရဲ့ instance method|`object::instanceMethod`|`message::toUpperCase`|
|Arbitrary object ရဲ့ instance method|`Class::instanceMethod`|`String::compareToIgnoreCase`|
|Constructor|`Class::new`|`Person::new`|

## Lambda vs Method Reference ဘယ်အချိန်သုံးရမလဲ

- **Lambda body ထဲမှာ logic ပါဝင်ရင်** (calculation, condition စသဖြင့်) → **Lambda** ကိုသုံးပါ

```java
n -> n * n + 1  // ဒါကို method reference နဲ့ ရေး၍မရ
```

- **Existing method တစ်ခုကို ခေါ်ရုံသက်သက်ဖြစ်ရင်** → **Method Reference** ကိုသုံးပါ

```java
s -> s.length()   ➜  String::length
```

## အကျဉ်းချုပ်

Method reference တွေက lambda expression ရဲ့ **readable version** တစ်မျိုးသာ ဖြစ်ပြီး၊ functionality အတူတူပါပဲ။ Code ကို ပိုတိုတောင်းအောင်၊ ပိုနားလည်လွယ်အောင် ရေးနိုင်စေတဲ့ syntactic sugar လို့ ဆိုနိုင်ပါတယ်။ Stream API နဲ့ Collections framework တွေမှာ အလွန်ကျယ်ကျယ်ပြန့်ပြန့် အသုံးများပါတယ်။