`Statement` class မှာ အသုံးများတဲ့ methods တွေကို အရင်ကြည့်ရအောင်။

---

# Statement Methods

```java
Statement stmt = connection.createStatement();
```

ဒီ `stmt` object ကနေ methods တွေကို ခေါ်သုံးတာပါ။

---

# 1. executeQuery()

**အသုံးပြုမှု**

Database ကနေ **SELECT** data ပြန်ယူတဲ့အခါ သုံးပါတယ်။

### Syntax

```java
ResultSet rs = stmt.executeQuery(sql);
```

### Example

```java
Statement stmt = conn.createStatement();

ResultSet rs = stmt.executeQuery(
    "SELECT * FROM students"
);

while (rs.next()) {
    System.out.println(rs.getString("name"));
}
```

### Return Type

```java
ResultSet
```

Database က data တွေကို `ResultSet` အဖြစ် ပြန်ပေးပါတယ်။

---

## Flow

```
Java

   |

executeQuery()

   |

Database

   |

SELECT

   |

ResultSet
```

---

# 2. executeUpdate()

INSERT, UPDATE, DELETE အတွက် သုံးပါတယ်။

### Syntax

```java
int rows = stmt.executeUpdate(sql);
```

### INSERT Example

```java
String sql =
"INSERT INTO students(name, age) VALUES('Aung',20)";

int rows = stmt.executeUpdate(sql);

System.out.println(rows);
```

Output

```
1
```

ဆိုတာ

```
1 row inserted.
```

လို့ အဓိပ္ပာယ်ရပါတယ်။

---

### UPDATE Example

```java
int rows = stmt.executeUpdate(
"UPDATE students SET age=21 WHERE id=1");
```

---

### DELETE Example

```java
int rows = stmt.executeUpdate(
"DELETE FROM students WHERE id=1");
```

---

### Return Type

```java
int
```

Database မှာ

```
ဘယ်နှစ် row ပြောင်းသွားလဲ
```

ဆိုတာ ပြန်ပေးပါတယ်။

ဥပမာ

```
5
```

ဆိုရင်

```
5 rows updated.
```

---

# executeUpdate Flow

```
Java

   |

executeUpdate()

   |

Database

   |

INSERT

UPDATE

DELETE

   |

Return affected rows
```

---

# 3. execute()

ဒီ method က **အမျိုးအစားမသိတဲ့ SQL** ကို execute လုပ်တဲ့အခါ သုံးပါတယ်။

### Syntax

```java
boolean result = stmt.execute(sql);
```

### Example

```java
boolean result =
stmt.execute("SELECT * FROM students");
```

Return

```
true
```

ဘာလို့လဲ?

ResultSet ပြန်လာလို့။

---

နောက်တစ်ခု

```java
boolean result =
stmt.execute(
"UPDATE students SET age=22 WHERE id=1");
```

Return

```
false
```

ဘာလို့?

ResultSet မရှိဘူး။

Update Count ပဲရှိတယ်။

---

### Return Type

```java
boolean
```

```
true
```

ResultSet ရှိ

```
false
```

ResultSet မရှိ

---

# execute() Flow

```
SQL

      |

execute()

      |

SELECT ?

     / \

   Yes  No

   |     |

true  false
```

---

# 4. close()

Statement ကို ပိတ်ဖို့ သုံးပါတယ်။

### Syntax

```java
stmt.close();
```

### Example

```java
Statement stmt =
conn.createStatement();

ResultSet rs =
stmt.executeQuery(
"SELECT * FROM students");

stmt.close();
```

ဒီလို မပိတ်ရင်

```
Database Resource
```

တွေ မလွတ်ဘူး။

Memory Leak ဖြစ်နိုင်ပါတယ်။

---

# 5. getResultSet()

`execute()` သုံးပြီးတဲ့နောက် ResultSet ကို ယူဖို့ သုံးပါတယ်။

```java
stmt.execute(sql);

ResultSet rs =
stmt.getResultSet();
```

အသုံးနည်းပါတယ်။

---

# 6. getUpdateCount()

`execute()` သုံးပြီး UPDATE ဖြစ်ခဲ့ရင်

```
ဘယ်နှစ် Row ပြောင်းလဲသွားလဲ
```

