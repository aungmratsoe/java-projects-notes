# Chapter 20 — QRGenerator `populateFieldsFromSelectedRow()`

## JTable Selection → Form Auto Fill → QR Preview Loading

Great **aung mrat**.

ဒီ method က `QRGenerator` Form ရဲ့ **JTable row selection handler** နဲ့ အဓိကဆက်စပ်နေပါတယ်။

အရင် Chapter 19 မှာ:

```java
tblStudents.getSelectionModel()
.addListSelectionListener(e -> {
    populateFieldsFromSelectedRow();
});
```

ဆိုပြီး JTable row ရွေးတဲ့အချိန် ဒီ method ကို ခေါ်ပါတယ်။

---

# 20.1 Method Purpose

ဒီ method ရဲ့ အလုပ်တွေက:

1. JTable မှာ ရွေးထားတဲ့ Student ကိုသိမ်းယူမယ်
    
2. Register Button ကို disable လုပ်မယ်
    
3. Generate Button ကို enable လုပ်မယ်
    
4. JTable data ကို TextField တွေထဲ ပြန်ထည့်မယ်
    
5. Student ရဲ့ QR PNG ရှိရင် Preview ပြမယ်
    
6. QR မရှိရင် "No QR Generated" ပြမယ်
    

---

# Complete Flow

```text
User Select Student Row

          |
          v

populateFieldsFromSelectedRow()

          |
          v

Get Selected Row Index

          |
          v

Enable Generate Button
Disable Register Button

          |
          v

Copy JTable Data → Form Fields

          |
          v

Check qrcodes/StudentID.png

          |
          +----------------+
          |                |
          v                v

      File Exists       No File

          |                |
          v                v

 Display QR          Show Message

```

---

# Chapter 20.2 — Get Selected Row

Code:

```java
int selectedRow = tblStudents.getSelectedRow();
```

---

`JTable` ထဲမှာ user ရွေးထားတဲ့ row number ကိုယူပါတယ်။

Example:

Table:

```
------------------------------------------------
No | StudentID | Name
------------------------------------------------
0  | ST001     | Aung
1  | ST002     | Mg Mg
2  | ST003     | Su Su
------------------------------------------------
```

User က ST002 ကို click လုပ်ရင်:

```java
selectedRow = 1;
```

---

မရွေးထားရင်:

```java
selectedRow = -1;
```

ဖြစ်ပါတယ်။

---

# Chapter 20.3 — Check Row Selected

Code:

```java
if (selectedRow != -1)
```

---

Meaning:

"Student တစ်ယောက်ရွေးထားလား?"

---

## Case 1: Selected

```
selectedRow = 1
```

ဖြစ်ရင်:

```java
if block
```

ထဲဝင်မယ်။

---

## Case 2: Not Selected

```
selectedRow = -1
```

ဖြစ်ရင်:

```java
else
```

သွားမယ်။

---

# Chapter 20.4 — Enable Generate Button

Code:

```java
btnGenerate.setEnabled(true);
```

---

အစမှာ:

```java
btnGenerate.setEnabled(false);
```

လုပ်ထားခဲ့တယ်။

ဘာကြောင့်လဲ?

Student မရွေးရသေးလို့ပါ။

---

Flow:

Before:

```
No Student Selected


[ Generate QR ]  Disabled

```

---

After:

```
Student Selected


[ Generate QR ]  Enabled

```

---

# Chapter 20.5 — Disable Register Button

Code:

```java
btnRegister.setEnabled(false);
```

---

ဒီဟာက UI state control ဖြစ်ပါတယ်။

---

ဘာကြောင့် Register ကို disable လုပ်တာလဲ?

User က Student တစ်ယောက်ရွေးထားပြီးသားဆိုရင်:

- Register = New Student
    
- Generate = Existing Student
    

ဖြစ်လို့ပါ။

---

State:

```
No Selection:

Register  ON
Generate OFF


Selected:

Register  OFF
Generate ON

```

---

ဒါက Professional CRUD UI pattern ဖြစ်ပါတယ်။

---

# Chapter 20.6 — Load Student Data Into Form

## Student ID

```java
txtStdId.setText(
    getCellValue(selectedRow,1)
);
```

---

JTable Column:

```
Index

0 = No
1 = Student ID
2 = Name
3 = Age
4 = Gender
5 = Department
6 = Email
7 = Token

```

---

Example:

Table:

```
ST001 | Aung | Male | IT
```

Column 1:

```
ST001
```

ရမယ်။

---

TextField:

Before:

```
Student ID:

[           ]

```

After:

```
Student ID:

[ ST001 ]

```

---

# Name

Code:

```java
txtName.setText(
    getCellValue(selectedRow,2)
);
```

---

Result:

```
Name:

[Aung]

```

---

# Gender

Code:

```java
cbGender.setSelectedItem(
    getCellValue(selectedRow,4)
);
```

---

ComboBox ဖြစ်လို့:

```java
setText()
```

မသုံးဘူး။

---

Example:

Before:

```
Gender

[Select ▼]

```

After:

```
Gender

[Male ▼]

```

---

# Department

```java
txtDepartment.setText(
getCellValue(selectedRow,5)
);
```

---

# Email

```java
txtEmail.setText(
getCellValue(selectedRow,6)
);
```

---

အခုဆိုရင်:

JTable:

```
ST001
Aung
Male
IT
aung@gmail.com

```

က

Form:

```
Student ID:
ST001

Name:
Aung

Gender:
Male

Department:
IT

Email:
aung@gmail.com

```

ဖြစ်သွားပြီ။

---

