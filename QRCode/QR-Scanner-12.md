# Chapter 12 — Complete System Architecture

## DAO + Model + Utils + UI Layer + Data Flow


ဒီ Chapter က ဒီ QR Student Identity System ရဲ့ **Big Picture Architecture** ကို နားလည်ဖို့ အရေးကြီးဆုံး Chapter ပါ။

အခုအထိ ကျွန်တော်တို့ သီးခြားစီ လေ့လာခဲ့တယ်။

- `CryptoUtils` → Encryption / Decryption
    
- `QRUtils` → QR Generate
    
- `QRScanner` → Camera Scan + Verification
    
- `StudentDAO` → Database Access
    
- `Student` → Data Object
    

အခု အားလုံးကို ဘယ်လိုချိတ်ထားလဲ ကြည့်မယ်။

---

# 12.1 Project Package Structure

မင်း project structure ကို Professional Java Application ပုံစံနဲ့ကြည့်ရင်:

```
com.ams.qrcode

│
├── ui
│    │
│    ├── Home.java
│    ├── QRGenerator.java
│    └── QRScanner.java
│
├── model
│    │
│    └── Student.java
│
├── dao
│    │
│    ├── StudentDAO.java
│    └── StudentDAOInterface.java
│
├── utils
│    │
│    ├── CryptoUtils.java
│    └── QRUtils.java
│
└── database
     │
     └── DBConnection.java

```

---

ဒီ Architecture ကို Layer Architecture လို့ခေါ်တယ်။

```
        User
         |
         v

+----------------+
|      UI        |
|  Swing JFrame  |
+----------------+

         |
         v

+----------------+
|     DAO        |
| Database Logic |
+----------------+

         |
         v

+----------------+
|   Database     |
|    MySQL       |
+----------------+

```

---

# 12.2 Layer တစ်ခုချင်းစီ

## 1. UI Layer

Location:

```
com.ams.qrcode.ui
```

တာဝန်:

User နဲ့ interaction လုပ်တာ။

Example:

```
QRGenerator.java
QRScanner.java
Home.java
```

---

UI Layer က မလုပ်သင့်တာ:

❌ SQL Query မရေးရ

မကောင်းတဲ့ Code:

```java
private void saveStudent(){

    Connection con =
    DriverManager.getConnection(...);

    PreparedStatement ps =
    con.prepareStatement(
    "INSERT INTO student..."
    );

}
```

ဘာဖြစ်လဲ?

UI + Database ပေါင်းသွားတယ်။

---

Professional way:

```java
private void saveStudent(){

    studentDAO.insert(student);

}
```

---

UI က DAO ကိုပဲခေါ်မယ်။

---

# 12.3 Model Layer

Location:

```
model.Student
```

Model ဆိုတာ Database Table ကို represent လုပ်တာ။

---

Database:

```
student_table

+-------------+
| id          |
| student_id  |
| name        |
| email       |
| qr_token    |
+-------------+

```

---

Java Class:

```java
public class Student {

    private int id;

    private String studentId;

    private String name;

    private String email;

    private String qrToken;

}
```

---

Relationship:

```
Database Row

        |
        |
        v

Student Object

```

---

Example:

Database:

|student_id|name|
|---|---|
|ST001|Aung|

↓

Java:

```java
Student s = new Student();

s.setStudentId("ST001");

s.setName("Aung");

```

---

# 12.4 DAO Layer

DAO = Data Access Object

Location:

```
dao.StudentDAO
```

---

DAO ရဲ့တာဝန်:

Database နဲ့ စကားပြောပေးတာ။

---

Example:

UI:

```java
studentDAO.getStudentByStudentId("ST001");
```

↓

DAO:

```sql
SELECT *
FROM students
WHERE student_id='ST001';

```

↓

Database:

```
ST001
Aung
Computer Science

```

---

Architecture:

```
QRScanner

   |
   v

StudentDAO

   |
   v

MySQL

```

---

# 12.5 DAO Interface

မင်းမှာ:

```java
StudentDAOInterface
```

ရှိတယ်။

ဒါက Professional Pattern ပါ။

---

Example:

```java
public interface StudentDAOInterface {


Student getStudentByStudentId(String id);


void save(Student student);


}
```

---

Implementation:

```java
public class StudentDAO 
implements StudentDAOInterface {


@Override
public Student getStudentByStudentId(String id){

}

}

```

---

ဘာအကျိုးရှိလဲ?

Future မှာ Database ပြောင်းချင်ရင်:

Before:

```
MySQL

```

After:

```
PostgreSQL

```

UI ကို မပြောင်းရဘူး။

---

# 12.6 Utils Layer

Location:

```
utils
```

ဒီမှာ reusable tools တွေထားတယ်။

---

## CryptoUtils

တာဝန်:

