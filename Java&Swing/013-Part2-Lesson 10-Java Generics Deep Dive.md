# Part 2: Advanced Java Knowledge for Swing

# Lesson 10: Java Generics Deep Dive

## Building Type-Safe Professional Café POS Architecture

ဒီ Lesson မှာ **Java Generics** ကို လေ့လာပါမယ်။

Generics က Java Professional Developer ဖြစ်ဖို့ အရေးကြီးတဲ့ Concept တစ်ခုပါ။

အသုံးများတဲ့နေရာတွေ:

- Collection Framework (`List<T>`, `Map<K,V>`)
    
- DAO Pattern
    
- Repository Pattern
    
- Spring Framework
    
- Hibernate/JPA
    
- Reusable Components
    

Café POS System မှာ:

```text
Product Repository

Customer Repository

Order Repository

Employee Repository
```

အားလုံးအတွက် Code မထပ်ရေးအောင် Generics သုံးနိုင်ပါတယ်။

---

# 1. The Problem Without Generics

Java အဟောင်း Style:

```java
ArrayList products = new ArrayList();

products.add("Coffee");
products.add(1000);
products.add(true);
```

Problem:

ဒီ List ထဲမှာ ဘာ type တွေဝင်လဲ မသိပါဘူး။

---

Retrieve:

```java
String name =
(String) products.get(0);
```

Cast လုပ်ရပါတယ်။

Wrong type ဖြစ်ရင်:

```text
ClassCastException
```

ဖြစ်နိုင်ပါတယ်။

---

# 2. What are Generics?

Generics ဆိုတာ:

> Class, Interface, Method တွေမှာ Type ကို parameter အနေနဲ့ သတ်မှတ်ပေးတဲ့ feature ဖြစ်ပါတယ်။

Example:

```java
List<String>
```

Meaning:

ဒီ List ထဲမှာ String ပဲဝင်ရမယ်။

---

```java
List<Product>
```

Meaning:

Product object တွေပဲဝင်ရမယ်။

---

# 3. Generic Syntax

Basic:

```java
ClassName<Type>
```

Example:

```java
List<Product>
```

---

Common Type Names:

|Symbol|Meaning|
|---|---|
|T|Type|
|E|Element|
|K|Key|
|V|Value|
|N|Number|

---

# 4. Generics with ArrayList

Without:

```java
ArrayList list =
new ArrayList();
```

---

With:

```java
ArrayList<String> names =
new ArrayList<>();
```

Now:

Allowed:

```java
names.add("John");
```

Not allowed:

```java
names.add(100);
```

Compiler Error ဖြစ်ပါတယ်။

---

# 5. Type Safety

Example:

```java
List<Product> products =
new ArrayList<>();


products.add(
new Product()
);
```

Compiler က:

"ဒီ List မှာ Product ပဲရှိရမယ်"

လို့စောင့်ကြည့်ပါတယ်။

---

Benefit:

Runtime Error ↓

Compile Time Safety ↑

---

# 6. Generic Class

ကိုယ်ပိုင် Generic Class ရေးနိုင်ပါတယ်။

Example:

```java
public class Box<T>{


private T value;


public void set(T value){

this.value=value;

}


public T get(){

return value;

}


}

```

---

Use:

```java
Box<String> box =
new Box<>();

box.set("Coffee");


String item =
box.get();

```

---

Another Type:

```java
Box<Integer> number =
new Box<>();

number.set(100);

```

---

Same Class:

```text
Box<String>

Box<Integer>

Box<Product>
```

အသုံးပြုနိုင်ပါတယ်။

---

# 7. Generic Method

Class တစ်ခုလုံးမဟုတ်ဘဲ Method ကိုလည်း Generic လုပ်နိုင်ပါတယ်။

Example:

```java
public static <T> void print(T value){

System.out.println(value);

}

```

---

Usage:

```java
print("Coffee");

print(100);

print(new Product());

```

---

# 8. Café POS Example: Generic Repository

Problem:

Product DAO:

```java
class ProductDAO{

List<Product> findAll();

Product findById(int id);

}

```

Customer DAO:

```java
class CustomerDAO{

List<Customer> findAll();

Customer findById(int id);

}

```

Order DAO:

```java
class OrderDAO{

List<Order> findAll();

Order findById(int id);

}

```

---

Code ထပ်နေပါတယ်။

---

Solution:

Generic Repository:

```java
public interface Repository<T>{


List<T> findAll();


T findById(int id);


void save(T entity);


void delete(T entity);


}

```

---

Now:

Product:

```java
Repository<Product>
```

Customer:

```java
Repository<Customer>
```

Order:

```java
Repository<Order>
```

---

One design, many objects.

---

# 9. Generic Repository Implementation

```java
public class MyRepository<T>
implements Repository<T>{


private List<T> data =
new ArrayList<>();


@Override
public List<T> findAll(){

return data;

}


@Override
public void save(T entity){

data.add(entity);

}


}

```

---

Usage:

```java
Repository<Product> productRepo =
new MyRepository<>();


productRepo.save(
new Product()
);

```

---

# 10. Generic Interface

Example:

```java
interface Printer<T>{


void print(T data);


}

```

