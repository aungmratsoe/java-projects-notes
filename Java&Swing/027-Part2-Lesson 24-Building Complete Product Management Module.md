# Part 2: Advanced Java Swing + Café POS Project

# Lesson 24: Building Complete Product Management Module

## Professional Swing UI + CRUD + JTable + MVC Integration

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ **Café POS ရဲ့ ပထမဆုံး Real Functional Module** ဖြစ်တဲ့

> Product Management System

ကို တည်ဆောက်ပါမယ်။

ဒီတစ်ခါမှာ Database CRUD တင်မဟုတ်ဘဲ **Professional Desktop POS UI Design** ကိုပါ လေ့လာပါမယ်။

---

# 1. Professional Café POS UI Design Goal

Beginner Swing UI:

```
+----------------------+
| Name: [          ]   |
| Price:[          ]   |
|                      |
| [Save] [Delete]      |
|                      |
| JTable               |
+----------------------+

```

ဒါက Learning အတွက်ကောင်းပေမယ့် Production POS အတွက် မလုံလောက်ပါ။

---

Professional POS:

```
+------------------------------------------------+
| ☕ Café POS        User: Admin        Logout   |
+------------------------------------------------+

| Sidebar          | Product Management          |
|                  |                             |
| Dashboard        | Search: [____________] 🔍    |
| Products         |                             |
| Orders           | +-----------------------+   |
| Inventory        | | ID Name Price Status |   |
| Reports          | |----------------------|   |
| Settings         | | 1 Coffee 5000 Active |   |
|                  | +-----------------------+   |
|                  |                             |
|                  | [Add] [Edit] [Delete]      |
+------------------------------------------------+

```

---

# 2. Swing Professional Architecture

Our UI:

```
view

 |
 +-- MainFrame
 |
 +-- ProductPanel
 |
 +-- ProductDialog
 |
 +-- CustomTableModel
 |
 +-- UITheme


```

---

# 3. Add Modern Look Library

Java Swing default Look:

```
Metal UI

```

အရမ်းဟောင်းပါတယ်။

Professional Swing App တွေမှာ:

## FlatLaf

အသုံးများပါတယ်။

---

Maven:

```xml
<dependency>

    <groupId>com.formdev</groupId>

    <artifactId>flatlaf</artifactId>

    <version>3.6</version>

</dependency>
```

---

# 4. Application Theme Setup

Application.java

```java
import com.formdev.flatlaf.FlatLightLaf;


public class Application {


public static void main(String[] args){


FlatLightLaf.setup();


SwingUtilities.invokeLater(() -> {


new MainFrame()
.setVisible(true);


});


}

}

```

---

Result:

- Modern buttons
    
- Better fonts
    
- Better colors
    
- Rounded UI
    

---

# 5. Main Window Design

Create:

```
view

MainFrame.java

```

---

Code:

```java
public class MainFrame extends JFrame {


private JPanel contentPanel;



public MainFrame(){


setTitle(
"Café POS System"
);


setSize(
1200,
700
);


setLocationRelativeTo(null);


setDefaultCloseOperation(
EXIT_ON_CLOSE
);


initUI();


}



private void initUI(){


setLayout(
new BorderLayout()
);


add(
createSidebar(),
BorderLayout.WEST
);


contentPanel =
new JPanel(
new BorderLayout()
);


add(
contentPanel,
BorderLayout.CENTER
);


}



}

```

---

# 6. Sidebar Navigation

Professional POS needs menu.

Create:

```java
private JPanel createSidebar(){


JPanel panel =
new JPanel();


panel.setPreferredSize(
new Dimension(220,0)
);



panel.setLayout(
new GridLayout(8,1,10,10)
);



panel.add(
new JButton("Dashboard")
);


panel.add(
new JButton("Products")
);


panel.add(
new JButton("Orders")
);


panel.add(
new JButton("Inventory")
);



return panel;

}

```

---

UI:

```
+----------------+
| Dashboard      |
| Products       |
| Orders         |
| Inventory      |
| Reports        |
+----------------+

```

---

# 7. Product Management Panel

Structure:

```
ProductPanel


Header

Search Bar


Table


Action Buttons

```

---

Create:

```
view/product

ProductPanel.java

```

---

# 8. Product Panel Code

```java
public class ProductPanel extends JPanel {


private JTable table;


private ProductController controller;



public ProductPanel(
ProductController controller
){


this.controller=controller;


initUI();


loadData();


}


}

```

---

# 9. Header Section

```java
private JPanel createHeader(){


JPanel panel =
new JPanel(
new BorderLayout()
);



JLabel title =
new JLabel(
"Product Management"
);



title.setFont(
new Font(
"Arial",
Font.BOLD,
24
)
);



panel.add(
title,
BorderLayout.WEST
);



return panel;


}

```

---

# 10. Search Bar

POS systems need fast search.

```java
JTextField searchField =
new JTextField();


searchField.putClientProperty(
"JTextField.placeholderText",
"Search product..."
);

```

---

Result:

```
Search product...

```

---

# 11. JTable Professional Setup

Default JTable:

```
ugly

```

Need customization.

```java
table =
new JTable();


table.setRowHeight(
35
);


table.setFont(
new Font(
"Arial",
Font.PLAIN,
14
)
);


table.getTableHeader()
.setFont(
new Font(
"Arial",
Font.BOLD,
14
)
);

```

---

# 12. Product Table Columns

```
ID | Name | Category | Price | Status


```

