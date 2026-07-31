# Part 2: Advanced Java Knowledge for Swing

# Lesson 8: Optional Class Deep Dive

## Professional Null Handling in Java Applications

ဒီ Lesson မှာ Java 8 မှာ ပါလာတဲ့ **Optional Class** ကို လေ့လာပါမယ်။

Java Application တွေမှာ အများဆုံးဖြစ်တဲ့ Error တစ်ခုက:

```
NullPointerException
```

ဖြစ်ပါတယ်။

Café POS System လို Application တွေမှာ:

- Product မတွေ့ခြင်း
    
- User မရှိခြင်း
    
- Order မရှိခြင်း
    
- Database Result မရှိခြင်း
    

တွေကြောင့် Null ဖြစ်နိုင်ပါတယ်။

---

# 1. The Problem: NullPointerException

Example:

```java
Product product = productDAO.findById(1001);

System.out.println(
    product.getName()
);
```

Problem:

Database မှာ Product မရှိရင်:

```java
product = null;
```

ဖြစ်နိုင်ပါတယ်။

ဒါဆို:

```
NullPointerException
```

ဖြစ်ပါမယ်။

---

# 2. Traditional Null Check

Java အဟောင်း Style:

```java
Product product =
productDAO.findById(1001);


if(product != null){

    System.out.println(
        product.getName()
    );

}
else{

    System.out.println(
        "Product not found"
    );

}

```

အလုပ်လုပ်ပါတယ်။

ဒါပေမယ့် Code များလာရင်:

```java
if(x != null){

    if(y != null){

        if(z != null){

        }

    }

}

```

ဖြစ်လာပါတယ်။

ဒါကို:

```
Null Check Hell
```

လို့ခေါ်ပါတယ်။

---

# 3. What is Optional?

`Optional<T>` ဆိုတာ:

> Value ရှိနိုင်တယ်၊ မရှိနိုင်တယ်ဆိုတာကို Explicit ပြုလုပ်ဖို့ အသုံးပြုတဲ့ Container Object ဖြစ်ပါတယ်။

Structure:

```
Optional

  |
  |
----------------
|              |
Value          Empty

```

---

Example:

Value ရှိ:

```
Optional<Product>

    |
    |
 Product Object

```

Value မရှိ:

```
Optional.empty()

```

---

# 4. Creating Optional

## Optional.of()

Value သေချာရှိတဲ့အခါ:

```java
String name = "Coffee";


Optional<String> optional =
Optional.of(name);

```

---

But:

```java
String name = null;


Optional<String> optional =
Optional.of(name);

```

Result:

```
NullPointerException
```

---

# 5. Optional.ofNullable()

Real Application မှာ ပိုသုံးပါတယ်။

Value ရှိလည်းရ:

```java
String name="Coffee";

Optional<String> opt =
Optional.ofNullable(name);

```

---

Value null ဖြစ်လည်းရ:

```java
String name=null;


Optional<String> opt =
Optional.ofNullable(name);

```

Result:

```
Optional.empty()

```

---

# 6. Optional.empty()

Empty Optional create:

```java
Optional<String> empty =
Optional.empty();

```

Meaning:

```
No Value

```

---

# 7. Basic Optional Example

```java
Optional<String> name =
Optional.ofNullable("Coffee");


System.out.println(
name
);

```

Output:

```
Optional[Coffee]

```

---

# 8. isPresent()

Check value ရှိမရှိ:

```java
if(name.isPresent()){


System.out.println(
name.get()
);


}

```

Output:

```
Coffee

```

---

But:

Modern Java မှာ `isPresent()` + `get()` ကို အများကြီးမသုံးပါဘူး။

---

# 9. ifPresent()

Better way:

```java
name.ifPresent(
n -> System.out.println(n)
);

```

Meaning:

Value ရှိရင် execute လုပ်။

---

Example:

```java
product.ifPresent(
p -> showProduct(p)
);

```

---

# 10. orElse()

Value မရှိရင် default value ပေးရန်:

```java
String result =
name.orElse(
"Unknown"
);

```

---

Example:

```java
Optional<String> name =
Optional.empty();


System.out.println(
name.orElse("No Name")
);

```

Output:

```
No Name

```

---

# 11. orElseGet()

Difference:

`orElse()`

```java
name.orElse(
createDefault()
);

```

`createDefault()` က value ရှိလည်း execute ဖြစ်ပါတယ်။

---

`orElseGet()`:

```java
name.orElseGet(
() -> createDefault()
);

```

Value မရှိမှ execute ဖြစ်ပါတယ်။

---

Performance အတွက်:

```
orElseGet()
```

ပိုကောင်းပါတယ်။

---

# 12. orElseThrow()

Professional Application မှာ အရေးကြီးပါတယ်။

Example:

```java
Product product =
productOptional
.orElseThrow(
() -> new ProductNotFoundException(
"Product not found"
)
);

```

---

Meaning:

ရှိရင်ယူ

မရှိရင် Exception throw

---

# 13. Optional with Database DAO

