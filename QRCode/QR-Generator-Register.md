# Chapter 17 — QRGenerator Register Button

## Student Registration Flow (DAO + Model + Database Architecture)

Great **aung mrat**.

ဒီ method က `QRGenerator` Form ရဲ့ **Register Button** ကိုနှိပ်တဲ့အချိန် run ဖြစ်တဲ့ method ဖြစ်ပါတယ်။

အဓိကတာဝန်က:

> User ဖြည့်ထားတဲ့ Student Information ကိုယူ → Validate လုပ် → Database ထဲသိမ်း → Table Refresh → Form Clear

ဖြစ်ပါတယ်။

---

# 17.1 Method Declaration

```java
private void btnRegisterActionPerformed(java.awt.event.ActionEvent evt)
```

ဒီ method ကို NetBeans GUI Builder က auto generate လုပ်ပေးတာပါ။

---

Structure:

```text
User Click Register Button

          |
          v

btnRegisterActionPerformed()

          |
          v

Read Form Data

          |
          v

Validate Input

          |
          v

Check Duplicate ID

          |
          v

Create Student Object

          |
          v

Save Database

          |
          v

Refresh Table

          |
          v

Clear Form

```

---

# Chapter 17.2 — Reading Form Data

Code:

```java
String stdId = txtStdId.getText().trim();
String name = txtName.getText().trim();
String sex = cbGender.getSelectedItem().toString();
String email = txtEmail.getText().trim();
String department = txtDepartment.getText().trim();
Date selectedDate = dcDob.getDate();
```

ဒီအပိုင်းက UI Component တွေက value ယူတာပါ။

---

## Student ID

```java
String stdId = txtStdId.getText().trim();
```

Example:

TextField:

```
+----------------+
| STU001         |
+----------------+
```

ရလာမယ်:

```java
stdId = "STU001";
```

---

## Why `.trim()`?

Example:

User input:

```
"   STU001   "
```

without trim:

```java
"   STU001   "
```

ဖြစ်နေမယ်။

---

with trim:

```java
"STU001"
```

ဖြစ်သွားတယ်။

---

## Name

```java
String name = txtName.getText().trim();
```

Example:

```
Aung Mrat
```

Result:

```java
name="Aung Mrat";
```

---

## Gender ComboBox

```java
String sex =
cbGender.getSelectedItem().toString();
```

ComboBox:

```
+-----------+
| Male   ▼  |
+-----------+

```

Selected:

```
Male
```

Result:

```java
sex="Male";
```

---

## Date Picker

```java
Date selectedDate =
dcDob.getDate();
```

ဒီက:

```
JDateChooser
```

ကနေ Date Object ယူတာ။

Example:

```
10 March 1994
```

Result:

```java
Date object
```

---

# Chapter 17.3 — Input Validation

Code:

```java
if (
stdId.isEmpty()
||
name.isEmpty()
||
sex.isEmpty()
||
email.isEmpty()
||
department.isEmpty()
||
selectedDate == null
)
```

---

Meaning:

"Required field တစ်ခုခု မဖြည့်ထားရင်"

Error ပြမယ်။

---

Example:

User:

```
Student ID: STU001

Name:

Email:

DOB:
```

---

Condition:

```java
name.isEmpty()
```

true ဖြစ်မယ်။

---

Then:

```java
JOptionPane.showMessageDialog(...)
```

run မယ်။

---

Result:

```
-------------------------
Input Error

Please fill in all required fields
-------------------------
```

---

Then:

```java
return;
```

---

Important:

`return` က method ကို ချက်ချင်းရပ်တယ်။

Flow:

```text
Validation Failed

       |
       v

Show Error

       |
       v

return

       |
       v

Stop

```

---

# Chapter 17.4 — Try Catch Block

Code:

```java
try {

}
catch(Exception e){

}
```

---

Database operation တွေမှာ error ဖြစ်နိုင်လို့ သုံးတာ။

Example:

- Database down
    
- SQL error
    
- Connection fail
    
- NullPointerException
    

---

Structure:

```text
try

 |
 |
Database Work

 |
 |
Success


catch

 |
 |
Handle Error

```

---

# Chapter 17.5 — Duplicate Student ID Check

Code:

```java
Student existingStudent =
studentDAO.getStudentByStudentId(stdId);
```

---

ဒီနေရာမှာ DAO layer ကို သုံးတယ်။

Architecture:

```
QRGenerator

     |
     v

StudentDAOInterface

     |
     v

StudentDAO

     |
     v

MySQL Database

```

---

Example:

Database:

|id|student_id|name|
|---|---|---|
|1|STU001|Aung|
|2|STU002|Mya|

---

User input:

```
STU001
```

Query:

```sql
SELECT *
FROM students
WHERE student_id='STU001';
```

---

Result:

```java
existingStudent != null
```

ဖြစ်မယ်။

---

Then:

```java
if(existingStudent != null)
```

---

Meaning:

"ဒီ Student ID ရှိပြီးသားလား?"

---

ရှိရင်:

```
Student ID 'STU001'
is already taken by Aung
```

ပြမယ်။

---

# Chapter 17.6 — Creating Student Object

Code:

```java
Student student = new Student();
```

---

Model class object တစ်ခု create လုပ်တာ။

---

Before:

```
Database

(empty)

```

---

Create:

```
Student Object

{
 id:null,
 name:null,
 email:null
}

```

---

ပြီးရင် data ထည့်တယ်။

---

# Chapter 17.7 — Setting Student Data

## Student ID

```java
student.setStudentId(stdId);
```

Example:

```java
student.studentId="STU001";
```

---

## Name

```java
student.setName(name);
```

Result:

```java
name="Aung Mrat";
```

---

## Gender

```java
student.setSex(sex);
```

---

## Email

```java
student.setEmail(email);
```

---

## Department

```java
student.setDepartment(department);
```

---

## DOB

```java
student.setDob(selectedDate);
```

---

Now Object:

```
Student

{
 studentId:"STU001",

 name:"Aung Mrat",

 sex:"Male",

 email:"aung@gmail.com",

 department:"Computer Science",

 dob:"10-03-1994"
}

```

---

# Chapter 17.8 — QR Token Design Decision

ဒီ comment က အရေးကြီးတယ်။

```java
// DO NOT set qrToken here!
```

---

အဓိပ္ပါယ်:

Register လုပ်တဲ့အချိန်မှာ QR Token မဖန်တီးဘူး။

---

Old design:

```
Register

   |
   v

Create UUID Token

   |
   v

Save Student

   |
   v

Generate QR

```

---

Problem:

Token ရှိပေမယ့် QR မထုတ်ရသေးနိုင်တယ်။

---

New design:

```
Register

   |
   v

Save Student

(qrToken=null)


Later:

Generate QR Button

   |
   v

Create Token

   |
   v

Encrypt

   |
   v

Generate QR

```

---

ပိုကောင်းတဲ့ flow ဖြစ်တယ်။

---

# Chapter 17.9 — Save Student

Code:

```java
studentDAO.saveStudent(student);
```

---

DAO က Database ကို handle လုပ်တယ်။

Flow:

```
Student Object

       |
       v

DAO

       |
       v

INSERT SQL

       |
       v

MySQL

```

---

Example SQL:

```sql
INSERT INTO students
(
student_id,
name,
sex,
email,
department,
dob
)

VALUES
(
'STU001',
'Aung Mrat',
'Male',
'aung@gmail.com',
'CS',
'1994-03-10'
);

```

---

# Chapter 17.10 — Refresh Table

Code:

```java
loadStudentData("");
```

---

Register ပြီးရင် JTable ကို update လုပ်တာ။

Before:

```
JTable

(empty)

```

After:

```
--------------------------------
ID       Name
--------------------------------
STU001   Aung Mrat
--------------------------------

```

---

Parameter:

```java
""
```

ဆိုတာ:

"No search filter"

လို့ အဓိပ္ပါယ်ရနိုင်တယ်။

---

# Chapter 17.11 — Clear Form

Code:

```java
handleClear();
```

---

Purpose:

Input field တွေ reset လုပ်တယ်။

Before:

```
ID: STU001
Name: Aung
Email: xxx@gmail.com

```

After:

```
ID:
Name:
Email:

```

---

User Experience ကောင်းစေတယ်။

---

# Chapter 17.12 — Success Message

Code:

```java
JOptionPane.showMessageDialog(
this,
"Student registered successfully!",
"Success",
JOptionPane.INFORMATION_MESSAGE
);
```

---

Result:

```
----------------------
Success

Student registered successfully!
----------------------

```

---

# Chapter 17.13 — Exception Handling

Code:

```java
catch(Exception e)
```

---

Error ဖြစ်ရင်:

```java
logger.log(
Level.SEVERE,
"Failed to register student",
e
);
```

---

Log file မှာ:

```
SEVERE:
Failed to register student

SQLException...
```

လိုရမယ်။

---

User ကို:

```java
JOptionPane
```

နဲ့ပြတယ်။

---

# Complete Architecture Flow

ဒီ button တစ်ခုနှိပ်တဲ့အခါ:

```
             Register Button

                    |
                    v

          Get UI Components

                    |
                    v

          Validate Required Fields

                    |
                    v

          Check Student ID

                    |
                    v

          Create Student Model

                    |
                    v

              StudentDAO

                    |
                    v

              MySQL Database

                    |
                    v

          Refresh JTable

                    |
                    v

          Clear Input Fields

                    |
                    v

             Success Message

```

---

# Senior Developer Analysis

ဒီ method မှာ ပါတဲ့ Architecture Concepts:

|Concept|Usage|
|---|---|
|MVC|View → Controller → Model|
|DAO Pattern|Database separation|
|Validation|Prevent bad data|
|Exception Handling|Stability|
|Object Mapping|Form → Model|
|User Feedback|JOptionPane|
|Clean Flow|Maintainability|

---

ဒီ Register method က Simple CRUD မဟုတ်တော့ဘူး။

ဒါက Professional Desktop Application ရဲ့ pattern:

```
Swing UI
   |
Controller Logic
   |
DAO Layer
   |
Database

```

ဖြစ်နေပြီ။

နောက် Chapter 18 မှာ သင့် `Generate QR Code Button` ကို breakdown လုပ်ရင်:

```
Student Data
      |
Generate UUID Token
      |
AES Encryption
      |
ZXing QR Writer
      |
Save PNG
      |
Update qrToken Database

```

ဆိုတဲ့ Security Flow ကို အသေးစိတ်သွားမယ်။