```
Plain Text

    |
    v

AES Encryption

    |
    v

Cipher Text

```

---

Example:

```java
CryptoUtils.encrypt(data);
```

---

## QRUtils

တာဝန်:

QR Generate

```
String

 |

BitMatrix

 |

Image

```

---

Example:

```java
QRUtils.generateQRCode(
data,
300,
300
);

```

---

# 12.7 Complete QR Generation Flow

အခု အားလုံးချိတ်ကြည့်မယ်။

User:

```
Click Generate QR Button

```

---

Step 1

UI:

```java
QRGenerator.java
```

Student ရွေးတယ်။

---

Step 2

Create Token

```
UUID.randomUUID()

```

Result:

```
8fa92bd1
```

---

Step 3

Create Payload

```
StudentID:ST001
Token:8fa92bd1

```

---

Step 4

Encryption

```java
CryptoUtils.encrypt(payload)

```

Result:

```
A8sk92Ksl==
```

---

Step 5

QR Generate

```java
QRUtils.generateQRCode()

```

Result:

```
████████
██    ██
████████

```

---

Step 6

Save Image

```
qrcodes/ST001.png

```

---

Step 7

Save Token

Database:

```
ST001

qr_token:
8fa92bd1

```

---

# 12.8 Complete Scanner Flow

အခု Reverse Flow:

```
Camera

 |
 v

QRScanner.java

 |
 v

ZXing Decode

 |
 v

Encrypted String


A8sk92Ksl==


 |
 v

CryptoUtils.decrypt()


 |
 v


StudentID:ST001
Token:8fa92bd1


 |
 v

StudentDAO


 |
 v

Database


 |
 v

Compare Token


 |
 v

Access Granted

```

---

# 12.9 Dependency Direction

Professional Architecture မှာ dependency direction အရေးကြီးတယ်။

မှန်တဲ့ပုံ:

```
UI
 |
 v
DAO
 |
 v
Database


UI
 |
 v
Utils


Model
 |
 ^
 |
DAO

```

---

မလုပ်သင့်တာ:

```
Database
     |
     v
UI

```

Database က UI ကို သိနေတယ်ဆိုရင် bad design.

---

# 12.10 Single Responsibility Principle (SOLID)

ဒီ project မှာ SRP ကိုသုံးထားတယ်။

## QRScanner

တာဝန်:

```
Scan QR
Verify Result

```

မလုပ်ရ:

```
Generate QR
Database Connection
AES Logic

```

---

## CryptoUtils

တာဝန်:

```
Encryption only

```

---

## QRUtils

တာဝန်:

```
QR Creation only

```

---

## StudentDAO

တာဝန်:

```
Database only

```

---

ဒါကြောင့်:

```
One Class

=

One Responsibility

```

---

# 12.11 Real World Flow Diagram

```
                 USER
                  |
                  |
                  v

          +---------------+
          | Swing UI      |
          | QRGenerator   |
          | QRScanner     |
          +---------------+

                  |
          ----------------
          |              |

          v              v

    +-----------+    +-----------+
    | Crypto    |    | QR Utils  |
    | AES       |    | ZXing     |
    +-----------+    +-----------+

          |
          |
          v

    +-------------+
    | DAO Layer   |
    | StudentDAO  |
    +-------------+

          |
          |
          v

    +-------------+
    | MySQL       |
    | Database    |
    +-------------+

```

---

# 12.12 Security Flow

ဒီ System ရဲ့ Security Model:

```
Original Data

StudentID
Token

    |
    |
    v

AES-128 Encryption

    |
    |
    v

QR Code


---------------------


Scanner

QR Code

    |
    v

Decrypt

    |
    v

Token Compare

    |
    v

Database Verify

```

---

# Chapter 12 Summary

ဒီ Project ရဲ့ Professional Structure:

|Layer|Class|Responsibility|
|---|---|---|
|UI|QRScanner|Camera + User Interface|
|UI|QRGenerator|Create QR|
|Model|Student|Data Object|
|DAO|StudentDAO|Database CRUD|
|Utils|CryptoUtils|AES Security|
|Utils|QRUtils|QR Creation|
|Database|MySQL|Storage|

---

**Senior Developer View**

မင်းရေးထားတဲ့ Project က ရိုးရိုး Swing Project မဟုတ်တော့ဘူး။

အခု Architecture က:

```
Swing MVC Style
+
DAO Pattern
+
Utility Layer
+
AES Security
+
ZXing Integration
```

ဖြစ်နေပြီ။

---

နောက် Chapter 13 မှာတော့:

# Chapter 13 — QRGenerator.java Complete Breakdown

မှာ

```
Button Click
 |
Generate Token
 |
AES Encrypt
 |
Create QR Image
 |
Display JLabel
 |
Save PNG
 |
Update Database
```

ကို line-by-line လေ့လာမယ်။