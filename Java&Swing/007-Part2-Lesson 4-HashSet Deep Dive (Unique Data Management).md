# Part 2: Advanced Java Knowledge for Swing

# Lesson 4: HashSet Deep Dive (Unique Data Management)

ဒီ Lesson မှာ **HashSet** ကို လေ့လာပါမယ်။

HashSet က HashMap နဲ့ ဆက်စပ်နေပြီး Java Collection Framework ထဲမှာ အရေးကြီးတဲ့ Data Structure တစ်ခုဖြစ်ပါတယ်။

Real-world Café POS System မှာ:

- Duplicate product tag မဖြစ်အောင်
    
- User Role မထပ်အောင်
    
- Permission မထပ်အောင်
    
- Unique data တွေကို manage လုပ်ရန်
    

အသုံးများပါတယ်။

---

# 1. What is Set?

Collection Framework မှာ:

```
Collection

    |
    |
----------------
|              |
List           Set
```

List:

- Ordered
    
- Duplicate allowed
    

Example:

```
Coffee
Coffee
Cake
Tea
```

---

Set:

- Duplicate မခွင့်ပြု
    
- Unique data only
    

Example:

```
Coffee
Cake
Tea
```

Duplicate Coffee ကို ဖယ်ရှားပေးပါတယ်။

---

# 2. What is HashSet?

HashSet ဆိုတာ:

> Unique elements တွေကို hash table အသုံးပြုပြီး သိမ်းတဲ့ Collection ဖြစ်ပါတယ်။

Example:

```java
HashSet<String> products =
new HashSet<>();


products.add("Coffee");
products.add("Cake");
products.add("Tea");
```

Data:

```
Coffee
Cake
Tea
```

---

Duplicate ထည့်ကြည့်မယ်:

```java
products.add("Coffee");
```

Result:

```
Coffee
Cake
Tea
```

Coffee ထပ်မထည့်ပါဘူး။

---

# 3. HashSet Internal Working

Important:

HashSet internally uses:

```
HashMap
```

Structure:

```
HashSet

    |
    |
 HashMap

    |
    |
Key       Value

Coffee    PRESENT
Cake      PRESENT

```

---

Example:

```java
HashSet<String> set =
new HashSet<>();

set.add("Coffee");
```

Internally:

```
HashMap

Key:
Coffee

Value:
PRESENT

```

---

# 4. Creating HashSet

Example:

```java
import java.util.HashSet;


public class Test {


public static void main(String[] args){


HashSet<String> menu =
new HashSet<>();


menu.add("Coffee");
menu.add("Cake");
menu.add("Tea");


System.out.println(menu);


}

}
```

Output:

```
[Tea, Coffee, Cake]
```

---

Important:

HashSet order မထိန်းပါဘူး။

---

# 5. HashSet Basic Methods

## add()

Data ထည့်ခြင်း

```java
set.add("Coffee");
```

---

## remove()

```java
set.remove("Coffee");
```

---

## contains()

Check ရှိ/မရှိ

```java
if(set.contains("Coffee")){

System.out.println("Available");

}
```

---

## size()

```java
int count =
set.size();
```

---

## clear()

```java
set.clear();
```

အားလုံးဖျက်ပါတယ်။

---

# 6. HashSet Does Not Maintain Order

Example:

```java
HashSet<Integer> numbers =
new HashSet<>();

numbers.add(5);
numbers.add(1);
numbers.add(10);
numbers.add(3);
```

Output:

```
1
3
5
10
```

လို့ အမြဲမဖြစ်ပါဘူး။

HashSet ရဲ့ order ကို မယုံကြည်ရပါ။

---

If order လိုချင်ရင်:

Use:

```
LinkedHashSet
```

---

If sorted လိုချင်ရင်:

Use:

```
TreeSet
```

---

# 7. HashSet vs ArrayList

## ArrayList

```
Coffee
Cake
Coffee
Tea
```

Allow duplicate.

---

## HashSet

```
Coffee
Cake
Tea
```

No duplicate.

---

Comparison:

|Feature|ArrayList|HashSet|
|---|---|---|
|Duplicate|Allowed|Not Allowed|
|Order|Maintained|Not guaranteed|
|Index|Yes|No|
|Search|O(n)|O(1) average|

---

# 8. HashSet Internal hashCode() and equals()

HashSet မှာ:

```
hashCode()

+

equals()

```

ကို အသုံးပြုပါတယ်။

---

Example:

```java
HashSet<Product> products =
new HashSet<>();
```

Product Object ဖြစ်ရင်:

`hashCode()` နဲ့ `equals()` override လုပ်ရပါတယ်။

---

# 9. Custom Object with HashSet

Product:

```java
public class Product {


private int id;

private String name;


public Product(
int id,
String name
){

this.id=id;
this.name=name;

}


@Override
public int hashCode(){

return id;

}


@Override
public boolean equals(
Object obj
){

Product p =
(Product)obj;


return this.id==p.id;

}


}
```

---

Usage:

```java
HashSet<Product> products =
new HashSet<>();


products.add(
new Product(1,"Coffee")
);


products.add(
new Product(1,"Coffee")
);

```

Result:

```
Only one Product
```

---

# 10. Why Override equals()?

Without override:

```java
Product p1 =
new Product(1,"Coffee");


Product p2 =
new Product(1,"Coffee");

```

Java sees:

```
p1 != p2
```

because memory address different.

---

Override ပြီးရင်:

