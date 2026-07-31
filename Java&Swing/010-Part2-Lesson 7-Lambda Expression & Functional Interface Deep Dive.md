# Part 2: Advanced Java Knowledge for Swing

# Lesson 7: Lambda Expression & Functional Interface Deep Dive

## Modern Java Programming Style

ဒီ Lesson မှာ **Lambda Expression** နဲ့ **Functional Interface** ကို အသေးစိတ်လေ့လာပါမယ်။

Java Swing + Café POS System မှာ:

- Button Event Handling
    
- Stream API
    
- Sorting
    
- Filtering
    
- Callback Design
    
- Asynchronous Programming
    

တွေမှာ Lambda ကို အများကြီးအသုံးပြုပါတယ်။

---

# 1. Why Lambda Expression?

Java 7 အရင်က:

```java
button.addActionListener(
    new ActionListener(){

        @Override
        public void actionPerformed(ActionEvent e){

            saveOrder();

        }

    }
);
```

Code များပါတယ်။

---

Java 8 Lambda:

```java
button.addActionListener(
    e -> saveOrder()
);

```

ပိုတိုပြီး ဖတ်ရလွယ်ပါတယ်။

---

# 2. What is Lambda Expression?

Lambda ဆိုတာ:

> Anonymous Function (နာမည်မရှိတဲ့ function) ကို ရေးတဲ့ syntax ဖြစ်ပါတယ်။

Structure:

```java
(parameter) -> expression

```

or

```java
(parameter) -> {
    statements;
}

```

---

Example:

Normal Method:

```java
public int add(int a,int b){

    return a+b;

}

```

Lambda:

```java
(a,b) -> a+b

```

---

# 3. Lambda Syntax

## No Parameter

```java
() -> System.out.println("Hello");

```

---

## One Parameter

```java
name -> System.out.println(name);

```

Parentheses မလိုပါ။

---

## Multiple Parameters

```java
(a,b) -> a+b;

```

---

## Multiple Statements

```java
(a,b)->{

int result=a+b;

return result;

}

```

---

# 4. Why Lambda Needs Functional Interface?

Lambda တစ်ခုတည်းနဲ့ မအလုပ်လုပ်ပါဘူး။

Lambda က:

```id="j51c8k"
Functional Interface

```

လိုအပ်ပါတယ်။

---

# 5. What is Functional Interface?

Functional Interface ဆိုတာ:

> Abstract Method တစ်ခုပဲ ပါတဲ့ Interface ဖြစ်ပါတယ်။

Example:

```java
interface Calculator {


    int calculate(int a,int b);


}

```

ဒီမှာ:

```java
calculate()

```

တစ်ခုပဲရှိပါတယ်။

ဒါကြောင့် Functional Interface ဖြစ်ပါတယ်။

---

# 6. Using Lambda with Custom Interface

Without Lambda:

```java
Calculator c =
new Calculator(){

@Override
public int calculate(int a,int b){

return a+b;

}

};

```

---

With Lambda:

```java
Calculator c =
(a,b) -> a+b;

```

---

Usage:

```java
System.out.println(
c.calculate(10,20)
);

```

Output:

```
30
```

---

# 7. @FunctionalInterface Annotation

Professional code မှာ:

```java
@FunctionalInterface
interface Calculator {


int calculate(int a,int b);


}

```

အသုံးပြုပါတယ်။

---

ဒီ Annotation က:

"ဒီ Interface မှာ Method တစ်ခုပဲရှိရမယ်"

ဆိုတာ compiler ကိုပြောတာပါ။

---

# 8. Built-in Functional Interfaces

Java က အသင့်ပေးထားတဲ့ Functional Interface တွေရှိပါတယ်။

Package:

```java
java.util.function

```

Important 4 ခု:

1. Predicate
    
2. Function
    
3. Consumer
    
4. Supplier
    

---

# 9. Predicate

Purpose:

> Input ယူပြီး boolean ပြန်ပေးသည်။

Structure:

```java
Predicate<T>

T -> boolean

```

---

Example:

Check product price:

```java
Predicate<Product> expensive =
p -> p.getPrice()>5000;

```

---

Usage:

```java
if(expensive.test(product)){

System.out.println("Expensive");

}

```

---

Output:

```
Expensive
```

---

Real POS:

```java
Predicate<Product> lowStock =
p -> p.getStock()<10;

```

အသုံး:

```java
Low stock report

```

---

# 10. Function<T,R>

Purpose:

> Input တစ်ခုယူပြီး Result ပြန်ပေးသည်။

Structure:

```
T -> R
```

---

Example:

Product → Name

```java
Function<Product,String> getName =
p -> p.getName();

```

---

Usage:

```java
String name =
getName.apply(product);

```

---

Output:

```
Coffee
```

---

Real POS:

Product:

```
Product
   |
   |
 String
```

Convert:

```
Product -> Product Name
```

---

# 11. Consumer

Purpose:

> Input ယူပြီး ဘာမှ return မပြန်။

Structure:

```
T -> void

```

---

Example:

```java
Consumer<Product> printer =
p -> System.out.println(
p.getName()
);

```

---

Usage:

```java
printer.accept(product);

```

Output:

```
Coffee
```

---

Real POS:

Print receipt:

```java
Consumer<Order> printReceipt =
order -> print(order);

```