# Chapter 20.7 — Get Student ID

Code:

```java
String studentId =
txtStdId.getText().trim();
```

---

ဘာကြောင့် TextField ကနေ ပြန်ယူတာလဲ?

အခုလေးတင်:

```java
txtStdId.setText(...)
```

လုပ်ထားပြီးသားဖြစ်လို့။

---

Example:

```java
studentId="ST001";
```

---

# Chapter 20.8 — Locate QR Image File

Code:

```java
File imgFile =
new File(
"qrcodes/" + studentId + ".png"
);
```

---

Student ID:

```
ST001
```

ဆိုရင်:

Path:

```
qrcodes/ST001.png
```

ဖြစ်မယ်။

---

Project Structure:

```
Project

 |
 +-- qrcodes

        |
        +-- ST001.png
        +-- ST002.png

```

---

# Chapter 20.9 — Check QR Exists

Code:

```java
if (!studentId.isEmpty()
    &&
    imgFile.exists())
```

---

Condition ၂ ခုစစ်တယ်။

---

## Condition 1

```java
!studentId.isEmpty()
```

Meaning:

Student ID ရှိရမယ်။

---

## Condition 2

```java
imgFile.exists()
```

Meaning:

PNG file တကယ်ရှိရမယ်။

---

Example:

```
Student ID = ST001

File:
qrcodes/ST001.png

exists = true

```

ဆိုရင် QR ပြမယ်။

---

# Chapter 20.10 — Load QR Image

Code:

```java
ImageIcon icon =
new ImageIcon(
imgFile.getAbsolutePath()
);
```

---

Disk ကနေ image ဖတ်တယ်။

Example:

```
C:\Project\qrcodes\ST001.png

```

---

Image object ဖြစ်လာမယ်။

---

# Chapter 20.11 — Resize QR Image

Code:

```java
Image scaled =
icon.getImage()
.getScaledInstance(
DISPLAY_QR_SIZE,
DISPLAY_QR_SIZE,
Image.SCALE_SMOOTH
);
```

---

ဘာကြောင့် resize လုပ်တာလဲ?

Original QR:

```
1000 x 1000 px
```

ဖြစ်နိုင်တယ်။

JLabel:

```
200 x 200
```

ပဲရှိတယ်။

---

Resize:

```
1000x1000

       |
       v

200x200

```

---

`SCALE_SMOOTH`

က quality ကောင်းအောင် resize လုပ်ပေးတယ်။

---

# Chapter 20.12 — Display QR Preview

Code:

```java
lblQRcode.setIcon(
new ImageIcon(scaled)
);

lblQRcode.setText("");
```

---

Before:

```
+----------------+
| No QR Generated|
+----------------+

```

---

After:

```
+----------------+
|                |
|    █▀█ █▀█     |
|    QR CODE     |
|                |
+----------------+

```

---

# Chapter 20.13 — If QR Does Not Exist

Else:

```java
else {

    lblQRcode.setIcon(null);

    lblQRcode.setText(
        "No QR Generated"
    );

}
```

---

Meaning:

Student registered ဖြစ်ပေမယ့် QR မထုတ်ရသေးဘူး။

---

Example:

Database:

```
Student:
ST001

qrToken:
null

```

File:

```
qrcodes/ST001.png

(no file)

```

Result:

```
No QR Generated

```

---

# Chapter 20.14 — No Row Selected Case

Code:

```java
else {

btnGenerate.setEnabled(false);

btnRegister.setEnabled(true);

}
```

---

ဒီအခြေအနေ:

```
Nothing selected

```

---

UI:

```
Register  ON

Generate  OFF

```

---

ဘာကြောင့်?

User က Student အသစ်ထည့်ချင်နိုင်လို့။

---

# Full Interaction Example

## Step 1 — Open Form

```
Register  Enabled

Generate Disabled

Table Loaded

```

---

## Step 2 — Select Student

User click:

```
ST001
Aung
```

---

Method runs:

```
populateFieldsFromSelectedRow()

```

---

Result:

```
Student ID:
ST001

Name:
Aung


Register Disabled

Generate Enabled

```

---

## Step 3 — QR Exists

File:

```
qrcodes/ST001.png

```

ရှိရင်:

```
Preview QR

```

---

## Step 4 — QR Not Exists

```
No QR Generated

```

---

# Professional Architecture View

ဒီ method က MVC မှာ:

```
View Layer
(JTable + TextFields)

        |
        v

Controller Logic
(populateFieldsFromSelectedRow)

        |
        v

Model Data
(Student)

        |
        v

File System
(QR PNG)

```

---

# Senior Developer Analysis

ဒီ method မှာ ပါတဲ့ Concepts:

|Concept|Usage|
|---|---|
|Event Driven Programming|JTable selection|
|UI State Management|Enable/Disable buttons|
|Data Binding|JTable → TextField|
|File Validation|QR existence check|
|Image Processing|Resize preview|
|MVC Pattern|Separate UI logic|
|User Experience|Auto populate|

---

အခု `QRGenerator` Form ရဲ့ main flow ကိုကြည့်ရင်:

```
initCustomLogic()
        |
        |
        v
loadStudentData()

        |
        v

User Select Row

        |
        v

populateFieldsFromSelectedRow()

        |
        v

Generate Button Enabled

        |
        v

btnGenerateActionPerformed()

        |
        v

AES Encrypted QR Created

```

ဖြစ်ပါတယ်။

ဒီ flow က Student QR Identity System တစ်ခုအတွက် တော်တော် professional ဖြစ်တဲ့ design ဖြစ်ပါတယ်။