# Part 3: Café POS Real Implementation Phase

# Lesson 31: Café POS Project Setup

## Maven Project + Java 25 Configuration + Professional Package Structure

### (Java 25 + Swing + FlatLaf + MySQL + HikariCP + MVC)

ဒီ Lesson ကနေစပြီး **Design Phase ပြီးပြီး Implementation Phase** ကို စတင်ပါမယ်။

အခုကနေ:

> "Architecture ပြောနေတဲ့အဆင့်"  
> ↓  
> "တကယ် Code ရေးတဲ့အဆင့်"

ကို ဝင်ပါမယ်။

---

# Lesson 31 Goals

ဒီနေ့ Build လုပ်မယ့်အရာတွေ:

✅ Maven Project Create  
✅ Java 25 Configuration  
✅ pom.xml Setup  
✅ Professional Package Structure  
✅ Resource Folder Setup  
✅ Dependency Management  
✅ First Application Bootstrap  
✅ First Swing Window Test

---

# 1. Development Environment

Recommended:

|Tool|Version|
|---|---|
|Java|JDK 25|
|Build Tool|Maven 3.9+|
|Database|MySQL 8+|
|IDE|IntelliJ IDEA|
|UI|Swing + FlatLaf|
|Version Control|Git|

---

# 2. Verify Java 25

Terminal:

```bash
java --version
```

Expected:

```text
java 25

Java(TM) SE Runtime Environment

```

---

Maven:

```bash
mvn -version
```

Expected:

```text
Apache Maven 3.9.x

Java version: 25

```

---

# 3. Create Maven Project

Command:

```bash
mvn archetype:generate \
-DgroupId=com.cafe.pos \
-DartifactId=cafe-pos \
-DarchetypeArtifactId=maven-archetype-quickstart \
-DinteractiveMode=false

```

---

Project:

```text
cafe-pos

├── pom.xml

└── src

```

---

# 4. Final Project Structure

We will change to:

```text
cafe-pos

│
├── pom.xml
│
├── src
│
│   ├── main
│   │
│   │   ├── java
│   │   │
│   │   │   └── com
│   │   │       └── cafe
│   │   │           └── pos
│   │   │
│   │   └── resources
│   │
│   └── test
│
└── README.md


```

---

# 5. Create Package Structure

Inside:

```text
com.cafe.pos

```

Create:

```
app

config

database

exception

security

common

module

view

```

---

Full:

```text
com.cafe.pos


├── app

├── config

├── database

├── exception

├── security

├── common


├── module

│
│── auth

│── product

│── inventory

│── order

│── payment

│── report


└── view


```

---

# 6. Maven pom.xml

Replace your pom.xml:

```xml
<project 
xmlns="http://maven.apache.org/POM/4.0.0">


<modelVersion>
4.0.0
</modelVersion>


<groupId>
com.cafe.pos
</groupId>


<artifactId>
cafe-pos
</artifactId>


<version>
1.0-SNAPSHOT
</version>



<properties>


<maven.compiler.source>
25
</maven.compiler.source>


<maven.compiler.target>
25
</maven.compiler.target>


<project.build.sourceEncoding>
UTF-8
</project.build.sourceEncoding>


</properties>



<dependencies>


<!-- MySQL Driver -->

<dependency>

<groupId>
com.mysql
</groupId>

<artifactId>
mysql-connector-j
</artifactId>

<version>
9.4.0
</version>

</dependency>



<!-- HikariCP -->

<dependency>

<groupId>
com.zaxxer
</groupId>

<artifactId>
HikariCP
</artifactId>

<version>
6.3.0
</version>

</dependency>



<!-- FlatLaf UI -->

<dependency>

<groupId>
com.formdev
</groupId>

<artifactId>
flatlaf
</artifactId>

<version>
3.6
</version>

</dependency>



<!-- Logging -->


<dependency>

<groupId>
org.slf4j
</groupId>

<artifactId>
slf4j-api
</artifactId>

<version>
2.0.17
</version>

</dependency>



<dependency>

<groupId>
ch.qos.logback
</groupId>

<artifactId>
logback-classic
</artifactId>

<version>
1.5.18
</version>

</dependency>



<!-- Testing -->


<dependency>

<groupId>
org.junit.jupiter
</groupId>

<artifactId>
junit-jupiter
</artifactId>

<version>
5.12.2
</version>

<scope>
test
</scope>

</dependency>



</dependencies>


</project>

```

