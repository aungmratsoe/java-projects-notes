
ဟုတ်တယ် **aung mrat** — မင်းရဲ့ လက်ရှိ structure ကိုကြည့်ရင် **Multi-Module Maven (`common`, `client`, `server`) ခွဲဖို့ အရမ်းသင့်တော်တယ်**။

အထူးသဖြင့် မင်းက **Java Swing Client + RMI Server + MySQL** architecture သုံးထားတာဆိုတော့ ဒီလိုခွဲလိုက်ရင် architecture က အများကြီးသန့်သွားမယ်။

---

# 1. လက်ရှိ Project ကို အရင်နားလည်ကြည့်မယ်

လက်ရှိမှာ အားလုံးက project တစ်ခုထဲမှာရှိတယ်—

```text
com.ams.sm
│
├── client
│   ├── ConfigLoader
│   └── RmiConnectionManager
│
├── dao
│   ├── FamilyMemberDAO
│   ├── SpecializationDAO
│   ├── StudentDAO
│   └── impl
│
├── db
│   └── DBConnection
│
├── model
│   ├── FamilyMemberModel
│   ├── SpecializationModel
│   ├── StudentModel
│   └── StudentSpecializationModel
│
├── panel
│   ├── StudentRegister
│   ├── FamilyMember
│   └── Specialization
│
├── server
│   └── ServerMain
│
└── service
    ├── FamilyMemberService
    ├── SpecializationService
    ├── StudentService
    └── impl
```

ဒီ structure မှာ အဓိက problem က—

> **Client code နဲ့ Server code တွေဟာ တစ်ခုတည်းသော application/module ထဲမှာ ရောနေတယ်။**

ဒါကြောင့် `common / client / server` ခွဲတာက အရမ်းကောင်းတဲ့ next step ပါ။

---

# 2. မင်း Project အတွက် Recommend လုပ်မယ့် Structure

ကျွန်တော်ဆိုရင် ဒီလိုလုပ်မယ်—

```text
student-management-system
│
├── pom.xml                         ← Parent POM
│
├── common
│   ├── pom.xml
│   └── src
│       └── main
│           └── java
│               └── com
│                   └── ams
│                       └── sm
│                           ├── model
│                           │   ├── StudentModel.java
│                           │   ├── FamilyMemberModel.java
│                           │   ├── SpecializationModel.java
│                           │   └── StudentSpecializationModel.java
│                           │
│                           └── service
│                               ├── StudentService.java
│                               ├── FamilyMemberService.java
│                               └── SpecializationService.java
│
├── client
│   ├── pom.xml
│   └── src
│       └── main
│           ├── java
│           │   └── com
│           │       └── ams
│           │           └── sm
│           │               ├── client
│           │               │   ├── ConfigLoader.java
│           │               │   └── RmiConnectionManager.java
│           │               │
│           │               ├── panel
│           │               │   ├── Default.java
│           │               │   ├── FamilyMember.java
│           │               │   ├── Specialization.java
│           │               │   └── StudentRegister.java
│           │               │
│           │               └── ui
│           │                   └── Home.java
│           │
│           └── resources
│               └── config.properties
│
└── server
    ├── pom.xml
    └── src
        └── main
            ├── java
            │   └── com
            │       └── ams
            │           └── sm
            │               ├── dao
            │               │   ├── StudentDAO.java
            │               │   ├── FamilyMemberDAO.java
            │               │   ├── SpecializationDAO.java
            │               │   │
            │               │   └── impl
            │               │       ├── StudentDAOImpl.java
            │               │       ├── FamilyMemberDAOImpl.java
            │               │       └── SpecializationDAOImpl.java
            │               │
            │               ├── db
            │               │   └── DBConnection.java
            │               │
            │               ├── service
            │               │   └── impl
            │               │       ├── StudentServiceImpl.java
            │               │       ├── FamilyMemberServiceImpl.java
            │               │       └── SpecializationServiceImpl.java
            │               │
            │               └── server
            │                   └── ServerMain.java
            │
            └── resources
                └── specializations.sql
```

