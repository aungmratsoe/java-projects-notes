# Part 2: Advanced Java Knowledge for Swing

# Lesson 13: Java File I/O & Serialization Deep Dive

## Building Export, Backup & Configuration System for Café POS

ဒီ Lesson မှာ Java ရဲ့ **File Input/Output (I/O)** နဲ့ **Serialization** ကို လေ့လာပါမယ်။

Café POS System မှာ Database တစ်ခုတည်းကိုပဲ မမှီခိုပါဘူး။

Real-world Application တွေမှာ:

- Sales Report Export
    
- Receipt File Generate
    
- Backup Data
    
- Configuration Save
    
- Log File Writing
    
- Import/Export Feature
    

တွေလိုအပ်ပါတယ်။

---

# 1. What is File I/O?

I/O ဆိုတာ:

```
Input  = Data ကို ဖတ်ခြင်း
Output = Data ကို ရေးခြင်း
```

Java Flow:

```
Application

    |
    |
    v

File System

    |
    |
    v

Data Storage

```

---

Example:

POS System:

```
Sales Data

    |
    v

sales_report.csv

```

---

# 2. Java I/O Package

Main Packages:

```java
java.io
```

and

```java
java.nio.file
```

---

Old I/O:

```
File
FileReader
FileWriter
BufferedReader
BufferedWriter
```

---

Modern I/O:

```
Path
Files
```

(Java 7+)

---

# 3. File Class

File object က File ကို represent လုပ်ပါတယ်။

Example:

```java
import java.io.File;


File file =
new File("sales.txt");

```

---

Check File Exists:

```java
if(file.exists()){

System.out.println("Found");

}

```

---

Get Information:

```java
file.getName();

file.length();

file.getPath();

```

---

# 4. Creating File

Example:

```java
File file =
new File("report.txt");


file.createNewFile();

```

Result:

```
project

 |
 |
 report.txt

```

---

# 5. Writing File

## FileWriter

Example:

```java
FileWriter writer =
new FileWriter(
"sales.txt"
);


writer.write(
"Coffee 5000"
);


writer.close();

```

---

File:

```
sales.txt


Coffee 5000

```

---

# 6. Problem with FileWriter

Large data ရေးရင်:

- Slow
    
- Many disk access
    

ဖြစ်နိုင်ပါတယ်။

Solution:

```
BufferedWriter
```

---

# 7. BufferedWriter

Example:

```java
BufferedWriter writer =
new BufferedWriter(
new FileWriter(
"sales.txt"
)
);


writer.write(
"Latte 6000"
);


writer.newLine();


writer.close();

```

---

Advantages:

- Faster
    
- Less disk operation
    

---

# 8. Reading File

## FileReader

Example:

```java
FileReader reader =
new FileReader(
"sales.txt"
);


int data;


while(
(data = reader.read())
!=-1
){

System.out.print(
(char)data
);

}

reader.close();

```

---

# 9. BufferedReader

Better:

```java
BufferedReader reader =
new BufferedReader(
new FileReader(
"sales.txt"
)
);


String line;


while(
(line=reader.readLine())
!=null
){

System.out.println(line);

}


reader.close();

```

---

# 10. try-with-resources

Professional Java မှာ:

Old:

```java
FileReader reader =
new FileReader();


reader.close();

```

Problem:

Exception ဖြစ်ရင် close မလုပ်နိုင်။

---

Modern:

```java
try(
BufferedReader reader =
new BufferedReader(
new FileReader(
"sales.txt"
)
)
){


String line =
reader.readLine();


}

```

---

Java က automatically close လုပ်ပေးပါတယ်။

---

# 11. Java NIO Files API

Modern Java မှာ:

```
java.nio.file.Files
```

ကို ပိုသုံးပါတယ်။

---

# 12. Reading Entire File

Example:

```java
Path path =
Paths.get(
"sales.txt"
);


String data =
Files.readString(path);

```

---

Output:

```
Coffee 5000

Cake 7000

```

---

# 13. Writing File with Files

Example:

```java
Path path =
Paths.get(
"report.txt"
);


Files.writeString(
path,
"Daily Report"
);

```

---

Simple and clean.

---

# 14. Café POS: Export CSV Report

Requirement:

Daily Sales Export:

```
sales_2026_07_31.csv


ID,Product,Amount

1,Coffee,5000

2,Cake,7000

```

---

Code:

```java
public void exportCSV(
List<Sale> sales
)
throws IOException{


BufferedWriter writer =
Files.newBufferedWriter(
Paths.get(
"sales.csv"
)
);


writer.write(
"ID,Product,Amount"
);


writer.newLine();


for(Sale s:sales){


writer.write(
s.getId()
+","
+s.getProduct()
+","
+s.getAmount()
);


writer.newLine();

}


writer.close();

}

```

---

# 15. File Append

Existing file ထဲကို ထပ်ရေးချင်ရင်:

```java
FileWriter writer =
new FileWriter(
"log.txt",
true
);

```

`true`

means:

```
append mode
```

---

Example:

```
log.txt

10:00 Login

10:05 Sale

10:10 Logout

```

---

# 16. Directory Handling

Create Folder:

```java
Files.createDirectory(
Paths.get("backup")
);

```

---

Check:

```java
Files.exists(path);

```

---

Delete:

```java
Files.delete(path);

```

---

# 17. Serialization Concept

Serialization ဆိုတာ:

> Java Object ကို byte stream အဖြစ်ပြောင်းပြီး File ထဲသိမ်းခြင်း ဖြစ်ပါတယ်။

Flow:

```
Object

   |
   |
serialize()

   |
   v

Binary File

```

---

