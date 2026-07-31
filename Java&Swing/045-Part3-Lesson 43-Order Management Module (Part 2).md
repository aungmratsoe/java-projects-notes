# Part 3: Café POS Real Implementation Phase

# Lesson 43: Order Management Module (Part 2)

## Product Selection + Cart UI + Checkout Transaction

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Café POS ရဲ့ **Cashier Selling Screen** ကို တကယ်အလုပ်လုပ်အောင် တည်ဆောက်ပါမယ်။

အခုအထိ:

```text
Product Module          ✅
Inventory Module        ✅
Recipe System           ✅
Cart Architecture       ✅
Order Entity            ✅
```

ရှိပြီးပါပြီ။

ဒီနေ့ပြီးရင်:

```text
Cashier

   ↓

Search Product

   ↓

Select Product

   ↓

Add To Cart

   ↓

Change Quantity

   ↓

Calculate Total

   ↓

Checkout

   ↓

Save Order Database

```

ဖြစ်လာပါမယ်။

---

# 1. POS Screen Final Design

Cashier Screen:

```text
+--------------------------------------------------------------+

|                    CAFÉ POS SELLING                          |

+--------------------------------------------------------------+


 Products                         Cart


 Search:

[ Latte____________ ]


+----------------+        +-------------------------------+

| Product        |        | Product | Qty | Price          |

|----------------|        |-------------------------------|

| Latte          |        | Latte   | 2   | 10000          |

| Cappuccino     |        | Cake    | 1   | 4000           |

| Burger         |        |                               |

+----------------+        +-------------------------------+



                         Total:

                         14000 MMK



                         [CHECKOUT]


+--------------------------------------------------------------+

```

---

# 2. Order View Package

Update:

```text
module/order/view


OrderPanel.java

ProductSelectionTableModel.java

CartTableModel.java

```

---

# 3. Product Selection Table Model

Products ဘက်:

```text
ID | Product | Price

1     Latte     5000

2     Cake      4000

```

---

Create:

`ProductSelectionTableModel.java`

```java
package com.cafe.pos.module.order.view;


import javax.swing.table.AbstractTableModel;

import java.util.*;

import com.cafe.pos.module.product.model.Product;



public class ProductSelectionTableModel
extends AbstractTableModel {



private final String[] columns =
{
"ID",
"Product",
"Price"
};



private List<Product> products =
new ArrayList<>();



public void setProducts(
List<Product> products
){

this.products = products;

fireTableDataChanged();

}



public Product getProduct(
int row
){

return products.get(row);

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

case 0 ->
product.id();


case 1 ->
product.name();


case 2 ->
product.price();


default ->
null;


};


}

}

```

---

# 4. Cart Quantity Update

အခု Cart မှာ:

```text
Latte x1

```

ရှိပြီး User က ထပ်နှိပ်ရင်:

Wrong:

```text
Latte x1

Latte x1

```

Correct:

```text
Latte x2

```

ဖြစ်ရမယ်။

---

# 5. Update CartService

Before:

```java
items.add(item);

```

---

After:

```java
public void addItem(
CartItem newItem
){


for(int i=0;
i<items.size();
i++){


CartItem item =
items.get(i);



if(
item.productId()
.equals(
newItem.productId()
)
){



items.set(

i,

new CartItem(

item.productId(),

item.productName(),

item.price(),

item.quantity()
+
newItem.quantity()

)

);


return;


}



}


items.add(newItem);


}

```

---

# 6. Remove Cart Item

Add:

```java
public void removeItem(
Long productId
){


items.removeIf(
item ->
item.productId()
.equals(productId)
);


}

```

---

# 7. OrderPanel Design

Create:

```java
public class OrderPanel
extends JPanel {


private JTable productTable;


private JTable cartTable;


private JButton checkoutButton;


}

```

---

Layout:

Use:

```java
JSplitPane

```

because:

```text
Left

Products


Right

Cart

```

---

Code:

```java
JSplitPane splitPane =
new JSplitPane(
JSplitPane.HORIZONTAL_SPLIT,
productScroll,
cartScroll
);


add(
splitPane,
BorderLayout.CENTER
);

```