ဒါက မင်း project အတွက် အတော် clean ဖြစ်တဲ့ architecture ပါ။

---

# 3. အရေးကြီးဆုံး Rule

ဒီ architecture ကို နားလည်ဖို့ ဒီ rule တစ်ခုကို မှတ်ထားပါ။

```text
COMMON
  ↑
  │
  ├──────── CLIENT
  │
  └──────── SERVER
```

**Common ကို Client နဲ့ Server နှစ်ခုလုံးသုံးမယ်။**

ဒါပေမယ့်—

```text
CLIENT  ─────X─────> SERVER implementation
SERVER  ─────X─────> CLIENT UI
```

ဆိုတဲ့ dependency မဖြစ်သင့်ဘူး။

---

# 4. `common` ထဲမှာ ဘာထည့်မလဲ?

Common ဆိုတာ—

> Client နဲ့ Server နှစ်ခုလုံး မဖြစ်မနေသိထားရတဲ့ classes

တွေထားတဲ့နေရာပါ။

မင်း project မှာ အဓိကအားဖြင့် **Model + Service Interface** ဖြစ်ပါတယ်။

---

## Model

ဒီ 4 ခုကို `common` ထဲထည့်ပါ။

```text
common
└── model
    ├── StudentModel.java
    ├── FamilyMemberModel.java
    ├── SpecializationModel.java
    └── StudentSpecializationModel.java
```

ဘာကြောင့်လဲ?

Client က server ကို—

```java
StudentModel
```

ပို့ရနိုင်တယ်။

Server က client ကို—

```java
StudentModel
```

ပြန်ပို့ရနိုင်တယ်။

ဒါကြောင့် နှစ်ဖက်လုံး Model ကို သိဖို့လိုတယ်။

---

# 5. Service Interface တွေလည်း Common ထဲ

ဒီဟာက **RMI architecture မှာ အရေးကြီးဆုံး** ပါ။

မင်းမှာ—

```java
StudentService
FamilyMemberService
SpecializationService
```

ရှိတယ်။

ဒီ interfaces တွေကို `common` ထဲထားပါ။

```text
common
└── service
    ├── StudentService.java
    ├── FamilyMemberService.java
    └── SpecializationService.java
```

ဥပမာ—

```java
public interface StudentService extends Remote {

    StudentModel getStudentById(int id)
            throws RemoteException;

    boolean saveStudent(StudentModel student)
            throws RemoteException;

}
```

Client က ဒီ interface ကိုသိရမယ်။

Server လည်း ဒီ interface ကိုသိရမယ်။

---

# 6. `server` ထဲမှာ ဘာတွေထားမလဲ?

Server ဆိုတာ—

> Database နဲ့ အလုပ်လုပ်တဲ့အပိုင်း

ဖြစ်ပါတယ်။

ဒါကြောင့်—

### DAO

```text
server
└── dao
    ├── StudentDAO
    ├── FamilyMemberDAO
    ├── SpecializationDAO
    └── impl
```

### Database

```text
server
└── db
    └── DBConnection.java
```

### Service Implementation

```text
server
└── service
    └── impl
        ├── StudentServiceImpl
        ├── FamilyMemberServiceImpl
        └── SpecializationServiceImpl
```

### RMI Server

```text
server
└── server
    └── ServerMain.java
```

---

# 7. ဒီအပိုင်းကို သေချာနားလည်ထား

ဥပမာ—

```java
StudentService
```

က **common** မှာရှိတယ်။

ဒါပေမယ့်—

```java
StudentServiceImpl
```

က **server** မှာရှိတယ်။

Architecture က—

```text
                 COMMON
                    │
            StudentService
                    │
             ┌──────┴──────┐
             │             │
          CLIENT         SERVER
             │             │
             │      StudentServiceImpl
             │             │
             │          StudentDAO
             │             │
             │        DBConnection
             │             │
             │          MySQL
             │
       Swing UI
```

ဒီဟာက မင်းရဲ့ project architecture ရဲ့ core idea ပါ။

---

# 8. `client` ထဲမှာ ဘာတွေထားမလဲ?

