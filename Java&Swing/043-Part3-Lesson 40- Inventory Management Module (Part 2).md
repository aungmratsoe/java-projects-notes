# Part 3: Café POS Real Implementation Phase

# Lesson 40: Inventory Management Module (Part 2)

## Complete Stock CRUD + JTable + Stock Adjustment + Low Stock Alert

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Inventory Module ကို **Backend Design ကနေ Professional Swing UI အထိ** ဆက်တည်ဆောက်ပါမယ်။

အခုအထိ:

```text
Inventory Module

Ingredient Entity        ✅
Inventory Entity         ✅
Stock Movement Entity    ✅
Repository Design        ✅
Service Foundation       ✅
```

ပြီးပါပြီ။

ဒီ Lesson ပြီးရင်:

```text
Dashboard

   ↓

Inventory Menu

   ↓

Inventory Management Screen

   ↓

View Stock

   ↓

Add Stock

   ↓

Adjust Stock

   ↓

Low Stock Alert

```

အထိ အလုပ်လုပ်ပါမယ်။

---

# 1. Inventory UI Architecture

Professional Flow:

```text
                 InventoryPanel

                       |

                       |

             InventoryController

                       |

                       |

              InventoryService

                       |

        --------------------------------

        |                              |

InventoryRepository        StockMovementRepository

                       |

                       |

                    MySQL

```

---

# 2. Inventory View Package

Create:

```text
module/inventory/view


├── InventoryPanel.java

├── InventoryTableModel.java

├── StockDialog.java

└── StockHistoryDialog.java

```

---

# 3. Inventory Table Design

UI:

```text
+------------------------------------------------+

 Inventory Management


 Search:

 [ Coffee____________ ]


+------------------------------------------------+

| Ingredient | Unit | Quantity | Status          |

|------------------------------------------------|

| Coffee Bean| gram | 5000     | NORMAL          |

| Milk       | ml   | 300      | LOW STOCK       |

| Sugar      | gram | 2000     | NORMAL          |

+------------------------------------------------+


[ Add Stock ]

[ Adjustment ]

[ History ]


+------------------------------------------------+

```

---

# 4. InventoryTableModel

Create:

```text
InventoryTableModel.java

```

---

Code:

```java
package com.cafe.pos.module.inventory.view;


import javax.swing.table.AbstractTableModel;

import java.util.*;

import com.cafe.pos.module.inventory.model.Inventory;



public class InventoryTableModel
extends AbstractTableModel {



private final String[] columns =
{
"Ingredient",
"Unit",
"Quantity",
"Status"
};



private List<Inventory> inventories =
new ArrayList<>();



public void setData(
List<Inventory> inventories
){

this.inventories =
inventories;


fireTableDataChanged();

}



@Override
public int getRowCount(){

return inventories.size();

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


Inventory inventory =
inventories.get(row);



return switch(column){


case 0 ->
inventory.ingredient()
.name();


case 1 ->
inventory.ingredient()
.unit();


case 2 ->
inventory.quantity();


case 3 ->
inventory.quantity()
<
inventory.ingredient()
.minimumStock()
?
"LOW STOCK"
:
"NORMAL";


default ->
null;

};


}


}

```

---

# 5. InventoryPanel

Create:

```text
InventoryPanel.java

```

---

Code:

```java
package com.cafe.pos.module.inventory.view;


import javax.swing.*;

import java.awt.*;



public class InventoryPanel
extends JPanel {



private JTable table;


private InventoryTableModel model;



public InventoryPanel(){


setLayout(
new BorderLayout()
);



initialize();


}



private void initialize(){


model =
new InventoryTableModel();


table =
new JTable(model);



table.setRowHeight(35);



JButton addStock =
new JButton(
"Add Stock"
);



JButton adjust =
new JButton(
"Adjustment"
);



JButton history =
new JButton(
"History"
);



JPanel bottom =
new JPanel();



bottom.add(addStock);

bottom.add(adjust);

bottom.add(history);



add(
new JScrollPane(table),
BorderLayout.CENTER
);



add(
bottom,
BorderLayout.SOUTH
);



}



}

```

---

# 6. Load Inventory Data

Controller:

Create:

```text
InventoryController.java

```

---

Code:

```java
public class InventoryController {



private final InventoryService service;



public InventoryController(
InventoryService service
){

this.service =
service;

}



public List<Inventory> getInventory(){

return service.getInventory();

}


}

```

---

# 7. Service Layer

Add:

```java
public List<Inventory> getInventory(){

return inventoryRepository.findAll();

}

```

---

# 8. Display JTable Data