ဆိုတာ ယူပါတယ်။

```java
stmt.execute(sql);

int count =
stmt.getUpdateCount();
```

---

# 7. addBatch()

SQL တွေကို တစ်ခုချင်းမပို့ဘဲ စုထားဖို့ သုံးပါတယ်။

```java
stmt.addBatch(
"INSERT INTO students VALUES(1,'Aung')");

stmt.addBatch(
"INSERT INTO students VALUES(2,'Mg Mg')");
```

Database မပို့သေးပါဘူး။

Memory ထဲမှာ သိမ်းထားပါတယ်။

---

# 8. executeBatch()

`addBatch()` နဲ့ စုထားတဲ့ SQL တွေကို တစ်ခါတည်း execute လုပ်ပါတယ်။

```java
stmt.executeBatch();
```

---

Flow

```
addBatch()

↓

addBatch()

↓

addBatch()

↓

executeBatch()

↓

Database
```

Performance ပိုကောင်းပါတယ်။

---

# 9. clearBatch()

Batch ထဲမှာရှိတဲ့ SQL တွေကို ဖျက်ပါတယ်။

```java
stmt.clearBatch();
```

---

# 10. cancel()

Run နေတဲ့ SQL ကို ရပ်ဖို့ သုံးပါတယ်။

```java
stmt.cancel();
```

ဥပမာ

```
SELECT
```

က ၁၀ မိနစ်လောက် ကြာနေတယ်ဆိုရင်

```
cancel()
```

နဲ့ ရပ်နိုင်ပါတယ်။

---

# 11. setQueryTimeout()

SQL ကို အများဆုံး ဘယ်နှစ်စက္ကန့် စောင့်မလဲ သတ်မှတ်ပါတယ်။

```java
stmt.setQueryTimeout(5);
```

အဓိပ္ပာယ်က

```
5 seconds ထက်ကျော်ရင်

Query Cancel
```

---

# 12. getConnection()

ဒီ Statement ကို ဘယ် `Connection` ကနေ create လုပ်ထားတာလဲဆိုတာ ပြန်ယူပါတယ်။

```java
Connection c =
stmt.getConnection();
```

---

# Summary Table

|Method|Return Type|အသုံးပြုမှု|
|---|---|---|
|`executeQuery(String sql)`|`ResultSet`|`SELECT` အတွက်|
|`executeUpdate(String sql)`|`int`|`INSERT`, `UPDATE`, `DELETE` အတွက်|
|`execute(String sql)`|`boolean`|SQL အမျိုးအစားမသေချာတဲ့အခါ|
|`close()`|`void`|Statement ပိတ်ရန်|
|`getResultSet()`|`ResultSet`|`execute()` ပြီးနောက် ResultSet ယူရန်|
|`getUpdateCount()`|`int`|`execute()` ပြီးနောက် affected rows ယူရန်|
|`addBatch(String sql)`|`void`|SQL များကို batch အဖြစ် စုရန်|
|`executeBatch()`|`int[]`|Batch SQL များကို တစ်ခါတည်း execute လုပ်ရန်|
|`clearBatch()`|`void`|Batch ကို ရှင်းရန်|
|`cancel()`|`void`|Run နေသော query ကို ရပ်ရန်|
|`setQueryTimeout(int seconds)`|`void`|Query timeout သတ်မှတ်ရန်|
|`getConnection()`|`Connection`|မူလ Connection ကို ပြန်ယူရန်|

## Interview မှာ အရေးကြီးဆုံး Methods (95% အသုံးများ)

Java Developer တစ်ယောက်အနေနဲ့ နေ့စဉ်အလုပ်မှာ အများဆုံးအသုံးပြုတာကတော့

1. `executeQuery()` → `SELECT`
    
2. `executeUpdate()` → `INSERT`, `UPDATE`, `DELETE`
    
3. `execute()` → SQL အမျိုးအစားမသေချာတဲ့အခါ
    
4. `close()` → Resource ပိတ်ရန်
    

ကျန်တဲ့ methods (`addBatch()`, `executeBatch()`, `setQueryTimeout()`, `cancel()` စတာတွေ) ကိုတော့ လိုအပ်တဲ့အခြေအနေတွေမှာ အသုံးပြုကြပါတယ်။