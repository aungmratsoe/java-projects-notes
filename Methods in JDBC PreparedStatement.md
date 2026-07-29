အခု `PreparedStatement` ရဲ့ methods တွေကို အသေးစိတ်လေ့လာရအောင်။

**အရေးကြီးတာတစ်ခုကို အရင်နားလည်ထားပါ။**

> `PreparedStatement` က `Statement` ကို **inherit** လုပ်ထားတာဖြစ်ပါတယ်။

ဒါကြောင့်

```text
Statement
     ▲
     │
PreparedStatement
```

ဆိုတဲ့ relationship ရှိပါတယ်။

အဲ့ဒါကြောင့် `PreparedStatement` မှာ

- Statement ရဲ့ methods တွေလည်း ရှိတယ်။
    
- သူ့ကိုယ်ပိုင် methods တွေလည်း ထပ်ရှိတယ်။
    

ဥပမာ

```java
PreparedStatement ps =
    conn.prepareStatement(sql);

ps.executeQuery();      // PreparedStatement
ps.executeUpdate();     // PreparedStatement
ps.close();             // Statement က inherited
ps.setString(...);      // PreparedStatement only
ps.setInt(...);         // PreparedStatement only
```

---

# PreparedStatement Methods

## 1. setString()

String Value ထည့်ဖို့ သုံးပါတယ်။

### Syntax

```java
ps.setString(parameterIndex, value);
```

### Example

```java
String sql =
"SELECT * FROM students WHERE name=?";

PreparedStatement ps =
conn.prepareStatement(sql);

ps.setString(1, "Aung");
```

ဒီမှာ

```text
?
```

ကို

```text
Aung
```

နဲ့ အစားထိုးလိုက်တာပါ။

Database ထဲမှာ

```sql
WHERE name='Aung'
```

လိုအလုပ်လုပ်ပါတယ်။

---

## Parameter Index ဆိုတာဘာလဲ?

```java
SELECT *
FROM students
WHERE name=?
AND age=?
```

ဒီမှာ

```text
?      ?
1      2
```

ပထမ `?` က

```java
ps.setString(1,"Aung");
```

ဒုတိယ `?`

```java
ps.setInt(2,20);
```

---

## 2. setInt()

Integer ထည့်ဖို့

```java
ps.setInt(1,10);
```

Example

```java
String sql =
"SELECT * FROM students WHERE id=?";

PreparedStatement ps =
conn.prepareStatement(sql);

ps.setInt(1,5);
```

Database

```sql
id=5
```

---

## 3. setDouble()

```java
ps.setDouble(1,99.5);
```

ဥပမာ

```sql
price=99.5
```

---

## 4. setBoolean()

```java
ps.setBoolean(1,true);
```

Database

```text
true
```

---

## 5. setLong()

```java
ps.setLong(1,500000L);
```

---

## 6. setFloat()

```java
ps.setFloat(1,3.14f);
```

---

## 7. setDate()

SQL Date ထည့်ဖို့

```java
ps.setDate(1,
Date.valueOf("2026-07-29"));
```

---

## 8. setTimestamp()

Date + Time

```java
ps.setTimestamp(
1,
Timestamp.valueOf(
"2026-07-29 12:30:00"));
```

---

## 9. setNull()

NULL ထည့်ဖို့

```java
ps.setNull(
1,
Types.VARCHAR);
```

Database

```text
NULL
```

ဖြစ်သွားပါတယ်။

---

## 10. clearParameters()

Parameter တွေကို Reset လုပ်ပါတယ်။

```java
ps.clearParameters();
```

ဥပမာ

```java
ps.setString(1,"Aung");

ps.clearParameters();

ps.setString(1,"Mg Mg");
```

---

# Execute Methods

PreparedStatement မှာ

SQL String မလိုတော့ပါဘူး။

ဘာကြောင့်လဲ?

Prepare လုပ်ပြီးသား။

---

## 11. executeQuery()

```java
ResultSet rs =
ps.executeQuery();
```

**Statement နဲ့ ကွာတာ**

Statement

```java
stmt.executeQuery(sql);
```

PreparedStatement

```java
ps.executeQuery();
```

SQL မထည့်တော့ပါဘူး။

---

## Example

```java
String sql =
"SELECT * FROM students WHERE id=?";

PreparedStatement ps =
conn.prepareStatement(sql);

ps.setInt(1,1);

ResultSet rs =
ps.executeQuery();
```

---

## Flow

```text
Prepare SQL

↓

Set Parameter

↓

Execute Query

↓

ResultSet
```

---

# 12. executeUpdate()

INSERT

UPDATE

