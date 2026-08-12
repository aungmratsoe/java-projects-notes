## သင့် Project ကို RMI ဖြင့် Client-Server ခွဲနည်း 🇲🇲

### လက်ရှိ Project Structure

QRCode (Single App)

├── model/        → Student.java, User.java

├── dao/          → Database logic (StudentDAO, UserDAO)

├── db/           → Database connection

├── ui/           → Swing UI (Home, Login, QRGenerator, QRScanner...)

├── utils/        → QRUtils, CryptoUtils, PasswordUtils

└── exceptions/   → Custom exceptions

---

### RMI ဆိုတာ ဘာလဲ?

**RMI (Remote Method Invocation)** ဆိုတာ Java မှာ တစ် machine ပေါ်ရှိ program က ကွန်ရက် (network) မှတဆင့် အခြား machine တစ်ခုပေါ်ရှိ method တွေကို ခေါ်သုံးနိုင်တဲ့ နည်းပညာပါ။

[Client PC]  ←—— Network ——→  [Server PC]

  Swing UI                      Database

  (ကျောင်းသား ကြည့်/တင်)        (MySQL + DAO)

---

### ဘာတွေ ခွဲရမလဲ?

|ဘာ|ဘယ်မှာ ထားမလဲ|ရှင်းချက်|
|---|---|---|
|`model/` (Student, User)|**Client + Server နှစ်ဖက်လုံး**|Network ကတဆင့် object ပို့ရ|
|`dao/` (DAO interfaces)|**Client + Server နှစ်ဖက်လုံး**|Interface သာ Client ဆီ|
|`dao/` (DAO implementations)|**Server သာ**|DB logic ကို client မမြင်ရ|
|`db/` (Database connection)|**Server သာ**|DB ပေါ်တည်|
|`ui/` (Swing forms)|**Client သာ**|User interface|
|`utils/` (QRUtils, CryptoUtils)|အပေါ်မူတည်|QR generate → Client, DB verify → Server|

---

### Projects ၃ ခု ဖန်တီးရမည်

📁 QRCode-Common/    ← Model + Interfaces ( နှစ်ဖက် share)

📁 QRCode-Server/    ← DAO implementation + DB + RMI server

📁 QRCode-Client/    ← Swing UI + RMI client

---

### အဆင့်ဆင့် လုပ်နည်း

#### **အဆင့် ၁** — RMI Interface တွေ ပြင်ဆင်ပါ

သင့်မှာ ရှိပြီးသား StudentDAOInterface နဲ့ UserDAOInterface ကို RMI interface အဖြစ် ပြောင်းပါ — `Remote` နဲ့ `RemoteException` ထည့်ရမည်:

java

```java
// QRCode-Common ထဲမှာ

import java.rmi.Remote;

import java.rmi.RemoteException;

public interface StudentService extends Remote {

    void addStudent(Student student) throws RemoteException, DataAccessException;

    void updateStudent(Student student) throws RemoteException, DataAccessException;

    void deleteStudent(String studentId) throws RemoteException, DataAccessException;

    Student getStudentByStudentId(String studentId) throws RemoteException, DataAccessException;

    List<Student> getAllStudents() throws RemoteException, DataAccessException;

    List<Student> searchStudents(String keyword) throws RemoteException, DataAccessException;

    // ... အခြားများ

}
```

```java
public interface UserService extends Remote {

    boolean registerUser(User user) throws RemoteException, DataAccessException;

    User loginUser(String email, String password) throws RemoteException, DataAccessException;

}
```

> **အရေးကြီး:** Student နဲ့ User class တွေ `Serializable` implement လုပ်ရမည် — network ကတဆင့် object ပို့လို့ရဖို့

java

```java
// Student.java မှာ ဖြည့်ရမည်

public class Student implements Serializable {

    private static final long serialVersionUID = 1L;

    // ... ကျန်သောကုဒ်

}
```

#### **အဆင့် ၂** — Server ဖက် (RMI Server)

java

