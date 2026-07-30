**Java Stream API ပေါင်းစပ်အသုံးပြုနည်း** (Practical Combined Usage)

Stream API ကို တစ်ခုချင်း မဟုတ်ဘဲ **ပေါင်းစပ်သုံးတဲ့** နည်းလမ်းတွေ ဥပမာတွေ အသေးစိတ် ရှင်းပြပါမယ်။

### 1. Basic Structure

```java
list.stream()
    .filter(...)      // စစ်ထုတ်
    .map(...)         // ပြောင်းလဲ
    .sorted(...)      // စဥ်
    .distinct()       // ထပ်နေတာ ဖယ်
    .limit(...)       // အရေအတွက် ကန့်သတ်
    .collect(...)     // ရလဒ်စုစည်း
```

### 2. အသုံးများဆုံး ပေါင်းစပ်ဥပမာ

**ဥပမာ ၁: Employee စာရင်း စီမံခြင်း**

```java
record Employee(String name, String dept, int salary, int age) {}

List<Employee> employees = List.of(
    new Employee("အောင်ကိုကို", "IT", 1200000, 28),
    new Employee("မြမြအောင်", "HR", 800000, 35),
    new Employee("စိုင်းစိုင်း", "IT", 1500000, 30),
    new Employee("သီသီ", "Finance", 900000, 25),
    new Employee("မောင်မောင်", "IT", 1100000, 29)
);

// ပေါင်းစပ်အသုံး
List<String> result = employees.stream()
    .filter(e -> e.dept().equals("IT"))                    // IT ဌာနပဲ
    .filter(e -> e.salary() > 1000000)                     // လစာ ၁၀ သိန်းကျော်
    .sorted(Comparator.comparingInt(Employee::salary).reversed()) // လစာအများဆုံး ဦးစားပေး
    .map(Employee::name)                                   // နာမည်ပဲ ထုတ်
    .collect(Collectors.toList());

System.out.println(result);
```

**ဥပမာ ၂: စာရင်း ခွဲခြမ်း စုစည်း (Grouping)**

```java
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::dept, 
        Collectors.counting()
    ));

Map<String, Integer> totalSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::dept,
        Collectors.summingInt(Employee::salary)
    ));
```

**ဥပမာ ၃: ရှုပ်ထွေးတဲ့ ပေါင်းစပ်အသုံး**

```java
List<String> topNames = employees.stream()
    .filter(e -> e.age() >= 25 && e.age() <= 35)
    .peek(e -> System.out.println("Processing: " + e.name())) // debugging
    .sorted(Comparator.comparingInt(Employee::salary).reversed())
    .limit(3)
    .map(e -> e.name().toUpperCase())
    .collect(Collectors.toList());
```

### 3. အသုံးဝင်ဆုံး Collectors

- `Collectors.toList()`
- `Collectors.toSet()`
- `Collectors.toMap()`
- `Collectors.groupingBy()`
- `Collectors.partitioningBy()` → true/false ခွဲ
- `Collectors.joining(", ")`
- `Collectors.summarizingInt()` → sum, avg, max, min တစ်ခါတည်း

### 4. Parallel Stream (အမြန်ဆုံး)

```java
long sum = employees.parallelStream()
    .filter(e -> e.dept().equals("IT"))
    .mapToInt(Employee::salary)
    .sum();
```

### 5. သတိထားရမယ့်အချက်များ

- Stream ကို **တစ်ကြိမ်တည်း** သုံးနိုင်တယ် (reuse မရ)
- `forEach()` ကို အဓိက ရလဒ်ထုတ်ဖို့ မသုံးပါနဲ့ → `collect()` ကို ဦးစားပေး
- Null value ရှိနိုင်ရင် `filter(Objects::nonNull)` သုံးပါ
- ရှုပ်ထွေးရင် အဆင့်ဆင့် ခွဲရေးပါ ( readability အတွက်)

---

**လိုချင်တဲ့ ဥပမာ အမျိုးအစား ပြောပါ**:

- Student စာရင်း
- Product စျေးနှုန်း စီမံခြင်း
- File ဖတ်ပြီး စီမံခြင်း
- Complex grouping + multi-level sorting

ဘယ်လို ဥပမာ လိုချင်လဲ ပြောပြပါ၊ အတိအကျ ရေးပေးပါမယ်။

---

**Stream Pipeline အဆင့်ဆင့် စဉ်းစားနည်း** (Step-by-Step Thinking)

Stream Pipeline ကို ကောင်းကောင်း ရေးနိုင်ဖို့ **အဆင့်ဆင့် စဉ်းစားတဲ့ နည်းလမ်း** ကို ရှင်းပြပါမယ်။

### Stream Pipeline ကို ၆ အဆင့် ခွဲစဉ်းစားနည်း

