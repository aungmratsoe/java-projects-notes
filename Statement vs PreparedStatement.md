JDBC (Java Database Connectivity) တွင် **Statement** နှင့် **PreparedStatement** တို့သည် SQL queries များ run ရန် အသုံးပြုသည့် Object များ ဖြစ်ကြသည်။ ၎င်းတို့၏ အဓိက ကွာခြားချက်မှာ **Performance (လုပ်ဆောင်ချက် မြန်ဆန်မှု)** နှင့် **Security (လုံခြုံရေး)** တို့ ဖြစ်သည်။

အောက်တွင် ၎င်းတို့၏ အဓိက ကွာခြားချက်များကို ရှင်းလင်းစွာ နှိုင်းယှဉ်ဖော်ပြပေးထားပါသည် -

၁။ အဓိက ကွာခြားချက်များ နှိုင်းယှဉ်ချက် (Comparison Table)

|အချက်အလက် (Feature)|Statement|PreparedStatement|
|---|---|---|
|**Execution (အလုပ်လုပ်ပုံ)**|SQL Query ကို Run တိုင်း အသစ်ပြန်လည် Compile လုပ်သည်။|SQL Query ကို ကြိုတင် Compile လုပ်ထားပြီး Parametric values များကိုသာ လဲလှယ်သည်။|
|**Performance (အရှိန်အဟုန်)**|Query တစ်ခုတည်းကို ခဏခဏ Run ပါက ပို၍ နှေးသည်။|Query တစ်ခုတည်းကို values ပြောင်းပြီး ခဏခဏ Run ပါက ပို၍ မြန်သည်။|
|**Security (လုံခြုံရေး)**|**SQL Injection** Vulnerability (အားနည်းချက်) ရှိသည်။|**SQL Injection** ကို အပြည့်အဝ ကာကွယ်ပေးနိုင်သည်။|
|**Syntax (ကုဒ်ပုံစံ)**|String Concatenation (`+` လက္ခဏာ) ဖြင့် values များကို ပေါင်းစပ်ရသည်။|Placeholder (`?` လက္ခဏာ) ကို အသုံးပြုသည်။|
|**Caching (မှတ်ဉာဏ်သိမ်းဆည်းမှု)**|Database မှ Query ကို Cache မလုပ်ပါ။|Database မှ ကြိုတင် Compile လုပ်ထားသော Query ကို Cache လုပ်ထားနိုင်သည်။|

---

၂။ အသေးစိတ် ရှင်းလင်းချက် (Detailed Explanation)

Statement ဆိုတာဘာလဲ။ 