Client က Database ကို **တိုက်ရိုက်မထိရဘူး**။

Client မှာ—

```text
client
├── client
│   ├── ConfigLoader.java
│   └── RmiConnectionManager.java
│
├── panel
│   ├── Default.java
│   ├── FamilyMember.java
│   ├── Specialization.java
│   └── StudentRegister.java
│
└── ui
    └── Home.java
```

ထားပါ။

အဓိက—

```text
Swing UI
   ↓
RmiConnectionManager
   ↓
RMI
   ↓
Server
```

ဖြစ်ရမယ်။

---

# 9. DAO ကို Client ထဲမထားပါနဲ့

ဒီဟာကို အထူးသတိထားပါ။

❌ မလုပ်သင့်တာ—

```text
client
└── dao
    └── StudentDAOImpl
```

ဒါမှမဟုတ်—

```java
// Client
Connection conn = DriverManager.getConnection(...);
```

လို Database connection တိုက်ရိုက်လုပ်တာ။

Client က—

```text
StudentRegister
      ↓
StudentService
      ↓
RMI
      ↓
StudentServiceImpl
      ↓
StudentDAO
      ↓
MySQL
```

ဖြစ်သင့်ပါတယ်။

---

# 10. `DAO Interface` က ဘယ်မှာထားမလဲ?

ဒီနေရာမှာ architecture ပေါ်မူတည်ပြီး ရွေးချယ်စရာရှိတယ်။

မင်းရဲ့ လက်ရှိ project အတွက် **DAO interface + DAO implementation နှစ်ခုလုံးကို server ထဲထားဖို့** recommend လုပ်တယ်။

```text
server
└── dao
    ├── StudentDAO.java
    ├── FamilyMemberDAO.java
    ├── SpecializationDAO.java
    │
    └── impl
        ├── StudentDAOImpl.java
        ├── FamilyMemberDAOImpl.java
        └── SpecializationDAOImpl.java
```

ဘာလို့လဲဆိုတော့ DAO က database implementation detail ဖြစ်လို့ပါ။

Client က DAO ကို သိစရာမလိုပါဘူး။

---

# 11. `specializations.sql` ဘယ်မှာထားမလဲ?

ဒါက database/server-side resource ဖြစ်တဲ့အတွက်—

```text
server
└── src
    └── main
        └── resources
            └── specializations.sql
```

မှာထားပါ။

Client မလိုပါဘူး။

---

# 12. `config.properties` ကတော့?

မင်းမှာ—

```text
resources
└── config.properties
```

ရှိတယ်။

အထဲမှာ ဘာတွေပါလဲဆိုတာပေါ်မူတည်တယ်။

ဥပမာ—

```properties
server.host=localhost
server.port=1099
```

လို RMI configuration ဆိုရင် **client** မှာထားလို့ရတယ်။

```text
client
└── resources
    └── config.properties
```

ဒါပေမယ့်—

```properties
db.url=...
db.username=...
db.password=...
```

ဆိုရင် **server** မှာထားတာပိုသင့်တော်ပါတယ်။

Database credential ကို client ထဲ မထည့်သင့်ပါဘူး။

---

# 13. `Sm.java` က ဘယ်မှာထားမလဲ?

လက်ရှိ—

```text
Sm.java
```

က application entry point ဖြစ်မယ်ထင်တယ်။

ဥပမာ—

```java
public class Sm {
    public static void main(String[] args) {
        new Home().setVisible(true);
    }
}
```

ဆိုရင် `client` ထဲထားပါ။

```text
client
└── src
    └── main
        └── java
            └── com.ams.sm
                └── Sm.java
```

Server entry point ကတော့—

```text
server
└── ServerMain.java
```

ဖြစ်မယ်။

ဒါဆို application နှစ်ခု ဖြစ်သွားမယ်—

```text
SERVER APPLICATION
        │
        └── ServerMain.java

CLIENT APPLICATION
        │
        └── Sm.java
```

ဒါက တကယ်ကောင်းတဲ့ separation ပါ။

---

# 14. Maven Parent POM

