# Part 3: Café POS Real Implementation Phase

# Lesson 53: Inventory Management Advanced Part 4

## Complete Swing UI Integration + MVC Controller + Real Database Binding

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Supplier / Purchase Order / Stock Receiving Module တွေကို **Database နဲ့ တကယ်ချိတ်ထားတဲ့ Professional Swing Application** ဖြစ်အောင် တည်ဆောက်ပါမယ်။

အခုအထိ:

```text
Supplier Model                 ✅
Supplier Repository            ✅
Purchase Order Design          ✅
JDBC Transaction Layer         ✅
Stock Receiving Logic          ✅
```

ရှိပြီးပါပြီ။

ဒီနေ့မှာ:

```text
Swing JTable

      ↓

TableModel

      ↓

Controller

      ↓

Service

      ↓

Repository

      ↓

MySQL

```

Architecture ကို အပြည့်အဝ Connect လုပ်ပါမယ်။

---

# 1. Why Swing MVC Integration?

Beginner Swing Code:

```java
button.addActionListener(e -> {

    database.save();

});

```

Problem:

```
❌ UI knows database
❌ Hard to test
❌ Difficult maintenance
❌ Code becomes huge
```

---

Professional:

```
SupplierPanel

      |
      |
SupplierController

      |
      |
SupplierService

      |
      |
SupplierRepository

      |
      |
MySQL
```

---

# 2. Final Supplier Module Architecture

Complete:

```
module/inventory/supplier


model

 Supplier.java


repository

 SupplierRepository.java
 SupplierRepositoryImpl.java


service

 SupplierService.java


controller

 SupplierController.java


view

 SupplierPanel.java
 SupplierTableModel.java

```

---

# PART 1

# Supplier Controller

---

# 3. Controller Responsibility

Controller က:

```
UI Event

   ↓

Call Service

   ↓

Return Result

   ↓

Update UI

```

Controller မလုပ်သင့်တာ:

```
❌ SQL

❌ Connection

❌ JTable Logic

```

---

# 4. Create SupplierController

```java
package com.cafe.pos.module.inventory.supplier.controller;


public class SupplierController {


    private final SupplierService service;


    public SupplierController(
            SupplierService service
    ){

        this.service = service;

    }



    public void save(
            Supplier supplier
    ){

        service.create(supplier);

    }



    public List<Supplier> getSuppliers(){

        return service.findAll();

    }



    public void delete(
            Long id
    ){

        service.delete(id);

    }


}
```

---

# 5. Dependency Flow

Object Creation:

```
SupplierRepositoryImpl


        ↓


SupplierService


        ↓


SupplierController


        ↓


SupplierPanel

```

---

Example:

```java
SupplierRepository repo =
new SupplierRepositoryImpl(dataSource);



SupplierService service =
new SupplierService(repo);



SupplierController controller =
new SupplierController(service);


```

---

# PART 2

# JTable Professional Binding

---

# 6. Why Not DefaultTableModel?

Beginner:

```java
DefaultTableModel model =
new DefaultTableModel();

```

Problem:

```
❌ Business object မရှိ

❌ Type safety မရှိ

❌ Maintenance ခက်

```

---

Professional:

```
SupplierTableModel

extends AbstractTableModel

```

---

# 7. Create SupplierTableModel

```java
public class SupplierTableModel

extends AbstractTableModel {



private final String[] columns =
{
"ID",
"Name",
"Phone",
"Email"
};



private List<Supplier> suppliers =
new ArrayList<>();



public void setData(
List<Supplier> suppliers
){

this.suppliers = suppliers;

fireTableDataChanged();

}


@Override
public int getRowCount(){

return suppliers.size();

}



@Override
public int getColumnCount(){

return columns.length;

}



@Override
public Object getValueAt(
int row,
int column
){


Supplier s =
suppliers.get(row);



return switch(column){

case 0 -> s.id();

case 1 -> s.name();

case 2 -> s.phone();

case 3 -> s.email();


default -> "";

};


}


@Override
public String getColumnName(
int column
){

return columns[column];

}


}

```

---

# 8. Why AbstractTableModel?

Because:

```
Database Object

       ↓

TableModel

       ↓

JTable

```

Mapping layer ဖြစ်ပါတယ်။

---

# PART 3

# SupplierPanel Database Binding

---

# 9. SupplierPanel Fields

```java
public class SupplierPanel 
extends JPanel {



private JTable table;


private SupplierTableModel model;


private SupplierController controller;


private JTextField nameField;


private JButton saveButton;


}

```

---