Statement ကို SQL query အသေများ (Static SQL queries) ဖြစ်သည့် `SELECT * FROM users` ကဲ့သို့သော variable မပါသည့် နေရာများတွင် သုံးရန် သင့်တော်သည်။ query ကို မောင်းနှင်သည့်အခါတိုင်း Database က SQL ကို အသစ်ထပ်မံ စစ်ဆေး (Parse) ပြီး Compile လုပ်ရသောကြောင့် တစ်ကြိမ်ထက်မက သုံးလျှင် အချိန်ပိုကြာတတ်သည်။ [[1]

- **ဥပမာ ကုဒ်စတိုင် -**

```java
String query = "SELECT * FROM users WHERE username = '" + name + "' AND password = '" + pass + "'";
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);
```

Use code with caution.

_(⚠️ သတိပြုရန် - အထက်ပါပုံစံသည် User ဘက်မှ `' OR '1'='1` ကဲ့သို့သော SQL Injection Hacker Code များ ရိုက်ထည့်ပါက Database ကျိုးပေါက်နိုင်ပါသည်။)_ 

PreparedStatement ဆိုတာဘာလဲ။ 

PreparedStatement ကို dynamic ဖြစ်သော (တစ်ခုနှင့်တစ်ခု တန်ဖိုးမတူဘဲ ပြောင်းလဲနေသည့် variable ပါသော) SQL queries များတွင် သုံးသည်။ ၎င်းသည် query template ကို ကြိုတင် Compile လုပ်ထားပြီး variable နေရာတွင် `?` (Placeholders) ကို အစားထိုးကာ သတ်မှတ်ပေးသည်။ 

- **ဥပမာ ကုဒ်စတိုင် -**

```java
String query = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement pstmt = connection.prepareStatement(query);
pstmt.setString(1, name);
pstmt.setString(2, pass);
ResultSet rs = pstmt.executeQuery();
```

Use code with caution.

_(✅ အကျိုးကျေးဇူး - `?` နေရာတွင် ဝင်လာသမျှ data တန်ဖိုးများကို text သက်သက်အဖြစ်သာ သတ်မှတ်ပြီး input ကို database က သန့်စင် (escape) လိုက်သောကြောင့် SQL Injection လုံးဝ မဖြစ်နိုင်တော့ပါ။)_ 

---

၃။ မည်သည့်အချိန်တွင် ဘာကို သုံးသင့်သလဲ။

- **PreparedStatement ကို အမြဲသုံးပါ -** Application တည်ဆောက်ရာတွင် Security နှင့် Speed သည် အရေးကြီးဆုံးဖြစ်သောကြောင့် လက်တွေ့ လုပ်ငန်းခွင် (Real-world projects) များတွင် **PreparedStatement ကိုသာ standard အဖြစ် သတ်မှတ်၍ အမြဲသုံးစွဲသင့်ပါသည်။** 
- **Statement ကို ခန့်မှန်းရလွယ်သော နေရာတွင်သာ သုံးပါ -** parameter (variable) လုံးဝ မပါဝင်ဘဲ application run ချိန်တွင် တစ်ကြိမ်သာ သုံးမည့် static query မျိုး (ဥပမာ - application စဖွင့်ချင်း table အသစ်ဆောက်သည့် `CREATE TABLE ...` မျိုး) တွင်သာ Statement ကို သုံးသင့်သည်။ [[1]

---
အရမ်းကောင်းတဲ့မေးခွန်းပါ။ Java Interview မှာလည်း **Statement** နဲ့ **PreparedStatement** က အမြဲလိုလို မေးခံရတဲ့ topic ဖြစ်ပါတယ်။

ဒီနေ့တော့ **"Statement နဲ့ PreparedStatement ဘယ်လိုအလုပ်လုပ်လဲ?"** ကို JDBC အတွင်းမှာ **Senior Java Developer** တစ်ယောက်လို မြင်နိုင်အောင် မြန်မာလို အသေးစိတ်ရှင်းပြမယ်။

---

# JDBC ဆိုတာဘာလဲ?

JDBC (Java Database Connectivity) ဆိုတာ

> Java Program နဲ့ Database (MySQL, PostgreSQL, Oracle...) ကို ဆက်သွယ်ပေးတဲ့ API ဖြစ်ပါတယ်။

Program က Database ကို ပြောချင်တဲ့အရာက SQL ဖြစ်ပါတယ်။

ဥပမာ

```
SELECT * FROM students;
```

ဒါကို Java က Database ဆီပို့ရပါတယ်။

ဒီနေရာမှာ အသုံးဝင်လာတာက

```
Statement
PreparedStatement
CallableStatement
```

ဒီနေ့တော့ Statement နဲ့ PreparedStatement ကို လေ့လာမယ်။

---

# JDBC Flow

```
Java Program

      │

      ▼

Connection

      │

      ▼

Statement / PreparedStatement

      │

      ▼

MySQL Server

      │

      ▼

Execute SQL

      │

      ▼

ResultSet
```

ဒါက JDBC အလုပ်လုပ်တဲ့ Flow ဖြစ်ပါတယ်။

---

# Step 1

Connection ရယူတယ်

```java
Connection conn =
DriverManager.getConnection(
"jdbc:mysql://localhost:3306/studentdb",
"root",
"1234");
```

ဒီမှာ

Java Program က

```
Hello MySQL!

I want to connect.
```

လို့ Database ကို ပြောတာပါ။

Connection ရသွားပြီ။

---

# Step 2

Statement Create လုပ်တယ်

```java
Statement stmt = conn.createStatement();
```

ဒီ Statement က

SQL ကို Database ဆီပို့မယ့် Messenger ဖြစ်ပါတယ်။

```
Java
   │
   ▼
Statement
   │
   ▼
Database
```

---

# Statement Example

```java
String sql =
"SELECT * FROM students";

ResultSet rs = stmt.executeQuery(sql);
```

ဒီမှာ

Java က

```
SELECT * FROM students
```

ကို String အဖြစ် Database ဆီပို့လိုက်တာပါ။

---

# Database အတွင်းမှာဘာဖြစ်လဲ?

Database က

```
SELECT * FROM students
```

ကို ရလာတယ်။

အဲ့ဒီ SQL ကို

**Parse**

↓

**Compile**

↓

**Optimize**

↓

**Execute**

လုပ်ပါတယ်။

ပြီးရင်

Result ကို Java ဆီပြန်ပို့ပါတယ်။

```
Student1

Student2

Student3
```

---

# Statement Diagram

```
Java

 |

 | SQL String

 ▼

Statement

 |

 ▼

Database

 |

 ▼

Parse

Compile

Optimize

Execute

 |

 ▼

ResultSet
```

---

# Statement အားနည်းချက်

Statement မှာ

SQL ကို

String နဲ့ပဲဆောက်ရတယ်။

ဥပမာ

```java
String name = "Aung";

String sql =
"SELECT * FROM students WHERE name='"
+ name + "'";
```

Database ဆီသွားတဲ့ SQL က

```
SELECT * FROM students
WHERE name='Aung'
```

အဆင်ပြေပါတယ်။

ဒါပေမယ့် User က

```
'Aung' OR 1=1 --
```

ထည့်လိုက်ရင်

SQL က

```
SELECT *
FROM students
WHERE name=''
OR 1=1 --
```

ဖြစ်သွားတယ်။

---

# SQL Injection

```
WHERE name=''

OR 1=1
```

`1=1` က အမြဲ True ဖြစ်ပါတယ်။

ဒါဆို

Database က

Student အားလုံးကို ပြန်ပေးလိုက်တယ်။

ဒါကို

**SQL Injection Attack**

လို့ခေါ်ပါတယ်။

Statement ရဲ့ အကြီးဆုံး Problem ဖြစ်ပါတယ်။

---

# Statement Performance

နောက်ပြဿနာတစ်ခုက

အကြိမ် 1000 Query ထုတ်ရင်

Database က

အကြိမ် 1000

Parse

Compile

Optimize

လုပ်ရတယ်။

ဥပမာ

```
SELECT * FROM students WHERE id=1

SELECT * FROM students WHERE id=2

SELECT * FROM students WHERE id=3

SELECT * FROM students WHERE id=4
```

Database အမြင်မှာ

ဒါတွေဟာ SQL အသစ်အသစ်တွေလို့ မြင်ပါတယ်။

ဒါကြောင့်

```
Parse

Compile

Optimize
```

ကို အကြိမ်တိုင်း ထပ်လုပ်ရတယ်။

---

# ဒီပြဿနာကို ဖြေရှင်းတာ PreparedStatement

PreparedStatement က

SQL Structure ကို အရင် Prepare လုပ်ထားတယ်။

ဥပမာ

```java
String sql =
"SELECT * FROM students WHERE id=?";
```

ဒီမှာ

```
?
```

က Placeholder ဖြစ်ပါတယ်။

---

PreparedStatement Create

```java
PreparedStatement ps =
conn.prepareStatement(sql);
```

ဒီအချိန်မှာ

Database က

```
SELECT * FROM students
WHERE id=?
```

ကို

Parse

Compile

Optimize

လုပ်ပြီး

Memory ထဲမှာ သိမ်းထားတယ်။

```
Execution Plan
```

လို့ခေါ်ပါတယ်။

---

Parameter ထည့်မယ်

```java
ps.setInt(1,5);
```

ဆိုတာ

```
? = 5
```

လို့ ပြောတာပါ။

---

Execute

```java
ResultSet rs =
ps.executeQuery();
```

Database ဆီသွားတဲ့ SQL က

```
SELECT *
FROM students
WHERE id=5
```

ဖြစ်ပါတယ်။

---

# ဒါဆို Parse ပြန်လုပ်လား?

မလုပ်တော့ပါဘူး။

Execution Plan ရှိပြီးသား။

Parameter ပဲပြောင်းတယ်။

```
id=5

id=6

id=7

id=8
```

---

Database က

```
Execution Plan
```

ကိုပဲ သုံးပါတယ်။

ဒါကြောင့်

ပိုမြန်ပါတယ်။

---

# PreparedStatement Diagram

```
Java

 |

 ▼

PreparedStatement

 |

 ▼

SELECT *
FROM students
WHERE id=?

 |

 ▼

Database

 |

 ▼

Prepare Once

Parse

Compile

Optimize

 |

 ▼

Execution Plan

 |

 ▼

setInt()

 |

 ▼

Execute

 |

 ▼

Result
```

---

# SQL Injection ဘာကြောင့်မဖြစ်တာလဲ?

ဥပမာ

```java
String sql =
"SELECT * FROM students WHERE name=?";
```

```java
ps.setString(1,
"' OR 1=1 --");
```

Database က

ဒါကို SQL အဖြစ် မမြင်ပါဘူး။

Text အဖြစ်ပဲ မြင်ပါတယ်။

တကယ်သွားတာက

```
Name

=

"' OR 1=1 --"
```

ဆိုတဲ့ String Value ပါ။

ဒါကြောင့်

SQL Structure မပြောင်းနိုင်ပါဘူး။

SQL Injection မဖြစ်တော့ပါဘူး။

---

# Statement vs PreparedStatement

|Statement|PreparedStatement|
|---|---|
|SQL ကို String နဲ့တည်ဆောက်|SQL Template + `?` Placeholder|
|SQL Injection ဖြစ်နိုင်|SQL Injection ကို ကာကွယ်နိုင်|
|Parse/Compile အကြိမ်တိုင်း|Prepare တစ်ခါလုပ်ပြီး ပြန်သုံးနိုင်|
|Performance နည်း|Performance ပိုကောင်း|
|Parameter မထောက်ပံ့|`setInt()`, `setString()` စတဲ့ parameter methods ပါဝင်|

---

# Real Example (Statement)

```java
Connection conn = DriverManager.getConnection(url, user, password);

Statement stmt = conn.createStatement();

String sql = "SELECT * FROM students WHERE id=1";

ResultSet rs = stmt.executeQuery(sql);

while (rs.next()) {
    System.out.println(rs.getString("name"));
}
```

**Flow**

```
Create Statement

↓

Send SQL String

↓

Database Parse

↓

Compile

↓

Execute

↓

Return Result
```

---

# Real Example (PreparedStatement)

```java
Connection conn = DriverManager.getConnection(url, user, password);

String sql = "SELECT * FROM students WHERE id=?";

PreparedStatement ps = conn.prepareStatement(sql);

ps.setInt(1, 1);

ResultSet rs = ps.executeQuery();

while (rs.next()) {
    System.out.println(rs.getString("name"));
}
```

**Flow**

```
Prepare SQL

↓

Database builds Execution Plan

↓

Set Parameter

↓

Execute

↓

Return Result
```

---

# ဘယ်အချိန်မှာ Statement သုံးသင့်လဲ?

တကယ်တော့ **Production Project** တွေမှာ Statement ကို သုံးတာ အလွန်ရှားပါတယ်။

သုံးနိုင်တဲ့ အခြေအနေတွေကတော့

- Parameter မလိုတဲ့ SQL
    
- Test code
    
- Database administration scripts
    
- Learning purpose
    

ဥပမာ

```java
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT NOW()");
```

---

# ဘယ်အချိန်မှာ PreparedStatement သုံးသင့်လဲ?

နေ့စဉ် CRUD (Create, Read, Update, Delete) အလုပ်အားလုံးမှာ သုံးသင့်ပါတယ်။

```java
INSERT INTO students ...

UPDATE students ...

DELETE FROM students ...

SELECT * FROM students WHERE id=?

SELECT * FROM students WHERE email=?
```

သင့်ရဲ့ **Student Management System** မှာလည်း `PreparedStatement` ကို အဓိကသုံးသင့်ပါတယ်။ User က TextField ကနေ ထည့်တဲ့ data တွေကို `setString()`, `setInt()` စတဲ့ methods တွေနဲ့ parameter အဖြစ်ပို့တာက လုံခြုံပြီး performance လည်း ပိုကောင်းပါတယ်။

---

# Interview မှာ ဖြေဖို့ အတိုချုပ်

> **Statement** က SQL ကို String အနေနဲ့ တိုက်ရိုက် execute လုပ်တာဖြစ်ပြီး SQL Injection ဖြစ်နိုင်သလို repeated execution မှာ performance မကောင်းပါဘူး။ **PreparedStatement** က SQL Template နဲ့ Placeholder (`?`) ကို သုံးပြီး parameter တွေကို သီးခြားပို့တဲ့အတွက် SQL Injection ကို ကာကွယ်နိုင်ပြီး execution plan ကို reuse လုပ်နိုင်လို့ performance ပိုကောင်းပါတယ်။

---

နောက်သင်ခန်းစာမှာ **PreparedStatement အတွင်း `setString()`, `setInt()`, `executeQuery()`, `executeUpdate()` ဘယ်လို အတွင်းပိုင်းအလုပ်လုပ်သလဲ** နဲ့ **ResultSet ရဲ့ Cursor ဘယ်လိုရွေ့သလဲ** ကို Diagram တွေ၊ Memory ပုံစံတွေနဲ့ အသေးစိတ်ဆက်ရှင်းပြပေးနိုင်ပါတယ်။