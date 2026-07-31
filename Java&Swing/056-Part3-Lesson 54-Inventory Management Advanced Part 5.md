# Part 3: Café POS Real Implementation Phase

# Lesson 54: Inventory Management Advanced Part 5

## Complete Purchase Order UI + Ingredient Selection + Professional JTable Cart Design

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Purchase Order Module ကို **တကယ်အသုံးပြုနိုင်တဲ့ Professional Swing Feature** အဖြစ် တည်ဆောက်ပါမယ်။

အခုအထိ:

```text
Supplier Management          ✅
Supplier CRUD                ✅
Repository Pattern           ✅
Service Layer                ✅
Controller Layer             ✅
JTable Binding               ✅
Transaction Concept          ✅
```

ရှိပြီးပါပြီ။

ဒီနေ့:

```text
Purchase Order Screen

        ↓

Select Supplier

        ↓

Select Ingredient

        ↓

Enter Quantity

        ↓

Add To Cart JTable

        ↓

Calculate Total

        ↓

Create Purchase Order

        ↓

Save Transaction

```

ကို Build ပါမယ်။

---

# 1. Purchase Order UI Real World Design

Restaurant Manager Workflow:

Example:

```
Manager Login


        ↓


Inventory


        ↓


Create Purchase Order


        ↓


Supplier:
ABC Coffee Supplier


        ↓


Add Items:


Coffee Bean     20 kg

Milk            50 L

Sugar           10 kg


        ↓


CREATE PO


```

---

# 2. Purchase Order Module Structure

Complete:

```
module/inventory/purchase


model

    PurchaseOrder.java

    PurchaseOrderItem.java


service

    PurchaseOrderService.java


controller

    PurchaseOrderController.java


view

    PurchaseOrderPanel.java

    PurchaseCartTableModel.java


```

---

# 3. Purchase Order Panel Layout

Professional UI:

```
+------------------------------------------------+

              CREATE PURCHASE ORDER


Supplier:


[ ABC Coffee Supplier          ▼ ]


Ingredient:


[ Coffee Bean                  ▼ ]


Quantity:


[ 20 ]


Unit Cost:


[ 5000 ]


              [ ADD ITEM ]


-------------------------------------------------


Purchase Items


+-----------------------------------------------+

| Ingredient | Qty | Cost | Amount              |

|-----------------------------------------------|

| Coffee Bean| 20  |5000  |100000              |

| Milk       | 50  |1000  |50000               |

+-----------------------------------------------+



Total:

150000 MMK



              [ CREATE PO ]


+------------------------------------------------+

```

---

# PART 1

# Loading Supplier Data

---

# 4. Supplier ComboBox

Need:

```java
JComboBox<Supplier> supplierCombo;

```

---

But JComboBox display:

```
Supplier Object

        ↓

Display Name

```

Need custom renderer.

---

# 5. SupplierRenderer

Create:

```java
public class SupplierRenderer

extends DefaultListCellRenderer {


@Override

public Component getListCellRendererComponent(

JList<?> list,

Object value,

int index,

boolean selected,

boolean focused

){


super.getListCellRendererComponent(

list,

value,

index,

selected,

focused

);



if(value instanceof Supplier supplier){


setText(
supplier.name()
);


}



return this;

}


}

```

---

Usage:

```java
supplierCombo
.setRenderer(
new SupplierRenderer()
);

```

---

# 6. Load Suppliers

```java
private void loadSuppliers(){


List<Supplier> suppliers =

supplierController
.getSuppliers();



for(Supplier s : suppliers){


supplierCombo.addItem(s);


}



}

```

---

# PART 2

# Ingredient Selection

---

# 7. Ingredient ComboBox

Need:

```java
JComboBox<Ingredient>

ingredientCombo;

```

Example:

```
Coffee Bean

Milk

Sugar

Chocolate

```

---

# 8. Ingredient Model

Example:

```java
public record Ingredient(


Long id,


String name,


String unit,


double costPrice


){

@Override

public String toString(){

return name;

}


}

```

---

Because:

```java
JComboBox

uses toString()

```

---

# PART 3

# Quantity Input

---

# 9. Quantity Field

```java
JTextField quantityField;

```

Example:

```
20

```

---

Validation:

```java
double qty;


try{


qty =
Double.parseDouble(
quantityField.getText()
);


}
catch(NumberFormatException e){


throw new ValidationException(
"Invalid quantity"
);


}

```

---

# PART 4

# Purchase Cart Design

---

# 10. Why Purchase Cart?

Similar to Sales Cart.

Wrong:

```
Click Add Item

        ↓

Immediately Save Database

```

Problem:

```
User changes mind

Cannot edit

Cannot remove

```

---

Correct:

```
Temporary Cart


Coffee Bean 20kg

Milk 50L


        ↓


CREATE PO


        ↓


Database

```

---

# 11. PurchaseCartItem

```java
public record PurchaseCartItem(


Ingredient ingredient,


double quantity


){



public double amount(){


return ingredient.costPrice()

*
quantity;


}


}

```