---

# 12. Supplier

Purpose:

> Input မယူဘဲ Value ပြန်ပေး။

Structure:

```
() -> T

```

---

Example:

```java
Supplier<String> time =
() -> LocalDateTime.now()
.toString();

```

---

Usage:

```java
System.out.println(
time.get()
);

```

---

Real POS:

Generate Order Number:

```java
Supplier<String> orderNumber =
() -> "ORD-"+System.currentTimeMillis();

```

---

# 13. Comparison Table

|Interface|Input|Output|Usage|
|---|---|---|---|
|Predicate|T|boolean|Condition|
|Function|T|R|Convert|
|Consumer|T|void|Action|
|Supplier|none|T|Generate|

---

# 14. Lambda with Stream API

Previous:

```java
products.stream()
.filter(
p -> p.getPrice()>5000
)
.collect(Collectors.toList());

```

ဒီမှာ:

```java
p -> p.getPrice()>5000

```

က Predicate ဖြစ်ပါတယ်။

---

# 15. Sorting with Lambda

Example:

Price sort:

```java
products.sort(

(a,b)->

Double.compare(
a.getPrice(),
b.getPrice()
)

);

```

---

Before:

```
Cake 7000
Coffee 2000
Burger 9000
```

After:

```
Coffee
Cake
Burger
```

---

# 16. Lambda in Swing Event Handling

Old:

```java
button.addActionListener(
new ActionListener(){

public void actionPerformed(ActionEvent e){

save();

}

});

```

---

Modern:

```java
saveButton.addActionListener(
e -> save()
);

```

---

# 17. Custom Functional Interface in Café POS

Example:

Discount Calculator:

```java
@FunctionalInterface
interface Discount {


double calculate(double price);


}

```

---

Usage:

```java
Discount vipDiscount =
price -> price*0.9;


double result =
vipDiscount.calculate(10000);

```

Output:

```
9000
```

---

# 18. Lambda + Strategy Pattern

Professional Design Pattern:

Before:

```java
class DiscountService {


if(type=="VIP"){

}

else if(type=="MEMBER"){

}


}

```

Problem:

Condition များလာမယ်။

---

Lambda:

```java
DiscountStrategy strategy;


price -> price*0.8

```

Flexible ဖြစ်ပါတယ်။

---

# 19. Real Café POS Example

## Payment Strategy

Payment:

```
Cash
Card
Mobile Pay

```

Interface:

```java
@FunctionalInterface
interface Payment {


void pay(double amount);


}

```

---

Cash:

```java
Payment cash =
amount ->
System.out.println(
"Cash: "+amount
);

```

---

Card:

```java
Payment card =
amount ->
System.out.println(
"Card: "+amount
);

```

---

Usage:

```java
payment.pay(50000);

```

---

# 20. Lambda Limitations

## 1. Only Functional Interface

Wrong:

```java
interface Test{

void a();

void b();

}

```

Lambda မသုံးနိုင်ပါ။

---

## 2. Hard to Debug

Complex Lambda:

```java
stream
.filter(...)
.map(...)
.reduce(...)

```

များရင် Debug ခက်နိုင်ပါတယ်။

---

## 3. Avoid Very Long Lambda

Bad:

```java
x -> {

100 lines code

}

```

Method ခွဲရေးပါ။

---

# 21. Interview Questions

## Q1: What is Lambda Expression?

Answer:

Lambda is an anonymous function introduced in Java 8 to provide functional programming support.

---

## Q2: Why Functional Interface?

Because Lambda expression requires an interface with exactly one abstract method.

---

## Q3: Difference between Predicate and Function?

Predicate:

```
T -> boolean
```

Function:

```
T -> R
```

---

## Q4: Can Lambda replace all interfaces?

No.

Only functional interfaces.

---

# 22. Café POS Modern Architecture

Now our POS uses:

```
Database

    |
    v

Collection

    |
    v

Stream API

    |
    v

Lambda

    |
    |
----------------
Filter
Sort
Calculate
Event
----------------

    |
    v

Swing UI

```

---

# Practice Exercise

Create Functional Interfaces:

## 1. Discount

```java
Discount vip =
price -> price*0.8;

```

---

## 2. Product Filter

```java
Predicate<Product>

```

Find:

```
price > 5000

```

---

## 3. Receipt Printer

```java
Consumer<Order>

```

Print:

```
Order ID
Items
Total

```

---

# Lesson 7 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Lambda Expression  
✅ Functional Interface  
✅ @FunctionalInterface  
✅ Predicate  
✅ Function  
✅ Consumer  
✅ Supplier  
✅ Stream + Lambda  
✅ Swing Event Lambda  
✅ Strategy Pattern usage  
✅ Café POS examples

---

# Next Lesson

## Lesson 8: Optional Class Deep Dive

### Solving NullPointerException Professionally

သင်ယူမည့်အရာ:

- Why Optional was introduced
    
- Optional.of()
    
- Optional.ofNullable()
    
- Optional.empty()
    
- map()
    
- filter()
    
- orElse()
    
- orElseThrow()
    
- Optional with Database DAO
    
- Preventing NullPointerException in POS System
    

Example:

```java
Optional<Product> product =
productRepository.findById(1001);

```

ပြီးရင် Java Modern Error Handling ကို ဆက်သွားပါမယ်။