Deserialize:

```
Binary File

   |
   |
deserialize()

   |
   v

Object

```

---

# 18. Serializable Interface

Example:

```java
import java.io.Serializable;


class Product
implements Serializable{


private int id;

private String name;


}

```

---

Serializable:

Marker Interface ဖြစ်ပါတယ်။

Method မရှိပါ။

---

# 19. ObjectOutputStream

Write Object:

```java
Product product =
new Product();


ObjectOutputStream out =
new ObjectOutputStream(
new FileOutputStream(
"product.dat"
)
);


out.writeObject(product);


out.close();

```

---

Result:

```
product.dat

(binary)

```

---

# 20. ObjectInputStream

Read Object:

```java
ObjectInputStream in =
new ObjectInputStream(
new FileInputStream(
"product.dat"
)
);


Product product =
(Product)
in.readObject();


in.close();

```

---

# 21. Café POS Backup System

Example:

Backup:

```
All Products

All Customers

All Sales


        |

        v


backup.dat

```

---

Model:

```java
class DatabaseBackup
implements Serializable{


List<Product> products;

List<Sale> sales;


}

```

---

Save:

```java
ObjectOutputStream out =
new ObjectOutputStream(
new FileOutputStream(
"backup.dat"
)
);


out.writeObject(
backup
);

```

---

Restore:

```java
DatabaseBackup backup =
(DatabaseBackup)
in.readObject();

```

---

# 22. serialVersionUID

Professional Topic.

Example:

```java
private static final long
serialVersionUID = 1L;

```

---

Why?

Class structure change ဖြစ်တဲ့အခါ compatibility စစ်ရန်။

---

Example:

Version 1:

```
Product

id
name

```

Version 2:

```
Product

id
name
price

```

---

# 23. transient Keyword

Some data မသိမ်းချင်ရင်:

Example:

```java
class User
implements Serializable{


String username;


transient String password;


}

```

---

Serialize:

```
username
saved


password
ignored

```

---

Security အတွက်သုံးပါတယ်။

---

# 24. Serialization vs Database

|Serialization|Database|
|---|---|
|Object save|Structured data|
|Small backup|Large data|
|Java specific|Multi-language|
|Fast local|Scalable|

---

Real Application:

Use both.

```
MySQL

   |
Main Data


Backup File

   |
Emergency Restore

```

---

# 25. Configuration File

POS Setting:

```
config.properties


database.url=localhost

theme=dark

printer=A4

```

---

Java:

```java
Properties prop =
new Properties();


prop.load(
new FileInputStream(
"config.properties"
)
);


String theme =
prop.getProperty(
"theme"
);

```

---

# 26. Logging System

Professional Application:

```
Application

   |

Log File


application.log

```

---

Example:

```java
Files.writeString(
path,
"Order Created"
);

```

---

(Real projects use Logging Frameworks like Log4j/SLF4J)

---

# 27. Café POS File Architecture

Professional:

```
CafePOS

 |

 |--database

 |

 |--backup

 |     |
 |     backup.dat

 |

 |--reports

 |     |
 |     sales.csv

 |

 |--logs

       |
       app.log

```

---

# 28. Exception Handling with File I/O

File operations fail:

Example:

```java
try{


Files.readString(path);


}
catch(IOException e){


throw new FileOperationException(
"Cannot read report"
);


}

```

---

Custom Exception:

```java
public class FileOperationException
extends RuntimeException{


public FileOperationException(
String message
){

super(message);

}

}

```

---

# 29. File I/O Best Practices

## 1. Always close resources

Use:

```
try-with-resources
```

---

## 2. Use NIO for modern applications

Prefer:

```java
Files

Path

```

---

## 3. Don't store sensitive data directly

Avoid:

```
password.txt

```

---

## 4. Handle IOException properly

Never:

```java
catch(Exception e){

}

```

---

# 30. Interview Questions

## Q1: Difference between FileReader and BufferedReader?

FileReader:

- Direct reading
    

BufferedReader:

- Faster
    
- Reads line by line
    

---

## Q2: What is Serialization?

Converting object into byte stream.

---

## Q3: Why Serializable interface?

To allow object serialization.

---

## Q4: transient keyword?

Prevent field serialization.

---

# Practice Project

Build Café POS Export & Backup Module:

Create:

```
ReportExporter

BackupManager

ConfigManager

LogManager

```

Features:

### 1. Export Sales

```
sales.csv
```

### 2. Backup Data

```
backup.dat
```

### 3. Restore Backup

```
backup.dat
    |
Objects
```

### 4. Save Settings

```
config.properties

```

Use:

- Files API
    
- BufferedWriter
    
- Serialization
    
- Exception Handling
    

---

# Lesson 13 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ File Class  
✅ FileWriter  
✅ BufferedWriter  
✅ BufferedReader  
✅ Files API  
✅ Path  
✅ CSV Export  
✅ Directory Handling  
✅ Serialization  
✅ Serializable  
✅ ObjectOutputStream  
✅ ObjectInputStream  
✅ transient  
✅ serialVersionUID  
✅ Backup Architecture  
✅ POS Export System

---

# Next Lesson

# Lesson 14: Java Networking & API Communication

## Connecting Café POS with External Services

သင်ယူမည့်အရာ:

- Socket Programming
    
- HTTP Client
    
- REST API Concept
    
- JSON Communication
    
- API Request/Response
    
- Online Payment Integration Concept
    
- Server Communication
    

Example:

```
POS Application

      |

 REST API

      |

Payment Server

```

ပြီးရင် Desktop POS ကို Network Application အဖြစ် Design လုပ်နိုင်ပါမယ်။