---

Example:

```
Coffee Bean


Cost:

5000


Qty:

20


Amount:

100000

```

---

# PART 5

# JTable Cart Model

---

# 12. PurchaseCartTableModel

```java
public class PurchaseCartTableModel

extends AbstractTableModel {


private final String[] columns =

{

"Ingredient",

"Quantity",

"Cost",

"Amount"

};



private List<PurchaseCartItem> items =
new ArrayList<>();


public void setItems(
List<PurchaseCartItem> items
){

this.items = items;


fireTableDataChanged();

}


@Override

public int getRowCount(){

return items.size();

}


@Override

public int getColumnCount(){

return columns.length;

}



@Override

public Object getValueAt(
int row,
int col
){


PurchaseCartItem item =
items.get(row);



return switch(col){


case 0 -> item.ingredient().name();


case 1 -> item.quantity();


case 2 -> item.ingredient().costPrice();


case 3 -> item.amount();


default -> null;


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

# PART 6

# Add Item Button

---

# 13. Add Item Flow

```
Click ADD


 ↓


Get Ingredient


 ↓


Get Quantity


 ↓


Create CartItem


 ↓


Add List


 ↓


Refresh JTable


```

---

Code:

```java
addButton.addActionListener(e -> {



Ingredient ingredient =

(Ingredient)
ingredientCombo
.getSelectedItem();



double qty =

Double.parseDouble(
quantityField.getText()
);



PurchaseCartItem item =

new PurchaseCartItem(

ingredient,

qty

);



cart.add(item);



tableModel
.setItems(cart);



});

```

---

# PART 7

# Calculate Total

---

# 14. Total Calculation

Formula:

```
Total

=

Item Amount Sum

```

---

Service:

```java
public double calculateTotal(
List<PurchaseCartItem> items
){


return items.stream()

.mapToDouble(
PurchaseCartItem::amount
)

.sum();


}

```

---

Example:

```
Coffee Bean

100000


Milk

50000


-------------

Total

150000

```

---

# PART 8

# Create Purchase Order

---

# 15. Button Flow

```
CREATE PO


     ↓


Validate


     ↓


Create PurchaseOrder


     ↓


Create Items


     ↓


Controller


     ↓


Service


     ↓


Transaction


     ↓


Database


```

---

# 16. Create Order Object

```java
Supplier supplier =

(Supplier)
supplierCombo
.getSelectedItem();



PurchaseOrder order =

new PurchaseOrder(

null,

supplier.id(),

PurchaseStatus.CREATED,

total

);

```

---

# 17. Controller Call

```java
controller.create(

order,

cart

);

```

---

# PART 9

# Purchase Order Transaction

Inside Service:

```
BEGIN


INSERT purchase_orders


        ↓


GET GENERATED ID


        ↓


INSERT ITEMS


        ↓


COMMIT


```

---

Code Concept:

```java
transactionManager.execute(con -> {



Long id =

repository.saveOrder(
con,
order
);



repository.saveItems(
con,
id,
items
);



});

```

---

# PART 10

# Error Handling

Possible Errors:

## Empty Cart

```java
if(cart.isEmpty()){


throw new ValidationException(

"Add items first"

);


}

```

---

## Invalid Quantity

```
Quantity must be greater than zero

```

---

## No Supplier Selected

```
Please select supplier

```

---

# PART 11

# Complete Purchase Order Flow

Now:

```
Supplier ComboBox


        ↓


Ingredient ComboBox


        ↓


Quantity Input


        ↓


Purchase Cart JTable


        ↓


Calculate Total


        ↓


Create PO


        ↓


Transaction Save


        ↓


Waiting Receiving


```

---

# Current Café POS Level

After Lesson 54:

```
Java 25                       ✅

Swing MVC                     ✅

Supplier Management           ✅

Purchase Order UI             ✅

JTable Cart Design             ✅

ComboBox Binding              ✅

Transaction Architecture      ✅

Inventory ERP Flow            ✅

```

---

# Practice Task

Implement:

### UI

1. PurchaseOrderPanel
    
2. Supplier ComboBox
    
3. Ingredient ComboBox
    
4. Quantity Input
    
5. Cart JTable
    

### Logic

6. PurchaseCartItem
    
7. PurchaseCartTableModel
    
8. Total Calculation
    
9. Create PO Button
    

### Database

10. Transaction Save
    

---

# Next Lesson

# Lesson 55: Inventory Management Advanced Part 6

## Stock Receiving UI + Inventory Update + Stock Movement History

Next we complete the second half:

```
Purchase Order

        ↓

Supplier Delivery

        ↓

Receive Stock Screen

        ↓

Update Inventory

        ↓

Stock Movement History

        ↓

Inventory Report

```

ဒီအဆင့်ပြီးရင် Café POS Inventory Module က **Real Restaurant ERP Workflow Complete** ဖြစ်ပါမယ်။