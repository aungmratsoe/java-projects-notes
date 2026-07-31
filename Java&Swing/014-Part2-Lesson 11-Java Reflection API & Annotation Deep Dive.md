# Part 2: Advanced Java Knowledge for Swing

# Lesson 11: Java Reflection API & Annotation Deep Dive

## Understanding How Spring, Hibernate, and Frameworks Work Internally

ဒီ Lesson မှာ **Reflection API** နဲ့ **Annotation** ကို လေ့လာပါမယ်။

ဒီ Concept တွေက Beginner Level မှာ မတွေ့ရပေမယ့် Professional Java Developer ဖြစ်ဖို့ အရေးကြီးပါတယ်။

အသုံးများတဲ့နေရာတွေ:

- Spring Framework
    
- Hibernate / JPA
    
- Dependency Injection
    
- ORM Mapping
    
- Testing Framework
    
- Serialization
    
- Custom Framework Development
    

Café POS System မှာလည်း:

- Database Table Mapping
    
- Automatic Validation
    
- Logging
    
- Permission Checking
    

တွေ တည်ဆောက်နိုင်ပါတယ်။

---

# 1. What is Reflection API?

Reflection ဆိုတာ:

> Program running ဖြစ်နေတဲ့အချိန်မှာ Class, Method, Field တွေရဲ့ information ကို ကြည့်ရှု၊ ပြောင်းလဲ၊ အသုံးပြုနိုင်တဲ့ Java feature ဖြစ်ပါတယ်။

Normal Java:

```java
Product product =
new Product();

product.getName();

```

Compiler အချိန်မှာ သိပြီးသား:

```
Product Class
    |
    |
getName()
```

---

Reflection:

Runtime မှာ:

```
Application Running

        |
        v

Find Class

        |
        v

Find Methods

        |
        v

Execute Method

```

---

# 2. Why Reflection?

Normal Code:

```java
Product p =
new Product();

```

Problem:

Class name ကို သိထားရမယ်။

---

Framework တွေမှာ:

```
User creates:

Product.java

Framework:

"I don't know Product class before"

```

ဒါကြောင့် Reflection သုံးပါတယ်။

---

# 3. Getting Class Information

Java မှာ Class Object ရှိပါတယ်။

Example:

```java
Class<Product> clazz =
Product.class;

```

---

Alternative:

```java
Product p =
new Product();


Class clazz =
p.getClass();

```

---

String name:

```java
Class clazz =
Class.forName(
"com.pos.Product"
);

```

---

# 4. Class Metadata

Example:

```java
class Product {


private int id;

private String name;


public void save(){

}

}

```

Reflection:

```java
Class clazz =
Product.class;

```

သိနိုင်တာတွေ:

```
Class Name

Fields

Methods

Constructors

Annotations

Modifiers

```

---

# 5. Get Class Name

```java
Class clazz =
Product.class;


System.out.println(
clazz.getName()
);

```

Output:

```
com.pos.Product
```

---

# 6. Getting Fields

Product:

```java
class Product{


private int id;

private String name;


}

```

Reflection:

```java
Field[] fields =
clazz.getDeclaredFields();


for(Field f:fields){

System.out.println(
f.getName()
);

}

```

Output:

```
id
name
```

---

# 7. Access Private Field

Normally:

```java
product.name

```

Not allowed.

Because:

```java
private

```

Reflection:

```java
Field field =
clazz.getDeclaredField(
"name"
);


field.setAccessible(true);

```

Now access:

```java
field.set(
product,
"Coffee"
);

```

---

# 8. Getting Methods

Example:

```java
Method[] methods =
clazz.getDeclaredMethods();


for(Method m:methods){

System.out.println(
m.getName()
);

}

```

Output:

```
save
getName
setName

```

---

# 9. Invoke Method Dynamically

Normal:

```java
product.save();

```

Reflection:

```java
Method method =
clazz.getMethod(
"save"
);


method.invoke(product);

```

Same result.

---

# 10. Creating Object Dynamically

Normal:

```java
Product p =
new Product();

```

Reflection:

```java
Class clazz =
Product.class;


Product p =
(Product)
clazz
.getDeclaredConstructor()
.newInstance();

```

---

Framework Example:

Spring:

```
@Component

class ProductService

        |

Spring Reflection

        |

new ProductService()

```

---

# 11. Reflection and Dependency Injection

Example:

```java
class OrderService{


private PaymentService paymentService;


}

```

Spring sees:

```
OrderService

Field:

paymentService

```

Reflection:

```
Create PaymentService

Inject into field

```

---

# 12. What are Annotations?

Annotation ဆိုတာ:

> Code အပေါ်မှာ metadata information ထည့်တဲ့ feature ဖြစ်ပါတယ်။

Example:

```java
@Override
public String toString(){

}

```

---

Annotation syntax:

```java
@interface

```

---

Examples:

```java
@Override

@Entity

@Table

@Component

@Autowired

```

---

# 13. Built-in Annotations

## @Override

Compiler ကို:

"Parent method override လုပ်နေတယ်"

ပြောတာ။

Example:

```java
@Override
public String toString(){

return "";

}

```

---

## @Deprecated

Old API:

```java
@Deprecated
void oldMethod(){

}

```

Warning ပြပါတယ်။

---

## @SuppressWarnings

Warning ပိတ်ရန်:

```java
@SuppressWarnings("unchecked")

```

---

# 14. Custom Annotation

ကိုယ်တိုင် Annotation ရေးနိုင်ပါတယ်။

