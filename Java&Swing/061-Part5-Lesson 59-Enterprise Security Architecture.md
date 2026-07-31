# Part 5: Café POS System Architecture Phase

# Lesson 59: Enterprise Security Architecture

## Authentication + Authorization + Database Security

### (Java 25 + Swing + MVC + JDBC + MySQL)

ဒီ Lesson မှာ Café POS System ကို **Security Level** တင်ပါမယ်။

အခုအထိ System မှာ:

```text
Product Management             ✅

Order Management               ✅

Payment                        ✅

Inventory ERP                  ✅

Database Architecture          ✅

Transaction Management         ✅

```

ရှိပါပြီ။

ဒါပေမယ့် Real Enterprise Application မှာ မေးရမယ့် Question တွေက:

```
ဘယ်သူ Login ဝင်ထားလဲ?

ဘယ်သူ Order ဖျက်ခွင့်ရှိလဲ?

ဘယ်သူ Price ပြောင်းခွင့်ရှိလဲ?

ဘယ်သူ Stock Adjust လုပ်နိုင်လဲ?

ဘယ်သူ Report ကြည့်နိုင်လဲ?

```

ဖြစ်ပါတယ်။

---

# Lesson 59 Goals

ဒီနေ့:

```text
1. Authentication System

2. User Management

3. Role Based Access Control (RBAC)

4. Password Security

5. Permission System

6. Audit Logging

7. Database Security

```

တည်ဆောက်ပါမယ်။

---

# PART 1

# Authentication vs Authorization

---

# 1. Authentication ဆိုတာဘာလဲ?

Authentication =

> "မင်းဘယ်သူလဲ?"

စစ်တာ

Example:

Login:

```
Username:

admin


Password:

******

```

System:

```
Username exists?

Password correct?

```

Answer:

```
YES

↓

Login Success

```

---

# 2. Authorization ဆိုတာဘာလဲ?

Authorization =

> "မင်းဘာလုပ်ခွင့်ရှိလဲ?"

စစ်တာ

Example:

User:

```
Cashier

```

Can:

```
Create Order ✅

Receive Payment ✅

```

Cannot:

```
Delete Product ❌

Change Cost Price ❌

Adjust Stock ❌

```

---

# 3. Difference

||Authentication|Authorization|
|---|---|---|
|Question|Who are you?|What can you do?|
|Example|Login|Permission|
|Layer|Security|Business Rule|

---

# PART 2

# User Database Design

---

# 4. users Table

Create:

```sql
CREATE TABLE users
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


username VARCHAR(50) UNIQUE NOT NULL,


password_hash VARCHAR(255) NOT NULL,


full_name VARCHAR(100),


role_id BIGINT,


active BOOLEAN DEFAULT TRUE,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP


);

```

---

Example:

```
users


id

1


username

admin


password_hash

8f4a9d...


role_id

1


```

---

# 5. Why Store Password Hash?

Wrong:

```text
password


admin123

```

Database leak ဖြစ်ရင်:

```
Everyone knows password

```

---

Correct:

```
Original Password


admin123


        ↓


Hash Algorithm


        ↓


$2a$10$8f73h...

```

---

# PART 3

# Password Hashing

---

# 6. Use BCrypt

Professional Java Application:

```
Password

        ↓

BCrypt

        ↓

Hash

```

---

Dependency:

Maven:

```xml
<dependency>

    <groupId>
    org.mindrot
    </groupId>

    <artifactId>
    jbcrypt
    </artifactId>

    <version>
    0.4
    </version>

</dependency>

```

---

# 7. Password Encoder

Create:

```
security/password

PasswordEncoder.java

```

---

Code:

```java
public class PasswordEncoder {


public String encode(
String password
){


return BCrypt.hashpw(

password,

BCrypt.gensalt()

);


}



public boolean matches(

String password,

String hash

){


return BCrypt.checkpw(

password,

hash

);


}


}

```

---

Example:

Input:

```
admin123

```

Output:

```
$2a$10$X7ab83k....

```

---

# PART 4

# Login Flow

---

# 8. Login Architecture

```
LoginPanel


    ↓


AuthController


    ↓


AuthService


    ↓


UserRepository


    ↓


Database


```

---

# 9. Login Screen

Swing:

```
+--------------------------+

        CAFÉ POS LOGIN


Username:


[____________]


Password:


[____________]



        [ LOGIN ]


+--------------------------+

```

---

# 10. Login Model

```java
public record User(


Long id,


String username,


String passwordHash,


Role role


){

}

```

---

# 11. Authentication Service

```java
public class AuthService {



private final UserRepository repository;


private final PasswordEncoder encoder;




public User login(

String username,

String password

){



User user =

repository.findByUsername(
username
);



if(user == null){


throw new AuthenticationException(

"User not found"

);


}



if(!encoder.matches(

password,

user.passwordHash()

)){


throw new AuthenticationException(

"Invalid password"

);

}



return user;


}


}

```

