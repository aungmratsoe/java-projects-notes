# Part 3: Café POS Real Implementation Phase

# Lesson 52: Inventory Management Advanced Part 3

## Complete JDBC Repository Implementation + Database Transaction Layer

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson က Café POS Project ရဲ့ **Database Engineering အပိုင်း** ဖြစ်ပါတယ်။

အခုအထိ ကျွန်တော်တို့မှာ:

```text
Supplier Module              ✅
Purchase Order Design        ✅
Stock Receiving Design       ✅
Inventory Workflow           ✅
Transaction Concept          ✅
```

ရှိပါပြီ။

ဒါပေမယ့် Professional Application တစ်ခုမှာ Design သိရုံနဲ့ မလုံလောက်ပါဘူး။

တကယ်ရေးရမယ့်အရာက:

```text
Swing UI

   ↓

Controller

   ↓

Service

   ↓

Repository

   ↓

JDBC

   ↓

MySQL

```

ဒီ Layer တစ်ခုချင်းစီကို Correct Implementation လုပ်ရပါမယ်။

---

# 1. Why Repository Layer Important?

Beginner Code:

```java
button.addActionListener(e -> {


Connection con =
DriverManager.getConnection(...);


PreparedStatement ps =
con.prepareStatement(
"INSERT INTO suppliers..."
);


});

```

Problem:

```text
❌ UI ထဲမှာ Database Code

❌ Testing Difficult

❌ Maintenance Difficult

❌ Business Logic Mixed

```

---

Professional:

```text
SupplierPanel


      ↓


SupplierController


      ↓


SupplierService


      ↓


SupplierRepository


      ↓


MySQL

```

---

# 2. JDBC Layer Responsibility

Repository က:

```text
Repository Responsibilities:


✔ SQL Query

✔ PreparedStatement

✔ ResultSet Mapping

✔ Database Connection

✔ CRUD Operation


```

မလုပ်သင့်တာ:

```text
❌ Validation

❌ Business Rule

❌ UI Logic

```

---

# PART 1

# Supplier JDBC Repository

---

# 3. SupplierRepository Interface

Already:

```java
public interface SupplierRepository {


void save(
Supplier supplier
);



List<Supplier> findAll();



Supplier findById(
Long id
);



void update(
Supplier supplier
);



void delete(
Long id
);


}

```

---

# 4. JDBC Implementation

Create:

```text
repository/jdbc


SupplierRepositoryImpl.java

```

---

Structure:

```java
public class SupplierRepositoryImpl

implements SupplierRepository {



private final DataSource dataSource;


public SupplierRepositoryImpl(
DataSource dataSource
){

this.dataSource =
dataSource;

}


}

```

---

# 5. INSERT Supplier

SQL:

```sql
INSERT INTO suppliers

(
name,
phone,
email,
address
)

VALUES
(
?,
?,
?,
?
);

```

---

Java:

```java
@Override
public void save(
Supplier supplier
){



String sql = """

INSERT INTO suppliers
(name,phone,email,address)

VALUES (?,?,?,?)

""";



try(
Connection con =
dataSource.getConnection();

PreparedStatement ps =
con.prepareStatement(sql)

){



ps.setString(
1,
supplier.name()
);



ps.setString(
2,
supplier.phone()
);



ps.setString(
3,
supplier.email()
);



ps.setString(
4,
supplier.address()
);



ps.executeUpdate();



}

catch(SQLException e){


throw new DatabaseException(
"Cannot save supplier",
e
);


}



}

```

---

# 6. Why PreparedStatement?

Wrong:

```java
String sql =
"SELECT * FROM supplier WHERE name='"
+
name
+
"'";

```

Problem:

```text
SQL Injection

```

---

Correct:

```java
PreparedStatement


WHERE name = ?

```

Database Safe ဖြစ်ပါတယ်။

---

# 7. SELECT All Suppliers

SQL:

```sql
SELECT *

FROM suppliers

ORDER BY id DESC;

```

---

Java:

```java
@Override
public List<Supplier> findAll(){



List<Supplier> list =
new ArrayList<>();



String sql =
"SELECT * FROM suppliers";



try(

Connection con =
dataSource.getConnection();

PreparedStatement ps =
con.prepareStatement(sql);

ResultSet rs =
ps.executeQuery();

){



while(rs.next()){



Supplier supplier =
new Supplier(


rs.getLong("id"),


rs.getString("name"),


rs.getString("phone"),


rs.getString("email"),


rs.getString("address")

);



list.add(supplier);



}



}



return list;


}

```

---

# 8. ResultSet Mapping

Database:

```text
id

name

phone

```

↓

Java Object:

```java
Supplier


id

name

phone

```

ဒီ Process ကို:

```
Object Relational Mapping

(ORM)

```

လို့ခေါ်ပါတယ်။

---

# PART 2

# Purchase Order Repository

---

# 9. Purchase Order Challenge

Purchase Order မှာ:

```text
purchase_orders

        1


purchase_order_items

        many

```

ရှိပါတယ်။

Example:

PO:

```text
PO-1001


Supplier:

ABC

```

Items:

```text
Coffee Bean 20kg

Milk 50L

```

---

ဒါကြောင့် Insert ကို:

```text
Step 1

Insert Header


Step 2

Get Generated ID


Step 3

Insert Items


```

လုပ်ရပါတယ်။

---

# 10. Generated Keys

Need:

```java
Statement.RETURN_GENERATED_KEYS

```

---

Example:

```java
PreparedStatement ps =

con.prepareStatement(

sql,

Statement.RETURN_GENERATED_KEYS

);

```

---

After Insert:

```java
ResultSet keys =

ps.getGeneratedKeys();


keys.next();


Long id =

keys.getLong(1);

```

---

# 11. Purchase Order Transaction

Important:

PO Save = Multiple SQL

```text
INSERT purchase_orders


INSERT purchase_order_items


```

တစ်ခု Fail ဖြစ်ရင်:

```text
Rollback

```

လုပ်ရမယ်။

---

# 12. Transaction Manager

Create:

```text
database


TransactionManager.java

```

---

Code:

```java
public class TransactionManager {



public void execute(

ConnectionTask task

){



Connection con=null;


try{


con =
dataSource
.getConnection();



con.setAutoCommit(false);



task.execute(con);



con.commit();



}

catch(Exception e){



if(con!=null)

con.rollback();



throw e;


}


}

}

```

---

# 13. ConnectionTask

Functional Interface:

```java
@FunctionalInterface

public interface ConnectionTask {


void execute(Connection con)
throws Exception;


}

```

---

Usage:

```java
transactionManager.execute(con -> {



savePurchaseOrder(
con,
order
);



saveItems(
con,
items
);



});

```

---

# 14. Purchase Order Save

Flow:

```text
BEGIN


INSERT purchase_order


       |

       |

Get PO ID


       |

       |

INSERT items


       |

       |

COMMIT


```

---

# 15. Batch Insert

Instead of:

```java
for(Item item:items){

insert(item);

}

```

Slow ဖြစ်ပါတယ်။

Use:

```java
ps.addBatch();

ps.executeBatch();

```

---

Example:

```java
for(
PurchaseOrderItem item:
items
){


ps.setLong(
1,
item.ingredientId()
);


ps.addBatch();


}



ps.executeBatch();

```

---

# PART 3

# Stock Receiving Repository

---

# 16. Receiving Transaction

When receive:

Need:

```text
1. Insert receiving


2. Update inventory


3. Insert stock movement


4. Update PO status


```

---

One Transaction:

```text
BEGIN


INSERT receiving


UPDATE inventory


INSERT movement


UPDATE purchase_order


COMMIT


```

---

# 17. Inventory Update SQL

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


ps.executeUpdate();

```

---

# 18. Stock Movement Insert

SQL:

```sql
INSERT INTO stock_movements

(
ingredient_id,
type,
quantity
)

VALUES
(
?,
?,
?
);

```

---

Example:

```text
ingredient:

Coffee Bean


type:

PURCHASE_RECEIVE


quantity:

+20

```

---

# 19. Database Exception Design

Never:

```java
throw new RuntimeException();

```

Professional:

Create:

```java
public class DatabaseException

extends RuntimeException {



public DatabaseException(
String message,
Throwable cause
){

super(
message,
cause
);

}


}

```

---

Usage:

```java
throw new DatabaseException(

"Failed to save purchase order",

e

);

```

---

# 20. Complete Repository Flow

Final Architecture:

```text
Swing


 ↓


Controller


 ↓


Service


 ↓


Repository Interface


 ↓


Repository Implementation


 ↓


Transaction Manager


 ↓


JDBC


 ↓


MySQL


```

---

# 21. Current Café POS Architecture Level

After Lesson 52:

```text
Java 25                     ✅

Swing MVC                   ✅

Service Layer               ✅

Repository Pattern          ✅

JDBC Layer                  ✅

PreparedStatement           ✅

Generated Keys              ✅

Batch Insert                ✅

Transaction Management      ✅

Rollback Strategy           ✅

```

---

# Practice Task

Implement:

## Supplier

1. SupplierRepositoryImpl
    
2. save()
    
3. findAll()
    
4. update()
    
5. delete()
    

## Purchase Order

6. Generated Key Handling
    
7. Transaction Save
    
8. Batch Insert
    

## Receiving

9. Inventory Update Transaction
    
10. Stock Movement Insert
    

---

# Next Lesson

# Lesson 53: Inventory Management Advanced Part 4

## Complete Swing UI Integration + MVC Controller + Real Database Binding

Next:

```text
JTable

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

ပြီးရင် Supplier / Purchase / Receiving Module ကို **Fully Working Application Feature** ဖြစ်အောင် ချိတ်ဆက်ပါမယ်။