# 10. Constructor

```java
public SupplierPanel(
SupplierController controller
){


this.controller =
controller;


model =
new SupplierTableModel();



table =
new JTable(model);



loadData();


}

```

---

# 11. Load Database Data

```java
private void loadData(){


List<Supplier> suppliers =

controller.getSuppliers();



model.setData(
suppliers
);


}

```

---

Flow:

```
Panel

 ↓

Controller

 ↓

Service

 ↓

Repository

 ↓

SELECT *

 ↓

JTable

```

---

# PART 4

# Save Button Complete Flow

---

# 12. Button Action

```java
saveButton.addActionListener(e -> {


Supplier supplier =

new Supplier(

null,

nameField.getText(),

phoneField.getText(),

emailField.getText(),

addressField.getText()

);



controller.save(
supplier
);



loadData();


});

```

---

# 13. What Happens?

Step by step:

```
User Click SAVE


       ↓


SupplierPanel


       ↓


SupplierController.save()


       ↓


SupplierService.create()


       ↓


SupplierRepository.save()


       ↓


INSERT SQL


       ↓


Database


       ↓


loadData()


       ↓


JTable Refresh

```

---

# PART 5

# Delete Operation

User selects row:

```
JTable Row

      ↓

Get ID

      ↓

Controller.delete()

```

---

Code:

```java
int row =
table.getSelectedRow();


Long id =
(Long)
model.getValueAt(
row,
0
);


controller.delete(id);


loadData();

```

---

# PART 6

# Purchase Order Swing Integration

---

# 14. Purchase Order MVC

Structure:

```
PurchaseOrderPanel


        ↓


PurchaseOrderController


        ↓


PurchaseOrderService


        ↓


PurchaseOrderRepository


        ↓


MySQL

```

---

# 15. Purchase Cart UI State

Remember:

Purchase Order is similar to Sales Cart.

```
PurchaseCart


Coffee Bean 20kg

Milk        50L

Sugar       10kg

```

---

# 16. PurchaseOrderController

```java
public class PurchaseOrderController {


private final PurchaseOrderService service;



public void create(
PurchaseOrder order,
List<PurchaseOrderItem> items
){


service.create(
order,
items
);


}



}

```

---

# PART 7

# Receiving UI Integration

---

# 17. Receiving Flow

```
User Select PO


       ↓


Load Items


       ↓


Enter Received Qty


       ↓


Click RECEIVE


       ↓


Transaction


       ↓


Inventory Updated


```

---

# 18. Receiving Controller

```java
public class StockReceivingController {


private final StockReceivingService service;



public void receive(
Long poId,
List<ReceivingItem> items
){


service.receive(
poId,
items
);


}



}

```

---

# PART 8

# Error Handling Integration

Example:

Database Error:

```
Duplicate Supplier

```

Repository:

```java
throw new DatabaseException(
"Supplier already exists",
e
);

```

---

Controller:

```java
try{


controller.save(supplier);


}
catch(DatabaseException e){


JOptionPane.showMessageDialog(

this,

e.getMessage()

);


}

```

---

# PART 9

# Complete Inventory Module Flow

Now:

```
              Swing


                |


          Controller


                |


          Service Layer


                |


        Repository Layer


                |


        Transaction Manager


                |


              JDBC


                |


              MySQL


```

---

# Current Café POS Architecture Level

After Lesson 53:

```
Java 25                      ✅

Swing MVC                    ✅

Supplier CRUD                ✅

JTable Binding               ✅

Controller Layer             ✅

Service Layer                ✅

Repository Layer             ✅

JDBC Integration             ✅

Transaction Handling         ✅

Purchase Module Architecture ✅

Receiving Architecture       ✅

```

---

# Practice Task

Build:

### Supplier

1. SupplierController
    
2. SupplierTableModel
    
3. SupplierPanel
    
4. Database Binding
    

### Purchase

5. PurchaseOrderController
    
6. Purchase Cart UI
    

### Receiving

7. Receiving Controller
    
8. Inventory Update UI
    

---

# Next Lesson

# Lesson 54: Inventory Management Advanced Part 5

## Complete Purchase Order UI + Product/Ingredient Selection + Professional JTable Cart Design

Next we build:

```
Purchase Order Screen

        ↓

Ingredient ComboBox

        ↓

Quantity Input

        ↓

JTable Cart

        ↓

Create PO

        ↓

Save Transaction

```

ဒီ Lesson ပြီးရင် Inventory Module က **Real Working Restaurant ERP Feature** ဖြစ်လာပါမယ်။