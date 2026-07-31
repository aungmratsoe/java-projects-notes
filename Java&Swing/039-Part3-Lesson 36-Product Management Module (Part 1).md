# Part 3: Café POS Real Implementation Phase

# Lesson 36: Product Management Module (Part 1)

## Product Entity + Repository + Service + CRUD Backend Architecture

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson ကနေ Café POS ရဲ့ **ပထမဆုံး Business Module** ကို စတင်ပါမယ်။

အခုအထိ ကျွန်တော်တို့မှာ:

```text
Application
      |
 Login
      |
 Dashboard
      |
 Navigation
```

ရှိပြီးပါပြီ။

အခုကနေ:

```text
Product Management

Database

    ↓

Entity

    ↓

Repository

    ↓

Service

    ↓

Controller

    ↓

Swing JTable UI

```

တည်ဆောက်ပါမယ်။

---

# 1. Product Module Requirements

Café POS မှာ Product ဆိုတာ:

ဥပမာ:

|Product|Category|Price|
|---|---|---|
|Coffee Latte|Coffee|5000|
|Cappuccino|Coffee|6000|
|Burger|Food|8000|
|Cake|Dessert|4000|

လိုမျိုး ဖြစ်ပါတယ်။

---

# Product Features

ဒီ Module မှာ:

✅ Create Product  
✅ Read Product List  
✅ Update Product  
✅ Delete Product  
✅ Search Product  
✅ Category Assignment  
✅ Active/Inactive Status

လုပ်ပါမယ်။

---

# 2. Product Module Architecture

Professional MVC:

```text
                 ProductPanel
                     |
                     |
              ProductController
                     |
                     |
              ProductService
                     |
                     |
          ProductRepository Interface
                     |
                     |
          MySQLProductRepository
                     |
                     |
                 MySQL


```

---

# 3. Package Structure

Create:

```text
module

└── product

    ├── model

    │    ├── Product.java

    │    └── Category.java


    ├── repository

    │    ├── ProductRepository.java

    │    └── MySQLProductRepository.java


    ├── service

    │    └── ProductService.java


    ├── controller

    │    └── ProductController.java


    └── view

         └── ProductPanel.java

```

---

# 4. Category Entity

Database:

```sql
categories

id

name

```

---

Create:

`Category.java`

```java
package com.cafe.pos.module.product.model;


public record Category(

Long id,

String name

){

}

```

---

# 5. Product Entity

Database:

```sql
products


id

category_id

name

price

active

created_at

```

---

Create:

`Product.java`

```java
package com.cafe.pos.module.product.model;


public record Product(

Long id,

Category category,

String name,

double price,

boolean active

){

}

```

---

# 6. Why Use Record?

Product object:

```java
Product p =
new Product(
1L,
coffeeCategory,
"Latte",
5000,
true
);

```

Immutable:

```text
id cannot change

name cannot change

price cannot change

```

---

# 7. Repository Layer

Repository Responsibility:

Only Database communication.

NOT:

❌ Validation

❌ UI

❌ Business rules

---

Interface:

`ProductRepository.java`

```java
package com.cafe.pos.module.product.repository;


import java.util.List;

import com.cafe.pos.module.product.model.Product;


public interface ProductRepository {


void save(Product product);


List<Product> findAll();


Product findById(Long id);


void update(Product product);


void delete(Long id);


}

```

---

# 8. CRUD Meaning

CRUD:

|Operation|Meaning|
|---|---|
|C|Create|
|R|Read|
|U|Update|
|D|Delete|

---

# 9. MySQL Product Repository

Create:

`MySQLProductRepository.java`

