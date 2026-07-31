# Part 3: Café POS Real Implementation Phase

# Lesson 38: Advanced Product Management Module (Part 3)

## Search + Validation + Category CRUD + Image Upload + Product Recipe Integration

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Product Module ကို **Basic CRUD ကနေ Professional POS Product System** အဖြစ် upgrade လုပ်ပါမယ်။

အခုအထိ:

```text
Product Module

Database        ✅
Entity          ✅
Repository      ✅
Service         ✅
Controller      ✅
JTable UI       ✅
CRUD            ✅
```

ရှိပြီးပါပြီ။

အခု:

```text
Product System

        +

Category Management

        +

Validation

        +

Search

        +

Image

        +

Recipe Integration

```

လုပ်ပါမယ်။

---

# 1. Professional Product Module Architecture

Final Design:

```text
                 ProductPanel
                      |
                      |
              ProductController
                      |
                      |
              ProductService
                      |
        -------------------------
        |                       |
 ProductRepository     CategoryRepository

        |
        |

     MySQL Database

```

---

# 2. Product Feature Requirements

Café POS Product:

Example:

```
Product:

Coffee Latte


Category:

Coffee


Price:

5000 MMK


Image:

latte.png


Recipe:

Coffee Bean 20g

Milk 100ml

Sugar 5g

```

---

# 3. Add Image Support

Database update:

products table:

```sql
ALTER TABLE products

ADD COLUMN image_path VARCHAR(255);

```

---

Why store path?

Wrong:

```
Image Binary inside Database

```

Problem:

- Database size ကြီး
    
- Backup slow
    
- Query slow
    

---

Professional:

```
Database

products

id
name
image_path


File System

/images/products

latte.png

```

---

# 4. Create Product Entity Update

Before:

```java
public record Product(

Long id,

Category category,

String name,

double price,

boolean active

){}
```

---

After:

```java
public record Product(

Long id,

Category category,

String name,

double price,

String imagePath,

boolean active

){

}

```

---

# 5. Image Upload Service

Create:

```
common

 └── FileService.java

```

---

Purpose:

```
Copy image

Generate filename

Return path

```

---

Code:

```java
package com.cafe.pos.common;


import java.io.*;

import java.nio.file.*;



public class FileService {


private FileService(){}



public static String saveImage(
File source
){


try{


Path folder =
Paths.get(
"images/products"
);



Files.createDirectories(
folder
);



Path target =
folder.resolve(
source.getName()
);



Files.copy(
source.toPath(),
target,
StandardCopyOption.REPLACE_EXISTING
);



return target.toString();


}
catch(Exception e){


throw new RuntimeException(
"Image upload failed",
e
);


}


}


}

```

---

# 6. Image Upload UI

Product Dialog:

Add button:

```
Image:

[ Choose File ]



Preview

+-----------+

|           |

| Image     |

|           |

+-----------+

```

---

Swing:

```java
JButton upload =
new JButton(
"Choose Image"
);


upload.addActionListener(e->{


JFileChooser chooser =
new JFileChooser();


int result =
chooser.showOpenDialog(this);



if(result ==
JFileChooser.APPROVE_OPTION){


File file =
chooser.getSelectedFile();


String path =
FileService.saveImage(file);


}


});

```

---

# 7. Product Validation System

Never allow:

```
Name = empty

Price = -500

Category = null

```

---

Create:

```
validation

ProductValidator.java

```

---

Code:

```java
public class ProductValidator {


public static void validate(
Product product
){


if(product.name()
.isBlank()){


throw new ValidationException(
"Product name required"
);


}



if(product.price() <=0){


throw new ValidationException(
"Invalid price"
);


}



if(product.category()==null){


throw new ValidationException(
"Category required"
);


}


}

}

```

---

# 8. Service Layer Update

Before:

```java
repository.save(product);

```

---

After:

```java
ProductValidator.validate(
product
);


repository.save(product);

```

---

Flow:

```
UI

 ↓

Controller

 ↓

Service

 ↓

Validator

 ↓

Repository

 ↓

Database

```

---

# 9. Category Management Module

Why separate?

Because:

```
Product

belongs to

Category

```

---

Example:

```
Coffee

 ├── Latte

 ├── Espresso


Food

 ├── Burger

 ├── Sandwich

```

---

Package:

```
module/category


├── model

├── repository

├── service

├── controller

└── view

```

---

# 10. Category Entity

```java
public record Category(

Long id,

String name

){

}

```

---

# 11. Category Repository

Interface:

```java
public interface CategoryRepository {


void save(Category category);


List<Category> findAll();


void delete(Long id);


}

```

---

# 12. Category CRUD SQL

Create:

```sql
INSERT INTO categories(name)

VALUES(?);

```

---

Read:

```sql
SELECT *

FROM categories;

```

---

Delete:

```sql
DELETE FROM categories

WHERE id=?;

```

---

# 13. Category ComboBox in Product Dialog

Instead of:

```
Category TextField

```

Use:

```
Category:

[ Coffee ▼ ]

```

---

Swing:

```java
JComboBox<Category> categoryBox =
new JComboBox<>();

```

---

Load:

```java
List<Category> categories =
categoryController.findAll();


for(Category c:categories){

categoryBox.addItem(c);

}

```

---

# 14. Product Search

UI:

```
Search:

[ Latte________ ]

```

---

Event:

```java
searchField
.getDocument()
.addDocumentListener(
new DocumentListener(){

public void insertUpdate(
DocumentEvent e
){

search();

}

}
);

```

---

Repository:

```sql
SELECT *

FROM products

WHERE name LIKE ?

```

---

Example:

Input:

```
lat

```

Result:

```
Latte

Chocolate Latte

```

---

# 15. Product Recipe Integration

Important Café Logic.

Example:

Selling:

```
1 Latte

```

Inventory:

```
Coffee Bean -20g

Milk -100ml

Sugar -5g

```

---

Database:

Already have:

```
product_recipes

product_id

ingredient_id

quantity

```

---

Relationship:

```
Product

   |

   |

Recipe

   |

   |

Ingredient

```

---

# 16. Product Details View

Professional POS:

Double click:

```
Latte


Price: 5000


Recipe:

Coffee Bean 20g

Milk 100ml

Sugar 5g


```

---

# 17. Product Module Final Structure

Now:

```
product


├── model

│    Product

│    Category


├── repository

│    ProductRepository

│    CategoryRepository


├── service

│    ProductService

│    CategoryService


├── validation

│    ProductValidator


├── controller

│    ProductController


└── view

     ProductPanel

     ProductDialog

```

---

# 18. Current Project Progress

Completed:

```
Java Setup              ✅

Database Layer          ✅

Migration               ✅

Authentication          ✅

Dashboard               ✅

Product CRUD            ✅

JTable                  ✅

Search                  ✅

Validation              ✅

Category Design         ✅

Image Support           ✅

Recipe Connection       ✅

```

---

# 19. Professional Lessons Learned

Important architecture rule:

UI should NOT know:

```
SQL

Database

File System

Business Rules

```

UI only:

```
Display

Collect Input

Send Event

```

---

# Next Lesson

# Lesson 39: Inventory Management Module (Part 1)

## Ingredient Entity + Stock Management + Inventory Transaction System

Next we build the most important Café POS feature:

```
Ingredient

      ↓

Inventory

      ↓

Stock Movement

      ↓

Recipe Consumption

      ↓

Automatic Stock Deduction

```

ပြီးရင် Order တစ်ခုရောင်းတဲ့အခါ **Inventory အလိုအလျောက် လျော့သွားတဲ့ Real POS Logic** ကို တည်ဆောက်ပါမယ်။