---

# 8. Add Product To Cart

User double clicks Product:

```java
productTable
.addMouseListener(

new MouseAdapter(){


@Override

public void mouseClicked(
MouseEvent e
){



if(e.getClickCount()==2){


int row =
productTable
.getSelectedRow();



Product product =
productModel
.getProduct(row);



cartService.addItem(

new CartItem(

product.id(),

product.name(),

product.price(),

1

)

);



cartModel.setItems(

cartService.getItems()

);


}


}


}

);

```

---

# 9. Cart Total Display

Create:

```java
private JLabel totalLabel;

```

---

Update:

```java
private void updateTotal(){


double total =
cartService.getTotal();



totalLabel.setText(

String.format(
"Total: %.2f MMK",
total
)

);


}

```

---

# 10. Checkout Button Flow

Button:

```java
checkoutButton
.addActionListener(e->{


checkout();


});

```

---

Method:

```java
private void checkout(){


if(
cartService
.getItems()
.isEmpty()
){


JOptionPane.showMessageDialog(
this,
"Cart is empty"
);


return;

}



orderController
.checkout(
cartService
.getItems()
);



}

```

---

# 11. OrderController

Create:

```java
public void checkout(
List<CartItem> items
){


service.checkout(items);


}

```

---

# 12. Save Order Database

Transaction:

```text
BEGIN


INSERT orders


       ↓


Get Order ID


       ↓


INSERT order_items


       ↓


Decrease Stock


       ↓


INSERT Payment


COMMIT

```

---

# 13. OrderRepository Save

Interface:

```java
Long save(
Order order
);

```

---

Implementation:

SQL:

```sql
INSERT INTO orders

(
user_id,
total_amount,
status
)

VALUES
(?,?,?)

```

---

Return generated ID:

```java
PreparedStatement.RETURN_GENERATED_KEYS

```

---

Example:

```java
ResultSet rs =
statement
.getGeneratedKeys();


rs.next();


Long orderId =
rs.getLong(1);

```

---

# 14. Save Order Items

SQL:

```sql
INSERT INTO order_items

(
order_id,
product_id,
quantity,
price
)

VALUES
(?,?,?,?)

```

---

Example:

```text
Order ID: 100


Latte

Qty 2

Price 5000


Cake

Qty 1

Price 4000

```

---

# 15. Complete Checkout Sequence

Final:

```text
CHECKOUT

   |

   |

OrderController

   |

   |

OrderService


   |

   |---- Save Order


   |

   |---- Save Items


   |

   |---- Consume Inventory


   |

   |---- Save Stock Movement


   |

   |---- Payment


   |

 COMMIT


```

---

# 16. Error Handling

Example:

Stock မလောက်:

```text
Coffee Bean insufficient

```

Throw:

```java
throw new StockException(
"Not enough Coffee Bean"
);

```

UI:

```java
catch(Exception e){

JOptionPane.showMessageDialog(
this,
e.getMessage()
);

}

```

---

# 17. Current POS Capability

Now:

```text
Login                  ✅

Dashboard              ✅

Product Management     ✅

Inventory              ✅

Recipe System          ✅

Cashier Screen         ✅

Cart                   ✅

Order Creation         🔄

Checkout Flow          🔄

```

---

# Practice Task

Implement:

1. Product Selection JTable
    
2. Cart JTable
    
3. Double Click Add Cart
    
4. Quantity Update
    
5. Total Calculation
    
6. Checkout Button
    

---

# Next Lesson

# Lesson 44: Order Management Module (Part 3)

## Complete Checkout + Payment System + Receipt Generation

Next:

```text
Cart

 ↓

Order

 ↓

Payment

 ↓

Receipt

 ↓

Print / PDF

```

ကို တည်ဆောက်ပါမယ်။

ပြီးရင် Café POS မှာ **Cashier တစ်ယောက် Customer Order ကို အဆုံးထိ Complete Sale လုပ်နိုင်ပါမယ်။**