---

Implementation:

```java
class ReceiptPrinter
implements Printer<Order>{


public void print(Order order){

System.out.println(
order.getTotal()
);

}

}

```

---

# 11. Generic Wildcards (?)

Advanced Topic.

Wildcard means:

Unknown Type.

Example:

```java
List<?> list;

```

Meaning:

Any type List.

Allowed:

```java
List<String>

List<Integer>

List<Product>

```

---

# 12. Upper Bound Wildcard (? extends)

Meaning:

"ဒီ Type သို့မဟုတ် သူ့ရဲ့ Subclass"

Syntax:

```java
? extends Type
```

---

Example:

```java
List<? extends Number>
```

Accept:

```java
List<Integer>

List<Double>

```

---

Real Example:

Calculate Total:

```java
public double sum(
List<? extends Number> numbers
){

double total=0;


for(Number n:numbers){

total += n.doubleValue();

}


return total;

}

```

---

# 13. Lower Bound (? super)

Meaning:

"ဒီ Type သို့မဟုတ် သူ့ရဲ့ Parent"

Syntax:

```java
? super Type
```

---

Example:

```java
List<? super Product>
```

Accept:

```text
Product

Object

```

---

# 14. PECS Rule

Professional Interview Topic.

PECS:

```text
Producer Extends

Consumer Super
```

---

If data produces:

```java
? extends
```

Example:

Read data:

```java
List<? extends Product>
```

---

If data consumes:

```java
? super
```

Example:

Write data:

```java
List<? super Product>
```

---

# 15. Generic Collections Examples

## List

```java
List<Product>
```

---

## Map

```java
Map<Integer,Product>
```

Meaning:

```text
Product ID -> Product Object
```

---

Example:

```java
Map<Integer,Product> products =
new HashMap<>();


products.put(
1001,
new Product()
);

```

---

## Set

```java
Set<String>
```

---

# 16. Generic Utility Class

Example:

```java
public class Converter{


public static <T> List<T> copy(
List<T> source
){

return new ArrayList<>(source);

}


}

```

---

# 17. Generic Exception

Can Exception be Generic?

No.

Wrong:

```java
class MyException<T>
extends Exception
{

}

```

Java does not allow generic Throwable classes.

---

# 18. Generic Array

Problem:

```java
T[] array =
new T[10];

```

Not allowed.

---

Solution:

```java
Object[] array =
new Object[10];

```

or:

```java
ArrayList<T>
```

သုံးပါ။

---

# 19. Real Café POS Architecture with Generics

Before:

```text
ProductDAO

CustomerDAO

OrderDAO

EmployeeDAO

```

---

After:

```text
              Repository<T>

                    |
        ----------------------

        |          |          |

    Product    Customer    Order

```

---

# 20. Swing Application Example

Table Model:

Generic Table Model:

```java
class GenericTableModel<T>
extends AbstractTableModel{


private List<T> data;


}

```

---

Use:

```java
GenericTableModel<Product>

GenericTableModel<Order>

```

---

One component:

Multiple screens.

---

# 21. Generics + Optional

Professional DAO:

```java
public interface Repository<T>{


Optional<T> findById(int id);


}

```

---

Usage:

```java
Optional<Product> product =
productRepo.findById(1001);

```

---

# 22. Generics + Stream API

Example:

```java
public <T> void print(
List<T> list
){

list.stream()
.forEach(
System.out::println
);

}

```

---

# 23. Interview Questions

## Q1: Why use Generics?

Answer:

To provide type safety and reusable code.

---

## Q2: Difference between Object and Generic?

Object:

```text
Runtime checking
Casting required
```

Generic:

```text
Compile-time checking
No casting
```

---

## Q3: What is Type Erasure?

Java compiler removes generic type information at runtime.

Example:

Compile:

```java
List<Product>
```

Runtime:

```java
List
```

---

## Q4: Can we create generic static variables?

No.

Example:

```java
static T value;
```

Not allowed.

---

# Practice Project

Refactor Café POS DAO Layer:

Create:

```java
Repository<T>
```

Methods:

```java
save(T)

findAll()

findById()

delete()

```

Implement:

```java
ProductRepository

CustomerRepository

OrderRepository

```

using:

- Generics
    
- Optional
    
- List
    
- Stream API
    

---

# Lesson 10 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ What are Generics  
✅ Type Safety  
✅ Generic Class  
✅ Generic Method  
✅ Generic Interface  
✅ Repository Pattern  
✅ Wildcard (?)  
✅ extends  
✅ super  
✅ PECS Rule  
✅ Generic DAO Architecture  
✅ Café POS Design Usage

---

# Next Lesson

# Lesson 11: Java Reflection API & Annotation Deep Dive

သင်ယူမည့်အရာ:

- What is Reflection?
    
- Class Metadata
    
- Creating Objects Dynamically
    
- Custom Annotation
    
- Runtime Processing
    
- How Spring/Hibernate use Reflection
    
- Building Framework-like Features
    

Example:

```java
@Table(name="products")
class Product{

}

```

ပြီးရင် Java Framework အလုပ်လုပ်ပုံကို နားလည်လာပါမယ်။