Example:

```java
public @interface Table {

String name();

}

```

---

Usage:

```java
@Table(
name="products"
)
class Product{


}

```

---

Meaning:

```
Product Class

maps to

products table

```

---

# 15. Reading Annotation using Reflection

Example:

```java
Class clazz =
Product.class;


Table table =
clazz.getAnnotation(
Table.class
);


System.out.println(
table.name()
);

```

Output:

```
products

```

---

# 16. Café POS ORM Example

Manual SQL:

```java
INSERT INTO products
(name,price)
VALUES
(?,?)

```

---

Framework style:

Entity:

```java
@Entity
@Table(
name="products"
)
class Product{


@Id
private int id;


@Column
private String name;


}

```

---

Framework Reflection:

```
Read Annotation

       |

Find Table

       |

Generate SQL

       |

Execute

```

---

# 17. Creating Validation Annotation

Example:

```java
@Retention(
RetentionPolicy.RUNTIME
)
@Target(
ElementType.FIELD
)
public @interface NotEmpty {


}

```

---

Usage:

```java
class Product{


@NotEmpty
private String name;


}

```

---

Runtime:

```
Reflection

   |

Find @NotEmpty

   |

Validate

```

---

# 18. Retention Policy

Important.

Annotation lifetime:

## SOURCE

Only source code.

```
Compiler

   |

Removed

```

---

## CLASS

Stored in .class file.

Runtime မရ။

---

## RUNTIME

Runtime မှာ access ရပါတယ်။

Framework တွေသုံးတာ:

```
@Retention(RUNTIME)

```

---

# 19. Target

Annotation ဘယ်မှာသုံးမလဲ:

Example:

```java
@Target(
ElementType.FIELD
)

```

Only Field.

---

Options:

```
TYPE       Class

METHOD     Method

FIELD      Field

PARAMETER  Parameter

```

---

# 20. Real Framework Example

Spring:

```java
@Component
class ProductService{

}

```

Spring Startup:

```
Scan Classes

      |

Find @Component

      |

Reflection

      |

Create Object

      |

Store Bean

```

---

Hibernate:

```java
@Entity
class Product{

}

```

Hibernate:

```
Read Entity

     |

Create SQL

     |

Map Database

```

---

# 21. Reflection Security Issues

Reflection powerful ဖြစ်တဲ့အတွက်:

Problems:

- Private access bypass
    
- Slower than normal calls
    
- Hard debugging
    

Example:

```java
field.setAccessible(true);

```

သတိထားသုံးရပါတယ်။

---

# 22. Reflection Performance

Normal:

```java
object.method();

```

Fast.

Reflection:

```java
method.invoke(object);

```

Slower.

Because:

```
Runtime lookup

Validation

Invocation

```

လုပ်ရပါတယ်။

---

# 23. Reflection Usage in Café POS

Possible Features:

## Automatic Table Mapping

```java
@Table("products")
class Product

```

Generate:

```sql
SELECT * FROM products

```

---

## Automatic Audit

Annotation:

```java
@Audit
void deleteProduct()

```

Reflection:

```
Before method

After method

Log

```

---

## Permission System

Annotation:

```java
@Permission(
"DELETE_PRODUCT"
)

void delete()

```

---

Runtime:

```
Check User Permission

Allow / Deny

```

---

# 24. Reflection vs Normal Programming

|Normal|Reflection|
|---|---|
|Compile time|Runtime|
|Fast|Slower|
|Type known|Dynamic|
|Simple|Complex|
|Application code|Framework code|

---

# 25. Interview Questions

## Q1: What is Reflection API?

Answer:

Reflection allows Java programs to inspect and manipulate classes, methods, and fields at runtime.

---

## Q2: Where is Reflection used?

Answer:

- Spring
    
- Hibernate
    
- JUnit
    
- Dependency Injection
    
- ORM
    

---

## Q3: Difference between Annotation and Reflection?

Annotation:

```
Metadata
```

Reflection:

```
Reads metadata at runtime
```

---

## Q4: Why Reflection is slower?

Because it performs runtime lookup and access checks.

---

# Practice Project

Build Mini ORM for Café POS.

Create:

```java
@Table(
name="products"
)

class Product{


@Id
int id;


@Column
String name;


@Column
double price;


}

```

Create:

```java
ORMProcessor

```

Features:

```
Read Class

Read Annotation

Generate SQL

Print SQL

```

Example Output:

```sql
SELECT id,name,price
FROM products;

```

---

# Lesson 11 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Reflection API  
✅ Class metadata  
✅ Field access  
✅ Method invocation  
✅ Dynamic object creation  
✅ Annotation concept  
✅ Custom Annotation  
✅ Runtime Annotation  
✅ Retention Policy  
✅ Target  
✅ Spring/Hibernate internal working  
✅ Café POS framework idea

---

# Next Lesson

# Lesson 12: Java Multithreading Advanced

## Building Background Processing for Swing POS

သင်ယူမည့်အရာ:

- Thread Lifecycle
    
- Runnable
    
- Callable
    
- Future
    
- ExecutorService
    
- Thread Pool
    
- Synchronization
    
- Atomic Variables
    
- BlockingQueue
    
- Producer Consumer Pattern
    

Café POS Example:

```
Swing UI Thread

        |

Order Processing Thread

        |

Database Thread

        |

Receipt Printing Thread

```

ပြီးရင် Swing Application ကို Professional Level နဲ့ Thread Safe ဖြစ်အောင် တည်ဆောက်ပါမယ်။