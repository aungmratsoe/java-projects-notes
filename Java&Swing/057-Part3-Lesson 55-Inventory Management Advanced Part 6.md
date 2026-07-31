# Part 3: Café POS Real Implementation Phase

# Lesson 55: Inventory Management Advanced Part 6

## Stock Receiving UI + Inventory Update + Stock Movement History

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Purchase Order Flow ရဲ့ နောက်ဆုံးအပိုင်းကို တည်ဆောက်ပါမယ်။

အခုအထိ:

```text
Supplier Management          ✅

Purchase Order Creation      ✅

Purchase Cart JTable         ✅

Purchase Transaction         ✅

Inventory Analytics          ✅

```

ရှိပြီးပါပြီ။

ဒီနေ့မှာ:

```text
Purchase Order

        ↓

Supplier Delivery

        ↓

Stock Receiving Screen

        ↓

Inventory Update

        ↓

Stock Movement Record

        ↓

Inventory History

```

ကို Complete လုပ်ပါမယ်။

---

# 1. Stock Receiving ဆိုတာဘာလဲ?

Purchase Order ဖန်တီးပြီးတာနဲ့ Stock တက်မသွားပါဘူး။

Example:

## Purchase Order

```text
PO No:

1001


Supplier:

ABC Coffee


Items:


Coffee Bean

20 kg


Milk

50 L

```

Status:

```text
CREATED

```

---

Supplier ပစ္စည်းလာပို့:

```text
Delivery:


Coffee Bean

20 kg


Milk

50 L

```

---

Manager Receive:

```text
Receive Stock


       ↓


Inventory +20kg


       ↓


PO Status = RECEIVED

```

---

# 2. Why Separate Receiving?

Wrong Design:

```text
Create PO

      ↓

Immediately Increase Stock

```

Problem:

Supplier မပို့သေးဘဲ Stock တက်သွားမယ်။

---

Professional:

```text
Purchase Order

        ↓

Waiting Delivery

        ↓

Receiving

        ↓

Stock Update

```

---

# 3. Stock Receiving Module Architecture

Complete:

```text
module/inventory/receiving


model

 StockReceiving.java

 ReceivingItem.java


repository

 StockReceivingRepository.java


service

 StockReceivingService.java


controller

 StockReceivingController.java


view

 StockReceivingPanel.java

```

---

# 4. Database Review

## stock_receiving

```sql
CREATE TABLE stock_receiving
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


purchase_order_id BIGINT,


received_by BIGINT,


received_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(purchase_order_id)

REFERENCES purchase_orders(id)


);

```

---

## stock_receiving_items

Need this table:

```sql
CREATE TABLE stock_receiving_items
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


receiving_id BIGINT,


ingredient_id BIGINT,


quantity DECIMAL(10,2),


FOREIGN KEY(receiving_id)

REFERENCES stock_receiving(id)


);

```

---

Why separate?

Because:

PO:

```text
Ordered

Coffee Bean 20kg

```

Receiving:

```text
Actually Received

Coffee Bean 18kg

```

ဖြစ်နိုင်ပါတယ်။

---

# 5. Model Design

## StockReceiving

```java
public record StockReceiving(


Long id,


Long purchaseOrderId,


Long receivedBy


){

}

```

---

## ReceivingItem

```java
public record ReceivingItem(


Long ingredientId,


String ingredientName,


double quantity


){

}

```

---

# PART 1

# Stock Receiving UI Design

---

# 6. Screen Design

```text
+------------------------------------------------+

              STOCK RECEIVING


Purchase Order:


[ PO-1001          ▼ ]



-------------------------------------------------


Items:


Ingredient       Ordered       Receive


Coffee Bean      20kg          [20]


Milk             50L           [50]



-------------------------------------------------



              [ RECEIVE STOCK ]


+------------------------------------------------+

```

---

# 7. Main Components

```java
public class StockReceivingPanel

extends JPanel {


private JComboBox<PurchaseOrder> poCombo;


private JTable itemTable;


private JButton receiveButton;


}

```

---

# 8. Load Pending Purchase Orders

Need only:

```text
CREATED

SENT

```

not:

```text
RECEIVED

```

---

SQL:

```sql
SELECT *

FROM purchase_orders

WHERE status != 'RECEIVED';

```

---

Controller:

```java
List<PurchaseOrder> orders =

controller
.findPendingOrders();

```

---

# 9. Purchase Order Selection

When user selects PO:

```text
PO-1001

       ↓

Load Items

       ↓

Display JTable

```

---

Code:

```java
poCombo.addActionListener(e -> {


PurchaseOrder po =

(PurchaseOrder)
poCombo
.getSelectedItem();



loadItems(po.id());


});

```

---

# PART 2

# Receiving JTable Model

---

# 10. ReceivingTableModel

```java
public class ReceivingTableModel

extends AbstractTableModel {


private String[] columns =
{

"Ingredient",

"Ordered",

"Receive"

};



private List<ReceivingRow> rows;


}

```

---