1. **Source (အရင်းအမြစ်)**
2. **Filtering (စစ်ထုတ်ခြင်း)**
3. **Transformation (ပြောင်းလဲခြင်း)**
4. **Ordering & Limiting (စဥ်ခြင်း & ကန့်သတ်ခြင်း)**
5. **Terminal Operation (အဆုံး လုပ်ဆောင်ချက်)**
6. **Collecting / Reduction (ရလဒ်စုစည်း)**

---

### လက်တွေ့ ဥပမာ အဆင့်ဆင့် စဉ်းစားပုံ

**မေးခွန်း**:  
"IT ဌာနမှ လစာ ၈ သိန်းကျော် ရတဲ့ ဝန်ထမ်းတွေထဲမှာ အသက် ၂၅ နှစ်အောက် ဖြစ်တဲ့ သူတွေကို လစာအများဆုံး အစဉ်လိုက် ၅ ယောက်ပဲ ယူပြီး နာမည်နဲ့ လစာ ပြန်ထုတ်ပေး။"

#### အဆင့်ဆင့် စဉ်းစားပုံ:

**Step 1: Source**

```java
employees.stream()           // သို့မဟုတ် parallelStream()
```

**Step 2: Filtering (စစ်ထုတ်မှု - အရေးအကြီးဆုံး)**

```java
.filter(e -> "IT".equals(e.getDept()))
.filter(e -> e.getSalary() > 800000)
.filter(e -> e.getAge() < 25)
```

**Step 3: Transformation (လိုအပ်ရင်ပဲ သုံး)**

```java
.map(e -> new SimpleEmployee(e.getName(), e.getSalary())) // လိုအပ်ရင်
// သို့မဟုတ် .mapToInt(Employee::getSalary) သုံးနိုင်
```

**Step 4: Ordering & Limiting**

```java
.sorted(Comparator.comparingInt(Employee::getSalary).reversed())
.limit(5)
```

**Step 5 & 6: Terminal + Collecting**

```java
.collect(Collectors.toList());
```

**ပြီးပြည့်စုံ ကုဒ်**:

```java
List<Result> top5 = employees.stream()
    .filter(e -> "IT".equals(e.getDept()))
    .filter(e -> e.getSalary() > 800000)
    .filter(e -> e.getAge() < 25)
    .sorted(Comparator.comparingInt(Employee::getSalary).reversed())
    .limit(5)
    .map(e -> new Result(e.getName(), e.getSalary()))
    .collect(Collectors.toList());
```

---

### ကောင်းမွန်တဲ့ Pipeline တစ်ခု ရေးတဲ့ အတွေးအခေါ်

|အဆင့်|မေးရမယ့်မေးခွန်း|သုံးရမယ့် Operation|
|---|---|---|
|Source|ဘယ်ကနေ စမလဲ?|stream(), list.stream()|
|Filter|ဘယ်အရာတွေ ပယ်မလဲ?|filter(), distinct()|
|Transform|ဒေတာကို ဘယ်လို ပြောင်းမလဲ?|map(), flatMap(), mapToInt()|
|Order/Limit|အစဉ်လိုက်ရင် ဘယ်လို စဥ်မလဲ?|sorted(), limit(), skip()|
|Collect|ဘယ်ပုံစံနဲ့ ရလဒ်လိုချင်လဲ?|collect(), forEach(), reduce()|

### အကြံပြုချက်များ

1. **Filter ကို အရင်ဆုံး ရေး** → မလိုအပ်တဲ့ ဒေတာ အရေအတွက်ကို လျှော့ချပါ (Performance ကောင်းမယ်)
2. **Map ကို နောက်မှ ရေး** → မလိုအပ်တဲ့ အရာတွေ ပြောင်းမနေအောင်
3. **Pipeline ကို အရမ်းရှည်ရင် ခွဲရေး** (Method ခွဲသုံး)
4. `peek()` ကို debugging အတွက် သုံးပါ
5. ရှုပ်ထွေးရင် **Intermediate Variable** နဲ့ ခွဲရေးပါ

**ဥပမာ (ခွဲရေးနည်း)**:

```java
private static Stream<Employee> itHighSalary(Stream<Employee> stream) {
    return stream.filter(e -> "IT".equals(e.getDept()))
                 .filter(e -> e.getSalary() > 1000000);
}
```

---

**လေ့ကျင့်ရန် မေးခွန်း**  
သင်ကြိုက်တဲ့ ဒေတာ (Student, Product, Order) တစ်ခုပြောပါ။ အဲဒီ အတွက် Stream Pipeline ကို အဆင့်ဆင့် ခွဲပြီး ရေးပေးပါမယ်။

ဘယ်လို ဥပမာ လိုချင်ပါသလဲ?