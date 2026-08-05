# Student & User DAO ကို RMI Service ပြောင်းခြင်း

သင့် `DAOInterface` ၂ ခုစလုံးကို base ယူပြီး Remote Service layer တည်ဆောက်ပေးပါမယ်။ `DBConnection` ကတော့ **ဘာမှ ပြင်စရာမလိုပါ** — Server ဘက်ကပဲ ကျန်နေရုံပါပဲ (Client ကို ဒီ class ကူးပေးစရာ လုံးဝမလို)။

## Step 1 — Model Classes ကို Serializable လုပ်

```java
// com.ams.qrcode.model.Student
public class Student implements java.io.Serializable {
    private static final long serialVersionUID = 1L;
    // fields များ ရှိသလို ချန်ထား — Serializable ပဲ ထပ်ထည့်
}
```

```java
// com.ams.qrcode.model.User
public class User implements java.io.Serializable {
    private static final long serialVersionUID = 1L;
    // fields များ ရှိသလို ချန်ထား
}
```

## Step 2 — Exception ကို Serializable စစ်

```java
// com.ams.qrcode.exceptions.DataAccessException
public class DataAccessException extends Exception implements java.io.Serializable {
    public DataAccessException(String message) { super(message); }
    public DataAccessException(String message, Throwable cause) { super(message, cause); }
}
```

## Step 3 — Remote Service Interfaces အသစ် (Server + Client နှစ်ဖက်စလုံး)

`com.ams.qrcode.service` package အသစ်ဖန်တီးပါ:

```java
// com.ams.qrcode.service.StudentService.java
package com.ams.qrcode.service;

import com.ams.qrcode.exceptions.DataAccessException;
import com.ams.qrcode.model.Student;
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.List;

public interface StudentService extends Remote {
    void addStudent(Student student) throws RemoteException, DataAccessException;
    void updateStudent(Student student) throws RemoteException, DataAccessException;
    void updateQrToken(String studentId, String newToken) throws RemoteException, DataAccessException;
    boolean deleteStudent(String studentId) throws RemoteException, DataAccessException;
    void saveStudent(Student student) throws RemoteException, DataAccessException;
    Student getStudentByStudentId(String studentId) throws RemoteException, DataAccessException;
    List<Student> getAllStudents() throws RemoteException, DataAccessException;
    List<Student> searchStudentsByName(String keyword) throws RemoteException, DataAccessException;
    List<Student> searchStudents(String keyword) throws RemoteException, DataAccessException;
}
```

```java
// com.ams.qrcode.service.UserService.java
package com.ams.qrcode.service;

import com.ams.qrcode.exceptions.DataAccessException;
import com.ams.qrcode.model.User;
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface UserService extends Remote {
    boolean registerUser(User user) throws RemoteException, DataAccessException;
    User loginUser(String email, String plainPassword) throws RemoteException, DataAccessException;
}
```

**Pattern သတိထားရန်**: သင့် DAO interface ရဲ့ method **အားလုံးကို ကူးယူပြီး `RemoteException` ကို method တိုင်းရဲ့ `throws` ထဲ ထပ်ထည့်ရုံပါပဲ**။ Method name/parameter/return type အားလုံး **တိတိကျကျ တူညီရပါမယ်**။

## Step 4 — Service Implementation (Server ဘက်ချည်း — DAO ကို Wrap)

သင့်ရဲ့ ရှိပြီးသား `StudentDAOInterface`, `UserDAOInterface` ကို implement လုပ်ထားတဲ့ class (ဥပမာ `StudentDAOImpl`, `UserDAOImpl`) ရှိပြီးသားလို့ ယူဆပါတယ် — ဒါတွေကို **ပြင်စရာလုံးဝမလို**ဘဲ အောက်ပါအတိုင်းပဲ wrap လုပ်ရပါမယ်:

```java
// com.ams.qrcode.service.impl.StudentServiceImpl.java
package com.ams.qrcode.service.impl;

import com.ams.qrcode.service.StudentService;
import com.ams.qrcode.dao.StudentDAOInterface;
import com.ams.qrcode.dao.StudentDAOImpl; // သင့်ရှိပြီးသား implementation
import com.ams.qrcode.exceptions.DataAccessException;
import com.ams.qrcode.model.Student;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.List;

public class StudentServiceImpl extends UnicastRemoteObject implements StudentService {

    private final StudentDAOInterface dao;

    public StudentServiceImpl() throws RemoteException {
        super();
        this.dao = new StudentDAOImpl(); // သင့် DAO class name အတိုင်း ပြင်ပါ
    }

    @Override
    public void addStudent(Student student) throws RemoteException, DataAccessException {
        dao.addStudent(student);
    }

    @Override
    public void updateStudent(Student student) throws RemoteException, DataAccessException {
        dao.updateStudent(student);
    }

    @Override
    public void updateQrToken(String studentId, String newToken) throws RemoteException, DataAccessException {
        dao.updateQrToken(studentId, newToken);
    }

    @Override
    public boolean deleteStudent(String studentId) throws RemoteException, DataAccessException {
        return dao.deleteStudent(studentId);
    }

    @Override
    public void saveStudent(Student student) throws RemoteException, DataAccessException {
        dao.saveStudent(student);
    }

    @Override
    public Student getStudentByStudentId(String studentId) throws RemoteException, DataAccessException {
        return dao.getStudentByStudentId(studentId);
    }

    @Override
    public List<Student> getAllStudents() throws RemoteException, DataAccessException {
        return dao.getAllStudents();
    }

    @Override
    public List<Student> searchStudentsByName(String keyword) throws RemoteException, DataAccessException {
        return dao.searchStudentsByName(keyword);
    }

    @Override
    public List<Student> searchStudents(String keyword) throws RemoteException, DataAccessException {
        return dao.searchStudents(keyword);
    }
}
```

