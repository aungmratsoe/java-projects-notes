# Part 3: Café POS Real Implementation Phase

# Lesson 37: Product Management Module (Part 2)

## Complete CRUD + Swing JTable + MVC Integration

### (Java 25 + Swing + FlatLaf + MVC + JDBC)

ဒီ Lesson မှာ Product Module ကို **Backend ကနေ Frontend အထိ ချိတ်ဆက်ပြီး Complete Feature** လုပ်ပါမယ်။

အခုအထိ:

```text
ProductPanel
       ❌

Product Backend
       ✅

Database
       ✅
```

ရှိပါတယ်။

ဒီနေ့ပြီးရင်:

```text
Dashboard

   ↓

Products Menu

   ↓

Product Management Screen

   ↓

JTable

   ↓

Add / Edit / Delete / Search

   ↓

MySQL Database

```

အထိ အလုပ်လုပ်ပါမယ်။

---

# 1. Product UI Architecture

Swing MVC Flow:

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
              ProductRepository
                      |
                      |
                   MySQL

```

---

# 2. Product View Structure

Create:

```text
module/product/view


ProductPanel.java

ProductTableModel.java

ProductDialog.java

```

---

# 3. Why JTable Model?

Beginner:

```java
table.addRow(
new Object[]{
"id",
"name",
"price"
}
);

```

Problem:

- Data control မရှိ
    
- Refresh ခက်
    
- MVC မဖြစ်
    

---

Professional:

```text
Database

   ↓

List<Product>

   ↓

TableModel

   ↓

JTable

```

---

# 4. Create ProductTableModel

File:

```text
ProductTableModel.java

```

---

Code:

```java
package com.cafe.pos.module.product.view;


import javax.swing.table.AbstractTableModel;

import java.util.*;

import com.cafe.pos.module.product.model.Product;



public class ProductTableModel
extends AbstractTableModel {


private final String[] columns =
{
"ID",
"Name",
"Category",
"Price",
"Status"
};



private List<Product> products =
new ArrayList<>();



public void setProducts(
List<Product> products
){

this.products = products;

fireTableDataChanged();

}



@Override
public int getRowCount(){

return products.size();

}



@Override
public int getColumnCount(){

return columns.length;

}



@Override
public String getColumnName(
int column
){

return columns[column];

}



@Override
public Object getValueAt(
int row,
int column
){


Product product =
products.get(row);


return switch(column){

case 0 -> product.id();

case 1 -> product.name();

case 2 -> product.category().name();

case 3 -> product.price();

case 4 -> product.active()
? "Active"
: "Inactive";

default -> null;

};


}


}

```

---

# 5. ProductPanel Design

UI:

```
+------------------------------------------------+

 Product Management


 Search: [____________] [Search]


 +--------------------------------------------+

 | ID | Name | Category | Price | Status       |

 |--------------------------------------------|

 | 1  | Latte| Coffee   |5000   | Active       |

 | 2  | Cake | Dessert  |4000   | Active       |

 +--------------------------------------------+


 [Add] [Edit] [Delete]


+------------------------------------------------+

```

---

# 6. Create ProductPanel

```java
package com.cafe.pos.module.product.view;


import javax.swing.*;

import java.awt.*;



public class ProductPanel
extends JPanel {



private JTable table;


private ProductTableModel model;



private JButton addButton;

private JButton editButton;

private JButton deleteButton;



public ProductPanel(){


setLayout(
new BorderLayout()
);



initialize();


}



private void initialize(){


model =
new ProductTableModel();


table =
new JTable(model);



addButton =
new JButton("Add");


editButton =
new JButton("Edit");


deleteButton =
new JButton("Delete");



JPanel top =
new JPanel(
new FlowLayout()
);


top.add(
new JLabel("Search")
);


top.add(
new JTextField(20)
);



add(
top,
BorderLayout.NORTH
);



add(
new JScrollPane(table),
BorderLayout.CENTER
);



JPanel bottom =
new JPanel();



bottom.add(addButton);

bottom.add(editButton);

bottom.add(deleteButton);



add(
bottom,
BorderLayout.SOUTH
);



}


}

```

---

# 7. Add Product Dialog

Separate Window:

Why?

Main Panel clean ဖြစ်စေဖို့။

Structure:

```
ProductPanel

      |

