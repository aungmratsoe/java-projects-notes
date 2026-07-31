# Part 3: Café POS Real Implementation Phase

# Lesson 51: Inventory Management Advanced Part 2

## Complete Supplier UI + Purchase Order Swing Module + Transaction Implementation

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ ပြီးခဲ့တဲ့ Lesson 50 မှာ Design လုပ်ခဲ့တဲ့ Inventory ERP Flow ကို **တကယ်အသုံးပြုနိုင်တဲ့ Swing Application Module** အဖြစ် တည်ဆောက်ပါမယ်။

အခုအထိ:

```text
Supplier Database Design        ✅

Purchase Order Design           ✅

Stock Receiving Concept         ✅

Inventory Update Flow           ✅
```

ရှိပါပြီ။

ဒီနေ့:

```text
Supplier Management UI

        ↓

Create Purchase Order

        ↓

Receive Stock

        ↓

Update Inventory

        ↓

Stock Movement

        ↓

Transaction Commit

```

ကို Coding Architecture နဲ့ ဆက်သွားပါမယ်။

---

# 1. Complete Inventory Workflow Review

Real Café Example:

## Step 1: Low Stock Alert

System:

```
Coffee Bean

Current: 2kg

Minimum: 5kg

⚠ LOW STOCK
```

---

## Step 2: Manager Creates PO

```
Supplier:

ABC Coffee


Items:

Coffee Bean 20kg

Milk 50L

```

Database:

```
purchase_orders

ID = 1001

Status = CREATED

```

---

## Step 3: Supplier Delivers

Delivery:

```
Coffee Bean 20kg

Milk 50L

```

---

## Step 4: Receive Stock

System:

```
Purchase Order

       ↓

Receiving

       ↓

Inventory + Quantity

       ↓

Stock Movement

```

---

# 2. Updated Package Structure

Full Inventory Module:

```
module/inventory


├── supplier

│
│── model
│     Supplier.java
│
│── repository
│     SupplierRepository.java
│
│── service
│     SupplierService.java
│
│── view
│     SupplierPanel.java



├── purchase

│
│── model
│     PurchaseOrder.java
│     PurchaseOrderItem.java
│
│── repository
│     PurchaseOrderRepository.java
│
│── service
│     PurchaseOrderService.java
│
│── view
│     PurchaseOrderPanel.java



├── receiving

│
│── service
│     StockReceivingService.java
│
│── view
│     StockReceivingPanel.java

```

---

# PART 1

# Supplier Management Swing UI

---

# 3. SupplierPanel Design

Professional CRUD Screen:

```
+------------------------------------------------+

 Supplier Management


 Name:

 [________________]


 Phone:

 [________________]


 Email:

 [________________]


 Address:

 [________________]


        [SAVE]

        [UPDATE]

        [DELETE]


-------------------------------------------------

ID     Name          Phone


1      ABC Coffee    09123456


2      Fresh Milk    09999999


+------------------------------------------------+

```

---

# 4. SupplierPanel Class

```java
public class SupplierPanel 
extends JPanel {


private JTable table;


private JTextField nameField;

private JTextField phoneField;

private JTextField emailField;


private JButton saveButton;


}
```

---

# 5. Layout Design

Use:

```
BorderLayout

       |

 ------------------

 |                |

Form             JTable


```

---

Code:

```java
setLayout(
new BorderLayout()
);


add(
createFormPanel(),
BorderLayout.WEST
);


add(
createTablePanel(),
BorderLayout.CENTER
);

```

---

# 6. Supplier Form Panel

```java
private JPanel createFormPanel(){


JPanel panel =
new JPanel(
new GridLayout(8,1)
);



panel.add(
new JLabel("Name")
);


panel.add(nameField);


panel.add(
new JLabel("Phone")
);


panel.add(phoneField);


panel.add(saveButton);



return panel;

}

```

---

# 7. Save Supplier Button

Flow:

```
Button Click

       ↓

Get Input

       ↓

Create Supplier Object

       ↓

SupplierController

       ↓

Service

       ↓

Repository

       ↓

Database

```

---

Code:

```java
saveButton.addActionListener(e->{


Supplier supplier =
new Supplier(

null,

nameField.getText(),

phoneField.getText(),

emailField.getText(),

addressField.getText()

);



controller.save(supplier);


});

```

---

# PART 2

# Purchase Order UI

---

# 8. Purchase Order Screen

Design:

```
+------------------------------------------------+

 Create Purchase Order


Supplier:


[ ABC Coffee Supplier ▼ ]


Ingredient:


[ Coffee Bean ▼ ]


Quantity:


[ 20 ] kg



             [ADD]


-----------------------------------------------


Current Items:


Coffee Bean       20kg

Milk              50L


-----------------------------------------------


Total:

500000 MMK



             [CREATE PO]


+------------------------------------------------+

```

---

# 9. Purchase Order State

Important:

Before saving:

```
Temporary Cart


```