```java
// com.ams.qrcode.service.impl.UserServiceImpl.java
package com.ams.qrcode.service.impl;

import com.ams.qrcode.service.UserService;
import com.ams.qrcode.dao.UserDAOInterface;
import com.ams.qrcode.dao.UserDAOImpl; // သင့်ရှိပြီးသား implementation
import com.ams.qrcode.exceptions.DataAccessException;
import com.ams.qrcode.model.User;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class UserServiceImpl extends UnicastRemoteObject implements UserService {

    private final UserDAOInterface dao;

    public UserServiceImpl() throws RemoteException {
        super();
        this.dao = new UserDAOImpl();
    }

    @Override
    public boolean registerUser(User user) throws RemoteException, DataAccessException {
        return dao.registerUser(user);
    }

    @Override
    public User loginUser(String email, String plainPassword) throws RemoteException, DataAccessException {
        return dao.loginUser(email, plainPassword);
    }
}
```

## Step 5 — Server Main Class (Service နှစ်ခုစလုံး Register)

```java
// com.ams.qrcode.server.ServerMain.java
package com.ams.qrcode.server;

import com.ams.qrcode.service.impl.StudentServiceImpl;
import com.ams.qrcode.service.impl.UserServiceImpl;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class ServerMain {
    public static void main(String[] args) {
        try {
            System.setProperty("java.rmi.server.hostname", "192.168.1.10"); // Server ရဲ့ WiFi IP

            Registry registry = LocateRegistry.createRegistry(1099);

            registry.rebind("StudentService", new StudentServiceImpl());
            registry.rebind("UserService", new UserServiceImpl());

            System.out.println("Server ready — StudentService, UserService bound on port 1099");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Step 6 — Client ဘက် Usage ဥပမာ (Login Screen)

```java
// Client ဘက် — Login logic
Registry registry = LocateRegistry.getRegistry("192.168.1.10", 1099);
UserService userService = (UserService) registry.lookup("UserService");
StudentService studentService = (StudentService) registry.lookup("StudentService");

// Login button click → SwingWorker ထဲမှာ
SwingWorker<User, Void> worker = new SwingWorker<>() {
    @Override
    protected User doInBackground() throws Exception {
        return userService.loginUser(emailField.getText(), new String(passwordField.getPassword()));
    }
    @Override
    protected void done() {
        try {
            User user = get();
            if (user != null) {
                // Main window ဖွင့်၊ studentService ကို panel တွေဆီ pass
            } else {
                JOptionPane.showMessageDialog(LoginFrame.this, "Email/Password မှားနေပါသည်");
            }
        } catch (Exception ex) {
            JOptionPane.showMessageDialog(LoginFrame.this, "Server ဆက်သွယ်မှု error: " + ex.getMessage());
        }
    }
};
worker.execute();
```

## ⚠️ လုံခြုံရေး သတိပေးချက် — Password Plain Text ပို့နေခြင်း

`loginUser(email, plainPassword)` ဟာ **password ကို plain text အတိုင်း network ပေါ်ကနေ ပို့** နေပါတယ်။ RMI default က **encryption မရှိပါ** (Part 5 က ရှင်းပြခဲ့သလိုပါပဲ) — WiFi ပေါ်က တခြားလူတစ်ယောက်က traffic sniff လုပ်ရင် password ကို ဖတ်လို့ရနိုင်ပါတယ်။ Production အတွက်ဆိုရင် **`SslRMIClientSocketFactory`/`SslRMIServerSocketFactory`** (Part 5 က ဥပမာအတိုင်း) ကို `UserServiceImpl` export လုပ်တဲ့နေရာမှာ ထည့်သင့်ပါတယ်။

## DBConnection အတွက် မှတ်ချက်

```java
private static final String USER = "root";
private static final String PASSWORD = "password";
```

ဒါတွေကို **hardcode** မထားဘဲ `.properties` file (သို့) environment variable ကနေ ဖတ်တာက ပိုကောင်းပါတယ် — Server code ကို git repo ထဲ commit လုပ်တဲ့အခါ DB password ပါသွားတတ်လို့ (Client ကို ဒီ file ကူးပေးစရာမလိုတာကတော့ RMI setup မှန်ကန်နေပါပြီ)။

---

`StudentDAOImpl`/`UserDAOImpl` (actual implementation class) ကို ပြပေးရင် **method logic ကို RMI-safe ဖြစ်အောင်** (SQLException handling, connection close စသည်) ပြန်စစ်ပေးနိုင်ပါတယ်။ SSL encryption ကို Login service အတွက် ထည့်ပုံ အသေးစိတ် ကူညီရမလား?