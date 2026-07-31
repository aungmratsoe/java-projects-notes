# Part 3: Café POS Real Implementation Phase

# Lesson 34: Building Authentication Module

## User Entity + BCrypt + Login Swing UI + Session Management

### (Java 25 + Swing + MVC + JDBC + MySQL + Security Architecture)

ဒီ Lesson က Café POS ရဲ့ **ပထမဆုံး Complete Feature Module** ဖြစ်ပါတယ်။

အခုကနေ User က:

```
Application Start

        ↓

Login Screen

        ↓

Enter Username / Password

        ↓

Database Verify

        ↓

Create Session

        ↓

Open Dashboard

```

အထိ အလုပ်လုပ်အောင် Build လုပ်ပါမယ်။

---

# 1. Authentication Module Architecture

Professional Flow:

```
                 LoginFrame
                     |
                     |
              LoginController
                     |
                     |
          AuthenticationService
                     |
                     |
             UserRepository
                     |
                     |
              Database Layer
                     |
                     |
                 MySQL

```

---

# 2. Authentication Package Structure

Create:

```
module

 └── auth

      ├── model

      │    ├── User.java
      │    └── Role.java

      ├── repository

      │    └── UserRepository.java

      ├── service

      │    └── AuthenticationService.java

      ├── controller

      │    └── LoginController.java

      └── view

           └── LoginFrame.java

```

---

# 3. Role Entity

Database:

```
roles

id

name

```

Java Record:

`Role.java`

```java
package com.cafe.pos.module.auth.model;


public record Role(

Long id,

String name

){

}

```

---

# 4. User Entity

Database:

```
users

id

username

password_hash

role_id

active

```

Create:

`User.java`

```java
package com.cafe.pos.module.auth.model;


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

# 5. Why Record?

Java 25 မှာ immutable object တွေအတွက်:

```
Less Code

Immutable

Thread Safe

Easy DTO

```

ဖြစ်ပါတယ်။

---

# 6. Password Security

Never store:

```
admin123

```

Database:

```
password_hash


$2a$12$8jd82jd82....

```

---

# 7. BCrypt Dependency

pom.xml:

```xml
<dependency>

<groupId>org.mindrot</groupId>

<artifactId>jbcrypt</artifactId>

<version>0.4</version>

</dependency>

```

---

# 8. Password Encoder Utility

Create:

```
security

 └── PasswordEncoder.java

```

---

Code:

```java
package com.cafe.pos.security;


import org.mindrot.jbcrypt.BCrypt;