---

# 7. Resource Structure

Create:

```text
src/main/resources


├── database.properties

├── application.properties

├── logback.xml

└── images

```

---

# 8. Application Configuration

File:

```
application.properties
```

Content:

```properties
app.name=Cafe POS System

app.version=1.0

app.author=Development Team

```

---

# 9. Database Configuration

File:

```
database.properties
```

Content:

```properties
db.url=jdbc:mysql://localhost:3306/cafe_pos

db.username=root

db.password=password

db.pool.size=10

```

---

# 10. Create Main Application Class

Location:

```
app/Application.java

```

---

Code:

```java
package com.cafe.pos.app;


import javax.swing.SwingUtilities;

import com.formdev.flatlaf.FlatLightLaf;


public class Application {


public static void main(String[] args){


setupUI();


startApplication();


}



private static void setupUI(){


FlatLightLaf.setup();


}



private static void startApplication(){


SwingUtilities.invokeLater(
()->{


System.out.println(
"Café POS Started"
);



}
);


}


}

```

---

Run:

Output:

```text
Café POS Started

```

---

# 11. Create First Swing Window

Create:

```
view/MainFrame.java

```

---

Code:

```java
package com.cafe.pos.view;


import javax.swing.*;


public class MainFrame
extends JFrame {



public MainFrame(){


setTitle(
"Café POS System"
);


setSize(
1200,
700
);


setLocationRelativeTo(null);


setDefaultCloseOperation(
EXIT_ON_CLOSE
);


initialize();


}



private void initialize(){


JLabel label =
new JLabel(
"Welcome to Café POS"
);



label.setHorizontalAlignment(
SwingConstants.CENTER
);



add(label);


}


}

```

---

Update Application:

```java
private static void startApplication(){


SwingUtilities.invokeLater(
()->{


new MainFrame()
.setVisible(true);



}
);


}

```

---

Run:

Result:

```
+--------------------------------+

        Welcome to Café POS


+--------------------------------+

```

---

# 12. Create Application Constants

Package:

```
common

Constants.java

```

---

Code:

```java
package com.cafe.pos.common;


public final class Constants {


private Constants(){}



public static final String APP_NAME =
"Café POS";


public static final String VERSION =
"1.0";


}

```

---

# 13. Why Constants Class?

Bad:

```java
setTitle(
"Café POS"
);

```

Repeated everywhere.

---

Good:

```java
setTitle(
Constants.APP_NAME
);

```

---

# 14. Add Git

Initialize:

```bash
git init

```

---

Create:

```
.gitignore

```

Content:

```text
target/

.idea/

*.iml

logs/

*.class

```

---

First commit:

```bash
git add .

git commit -m "initial project setup"

```

---

# 15. Project Dependency Flow

Now:

```text
Application

    |

MainFrame

    |

Swing UI


```

Later:

```text
Application


 |

Controllers


 |

Services


 |

Repositories


 |

Database

```

---

# 16. Current Project Status

Completed:

```
Java 25 Setup          ✅

Maven Project          ✅

Dependencies           ✅

Package Architecture   ✅

Resources              ✅

FlatLaf Theme           ✅

First Swing Window     ✅

Git Setup              ✅

```

---

# 17. Professional Rule

From now:

Never put business code inside:

❌ JFrame  
❌ JPanel  
❌ JButton ActionListener

Example Bad:

```java
button.addActionListener(e->{


SELECT * FROM products;


});

```

---

Correct:

```text
Button

↓

Controller

↓

Service

↓

Repository

↓

Database

```

---

# Lesson 31 Practice

Create manually:

1. Maven Project
    
2. Package Structure
    
3. pom.xml
    
4. Application.java
    
5. MainFrame.java
    
6. Configuration Files
    
7. Git Repository
    

---

# Next Lesson

# Lesson 32: Database Layer Implementation

## MySQL + HikariCP + Connection Management + Transaction Manager

Next we will build:

✅ Database Connection Pool  
✅ DatabaseManager  
✅ Connection Provider  
✅ Transaction Manager  
✅ SQL Migration Runner  
✅ Database Exception Handling

ပြီးရင် Café POS က **Java Code ↔ MySQL တကယ်ချိတ်ဆက်နိုင်တဲ့ Stage** ကို ရောက်ပါမယ်။