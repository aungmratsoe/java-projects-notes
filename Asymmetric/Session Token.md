ဒီဟာက **Enterprise Java** မှာ အသုံးအများဆုံး Design Pattern တစ်ခုပါ။

**Session Token က "User ဘယ်သူလဲ?" ကို သိဖို့သုံးတာ** ဖြစ်ပြီး **Permission Check** ကိုတော့ Server ဘက်မှာ လုပ်ပါတယ်။

---

# Overall Architecture

```text
                 Swing Client
                      │
             Login(username,password)
                      │
                      ▼
                RMI Server
                      │
          Verify Username/Password
                      │
             Generate Session Token
                      │
                      ▼
          token = "3c9baf2d-..."
                      │
                      ▼
           Return Token to Client
                      │
────────────────────────────────────────────
Every RMI Request

Client
    │
    ├── Session Token
    └── Business Data
           │
           ▼
Server
    │
    ├── Validate Token
    ├── Find User
    ├── Check Permission
    └── Execute Method
```

---

# Database Design

## users

```sql
id
username
password_hash
role_id
status
```

---

## roles

```sql
id
role_name
```

Example

|id|role_name|
|---|---|
|1|ADMIN|
|2|STAFF|
|3|TEACHER|

---

## permissions

```sql
id
permission_name
```

Example

```text
CREATE_STUDENT
UPDATE_STUDENT
DELETE_STUDENT
VIEW_REPORT
```

---

## role_permissions

```sql
role_id
permission_id
```

Example

|Role|Permission|
|---|---|
|ADMIN|ALL|
|STAFF|CREATE_STUDENT|
|STAFF|UPDATE_STUDENT|
|TEACHER|VIEW_REPORT|

---

# Session Class

```java
public class UserSession {

    private String token;

    private int userId;

    private String username;

    private String role;

    private Set<String> permissions;

    private LocalDateTime expireTime;

    // getters setters
}
```

---

# Session Manager

```java
public class SessionManager {

    private final Map<String, UserSession> sessions
            = new ConcurrentHashMap<>();

}
```

Login အောင်မြင်ရင်

```java
String token = UUID.randomUUID().toString();

UserSession session = new UserSession();

session.setToken(token);
session.setUserId(user.getId());
session.setUsername(user.getUsername());
session.setRole(user.getRole());
session.setPermissions(permissionDAO.findByRole(user.getRole()));

sessions.put(token, session);

return token;
```

ဥပမာ

```text
Token

↓

38af2d10-e55d-49b4-a3c5...
```

---

# Client

Login

```java
String token = service.login(username,password);
```

Client မှာ

```java
SessionContext.setToken(token);
```

သိမ်းထားတယ်။

---

# Every RMI Method

RMI Interface

```java
void addStudent(String token,
                Student student)
        throws RemoteException;
```

Client

```java
service.addStudent(
        SessionContext.getToken(),
        student
);
```

---

# Server

```java
public void addStudent(
        String token,
        Student student) {

    UserSession session =
            sessionManager.get(token);

    if(session == null){

        throw new SecurityException(
                "Invalid Session");

    }

}
```

---

# Permission Check

```java
if(!session.getPermissions()
        .contains("CREATE_STUDENT")){

    throw new SecurityException(
            "Permission Denied");

}
```

ပြီးမှ

```java
studentDAO.insert(student);
```

---

# Delete Student

```java
public void deleteStudent(
        String token,
        int studentId){
```

Server

```java
UserSession session =
        sessionManager.get(token);

if(session==null){

    throw new SecurityException();

}

if(!session.getPermissions()
        .contains("DELETE_STUDENT")){

    throw new SecurityException();

}

dao.delete(studentId);
```

---

# UI

Login ပြီးရင်

Server က

```java
UserInfo
```

လည်းပြန်ပို့လို့ရပါတယ်။

```java
public class LoginResponse {

    private String token;

    private String username;

    private String role;

    private Set<String> permissions;

}
```

Client

```java
LoginResponse response
        = service.login(...);
```

ပြီးရင်

```java
currentUser=response;
```

---

Button

```java
btnDelete.setEnabled(

currentUser.getPermissions()

.contains("DELETE_STUDENT")

);
```

---

## အရေးကြီးဆုံး

UI မှာ

```java
btnDelete.setEnabled(false);
```

လုပ်ထားတာက **User Experience** အတွက်ပဲ။

Security မဟုတ်ပါဘူး။

Server မှာ

```java
if(!permission){

    throw SecurityException();

}
```

**မဖြစ်မနေ စစ်ရပါတယ်။**

---

# Session Expiration

Session က Forever မထားသင့်ပါ။

```java
session.setExpireTime(

LocalDateTime.now()

.plusMinutes(30)

);
```

Method ခေါ်တိုင်း

```java
if(LocalDateTime.now()

.isAfter(session.getExpireTime())){

    sessions.remove(token);

    throw new SecurityException(
            "Session Expired");

}
```

---

# Logout

```java
service.logout(token);
```

Server

```java
sessions.remove(token);
```

Token ပျက်သွားပြီ။

---

# Professional Design

```text
                 Login
                   │
                   ▼
        Username + Password
                   │
                   ▼
        Verify Password Hash
                   │
                   ▼
          Generate UUID Token
                   │
                   ▼
         Store Session in Memory
                   │
         +-------------------+
         | userId            |
         | username          |
         | role              |
         | permissions       |
         | expireTime        |
         +-------------------+
                   │
                   ▼
        Return Token to Client
                   │
─────────────────────────────────────
        Every RMI Method Call
                   │
          Send Session Token
                   │
                   ▼
        Validate Token Exists?
                   │
                   ▼
        Check Session Expired?
                   │
                   ▼
        Check Permission?
                   │
             Yes        No
              │          │
              ▼          ▼
      Execute Method   SecurityException
```

## ကျွန်တော့်ရဲ့ အကြံပြုချက်

သင့် **Student Management System (Swing + RMI + MySQL)** အတွက် Login Method ကို `String` token တစ်ခုတည်း မပြန်ဘဲ `LoginResponse` DTO တစ်ခု ပြန်တာက ပိုကောင်းပါတယ်။

```java
public class LoginResponse {
    private String sessionToken;
    private UserDto user;
    private Set<String> permissions;
    private Instant expiresAt;
}
```

ဒီလိုဆို Client က Login ပြီးတာနဲ့ User Name, Role, Permission တွေကို UI မှာ တစ်ခါတည်း အသုံးပြုနိုင်ပြီး၊ RMI Method တိုင်းမှာတော့ `sessionToken` ကိုသာ ပို့ပြီး Server က Token ကို Verify လုပ်ကာ Permission ကို ပြန်စစ်ပေးနိုင်ပါတယ်။ ဒီ Design က Maintain လုပ်ရလည်း လွယ်ပြီး Enterprise Java Application တွေမှာ အသုံးများတဲ့ ပုံစံဖြစ်ပါတယ်။