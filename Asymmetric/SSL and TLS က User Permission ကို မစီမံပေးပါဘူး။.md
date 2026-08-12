ဒီမေးခွန်းက **Authentication** နဲ့ **Authorization** ကို ရောထားတာပါ။

အရေးကြီးဆုံး အချက်က

> **SSL/TLS နဲ့ Asymmetric Encryption က "ဘယ်သူလဲ" (Authentication) ကို ကူညီပေးတယ်။**
> 
> **"ဘာလုပ်ခွင့်ရှိလဲ" (Authorization) ကိုတော့ သင့် Application က ဆုံးဖြတ်ရပါတယ်။**

ဒါကြောင့် **SSL/TLS က User Permission ကို မစီမံပေးပါဘူး။**

---

# Professional Architecture

```text
                Swing Client
                      │
         Login(username/password)
                      │
                      ▼
              RMI over SSL/TLS
                      │
              Server Authenticate
                      │
          Check Database User Table
                      │
       +--------------+--------------+
       |                             |
    Login Fail                  Login Success
                                      │
                               Create Session
                                      │
                              Return Session Token
                                      │
                   Every RMI call includes Token
                                      │
                              Server verifies Token
                                      │
                           Check User Role/Permission
                                      │
                +---------------------+------------------+
                |                                        |
             Allowed                                Access Denied
```

---

# Step 1 - User Table

```sql
users
------------------------------
id
username
password_hash
role
status
```

ဥပမာ

|username|role|
|---|---|
|admin|ADMIN|
|teacher|TEACHER|
|staff|STAFF|

---

# Step 2 - Permission Table

```sql
permissions
----------------------------
id
permission_name
```

ဥပမာ

```
CREATE_STUDENT

UPDATE_STUDENT

DELETE_STUDENT

VIEW_REPORT

REGISTER_USER
```

---

# Step 3 - Role Permission

```sql
role_permission

role

permission
```

ဥပမာ

|Role|Permission|
|---|---|
|ADMIN|CREATE_STUDENT|
|ADMIN|DELETE_STUDENT|
|ADMIN|VIEW_REPORT|
|STAFF|CREATE_STUDENT|
|STAFF|VIEW_REPORT|
|TEACHER|VIEW_REPORT|

ဒီကို **RBAC (Role-Based Access Control)** လို့ခေါ်ပါတယ်။

---

# Step 4 - Login

Client

```java
service.login(username, password);
```

Server

```java
if (passwordCorrect) {

    String token = UUID.randomUUID().toString();

    sessions.put(token, user);

    return token;

}
```

Client က

```
f2a9d87e...
```

လို Session Token တစ်ခု ရမယ်။

---

# Step 5 - Every Request

Student Add

Client

```java
service.addStudent(token, student);
```

Server

```java
User user = sessionManager.getUser(token);

if(user == null){

    throw new SecurityException("Login required");

}
```

---

# Step 6 - Permission Check

```java
if(!user.hasPermission("CREATE_STUDENT")){

    throw new SecurityException("Permission denied");

}
```

ပြီးမှ

```java
studentDAO.insert(student);
```

---

# Example

Admin

```
Create Student ✔

Delete Student ✔

Update Student ✔

View Report ✔
```

Staff

```
Create Student ✔

Delete Student ❌

Update Student ✔

View Report ✔
```

Teacher

```
Create Student ❌

Delete Student ❌

Update Student ❌

View Report ✔
```

---

# Swing UI

Login ပြီးရင်

Server က

```java
User
```

Object ပြန်ပို့တယ်။

```java
class User{

    String username;

    String role;

    Set<String> permissions;

}
```

Client မှာ

```java
btnDelete.setVisible(
    currentUser.hasPermission("DELETE_STUDENT")
);
```

ဒါက UI အတွက်ပဲ ဖြစ်ပါတယ်။

⚠️ **ဒီတစ်ခုတည်းနဲ့ မလုံခြုံပါဘူး။**

---

# အရေးကြီးဆုံး Rule

Server မှာလည်း ထပ်စစ်ရပါတယ်။

```java
public void deleteStudent(String token, int id){

    User user = session.getUser(token);

    if(!user.hasPermission("DELETE_STUDENT")){

        throw new SecurityException();

    }

    dao.delete(id);

}
```

ဘာလို့လဲဆိုတော့

Client ကို Hack လုပ်ပြီး

```java
btnDelete.setVisible(true);
```

လုပ်လို့ရပါတယ်။

ဒါပေမယ့်

Server မှာ

Permission Check ရှိနေတော့

Delete မလုပ်နိုင်တော့ဘူး။

---

# SSL/TLS ဘယ်နေရာမှာ ပါလဲ?

SSL/TLS က

```
Username

Password

Session Token

Student Data
```

တွေကို

```
Encrypted
```

ဖြစ်အောင် လုပ်ပေးတာပါ။

Permission မစစ်ပါဘူး။

---

# Production-Level Architecture

```text
           Swing Client
                 │
          Login(username/password)
                 │
                 ▼
        RMI over SSL/TLS
                 │
                 ▼
      Authentication Service
                 │
      Verify Password (BCrypt/Argon2)
                 │
      Generate Session Token
                 │
                 ▼
        Session Manager
                 │
        Load User Role
                 │
        Load Permissions
                 │
                 ▼
      Business Service (StudentService)
                 │
      Permission Check (Server-side)
                 │
                 ▼
             Database
```

## သင့် Student Management System အတွက် အကြံပြုချက်

သင့် Project က Java Swing + RMI + MySQL ဖြစ်တဲ့အတွက် Enterprise Application တွေလို ဒီ Design ကို သုံးတာ အကောင်းဆုံးပါ။

1. **SSL/TLS** → Client ↔ Server Communication ကို ကာကွယ်ရန်။
    
2. **BCrypt သို့မဟုတ် Argon2** → Password ကို Hash သိမ်းရန်။
    
3. **Session Token** → Login အောင်မြင်ပြီးနောက် User ကို မှတ်သားရန်။
    
4. **RBAC (Role-Based Access Control)** → Admin, Staff, Teacher စတဲ့ Role တွေအလိုက် Permission သတ်မှတ်ရန်။
    
5. **Server-side Permission Check** → RMI Method တိုင်းမှာ Permission ကို Server က စစ်ဆေးရန်။
    
6. **Client UI** → Permission အလိုက် Button/Menu တွေကို ဖျောက်ပြပေးနိုင်ပေမယ့် Security ကို Server ကပဲ အာမခံရမယ်။
    

ဒီ Architecture က Banking System, Hospital Management System, ERP System နဲ့ Enterprise Java Application အများစုမှာ အသုံးပြုတဲ့ စံနည်းလမ်းဖြစ်ပါတယ်။