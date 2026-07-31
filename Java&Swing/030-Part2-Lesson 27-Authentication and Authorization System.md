# Part 2: Advanced Java Swing + Café POS Project

# Lesson 27: Authentication & Authorization System

## Secure Login for Café POS

### (Java 25 + Swing + MVC + JDBC + BCrypt + RBAC + Audit Logging)

ဒီ Lesson မှာ Café POS ကို **Multi-user Enterprise Application** အဖြစ် ပြောင်းပါမယ်။

အခုအချိန်ထိ:

- Product Management ✅
    
- Order System ✅
    
- Inventory System ✅
    
- Transaction Handling ✅
    

ပြီးပါပြီ။

ဒါပေမယ့် Production POS System မှာ:

- Cashier တစ်ယောက်
    
- Manager တစ်ယောက်
    
- Admin တစ်ယောက်
    

ရှိနိုင်ပါတယ်။

အားလုံးကို Access မပေးသင့်ပါ။

---

# 1. Authentication vs Authorization

ဒီနှစ်ခုကို မရောရပါ။

---

# Authentication

Question:

> "Who are you?"

Example:

```text
Username:
admin


Password:
******


System:

You are Admin

```

---

# Authorization

Question:

> "What can you do?"

Example:

```text
Admin

Can:
- Add User
- Delete Product
- View Report


Cashier

Can:
- Create Order
- Receive Payment

Cannot:
- Delete Product

```

---

# 2. Security Architecture

Complete Flow:

```text
          Login Screen

               |

        Authentication Service

               |

        User Repository

               |

             MySQL

               |

        Create Session

               |

        Authorization Check

               |

        Open POS Dashboard


```

---

# 3. User Database Design

Previous users table:

```sql
users

id

username

password_hash

role

created_at

```

---

Professional design:

Separate Role table.

Relationship:

```text
Users

  |

  |

Roles

```

---

# 4. Roles Table

Create:

```sql
CREATE TABLE roles(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(50) UNIQUE NOT NULL

);

```

---

Data:

```text
1 | ADMIN

2 | MANAGER

3 | CASHIER

```

---

# 5. Users Table

```sql
CREATE TABLE users(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


username VARCHAR(50)
UNIQUE NOT NULL,


password_hash VARCHAR(255)
NOT NULL,


role_id BIGINT NOT NULL,


active BOOLEAN DEFAULT TRUE,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,


FOREIGN KEY(role_id)

REFERENCES roles(id)

);

```

---

# 6. Why Password Hashing?

Never:

```java
password="123456"

```

Database:

```text
username | password

admin    | 123456

```

---

Problem:

Database leak ဖြစ်ရင်:

Everyone knows password.

---

Correct:

```text
Original:

mypassword123


Hash:

$2a$12$7Hsj38....
```

---

# 7. BCrypt

BCrypt features:

✅ One-way hashing

✅ Salt included

✅ Slow by design

✅ Resistant to brute force

---

Maven:

```xml
<dependency>

<groupId>org.mindrot</groupId>

<artifactId>jbcrypt</artifactId>

<version>0.4</version>

</dependency>

```

---

# 8. Password Hash Creation

Example:

```java
String hash =
BCrypt.hashpw(

"admin123",

BCrypt.gensalt()

);

```

---

Result:

```text
$2a$12$8F9s83jd82.....

```

---

Database stores only hash.

---

# 9. Password Verification

Login:

User enters:

```text
admin

admin123

```

---

System:

```java
boolean result =
BCrypt.checkpw(

inputPassword,

storedHash

);

```

---

If:

```text
true

Login Success

```

---

# 10. User Entity

Java 25 Record:

```java
public record User(

Long id,

String username,

String passwordHash,

Role role,

boolean active

){

}

```

---

Role:

```java
public record Role(

Long id,

String name

){

}

```

---

# 11. User Repository

Interface:

```java
public interface UserRepository {


User findByUsername(
String username
);


void save(User user);


}

```

---

# 12. MySQL User Repository

Query:

```sql
SELECT

u.id,

u.username,

u.password_hash,

r.id,

r.name


FROM users u


JOIN roles r

ON u.role_id=r.id


WHERE username=?;

```

---

Why JOIN?

Need:

```text
User

+

Role

```

---

# 13. Authentication Service

Business logic:

```java
public class AuthenticationService {


private final UserRepository repository;



public User login(

String username,

String password

){



User user =
repository.findByUsername(
username
);



if(user==null){

throw new AuthenticationException(
"User not found"
);

}



if(!BCrypt.checkpw(

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

# 14. Custom Security Exception

Package:

```text
exception

   |
   |
AuthenticationException.java

```

---

Code:

```java
public class AuthenticationException
extends RuntimeException {


public AuthenticationException(
String message
){

super(message);

}


}

```

---

# 15. Login UI Design

Professional:

```text
+--------------------------------+

          ☕ Café POS


 Username

 [________________]


 Password

 [________________]


        [ Login ]