အခု Parent `pom.xml` ကိုကြည့်ရအောင်။

```text
student-management-system
│
└── pom.xml
```

ဒီ POM က application မဟုတ်ဘူး။

**Parent / Aggregator** ဖြစ်တယ်။

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ams</groupId>
    <artifactId>student-management-system</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <modules>
        <module>common</module>
        <module>client</module>
        <module>server</module>
    </modules>

</project>
```

ဒီဟာက—

```text
Parent
  │
  ├── common
  ├── client
  └── server
```

ကို manage လုပ်ပေးမယ်။

---

# 15. Module Dependency က ဒီလိုဖြစ်မယ်

အရေးကြီးဆုံး diagram က ဒီဟာပါ။

```text
                  ┌──────────────┐
                  │    COMMON    │
                  │              │
                  │ Model        │
                  │ Service      │
                  │ Interfaces   │
                  └──────┬───────┘
                         ↑
                 ┌───────┴───────┐
                 │               │
                 │               │
          ┌──────┴──────┐ ┌─────┴──────┐
          │   CLIENT    │ │   SERVER   │
          │             │ │            │
          │ Swing UI    │ │ DAO        │
          │ RMI Client  │ │ DB         │
          │             │ │ Services   │
          └──────┬──────┘ └─────┬──────┘
                 │               │
                 └────── RMI ────┘
```

Dependency အနေနဲ့—

```text
client  → common
server  → common
```

ဖြစ်ပါတယ်။

ဒါပေမယ့်—

```text
client → server
```

ဆိုပြီး Maven dependency တိုက်ရိုက်ထားဖို့ မလိုပါဘူး။

RMI က runtime မှာ ဆက်သွယ်ပေးမှာပါ။

---

# 16. `common/pom.xml`

Common မှာ Java code သက်သက်နီးပါးဖြစ်လို့ ရိုးရှင်းပါတယ်။

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.ams</groupId>
        <artifactId>student-management-system</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>common</artifactId>

</project>
```

---

# 17. `client/pom.xml`

Client က common ကို dependency အဖြစ်သုံးမယ်။

```xml
<dependency>
    <groupId>com.ams</groupId>
    <artifactId>common</artifactId>
    <version>${project.version}</version>
</dependency>
```

ဥပမာ full structure—

```xml
<parent>
    <groupId>com.ams</groupId>
    <artifactId>student-management-system</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<artifactId>client</artifactId>

<dependencies>

    <dependency>
        <groupId>com.ams</groupId>
        <artifactId>common</artifactId>
        <version>${project.version}</version>
    </dependency>

</dependencies>
```

---

# 18. `server/pom.xml`

Server လည်း common ကို dependency လုပ်မယ်။

```xml
<parent>
    <groupId>com.ams</groupId>
    <artifactId>student-management-system</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<artifactId>server</artifactId>

<dependencies>

    <dependency>
        <groupId>com.ams</groupId>
        <artifactId>common</artifactId>
        <version>${project.version}</version>
    </dependency>

</dependencies>
```

Server ထဲမှာတော့ additional dependencies တွေရှိမယ်—

```text
server
 ├── common
 ├── MySQL Connector
 └── RMI-related dependencies
```

Client မှာ—

```text
client
 ├── common
 ├── FlatLaf
 ├── Swing dependencies
 └── RMI-related dependencies
```

လိုဖြစ်နိုင်ပါတယ်။

---

# 19. မင်းရဲ့ လက်ရှိ Classes တွေကို ဘယ်ရွှေ့မလဲ?

မင်းပေးထားတဲ့ list ကို တစ်ခုချင်း map လုပ်ကြည့်ရအောင်။

|Current|New Module|
|---|---|
|`Sm.java`|`client`|
|`client/ConfigLoader.java`|`client`|
|`client/RmiConnectionManager.java`|`client`|
|`dao/*`|`server`|
|`dao/impl/*`|`server`|
|`db/DBConnection.java`|`server`|
|`model/*`|**common**|
|`panel/*`|`client`|
|`server/ServerMain.java`|`server`|
|`service/*.java`|**common**|
|`service/impl/*`|`server`|
|`ui/*`|`client`|
|`specializations.sql`|`server/resources`|
|`config.properties`|`client` or `server` depending on contents|