public class PasswordEncoder {


private PasswordEncoder(){}



public static String encode(
String password
){


return BCrypt.hashpw(

password,

BCrypt.gensalt()

);


}



public static boolean matches(

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

# 9. Test BCrypt

Example:

```java
public class PasswordTest {


public static void main(String[] args){


String hash =
PasswordEncoder.encode(
"admin123"
);



System.out.println(hash);



System.out.println(

PasswordEncoder.matches(
"admin123",
hash
)

);


}


}

```

Output:

```
$2a$10$xxxxxxx

true

```

---

# 10. Insert Default Admin User

After migration:

Need user:

Role:

```
ADMIN

```

Password:

```
admin123

```

Generate hash:

Example:

```
$2a$10$abcxxxxxxxx

```

SQL:

```sql
INSERT INTO users

(username,password_hash,role_id)

VALUES

(
'admin',
'$2a$10$xxxxxx',
1
);

```

---

# 11. User Repository Interface

Create:

```
repository/UserRepository.java

```

Code:

```java
package com.cafe.pos.module.auth.repository;


import com.cafe.pos.module.auth.model.User;



public interface UserRepository {


User findByUsername(
String username
);


}

```

---

# 12. MySQL User Repository

Create:

```
MySQLUserRepository.java

```

Code:

```java
package com.cafe.pos.module.auth.repository;



import java.sql.*;

import com.cafe.pos.database.DatabaseManager;

import com.cafe.pos.module.auth.model.*;



public class MySQLUserRepository
implements UserRepository {



@Override

public User findByUsername(
String username
){


String sql = """

SELECT

u.id,

u.username,

u.password_hash,

u.active,

r.id,

r.name


FROM users u


JOIN roles r

ON u.role_id=r.id


WHERE u.username=?

""";



try(Connection con =
DatabaseManager.getConnection();


PreparedStatement ps =
con.prepareStatement(sql)){



ps.setString(
1,
username
);



ResultSet rs =
ps.executeQuery();



if(rs.next()){


Role role =
new Role(

rs.getLong(5),

rs.getString(6)

);



return new User(

rs.getLong(1),

rs.getString(2),

rs.getString(3),

role,

rs.getBoolean(4)

);


}



return null;



}
catch(Exception e){


throw new RuntimeException(e);


}


}


}

```

---

# 13. Authentication Exception

Create:

```
exception

AuthenticationException.java

```

Code:

```java
package com.cafe.pos.exception;


public class AuthenticationException
extends RuntimeException{


public AuthenticationException(
String message
){

super(message);

}


}

```

---

# 14. Authentication Service

Business Logic:

```
Check User

       +

Check Password

       +

Check Active

       +

Return User

```

Create:

`AuthenticationService.java`

```java
package com.cafe.pos.module.auth.service;


import com.cafe.pos.module.auth.model.User;

import com.cafe.pos.module.auth.repository.UserRepository;

import com.cafe.pos.security.PasswordEncoder;

import com.cafe.pos.exception.AuthenticationException;



public class AuthenticationService {



private final UserRepository repository;



public AuthenticationService(
UserRepository repository
){

this.repository=repository;

}




public User login(

String username,

String password

){


User user =
repository.findByUsername(username);



if(user==null){

throw new AuthenticationException(
"User not found"
);

}



if(!user.active()){

throw new AuthenticationException(
"Account disabled"
);

}



if(!PasswordEncoder.matches(

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

# 15. Session Management

After Login:

Need remember:

```
Current User

admin

ADMIN

```

---

Create:

```
security

 └── Session.java

```

---

Code:

```java
package com.cafe.pos.security;


import com.cafe.pos.module.auth.model.User;



public class Session {


private static User currentUser;



private Session(){}



public static void setUser(
User user
){

currentUser=user;

}



public static User getUser(){

return currentUser;

}



public static void clear(){

currentUser=null;

}


}

```

---

# 16. Login Controller

Create:

```
controller/LoginController.java

```

Code:

```java
package com.cafe.pos.module.auth.controller;


import com.cafe.pos.module.auth.model.User;

import com.cafe.pos.module.auth.service.AuthenticationService;

import com.cafe.pos.security.Session;



public class LoginController {


private final AuthenticationService service;



public LoginController(
AuthenticationService service
){

this.service=service;

}



public User login(
String username,
String password
){


User user =
service.login(
username,
password
);



Session.setUser(user);



return user;


}


}

```

---

# 17. Professional Login UI

Structure:

```
+--------------------------------+

             ☕


        Café POS Login


 Username:

 [____________]


 Password:

 [____________]


          [ LOGIN ]


+--------------------------------+

```

---

# 18. LoginFrame

Create:

```
view/LoginFrame.java

```

Code:

```java
public class LoginFrame
extends JFrame{


private JTextField username;

private JPasswordField password;

private JButton login;



public LoginFrame(){


setTitle(
"Café POS Login"
);


setSize(
400,
300
);


setLocationRelativeTo(null);


initialize();


}



private void initialize(){


username =
new JTextField();



password =
new JPasswordField();



login =
new JButton(
"LOGIN"
);



setLayout(
new GridLayout(3,2)
);



add(
new JLabel("Username")
);


add(username);


add(
new JLabel("Password")
);


add(password);


add(login);



}


}

```

---

# 19. Login Button Flow

```
Click Button


       |

Get Input


       |

LoginController


       |

AuthenticationService


       |

Database


       |

Session Created


       |

Dashboard Open


```

---

# 20. Dependency Injection

Application:

Create objects:

```java
UserRepository repo =
new MySQLUserRepository();



AuthenticationService service =
new AuthenticationService(repo);



LoginController controller =
new LoginController(service);

```

---

Flow:

```
Application

 creates

      ↓

Controller

      ↓

Service

      ↓

Repository

```

---

# 21. Current Authentication Status

Completed:

```
User Entity              ✅

Role Entity              ✅

BCrypt Security          ✅

User Repository          ✅

Authentication Service   ✅

Session Management       ✅

Login Architecture       ✅

```

---

# 22. Security Rules

Remember:

Never:

❌ Store plain password

❌ Login directly from JFrame

❌ SQL inside UI

❌ Static database connection

Always:

✅ BCrypt

✅ Service Layer

✅ Repository Pattern

✅ Session

---

# Practice Task

Implement:

1. Add BCrypt dependency
    
2. Create PasswordEncoder
    
3. Create User + Role Entity
    
4. Create UserRepository
    
5. Create AuthenticationService
    
6. Create LoginFrame
    
7. Test admin login
    

---

# Next Lesson

# Lesson 35: Professional Main Dashboard + Role Based Access Control (RBAC)

Next we build:

✅ Main Application Window  
✅ Sidebar Navigation  
✅ Admin/Cashier Menu Control  
✅ Permission System  
✅ Logout  
✅ User Session Display  
✅ FlatLaf Professional UI Theme

ပြီးရင် Café POS မှာ **Login → Dashboard → Module Navigation** အပြည့်အဝ အလုပ်လုပ်လာပါမယ်။