---

Create Model:

```java
DefaultTableModel model =
new DefaultTableModel(
new Object[]{
"ID",
"Name",
"Price",
"Status"
},
0
);


table.setModel(model);

```

---

# 13. Loading Database Data

Controller:

```java
public List<ProductDTO> findAll(){

return service.findAll();

}

```

---

Panel:

```java
private void loadData(){


List<ProductDTO> products =
controller.findAll();


model.setRowCount(0);



for(ProductDTO p:products){


model.addRow(
new Object[]{

p.id(),

p.name(),

p.price(),

p.active()

}

);


}


}

```

---

# 14. Button Panel

Professional buttons:

```
[ + Add ]

[ ✏ Edit ]

[ 🗑 Delete ]

[ Refresh ]

```

---

Code:

```java
JButton add =
new JButton(
"Add"
);


JButton edit =
new JButton(
"Edit"
);


JButton delete =
new JButton(
"Delete"
);

```

---

# 15. Product Dialog

Instead of another window:

```
Product Form


Name:

[ Coffee          ]


Price:

[ 5000            ]


[Save] [Cancel]

```

---

Create:

```
ProductDialog.java

```

---

Fields:

```java
private JTextField nameField;

private JTextField priceField;

```

---

# 16. Add Product

Flow:

```
User

 |

Click Add

 |

Dialog

 |

Controller

 |

Service

 |

Repository

 |

MySQL

```

---

Button:

```java
saveButton.addActionListener(e->{


controller.save(
createProduct()
);


dispose();


});

```

---

# 17. Validation

Before save:

```java
if(name.isBlank()){


throw new ValidationException(

"Product name required"

);


}

```

---

Price:

```java
if(price.compareTo(
BigDecimal.ZERO
)<=0){


throw new ValidationException(
"Invalid price"
);


}

```

---

# 18. Global Exception Handler in Swing

Never:

```java
catch(Exception e){

e.printStackTrace();

}

```

---

Use:

```java
try{


controller.save(product);


}
catch(AppException e){


GlobalExceptionHandler.handle(e);

}

```

---

User:

```
❌ Product price is invalid

```

Developer:

```
INV_001

Stack Trace

Log

```

---

# 19. Delete Product

Select row:

```java
int row =
table.getSelectedRow();

```

---

Get ID:

```java
Long id =
(Long)table.getValueAt(
row,
0
);

```

---

Confirm:

```java
int result =
JOptionPane.showConfirmDialog(
this,
"Delete Product?"
);

```

---

Delete:

```java
controller.delete(id);

```

---

# 20. Database Operation Threading

Wrong:

```
Click Save

 |

Database

 |

UI freezes

```

---

Correct:

```
Swing EDT

 |

Virtual Thread

 |

Database

```

---

Example:

```java
CompletableFuture.runAsync(() -> {


controller.save(product);


})

.thenRun(() -> {


SwingUtilities.invokeLater(() -> {


loadData();


});


});

```

---

# 21. Product Module Complete Flow

```
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

# 22. Professional UI Features to Add

Future improvements:

## JTable

✅ Sorting

✅ Filtering

✅ Pagination

## Form

✅ Validation

✅ Auto complete

✅ Image upload

## UX

✅ Loading indicator

✅ Toast notification

✅ Keyboard shortcut

---

# 23. Café POS Color Theme Example

Professional colors:

```
Primary:

Coffee Brown


Accent:

Orange


Background:

Light Gray


Success:

Green


Danger:

Red

```

---

# 24. Keyboard Shortcuts

POS cashier speed is important.

Example:

```
Ctrl + N

New Product


Ctrl + S

Save


Delete

Remove

```

---

# 25. Final Product Module Structure

```
product

├── ProductPanel.java

├── ProductDialog.java

├── ProductTableModel.java

├── ProductController.java

├── ProductService.java

├── ProductRepository.java

└── ProductDTO.java

```

---

# 26. Senior Developer Checklist

Product Module:

Architecture:

✅ MVC

✅ Service Layer

✅ Repository

Database:

✅ CRUD

✅ PreparedStatement

UI:

✅ FlatLaf

✅ JTable

✅ Dialog

✅ Validation

Performance:

✅ Async Database Call

Error:

✅ Global Handler

---

# Practice Task

Build:

## Product Management Screen

Features:

1. Display Products
    
2. Search Product
    
3. Add Product
    
4. Edit Product
    
5. Delete Product
    
6. Refresh Data
    
7. Database Integration
    
8. Exception Handling
    
9. Professional UI Theme
    

---

# Lesson 24 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Professional Swing UI Design  
✅ FlatLaf Theme  
✅ MainFrame Architecture  
✅ Sidebar Navigation  
✅ Product Management Panel  
✅ JTable Customization  
✅ Product Dialog  
✅ MVC Integration  
✅ CRUD UI Flow  
✅ Validation  
✅ Exception Handling in UI  
✅ Async Database Operation

---

# Next Lesson

# Lesson 25: Café POS Order Management Module

## Shopping Cart + Transaction + Inventory Update

Topics:

- Order UI Design
    
- Product Search
    
- Cart System
    
- JTable Cart
    
- Order Transaction
    
- Stock Deduction
    
- Receipt Generation
    
- Database Transaction Handling
    

ဒီ Lesson ပြီးရင် Café POS မှာ **တကယ်ရောင်းချနိုင်တဲ့ Checkout System** စတင်ရပါမယ်။