---

# PART 5

# Role Based Access Control (RBAC)

---

# 12. Role Design

Roles:

```
ADMIN

MANAGER

CASHIER

INVENTORY_STAFF

```

---

Database:

roles:

```sql
CREATE TABLE roles
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


name VARCHAR(50)

);

```

---

Example:

```
1 ADMIN

2 MANAGER

3 CASHIER

4 INVENTORY_STAFF

```

---

# PART 6

# Permission System

---

# 13. Why Permission?

Role တစ်ခုတည်းနဲ့ မလုံလောက်ပါ။

Example:

Manager:

```
Can view report

Can adjust stock

```

Cashier:

```
Can create order

Cannot adjust stock

```

---

Create:

```sql
CREATE TABLE permissions
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


name VARCHAR(100)

);

```

---

Example:

```
CREATE_ORDER

DELETE_ORDER

VIEW_REPORT

ADJUST_STOCK

MANAGE_USER

```

---

# 14. Role Permission Mapping

Many-to-Many:

```sql
CREATE TABLE role_permissions
(

role_id BIGINT,


permission_id BIGINT,


PRIMARY KEY(
role_id,
permission_id
)

);

```

---

Relationship:

```
Role

  |

  *

Role_Permission

  *

  |

Permission

```

---

# PART 7

# Permission Checking

---

# 15. Permission Service

```java
public class PermissionService {



public boolean hasPermission(

User user,

String permission

){


return repository.exists(

user.role().id(),

permission

);


}


}

```

---

Example:

Before Delete:

```java
if(
!permissionService.hasPermission(

user,

"DELETE_PRODUCT"

)

){


throw new SecurityException(

"No permission"

);

}

```

---

# PART 8

# Swing Security Integration

---

# 16. Hide Menu Based On Role

Example:

Admin Menu:

```
Users

Reports

Inventory

Products

Settings

```

---

Cashier Menu:

```
Orders

Payment

Receipt

```

---

Code:

```java
if(
user.role()
.equals(Role.ADMIN)

){


userMenu.setVisible(true);


}

```

---

# PART 9

# Audit Logging

---

# 17. Why Audit?

Restaurant Owner wants:

```
Who changed stock?


Who deleted order?


Who changed price?

```

---

Create:

```sql
CREATE TABLE audit_logs
(

id BIGINT AUTO_INCREMENT PRIMARY KEY,


user_id BIGINT,


action VARCHAR(100),


entity VARCHAR(100),


entity_id BIGINT,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

);

```

---

Example:

```
User:

admin


Action:

UPDATE_PRODUCT


Entity:

Product


ID:

10


Time:

2026-07-31

```

---

# 18. Audit Service

```java
public class AuditService {


public void log(

User user,

String action,

String entity,

Long id

){



repository.save(

new AuditLog(

user.id(),

action,

entity,

id

)

);


}


}

```

---

# PART 10

# Database Security Best Practices

---

## 1. Least Privilege

Wrong:

```
Application User:

root

```

---

Correct:

Create:

```
cafe_app_user

```

Only:

```
SELECT

INSERT

UPDATE

DELETE

```

---

## 2. PreparedStatement

Already implemented:

```
SQL Injection Protection ✅

```

---

## 3. Password Never Stored Plain Text

Use:

```
BCrypt

```

---

## 4. Database Backup

Daily:

```
mysqldump

```

---

# PART 11

# Final Security Architecture

```
                 User


                  |

                  |


              Login UI


                  |

                  |


          Authentication Service


                  |

                  |


          User + Password Hash


                  |

                  |


              RBAC


                  |

        -------------------


        Permission Check


                  |

                  |

             Business Action


                  |

                  |

             Audit Log


```

---

# Café POS Current Level

After Lesson 59:

```
Java 25 Architecture              ✅

Swing MVC                         ✅

Database Design                   ✅

Inventory ERP                     ✅

Transaction System                ✅

Authentication                    ✅

Password Hashing                  ✅

RBAC                              ✅

Permission System                 ✅

Audit Logging                     ✅

Database Security                 ✅

```

---

# Practice Task

Implement:

## Database

1. users table
    
2. roles table
    
3. permissions table
    
4. audit_logs table
    

## Java

5. PasswordEncoder
    
6. AuthService
    
7. PermissionService
    
8. AuditService
    

## Swing

9. LoginPanel
    
10. Role Based Menu
    

---

# Next Lesson

# Lesson 60: Café POS Final Enterprise Project

## Complete System Architecture + Deployment + Production Design

Next we combine everything:

```
Architecture Review

        ↓

Project Structure

        ↓

Production Configuration

        ↓

Database Migration

        ↓

Logging

        ↓

Testing Strategy

        ↓

Deployment

```

ဒီ Lesson ပြီးရင် Café POS Project ကို **Senior Java Developer / System Architect Portfolio Project Level** အဖြစ် Finalize လုပ်ပါမယ်။