ဒါက မင်း project အတွက် အဓိက migration map ပါ။

---

# 20. နောက်ဆုံး Structure

အဆုံးမှာ မင်း project က ဒီလိုမြင်ရမယ်—

```text
student-management-system
│
├── pom.xml
│
├── common
│   ├── pom.xml
│   │
│   └── src/main/java/com/ams/sm
│       │
│       ├── model
│       │   ├── StudentModel.java
│       │   ├── FamilyMemberModel.java
│       │   ├── SpecializationModel.java
│       │   └── StudentSpecializationModel.java
│       │
│       └── service
│           ├── StudentService.java
│           ├── FamilyMemberService.java
│           └── SpecializationService.java
│
├── client
│   ├── pom.xml
│   │
│   └── src/main
│       ├── java/com/ams/sm
│       │   ├── Sm.java
│       │   ├── client
│       │   │   ├── ConfigLoader.java
│       │   │   └── RmiConnectionManager.java
│       │   │
│       │   ├── panel
│       │   │   ├── Default.java
│       │   │   ├── FamilyMember.java
│       │   │   ├── Specialization.java
│       │   │   └── StudentRegister.java
│       │   │
│       │   └── ui
│       │       └── Home.java
│       │
│       └── resources
│           └── config.properties
│
└── server
    ├── pom.xml
    │
    └── src/main
        ├── java/com/ams/sm
        │   ├── dao
        │   │   ├── StudentDAO.java
        │   │   ├── FamilyMemberDAO.java
        │   │   ├── SpecializationDAO.java
        │   │   └── impl
        │   │       ├── StudentDAOImpl.java
        │   │       ├── FamilyMemberDAOImpl.java
        │   │       └── SpecializationDAOImpl.java
        │   │
        │   ├── db
        │   │   └── DBConnection.java
        │   │
        │   ├── service
        │   │   └── impl
        │   │       ├── StudentServiceImpl.java
        │   │       ├── FamilyMemberServiceImpl.java
        │   │       └── SpecializationServiceImpl.java
        │   │
        │   └── server
        │       └── ServerMain.java
        │
        └── resources
            └── specializations.sql
```

## ဒီ architecture ရဲ့ အဓိကအကျိုးကျေးဇူး

မင်း project ကို အခုလို ခွဲလိုက်တာနဲ့—

```text
                    Student Management
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          COMMON         CLIENT         SERVER
             │             │             │
          Models        Swing UI       Database
          RMI API       RMI Client     DAO
          Contracts                    Services
```

ဆိုပြီး **အမှန်တကယ် Client–Server Architecture** ဖြစ်လာပါပြီ။

ပြီးတော့ အနာဂတ်မှာ—

```text
                 COMMON
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Swing     Android    Web/API
       Client    Client     Client
          │         │         │
          └─────────┼─────────┘
                    ↓
                  SERVER
                    ↓
                 MySQL
```

လိုမျိုး Client အသစ်တွေထပ်ထည့်ဖို့လည်း လွယ်သွားမယ်။

**အထူးသဖြင့် မင်းအခုရှိပြီးသား project ကို အစကနေပြန်ရေးစရာမလိုဘူး။** လက်ရှိ classes တွေကို module တစ်ခုချင်း **ရွှေ့ → package/import ပြင် → POM dependency ချိတ် → compile/test** လုပ်ရုံပါ။

မင်းလိုချင်ရင် နောက်တစ်ဆင့်မှာ **မင်းပေးထားတဲ့ exact project ကို အခြေခံပြီး NetBeans မှာ `common`, `client`, `server` Maven modules သုံးခုကို ဘယ်လို create လုပ်ရမလဲ + `pom.xml` သုံးခုလုံးကို complete code + classes တွေ ဘယ်လိုရွှေ့ရမလဲ** ကို step-by-step လုပ်ပေးနိုင်တယ်။