ProductDialog

      |

Input Data

```

---

Create:

```text
ProductDialog.java

```

---

UI:

```
+--------------------+

 Product


 Name:

 [________]


 Price:

 [________]


 Category:

 [Coffee ▼]


 [SAVE]


+--------------------+

```

---

Code:

```java
public class ProductDialog
extends JDialog {


private JTextField nameField;

private JTextField priceField;



public ProductDialog(
JFrame parent
){

super(
parent,
"Add Product",
true
);



setSize(
400,
300
);



setLocationRelativeTo(
parent
);



initialize();


}



private void initialize(){


nameField =
new JTextField();



priceField =
new JTextField();



JButton save =
new JButton(
"Save"
);



setLayout(
new GridLayout(3,2)
);



add(
new JLabel("Name")
);


add(nameField);


add(
new JLabel("Price")
);


add(priceField);


add(save);


}


}

```

---

# 8. Product Service Update

Need Read All:

Interface:

```java
List<Product> findAll();

```

---

Service:

```java
public List<Product> getProducts(){


return repository.findAll();


}

```

---

# 9. Controller Update

Add:

```java
public List<Product> loadProducts(){


return service.getProducts();


}

```

---

# 10. Loading JTable Data

ProductPanel:

```java
private void loadData(){


List<Product> products =
controller.loadProducts();



model.setProducts(
products
);


}

```

---

Call:

```java
initialize();

loadData();

```

---

# 11. Delete Flow

User clicks:

```text
Delete Button

       ↓

Selected Row

       ↓

Confirm Dialog

       ↓

Controller

       ↓

Service

       ↓

Repository

       ↓

DELETE SQL

```

---

Confirmation:

```java
int result =
JOptionPane.showConfirmDialog(
this,
"Delete Product?"
);


if(result ==
JOptionPane.YES_OPTION){

// delete

}

```

---

# 12. JTable Professional Settings

Add:

```java
table.setRowHeight(35);


table.setAutoCreateRowSorter(
true
);


table.setSelectionMode(
ListSelectionModel.SINGLE_SELECTION
);

```

---

Benefits:

```
✔ Sort columns

✔ Better UI

✔ Single selection

```

---

# 13. Search Feature

Architecture:

```
Search Box

    ↓

Controller

    ↓

Repository

    ↓

SQL LIKE


```

---

SQL:

```sql
SELECT *

FROM products

WHERE name LIKE ?

```

Example:

```text
%lat%

```

Result:

```text
Latte

Chocolate Latte

```

---

# 14. Connect Dashboard

Sidebar:

Before:

```java
addButton(
"Products"
);

```

After:

```java
productsButton
.addActionListener(e->{


mainFrame.changeContent(

new ProductPanel()

);


});

```

---

# 15. Complete Product Flow

Now:

```
User

 ↓

ProductPanel

 ↓

Controller

 ↓

Service

 ↓

Repository

 ↓

HikariCP

 ↓

MySQL


```

---

# 16. Product Module Status

Completed:

```
Database Table          ✅

Entity                  ✅

Repository              ✅

Service                 ✅

Controller              ✅

JTable                  ✅

TableModel              ✅

Dialog UI               ✅

CRUD Architecture       ✅


```

---

# 17. Production Improvements (Next)

Real POS needs:

- Pagination
    
- Image Upload
    
- Barcode
    
- Category Management
    
- Price History
    
- Stock Integration
    
- Validation Messages
    
- Loading Animation
    

---

# Practice Task

Implement:

1. Add ProductPanel to Dashboard
    
2. Create ProductTableModel
    
3. Display products from database
    
4. Create Add Dialog
    
5. Connect Save Button
    
6. Test CRUD
    

---

# Next Lesson

# Lesson 38: Product Module (Part 3)

## Advanced Product Management

### Search + Validation + Category CRUD + Image Upload + Inventory Integration

Next we upgrade Product Module to a **Real Café POS Product System**:

✅ Category Management  
✅ Product Search  
✅ Input Validation  
✅ Image Handling  
✅ Price Rules  
✅ Product ↔ Inventory Recipe Connection

ပြီးရင် Inventory Module ကို စတင်ပါမယ်။