InventoryPanel:

```java
private void loadData(){


List<Inventory> data =
controller.getInventory();


model.setData(data);


}

```

---

Call:

```java
public InventoryPanel(){


initialize();


loadData();


}

```

---

# 9. Add Stock Feature

Real World Example:

Supplier delivers:

```text
Coffee Bean

+5000 gram

```

System:

```text
Inventory

5000 → 10000


Stock Movement

PURCHASE +5000

```

---

# 10. Stock Dialog UI

Design:

```text
+-------------------------+

 Add Stock


 Ingredient:

 [Coffee Bean ▼]


 Quantity:

 [5000]


       [SAVE]


+-------------------------+

```

---

Create:

```text
StockDialog.java

```

---

Fields:

```java
JComboBox<Ingredient> ingredientBox;


JTextField quantityField;


JButton saveButton;

```

---

# 11. Add Stock Flow

```text
Save Button


    ↓


Validate Quantity


    ↓


Controller


    ↓


InventoryService.increaseStock()


    ↓


Database Transaction


    ↓


Refresh JTable


```

---

# 12. Stock Adjustment

Example:

Physical Count:

System:

```text
Coffee Bean

5000g

```

Actual:

```text
4500g

```

Adjustment:

```text
-500g

```

---

Need:

```java
decreaseStock()

```

---

Service:

```java
public void decreaseStock(

Ingredient ingredient,

double amount

){


Inventory inventory =
repository.findByIngredientId(
ingredient.id()
);



if(
inventory.quantity()
<
amount
){

throw new RuntimeException(
"Not enough stock"
);

}



double newQuantity =
inventory.quantity()
-
amount;



repository.updateQuantity(

ingredient.id(),

newQuantity

);



}

```

---

# 13. Stock Movement Type

Adjustment:

```java
MovementType.ADJUSTMENT

```

---

Waste:

```java
MovementType.WASTE

```

---

Sale:

```java
MovementType.SALE

```

---

# 14. Stock History View

Why?

Manager wants:

```text
Who changed stock?

When?

Why?

How much?

```

---

UI:

```text
+--------------------------------+

Coffee Bean History


Date       Type       Qty


10:00      PURCHASE   +5000


12:00      SALE       -40


15:00      WASTE      -100


+--------------------------------+

```

---

# 15. Low Stock Alert System

Rule:

```text
Current Quantity

        <

Minimum Stock


        =


LOW STOCK

```

---

Example:

Ingredient:

```text
Milk

Current:
300 ml


Minimum:
1000 ml

```

Result:

```text
⚠ LOW STOCK

```

---

# 16. Notification

Simple:

```java
if(
quantity < minimumStock
){

JOptionPane.showMessageDialog(
this,
"Low Stock Warning"
);

}

```

---

# 17. Database Transaction

Important:

Add Stock:

```text
BEGIN


UPDATE inventory


INSERT stock_movements


COMMIT


```

---

Failure:

```text
ROLLBACK

```

---

# 18. Connect Dashboard Sidebar

Before:

```java
addButton(
"Inventory"
);

```

After:

```java
inventoryButton
.addActionListener(e->{


mainFrame.changeContent(

new InventoryPanel()

);


});

```

---

# 19. Inventory Module Status

Completed:

```text
Ingredient Model          ✅

Inventory Model           ✅

Stock Movement            ✅

Inventory JTable           ✅

Stock View                 ✅

Add Stock Design           ✅

Adjustment Logic           ✅

Low Stock Detection        ✅

History Design             ✅

```

---

# 20. Important POS Concept

Product ≠ Stock

Example:

Product:

```text
Latte

```

Stock:

```text
Coffee Bean
Milk
Sugar

```

---

Relationship:

```text
Product

    |

Product Recipe

    |

Ingredient

    |

Inventory

```

---

# Practice Task

Implement:

1. InventoryPanel
    
2. InventoryTableModel
    
3. InventoryController
    
4. Load Inventory JTable
    
5. Add Stock Dialog
    
6. Stock Adjustment
    

---

# Next Lesson

# Lesson 41: Inventory Management Module (Part 3)

## Automatic Stock Deduction with Product Recipe + Transaction Management

Next we implement the real Café POS logic:

```text
Customer Order

       ↓

Order Completed

       ↓

Read Product Recipe

       ↓

Calculate Ingredients

       ↓

Reduce Inventory

       ↓

Create Stock Movement

```

ဒီအဆင့်ပြီးရင် Café POS က **အမှန်တကယ် Restaurant System Logic** စတင်ဖြစ်လာပါမယ်။