# 11. Editable Receive Column

Important:

User can edit:

```text
Receive Column

```

---

Example:

```text
Ordered:

20


Receive:

18

```

---

Override:

```java
@Override

public boolean isCellEditable(

int row,

int column

){


return column == 2;


}

```

---

# 12. Validation

Before Receive:

Check:

```text
Receive <= Ordered

Receive > 0

```

---

Example:

Order:

```text
20kg

```

User:

```text
30kg

```

Error:

```text
Cannot receive more than ordered quantity

```

---

# PART 3

# Receiving Service

---

# 13. Receive Flow

Complete transaction:

```text
BEGIN


INSERT receiving


       ↓


GET receiving_id


       ↓


INSERT receiving_items


       ↓


UPDATE inventory


       ↓


INSERT stock movement


       ↓


UPDATE purchase order


       ↓


COMMIT

```

---

# 14. Service Method

```java
public void receiveStock(

Long poId,

List<ReceivingItem> items

){


transactionManager.execute(con -> {


Long receivingId =


repository.saveReceiving(

con,

poId

);



repository.saveItems(

con,

receivingId,

items

);



inventoryRepository
.increaseStock(

con,

items

);



stockMovementRepository
.save(

con,

items

);



repository.updateStatus(

con,

poId,

"RECEIVED"

);



});


}

```

---

# PART 4

# Inventory Update

---

# 15. Inventory Before

Database:

```text
ingredient_id

10


quantity

5 kg

```

---

Receive:

```text
+20 kg

```

---

After:

```text
25 kg

```

---

SQL:

```sql
UPDATE inventory

SET quantity = quantity + ?

WHERE ingredient_id = ?;

```

---

Java:

```java
ps.setDouble(
1,
quantity
);


ps.setLong(
2,
ingredientId
);

```

---

# PART 5

# Stock Movement History

Every stock change must be recorded.

Why?

Audit:

```text
Who changed stock?

When?

Why?

How much?

```

---

# 16. Stock Movement Table

```sql
CREATE TABLE stock_movements
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


ingredient_id BIGINT,


type VARCHAR(50),


quantity DECIMAL(10,2),


reference_id BIGINT,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

```

---

Example:

```text
Ingredient:

Coffee Bean


Type:

PURCHASE_RECEIVE


Quantity:

+20


Reference:

PO-1001

```

---

# 17. Movement Types

Enum:

```java
public enum StockMovementType {


PURCHASE_RECEIVE,


SALE_CONSUME,


ADJUSTMENT,


RETURN


}

```

---

# 18. Stock Movement Repository

```java
public interface StockMovementRepository {


void save(

Connection con,

StockMovement movement

);


}

```

---

# PART 6

# Complete Receiving Button

Button:

```java
receiveButton
.addActionListener(e -> {


List<ReceivingItem> items =

tableModel
.getReceivingItems();



controller.receive(

selectedPO.id(),

items

);



JOptionPane
.showMessageDialog(

this,

"Stock Received Successfully"

);



});

```

---

# PART 7

# Error Handling

## Case 1

Already Received:

```java
if(
po.status()
==
RECEIVED
){

throw new InventoryException(

"PO already received"

);

}

```

---

## Case 2

Quantity Error:

```java
if(
receiveQty > orderedQty
){

throw new InventoryException(

"Invalid quantity"

);

}

```

---

## Case 3

Database Failure:

```java
rollback();

```

---

# PART 8

# Complete Inventory ERP Flow

Now:

```text
LOW STOCK ALERT


       ↓


CREATE PURCHASE ORDER


       ↓


SEND SUPPLIER


       ↓


SUPPLIER DELIVERY


       ↓


RECEIVE STOCK


       ↓


INVENTORY UPDATE


       ↓


STOCK MOVEMENT


       ↓


REPORT UPDATE

```

---

# Current Café POS Level

After Lesson 55:

```text
Java 25                         ✅

Swing MVC                       ✅

Supplier Module                 ✅

Purchase Order Module           ✅

Stock Receiving Module          ✅

Inventory Update                ✅

Stock Movement History          ✅

Transaction Management          ✅

Audit Trail Concept             ✅

```

---

# Practice Task

Implement:

## UI

1. StockReceivingPanel
    
2. Purchase Order ComboBox
    
3. Receiving JTable
    
4. Editable Receive Column
    

## Backend

5. StockReceivingService
    
6. Inventory Increase
    
7. Stock Movement Save
    
8. Update PO Status
    

---

# Next Lesson

# Lesson 56: Inventory Management Final Part

## Inventory Audit + Stock Adjustment + Complete Inventory Dashboard

Next we build:

```text
Inventory Audit

        ↓

Stock Count

        ↓

Difference Detection

        ↓

Adjustment Entry

        ↓

Inventory Dashboard

        ↓

Final ERP Inventory Module

```

ဒီ Lesson ပြီးရင် Café POS Inventory System က **Professional Restaurant ERP Inventory Level** ပြည့်စုံသွားပါမယ်။