```
p1.equals(p2)

true

```

---

# 11. Real-world Café POS Usage

## Example 1: Product Category Tags

Product:

```
Coffee

Tags:

Hot
Drink
Popular

```

Java:

```java
HashSet<String> tags =
new HashSet<>();


tags.add("Hot");
tags.add("Drink");
tags.add("Popular");

```

Duplicate မဖြစ်နိုင်ပါဘူး။

---

# Example 2: User Roles

System:

```
User:

Admin


Roles:

CREATE_PRODUCT
DELETE_PRODUCT
VIEW_REPORT

```

Java:

```java
HashSet<String> permissions =
new HashSet<>();


permissions.add(
"CREATE_PRODUCT"
);

permissions.add(
"VIEW_REPORT"
);

```

---

Check:

```java
if(
permissions.contains("DELETE_PRODUCT")
){

allowDelete();

}

```

---

# Example 3: Remove Duplicate Data

Database Result:

```
Coffee
Cake
Coffee
Tea
Cake

```

Need:

```
Coffee
Cake
Tea

```

Solution:

```java
HashSet<String> uniqueProducts =
new HashSet<>(products);

```

---

# 12. HashSet Performance

Average:

|Operation|Performance|
|---|---|
|add()|O(1)|
|remove()|O(1)|
|contains()|O(1)|

Because:

```
Hash calculation
      |
      |
Direct bucket access
```

---

# 13. HashSet vs HashMap

Very Important.

## HashMap:

Stores:

```
Key -> Value
```

Example:

```
1001 -> Coffee
1002 -> Cake

```

---

## HashSet:

Stores:

```
Only Value
```

Example:

```
Coffee
Cake
Tea

```

---

Internally:

HashSet:

```
HashMap

Coffee -> PRESENT

```

---

# 14. HashSet vs TreeSet vs LinkedHashSet

Java Set Types:

```
Set

 |
 |
-----------------
|       |       |
HashSet LinkedHashSet TreeSet

```

---

## HashSet

```
Fastest

No order
```

---

## LinkedHashSet

```
Insertion order maintained

```

Example:

Input:

```
Coffee
Cake
Tea
```

Output:

```
Coffee
Cake
Tea
```

---

## TreeSet

Sorted order:

Example:

Input:

```
Tea
Coffee
Cake
```

Output:

```
Cake
Coffee
Tea
```

---

# 15. When to Use Which?

## HashSet

Use:

- Fast lookup
    
- Unique data
    

Example:

```
Permissions
Tags
IDs
```

---

## LinkedHashSet

Use:

- Unique + maintain insertion order
    

Example:

```
Recent searches
Menu display order

```

---

## TreeSet

Use:

- Unique + sorted
    

Example:

```
Price ranking
Alphabetical list

```

---

# 16. HashSet in Swing Application

Example:

Prevent duplicate product selection:

```java
HashSet<Integer> selectedProducts =
new HashSet<>();


selectedProducts.add(101);
selectedProducts.add(101);
selectedProducts.add(102);

```

Result:

```
101
102
```

---

# 17. Common Mistakes

## Mistake 1:

Expecting order:

Wrong:

```java
for(String s:set){

}
```

Expecting:

```
A
B
C
```

No guarantee.

---

## Mistake 2:

Using Object without equals/hashCode

Wrong:

```java
HashSet<Product>
```

without overriding methods.

---

## Mistake 3:

Using HashSet when duplicates are required

Example:

Sales History:

```
Coffee sale
Coffee sale
Coffee sale

```

HashSet မသုံးသင့်ပါ။

Use:

```
ArrayList
```

---

# 18. Interview Questions

## Q1: How HashSet prevents duplicates?

Answer:

HashSet uses:

1. hashCode()
    
2. equals()
    

to check duplicate objects.

---

## Q2: Does HashSet allow null?

Yes.

Example:

```java
set.add(null);
```

One null value allowed.

---

## Q3: Is HashSet synchronized?

No.

For multi-thread:

Use:

```
Collections.synchronizedSet()
```

or

```
ConcurrentHashMap.newKeySet()
```

---

# 19. Practice Project

Create Café POS Permission System.

Classes:

```
User

Permission

Role
```

Example:

Admin:

```
CREATE_PRODUCT
DELETE_PRODUCT
VIEW_REPORT
```

Cashier:

```
CREATE_ORDER
PAYMENT
```

Use:

```java
HashSet<String> permissions;
```

Functions:

```
addPermission()

removePermission()

hasPermission()

displayPermissions()

```

---

# Lesson 4 Summary

Today you learned:

✅ Set concept  
✅ HashSet internal HashMap relationship  
✅ Duplicate prevention  
✅ hashCode() and equals()  
✅ HashSet vs ArrayList  
✅ HashSet vs HashMap  
✅ LinkedHashSet / TreeSet  
✅ Real Café POS usage

---

# Next Lesson

## Lesson 5: Queue & PriorityQueue Deep Dive

We will build concepts for:

- Order Processing System
    
- Kitchen Queue
    
- Printer Queue
    
- FIFO
    
- Priority Ordering
    
- Real-time POS workflow
    

Example:

```
Customer Order

        ↓

Kitchen Queue

        ↓

Prepare Food

        ↓

Complete Order

```

ပြီးရင် Collections Framework ရဲ့ အရေးကြီးတဲ့အပိုင်းတွေဖြစ်တဲ့ **Map, Queue, Stream API** ဆက်သွားပါမယ်။