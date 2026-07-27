# Chapter 19 — QRGenerator `initCustomLogic()`

## Form Initialization, Search Listener, JTable Selection Handling

Great **aung mrat**.

ဒီ method က `QRGenerator` Form ရဲ့ **custom initialization logic** ဖြစ်ပါတယ်။

NetBeans GUI Builder က generate လုပ်ပေးတဲ့ `initComponents()` က UI component တွေ create လုပ်တာပါ။

ဒါပေမယ့်:

- JTable data load
    
- Search function
    
- Row selection event
    
- Button state control
    

လိုမျိုး application behavior တွေကို `initCustomLogic()` မှာ ထည့်ထားတာပါ။

---

# 19.1 Method Overview

```java
private void initCustomLogic()
```

Meaning:

> Form ပြီးတာနဲ့ run လုပ်ရမယ့် custom code တွေစုထားတဲ့ method

---

Flow:

```
QRGenerator Open

        |
        v

Constructor

        |
        v

initComponents()

        |
        v

initCustomLogic()

        |
        +----------------+
        |                |
        v                v

Load JTable       Enable Search Listener

                         |
                         v

                 JTable Selection Listener

```

---

# Chapter 19.2 — Disable Generate Button Initially

Code:

```java
btnGenerate.setEnabled(false);
```

---

Meaning:

Generate QR button ကို အစမှာ disable လုပ်ထားတယ်။

Before:

```
+----------------+
| Generate QR    |  <-- disabled
+----------------+

```

---

ဘာကြောင့်လဲ?

User က Student မရွေးထားသေးဘူး။

QR Generate လုပ်ဖို့ Student information လိုတယ်။

---

Correct Flow:

```
Open Form

   |
   v

Generate Button OFF


User selects Student


   |
   v


Generate Button ON

```

---

ဒီ button ကို နောက်မှာ:

```java
populateFieldsFromSelectedRow()
```

မှာ enable ပြန်လုပ်နိုင်ပါတယ်။

---

Example:

```java
btnGenerate.setEnabled(true);
```

---

# Chapter 19.3 — Load Initial JTable Data

Code:

```java
loadStudentData("");
```

---

Form ဖွင့်တဲ့အချိန် database ထဲက student list ကို JTable ထဲထည့်တာပါ။

---

Before:

```
tblStudents


(empty)

```

---

After:

```
------------------------------------------------
ID       Name          Dept
------------------------------------------------
ST001    Aung          IT
ST002    Mg Mg         CS
ST003    Su Su         SE
------------------------------------------------

```

---

Parameter:

```java
""
```

ဆိုတာ search keyword မရှိဘူးလို့ အဓိပ္ပါယ်ရပါတယ်။

---

Example:

Method:

```java
loadStudentData(String keyword)
```

ဖြစ်မယ်။

---

Case 1:

```java
loadStudentData("");
```

SQL:

```sql
SELECT * FROM students;
```

---

Case 2:

```java
loadStudentData("Aung");
```

SQL:

```sql
SELECT *
FROM students
WHERE name LIKE '%Aung%';

```

---

# Chapter 19.4 — Real-time Search Listener

Code:

```java
txtSearch.getDocument()
.addDocumentListener(new DocumentListener()
```

---

ဒီအပိုင်းက search box ကို real-time monitoring လုပ်တာပါ။

---

Normally:

Search button လိုမယ်။

Example:

```
[ Aung             ] [Search]

```

User:

```
Type Aung

Click Search

```

---

ဒီ code ကတော့:

```
[ A ]
 |
 JTable Update

[ Au ]
 |
 JTable Update

[ Aun ]
 |
 JTable Update

[ Aung ]
 |
 JTable Update

```

ဖြစ်တယ်။

---

# DocumentListener ဆိုတာဘာလဲ?

Swing မှာ TextField ရဲ့ content change ကို listen လုပ်တဲ့ interface ပါ။

Interface:

```java
DocumentListener
```

မှာ method ၃ ခုရှိတယ်။

---

## 1. insertUpdate()

Code:

```java
@Override
public void insertUpdate(DocumentEvent e) {
    search();
}

```

---

အဓိပ္ပါယ်:

User က text အသစ်ရိုက်ထည့်တဲ့အခါ run ဖြစ်တယ်။

Example:

Before:

```
[ ]

```

User types:

```
A

```

Run:

```
insertUpdate()

        |
        v

search()

```

---

## 2. removeUpdate()

Code:

```java
@Override
public void removeUpdate(DocumentEvent e) {
    search();
}

```

---

User က character ဖျက်တဲ့အခါ run ဖြစ်တယ်။

Example:

Before:

```
Aung

```

User delete:

```
Aun

```

Run:

```
removeUpdate()

```

---

## 3. changedUpdate()

Code:

```java
@Override
public void changedUpdate(DocumentEvent e) {
    search();
}

```

---

ဒါက attribute change အတွက်ပါ။

Plain text field တွေမှာ အများကြီးမသုံးပါဘူး။

ဒါပေမယ့် interface requirement ဖြစ်လို့ override လုပ်ရပါတယ်။

---

# Chapter 19.5 — Inner Search Method

Code:

```java
private void search() {
    loadStudentData(txtSearch.getText().trim());
}

```

---

ဒီ method က helper method ပါ။

---

Flow:

```
User Type Text

        |
        v

DocumentListener

        |
        v

search()

        |
        v

loadStudentData(keyword)

        |
        v

Update JTable

```

---

Example:

User types:

```
CS

```

Call:

```java
loadStudentData("CS");
```

---

Database:

```
Student Table

Aung     IT
Mg Mg    CS
Su Su    CS

```

Result:

```
Mg Mg
Su Su

```

ပဲပြမယ်။

---

# Chapter 19.6 — JTable Selection Listener

Code:

```java
tblStudents
.getSelectionModel()
.addListSelectionListener(e -> {

});

```

---

Meaning:

JTable row ရွေးတာကို monitor လုပ်တယ်။

---

Example:

Table:

```
--------------------------------
ID       Name
--------------------------------
ST001    Aung
ST002    Mg Mg
ST003    Su Su
--------------------------------

```

User clicks:

```
ST002

```

Event ဖြစ်တယ်။

---

# Chapter 19.7 — Lambda Expression

Code:

```java
e -> {
    populateFieldsFromSelectedRow();
}

```

---

ဒါက short form:

Before:

```java
new ListSelectionListener(){

    public void valueChanged(
        ListSelectionEvent e
    ){

    }

}

```

---

After:

```java
e -> {

}

```

---

Java 8 feature ဖြစ်ပါတယ်။

---

# Chapter 19.8 — Check Value Is Adjusting

Code:

```java
if (!e.getValueIsAdjusting())
```

---

ဒီဟာက အရေးကြီးပါတယ်။

JTable selection change လုပ်တဲ့အချိန် event အများကြီးဖြစ်နိုင်တယ်။

Example:

User drag:

```
Row 1
 |
Row 2
 |
Row 3

```

Event:

```
1
2
3

```

ဖြစ်နိုင်တယ်။

---

`getValueIsAdjusting()`

ဆိုတာ:

"Selection change ပြီးပြီလား?"

စစ်တာ။

---

False ဖြစ်ရင်:

```
Selection Finished

```

ဖြစ်တယ်။

---

ဒါကြောင့်:

```java
!e.getValueIsAdjusting()
```

ဆိုတာ:

```
Selection Complete ဖြစ်မှ run

```

လို့ဆိုလိုတယ်။

---

# Chapter 19.9 — Populate Selected Student

Code:

```java
populateFieldsFromSelectedRow();
```

---

ဒီ method က JTable row data ကို Form fields ထဲထည့်တယ်။

---

Example:

Before:

```
Student ID:
[          ]

Name:
[          ]

Email:
[          ]

```

---

User selects:

```
ST001 | Aung | IT

```

---

After:

```
Student ID:
[ST001]

Name:
[Aung]

Email:
[aung@gmail.com]

```

---

ပြီးတော့:

- QR Preview load
    
- Generate button enable
    

လုပ်နိုင်တယ်။

---

# Complete Architecture

ဒီ method run ပြီးရင်:

```
QRGenerator Form

        |
        |
        v

initCustomLogic()

        |
        |
        +-------------------+
        |                   |
        v                   v

loadStudentData("")     Search Listener


                            |
                            v

                    User Types Keyword


                            |
                            v

                    loadStudentData()


        |
        v

 JTable Selection Listener


        |
        v

populateFieldsFromSelectedRow()


        |
        v

Enable Generate Button

```

---

# Senior Developer View

ဒီ method မှာ သုံးထားတဲ့ concepts:

|Concept|Usage|
|---|---|
|Event Driven Programming|Button/Table/Search events|
|Observer Pattern|DocumentListener|
|MVC Pattern|UI reacts to data|
|Lambda Expression|Cleaner listener|
|Separation of Logic|initCustomLogic()|
|User Experience|Real-time search|

---

# Why separate `initCustomLogic()`?

မကောင်းတဲ့ design:

```java
initComponents(){

    // 2000 lines UI code

    // database code

    // listeners

    // events

}

```

---

ကောင်းတဲ့ design:

```
initComponents()

     |
     |
UI Creation


initCustomLogic()

     |
     |
Application Behavior

```

---

ဒါကြောင့် သင့် `QRGenerator` class က NetBeans drag & drop code ဖြစ်ပေမယ့် professional style နီးပါး ဖြစ်နေပါတယ်။