```java
// QRCode-Server ထဲမှာ

public class StudentServiceImpl extends UnicastRemoteObject implements StudentService {

    private StudentDAO dao = new StudentDAO(); // မင်းရဲ့ StudentDAO

    public StudentServiceImpl() throws RemoteException {}

    @Override

    public List<Student> getAllStudents() throws RemoteException, DataAccessException {

        return dao.getAllStudents(); // ဒါပဲ ခေါ်ရတာ

    }

    // ... အခြားများ

}
```

```java
// Server စတင်မယ့် main class

public class RMIServer {

    public static void main(String[] args) throws Exception {

        StudentService studentService = new StudentServiceImpl();

        UserService userService = new UserServiceImpl();

        Registry registry = LocateRegistry.createRegistry(1099); // port

        registry.rebind("StudentService", studentService);

        registry.rebind("UserService", userService);

        System.out.println("RMI Server started on port 1099...");

    }
}
```

#### **အဆင့် ၃** — Client ဖက် (Swing UI)

java

```java
// QRCode-Client ထဲမှာ — UI form တွေ ခေါ်တဲ့နည်း

public class RMIClient {

    private static StudentService studentService;

    private static UserService userService;

    public static void connect(String serverIP) throws Exception {

        Registry registry = LocateRegistry.getRegistry(serverIP, 1099);

        studentService = (StudentService) registry.lookup("StudentService");

        userService = (UserService) registry.lookup("UserService");

    }

    public static StudentService getStudentService() { return studentService; }

    public static UserService getUserService() { return userService; }

}

```

မင်းရဲ့ Swing UI 

QRScanner.java / 

QRGenerator.java ထဲမှာ DAO ခေါ်တဲ့ နေရာကို ဒီလို ပြောင်းရမည်:


```java
// ယခင် (direct DB)

StudentDAO dao = new StudentDAO();

Student s = dao.getStudentByStudentId(id);

// RMI ဖြင့် (network call)

Student s = RMIClient.getStudentService().getStudentByStudentId(id);

```
---

### Project Structure ပြောင်းလဲမှု Summary

📁 QRCode-Common/  (JAR လုပ်ပြီး နှစ်ဖက်မှာ dependency ထည့်)

├── model/

│   ├── Student.java  ← implements Serializable

│   └── User.java     ← implements Serializable

├── exceptions/

│   └── DataAccessException.java

└── rmi/
	├── StudentService.java  ← extends Remote (interface)

    └── UserService.java     ← extends Remote (interface)

📁 QRCode-Server/

├── dao/

│   ├── StudentDAO.java         ← DB logic (မပြောင်းဘဲ)

│   └── UserDAO.java

├── db/

│   └── DatabaseConnection.java

├── rmi/

│   ├── StudentServiceImpl.java ← implements StudentService

│   └── UserServiceImpl.java

└── RMIServer.java              ← main entry point

📁 QRCode-Client/

├── ui/

│   ├── Home.java, Login.java, QRGenerator.java, QRScanner.java...

└── RMIClient.java              ← server နဲ့ ချိတ်

---

### အကျဉ်းချုပ် — ဘာတွေ သတိထားမလဲ

|အချက်|ရှင်းချက်|
|---|---|
|`Serializable`|Model class တွေ network ကတဆင့် ပြေးနိုင်ဖို့ မဖြစ်မနေ ထည့်ရမည်|
|`RemoteException`|Remote interface method တိုင်းမှာ throw ရမည်|
|Port 1099|Java RMI default port, firewall ဖွင့်ထားဖို့|
|Common JAR|Model + Interface တွေကို jar လုပ်ပြီး Server/Client နှစ်ဖက်ထည့်မှ Java compile ဖြစ်မည်|
|Server IP|Client မှာ server IP address ထည့်ဖို့လိုမည်|

---

ဒီ project ကို RMI ဖြင့် ခွဲချင်မယ်ဆိုရင် **Netbeans မှာ Maven Multi-Module Project** (သို့မဟုတ်) **Project ၃ ခု သပ်သပ်** ဖန်တီးနိုင်ပါတယ်။ ဘယ် approach ကိုသွားမလဲ ဆုံးဖြတ်ပေးပါ — ကုဒ်တွေကို အသေအချာ generate လုပ်ပေးပါမည်! 🚀