```java
package com.cafe.pos.module.product.repository;


import java.sql.*;

import java.util.*;

import com.cafe.pos.database.DatabaseManager;

import com.cafe.pos.module.product.model.*;



public class MySQLProductRepository
implements ProductRepository {



@Override
public void save(Product product){


String sql="""

INSERT INTO products

(category_id,name,price,active)

VALUES(?,?,?,?)

""";


try(Connection con =
DatabaseManager.getConnection();

PreparedStatement ps =
con.prepareStatement(sql)){


ps.setLong(
1,
product.category().id()
);


ps.setString(
2,
product.name()
);


ps.setDouble(
3,
product.price()
);


ps.setBoolean(
4,
product.active()
);



ps.executeUpdate();


}
catch(Exception e){

throw new RuntimeException(e);

}


}


}

```

---

# 10. Repository Flow

When creating product:

```text
ProductService

      |

ProductRepository.save()

      |

SQL INSERT

      |

MySQL products table

```

---

# 11. Service Layer

Why Service?

Business rules live here.

Example:

Product price cannot:

```text
<= 0

```

---

Create:

`ProductService.java`

```java
package com.cafe.pos.module.product.service;


import com.cafe.pos.module.product.model.Product;

import com.cafe.pos.module.product.repository.ProductRepository;



public class ProductService {



private final ProductRepository repository;



public ProductService(
ProductRepository repository
){

this.repository =
repository;

}



public void create(
Product product
){



if(product.price() <= 0){

throw new RuntimeException(
"Price must be greater than zero"
);

}



repository.save(product);


}


}

```

---

# 12. Service Responsibility

Example:

Input:

```text
Product Name:
Latte

Price:
-500

```

Controller:

```text
send to Service

```

Service:

```java
if(price <=0)

reject

```

Database:

```text
Never receives invalid data

```

---

# 13. Custom Validation Exception

Create:

```text
exception

ValidationException.java

```

---

Code:

```java
package com.cafe.pos.exception;


public class ValidationException
extends RuntimeException {


public ValidationException(
String message
){

super(message);

}


}

```

---

Replace:

```java
throw new RuntimeException();

```

with:

```java
throw new ValidationException(
"Product price invalid"
);

```

---

# 14. Product Controller

Controller connects UI and Service.

Create:

`ProductController.java`

```java
package com.cafe.pos.module.product.controller;


import com.cafe.pos.module.product.model.Product;

import com.cafe.pos.module.product.service.ProductService;



public class ProductController {


private final ProductService service;



public ProductController(
ProductService service
){

this.service =
service;

}



public void createProduct(
Product product
){


service.create(product);


}


}

```

---

# 15. Dependency Injection

Application:

```java
ProductRepository repo =
new MySQLProductRepository();



ProductService service =
new ProductService(repo);



ProductController controller =
new ProductController(service);

```

---

Flow:

```text
Application

 creates

Controller

      ↓

Service

      ↓

Repository

```

---

# 16. Database Test

Insert:

```java
Category coffee =
new Category(
1L,
"Coffee"
);


Product latte =
new Product(
null,
coffee,
"Latte",
5000,
true
);


controller.createProduct(latte);

```

---

Database:

```text
products


id | name | price

1     Latte  5000

```

---

# 17. Product Module Status

Completed:

```text
Product Entity          ✅

Category Entity         ✅

Repository Design       ✅

Create Product Backend  ✅

Service Layer           ✅

Controller Layer        ✅

Validation              ✅

```

---

# 18. Professional Rule

Never:

```java
button.addActionListener(e->{


INSERT INTO products...


});

```

Wrong ❌

---

Correct:

```text
Button

 ↓

Controller

 ↓

Service

 ↓

Repository

 ↓

Database

```

---

# Next Lesson

# Lesson 37: Product Management Module (Part 2)

## Complete CRUD + JTable + Professional Swing UI Integration

Next we build:

✅ Product JTable  
✅ Table Model  
✅ Add Product Dialog  
✅ Edit Product  
✅ Delete Confirmation  
✅ Search Box  
✅ Controller Binding  
✅ FlatLaf Professional POS UI

ပြီးရင် Dashboard ထဲက **Products Menu ကို နှိပ်ပြီး Product Management တကယ်အသုံးပြုနိုင်ပါမယ်။**