Old DAO:

```java
public Product findById(int id){

    Product product=null;

    // database query

    return product;

}

```

Problem:

Caller မသိဘူး:

```
null?
value?
```

---

Better DAO:

```java
public Optional<Product> findById(
int id
){

Product product=null;


// query


return Optional.ofNullable(product);

}

```

---

Service:

```java
Product product =
productDAO
.findById(1001)
.orElseThrow(
() -> new ProductNotFoundException(
"Product does not exist"
)
);

```

---

# 14. Café POS Example

## Product Search

User:

```
Search Product ID: 1001

```

Flow:

```
Swing UI

   |
   v

Controller

   |
   v

Service

   |
   v

DAO

   |
   v

Optional<Product>

```

---

DAO:

```java
public Optional<Product> findProduct(
int id
){

return Optional.ofNullable(
database.find(id)
);

}

```

---

Service:

```java
public Product getProduct(
int id
){

return repository
.findProduct(id)
.orElseThrow(
() ->
new ProductNotFoundException(
"Product not found"
)
);

}

```

---

# 15. Optional map()

Data transform လုပ်ရန်:

Example:

```java
Optional<Product> product;


Optional<String> name =
product.map(
p -> p.getName()
);

```

---

Flow:

```
Product

   |
   |
map()

   |
   v

String Name

```

---

# 16. Optional filter()

Condition စစ်ရန်:

Example:

Only available products:

```java
Optional<Product> available =
product.filter(
p -> p.getStock()>0
);

```

---

Flow:

```
Product

 |
filter()

 |
Stock > 0

 |
Result

```

---

# 17. Optional Chaining

Example:

```java
String categoryName =
product
.map(Product::getCategory)
.map(Category::getName)
.orElse(
"Unknown"
);

```

---

Problem:

Old Java:

```java
product
.getCategory()
.getName();

```

Null ဖြစ်နိုင်ပါတယ်။

---

Optional:

Safe ဖြစ်ပါတယ်။

---

# 18. Optional with Stream

Example:

Find first product:

```java
Optional<Product> result =
products.stream()
.filter(
p -> p.getName()
.equals("Coffee")
)
.findFirst();

```

---

`findFirst()` က:

```java
Optional<T>
```

ပြန်ပေးပါတယ်။

---

# 19. Optional Best Practices

## Rule 1: Don't use Optional for fields

Bad:

```java
class Product{


Optional<String> name;


}

```

---

Use:

```java
class Product{


String name;


}

```

---

## Rule 2: Use Optional as Return Type

Good:

```java
Optional<Product>
findById();

```

---

## Rule 3: Don't call get() directly

Bad:

```java
optional.get();

```

Possible:

```
NoSuchElementException

```

---

Better:

```java
optional.orElseThrow();

```

---

# 20. Optional vs Null

|Null|Optional|
|---|---|
|Hidden possibility|Explicit|
|Runtime error|Controlled|
|Manual checking|Built-in methods|
|Less readable|More readable|

---

# 21. Real Café POS Architecture

Before:

```
Controller

   |
   |
null check

   |
Service

   |
DAO

```

---

After:

```
Controller

   |
Service

   |
DAO

   |
Optional<Product>

   |
orElseThrow()

   |
Custom Exception

```

---

# 22. Interview Questions

## Q1: Why Optional introduced?

Answer:

To represent absence of value and reduce NullPointerException.

---

## Q2: Difference between of() and ofNullable()?

`of()`

- Cannot accept null
    

`ofNullable()`

- Accepts null
    

---

## Q3: Should Optional replace all null?

No.

Use mainly for:

- Method return values
    
- Search results
    
- Database queries
    

---

## Q4: Difference between orElse and orElseGet?

orElse:

```
Always evaluates

```

orElseGet:

```
Only evaluates when empty

```

---

# Practice Project

Modify Café POS Product Repository:

Before:

```java
Product findById(int id);

```

After:

```java
Optional<Product> findById(int id);

```

Implement:

```java
getProduct()

searchProduct()

deleteProduct()

```

with:

- Optional
    
- Custom Exception
    
- Global Exception Handler
    

---

# Lesson 8 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ NullPointerException Problem  
✅ Optional Concept  
✅ Optional.of()  
✅ Optional.ofNullable()  
✅ empty()  
✅ ifPresent()  
✅ orElse()  
✅ orElseThrow()  
✅ map()  
✅ filter()  
✅ DAO Layer Usage  
✅ Professional Null Handling

---

# Next Lesson

# Lesson 9: Java Date & Time API Deep Dive

သင်ယူမည့်အရာ:

- LocalDate
    
- LocalTime
    
- LocalDateTime
    
- Date Formatting
    
- Date Comparison
    
- Sales Report Date Filtering
    
- Transaction Timestamp
    
- Café POS Daily Report System
    

Example:

```java
LocalDate today =
LocalDate.now();

```

ပြီးရင် POS System မှာ Date/Time Handling ကို Professional Level နဲ့ တည်ဆောက်ပါမယ်။