Similar to Sales Cart.

Example:

```
PurchaseCart


Coffee Bean 20kg

Milk        50L

```

---

# 10. PurchaseCartItem

Create:

```java
public record PurchaseCartItem(


Long ingredientId,


String ingredientName,


double quantity,


double cost


){

public double subtotal(){

return quantity * cost;

}


}

```

---

# 11. Purchase Cart Service

```java
public class PurchaseCartService {


private final List<PurchaseCartItem>
items =
new ArrayList<>();



public void add(
PurchaseCartItem item
){

items.add(item);

}



public double total(){


return items.stream()

.mapToDouble(
PurchaseCartItem::subtotal
)

.sum();


}



public void clear(){

items.clear();

}

}

```

---

# 12. Create Purchase Order

Button:

```
CREATE PO

```

Flow:

```
Purchase Cart

       ↓

PurchaseOrderService

       ↓

Save Purchase Order

       ↓

Save Items

       ↓

COMMIT

```

---

# 13. Purchase Order Transaction

Very Important.

Wrong:

```
Save Header

Commit


Save Items Failed

```

Problem:

```
PO exists without items

```

---

Correct:

```
BEGIN


INSERT purchase_orders


GET PO ID


INSERT purchase_items


COMMIT


```

---

# 14. JDBC Transaction Example

```java
Connection con =
dataSource.getConnection();


try{


con.setAutoCommit(false);



Long poId =
repository.saveHeader(
po
);



repository.saveItems(
poId,
items
);



con.commit();



}
catch(Exception e){


con.rollback();


throw e;


}

```

---

# PART 3

# Stock Receiving UI

---

# 15. Receiving Screen

```
+------------------------------------------------+

 Stock Receiving


Purchase Order:


[PO-1001]


---------------------------------


Item          Ordered    Receive


Coffee Bean   20kg       [20]


Milk          50L        [50]



             [RECEIVE]


+------------------------------------------------+

```

---

# 16. Receiving Logic

When Receive clicked:

```
Receive Button


      ↓


Validate Quantity


      ↓


Create Receiving Record


      ↓


Increase Inventory


      ↓


Create Stock Movement


      ↓


Update PO Status


```

---

# 17. Receiving Transaction

One transaction:

```
BEGIN


INSERT stock_receiving


UPDATE inventory


INSERT stock_movement


UPDATE purchase_orders


COMMIT

```

---

# 18. Inventory Increase Method

InventoryService:

Add:

```java
public void addStock(

Long ingredientId,

double quantity

){



Inventory inventory =
repository
.findByIngredientId(
ingredientId
);



double newQuantity =

inventory.quantity()
+
quantity;



repository.updateQuantity(

ingredientId,

newQuantity

);



}

```

---

# 19. Stock Movement Record

Example:

```
Movement:


Ingredient:

Coffee Bean


Type:

PURCHASE_RECEIVE


Quantity:

+20kg


User:

Manager


Time:

2026-07-31


```

---

# 20. Error Handling

Possible Errors:

## Case 1:

PO Already Received:

```
Purchase Order already completed

```

---

## Case 2:

Receive Quantity > Ordered:

Example:

Order:

```
20kg

```

Receive:

```
30kg

```

Throw:

```java
throw new InventoryException(

"Received quantity exceeds order"

);

```

---

# 21. Complete Inventory ERP Flow

Now:

```
LOW STOCK


    ↓


Purchase Order


    ↓


Supplier


    ↓


Delivery


    ↓


Stock Receiving


    ↓


Inventory Increase


    ↓


Stock Movement


    ↓


Reports Update


```

---

# 22. Current Café POS Status

After Lesson 51:

```
Authentication              ✅

Product Management           ✅

Inventory CRUD               ✅

Recipe System                ✅

Stock Deduction              ✅

Order System                 ✅

Payment                      ✅

Receipt                      ✅

Sales Report                 ✅

Profit Analysis              ✅

Supplier Management          ✅

Purchase Order UI            🔄

Stock Receiving UI           🔄

Transaction System           🔄

```

---

# Practice Task

Implement:

### Supplier

1. SupplierPanel
    
2. Supplier CRUD Controller
    
3. Supplier Service
    

### Purchase

4. PurchaseCartService
    
5. PurchaseOrderPanel
    
6. Purchase Transaction
    

### Receiving

7. StockReceivingPanel
    
8. Inventory addStock()
    
9. Stock Movement Creation
    

---

# Next Lesson

# Lesson 52: Inventory Management Advanced Part 3

## Complete JDBC Repository Implementation + Database Transaction Layer

Next we go deeper into:

```
Repository Layer

      ↓

PreparedStatement

      ↓

Generated Keys

      ↓

Batch Insert

      ↓

Transaction Manager

      ↓

Rollback Strategy

```

ဒီ Lesson ကနေ **Senior Java Backend Engineer Level Database Handling** ကို စတင်ဝင်ပါမယ်။