DELETE

အတွက်

```java
int rows =
ps.executeUpdate();
```

Example

```java
String sql =
"UPDATE students SET age=? WHERE id=?";

PreparedStatement ps =
conn.prepareStatement(sql);

ps.setInt(1,21);

ps.setInt(2,1);

int rows =
ps.executeUpdate();
```

Return

```text
1
```

ဆိုတာ

```text
1 row updated.
```

---

# 13. execute()

```java
boolean b =
ps.execute();
```

Statement နဲ့တူပါတယ်။

---

# 14. addBatch()

```java
ps.setInt(1,1);

ps.addBatch();

ps.setInt(1,2);

ps.addBatch();

ps.executeBatch();
```

တူညီတဲ့ SQL ကို parameter တွေပြောင်းပြီး batch execute လုပ်တဲ့အခါ အရမ်းအသုံးဝင်ပါတယ်။

---

# 15. executeBatch()

```java
int[] rows =
ps.executeBatch();
```

Return

```text
[1,1,1,1]
```

ဆိုတာ

Row တစ်ခုစီအတွက် result ဖြစ်ပါတယ်။

---

# 16. close()

```java
ps.close();
```

Resource ပိတ်ပါတယ်။

---

# 17. getMetaData()

Result Column Information

```java
ResultSetMetaData md =
ps.getMetaData();
```

---

# 18. getParameterMetaData()

Parameter အချက်အလက်

```java
ParameterMetaData pm =
ps.getParameterMetaData();
```

ဥပမာ

```sql
WHERE id=?
AND name=?
```

Parameter Count

```java
System.out.println(
pm.getParameterCount());
```

Output

```text
2
```

---

# Real CRUD Example

## INSERT

```java
String sql =
"INSERT INTO students(name, age) VALUES(?, ?)";

PreparedStatement ps =
conn.prepareStatement(sql);

ps.setString(1, "Aung");
ps.setInt(2, 20);

int rows = ps.executeUpdate();

System.out.println(rows + " row inserted.");
```

---

## UPDATE

```java
String sql =
"UPDATE students SET age=? WHERE id=?";

PreparedStatement ps =
conn.prepareStatement(sql);

ps.setInt(1, 22);
ps.setInt(2, 1);

ps.executeUpdate();
```

---

## DELETE

```java
String sql =
"DELETE FROM students WHERE id=?";

PreparedStatement ps =
conn.prepareStatement(sql);

ps.setInt(1, 1);

ps.executeUpdate();
```

---

## SELECT

```java
String sql =
"SELECT * FROM students WHERE id=?";

PreparedStatement ps =
conn.prepareStatement(sql);

ps.setInt(1, 1);

ResultSet rs = ps.executeQuery();

while (rs.next()) {
    System.out.println(rs.getString("name"));
}
```

---

# Statement vs PreparedStatement Methods

|Statement|PreparedStatement|
|---|---|
|`executeQuery(sql)`|`setString()`, `setInt()` စတဲ့ parameter methods ရှိ|
|`executeUpdate(sql)`|`executeQuery()` (SQL မလို)|
|`execute(sql)`|`executeUpdate()` (SQL မလို)|
|`close()`|`execute()`|
|`addBatch(sql)`|`addBatch()` (parameter တွေနဲ့)|
|`executeBatch()`|`executeBatch()`|
|`clearBatch()`|`clearParameters()` (PreparedStatement သီးသန့်)|

---

# Interview မှာ အရေးကြီးဆုံး PreparedStatement Methods

Java Developer တစ်ယောက်အနေနဲ့ နေ့စဉ်အသုံးအများဆုံး methods တွေကတော့

1. `setString()`
    
2. `setInt()`
    
3. `setLong()`
    
4. `setDouble()`
    
5. `setBoolean()`
    
6. `executeQuery()`
    
7. `executeUpdate()`
    
8. `close()`
    

---

## Senior Developer Tip

Production Project တွေမှာ `PreparedStatement` ကို `try-with-resources` နဲ့ တွဲသုံးတာက အကောင်းဆုံးဖြစ်ပါတယ်။ အဲဒီလိုဆို `close()` ကို ကိုယ်တိုင်မခေါ်လည်း Java က အလိုအလျောက် resource တွေကို ပိတ်ပေးပါတယ်။

```java
String sql = "SELECT * FROM students WHERE id=?";

try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setInt(1, 1);

    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("name"));
        }
    }
}
```

ဒီပုံစံက resource leak မဖြစ်အောင် ကာကွယ်ပေးပြီး production code တွေမှာ အသုံးအများဆုံး coding style ဖြစ်ပါတယ်။