+--------------------------------+

```

---

# 16. Login JFrame

```java
public class LoginFrame
extends JFrame{


private JTextField usernameField;


private JPasswordField passwordField;



}

```

---

# 17. Login Button Flow

```text
Click Login


    |

Get username/password


    |

Controller


    |

AuthenticationService


    |

Database


    |

Success


    |

Open MainFrame


```

---

# 18. Login Controller

```java
public class LoginController {


private final AuthenticationService service;



public User login(

String username,

String password

){


return service.login(
username,
password
);


}


}

```

---

# 19. Session Management

After login:

Need remember:

```text
Current User:

Admin

ID:1

Role:ADMIN

```

---

Create:

```text
security

   |

Session.java

```

---

Code:

```java
public class Session {


private static User currentUser;



public static void setUser(
User user
){

currentUser=user;

}



public static User getUser(){

return currentUser;

}


}

```

---

# 20. Authorization Check

Example:

Delete Product:

```java
if(
Session.getUser()
.role()
.name()
.equals("ADMIN")
){


deleteProduct();


}

```

---

Better:

Create:

```java
PermissionService

```

---

# 21. Permission Based Security

Roles:

```text
ADMIN

MANAGER

CASHIER

```

---

Permissions:

```text
PRODUCT_CREATE

PRODUCT_DELETE

REPORT_VIEW

ORDER_CREATE

PAYMENT_PROCESS


```

---

Database:

```text
roles


permissions


role_permissions

```

---

Relationship:

```text
Role

 |

Role Permission

 |

Permission

```

---

# 22. RBAC Architecture

Role Based Access Control:

```text
User

 |

Role

 |

Permissions

 |

Features

```

---

Example:

```text
Cashier

 |

ORDER_CREATE

PAYMENT_PROCESS

 |

Order Screen Enabled


```

---

# 23. Disable UI Components

Example:

Admin:

```java
deleteButton.setEnabled(true);

```

---

Cashier:

```java
deleteButton.setEnabled(false);

```

---

# 24. MainFrame Security

After login:

```java
User user =
Session.getUser();


if(user.role().name()
.equals("CASHIER")){


hideAdminMenu();


}

```

---

# 25. Audit Logging

Security requires tracking.

Example:

```text
10:20

Admin login


10:35

Deleted Product Coffee


11:00

Changed Price


```

---

Table:

```sql
CREATE TABLE audit_logs(

id BIGINT PRIMARY KEY AUTO_INCREMENT,


user_id BIGINT,


action VARCHAR(100),


description TEXT,


created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP


);

```

---

# 26. Audit Service

```java
public class AuditService {


public void log(

String action,

String description

){

repository.save(
new AuditLog(
...
)
);


}


}

```

---

# 27. Login Audit Example

After login:

```java
auditService.log(

"LOGIN",

"User logged in"

);

```

---

# 28. Security Exception Flow

Example:

Cashier tries delete:

```text
Click Delete


    |

Permission Check


    |

DENIED


    |

AuthorizationException


    |

Show Message


```

---

# 29. Exception Structure

Professional:

```text
RuntimeException

        |

   AppException

        |

 -----------------

 |               |

SecurityException BusinessException


        |

AuthenticationException


AuthorizationException

```

---

# 30. Complete Security Architecture

```text
                 Login UI

                    |

             Login Controller

                    |

          Authentication Service

                    |

             User Repository

                    |

                  MySQL


After Login:


                Session

                    |

           Permission Service

                    |

               UI Access


```

---

# 31. Production Security Checklist

Authentication:

✅ BCrypt Password

✅ Secure Login

✅ Session

Authorization:

✅ RBAC

✅ Permission Check

✅ Menu Control

Audit:

✅ Login History

✅ User Actions

✅ Security Logs

---

# 32. Practice Task

Build:

## Authentication Module

Create:

1. Role Table
    
2. User Table
    
3. BCrypt Utility
    
4. UserRepository
    
5. AuthenticationService
    
6. LoginFrame
    
7. Session Manager
    
8. Permission Service
    
9. Audit Logging
    

---

# Lesson 27 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Authentication Concept  
✅ Authorization Concept  
✅ BCrypt Password Hashing  
✅ Login Architecture  
✅ User Repository  
✅ Authentication Service  
✅ Session Management  
✅ RBAC Design  
✅ Permission System  
✅ Audit Logging  
✅ Security Exception Handling

---

# Next Lesson

# Lesson 28: Reporting System + Dashboard Design

## Enterprise Café POS Analytics

Topics:

- Sales Dashboard
    
- JTable Reports
    
- Charts
    
- Daily/Monthly Sales
    
- Java Swing Charts
    
- Export PDF
    
- Export Excel
    
- Reporting Service
    
- Query Optimization
    

ဒီ Lesson ပြီးရင် Café POS က **Business Intelligence ပါတဲ့ Professional POS System** ဖြစ်လာပါမယ်။