# Split QRCode into RMI Client + Server Projects

Run on **two computers** over a local network using Java RMI (Remote Method Invocation).

## Architecture

[Server Computer]                [Client Computer]

  MySQL Database        ←──        Swing UI

  QRCode-Server.jar    ←── RMI ──  QRCode-Client.jar

  (port 1099)                      (connects to server IP)

## Projects to Create

|Project|Location|Purpose|
|---|---|---|
|`QRCode-Common`|New folder|Shared: Model classes + RMI interfaces|
|`QRCode-Server`|New folder|DB logic + RMI server (runs on Server PC)|
|`QRCode-Client`|Modified|Swing UI + RMI client (runs on Client PC)|

---

## Proposed Changes

### QRCode-Common (New Project)

IMPORTANT

This is a shared library. Must be `mvn install` first, before building Server or Client.

#### [NEW] QRCode-Common/pom.xml

Plain Maven JAR project with `groupId = com.ams`, `artifactId = QRCode-Common`.

#### [MODIFY] Student.java (copy from original)

java

// Add Serializable

public class Student implements Serializable {

    private static final long serialVersionUID = 1L;

    // ... rest unchanged

}

#### [MODIFY] User.java (copy from original)

java

public class User implements Serializable {

    private static final long serialVersionUID = 2L;

}

#### [NEW] StudentService.java (RMI interface)

java

package com.ams.qrcode.rmi;

import java.rmi.Remote;

import java.rmi.RemoteException;

public interface StudentService extends Remote {

    void saveStudent(Student s) throws RemoteException, DataAccessException;

    void addStudent(Student s) throws RemoteException, DataAccessException;

    void updateStudent(Student s) throws RemoteException, DataAccessException;

    void updateQrToken(String studentId, String token) throws RemoteException, DataAccessException;

    boolean deleteStudent(String studentId) throws RemoteException, DataAccessException;

    Student getStudentByStudentId(String id) throws RemoteException, DataAccessException;

    List<Student> getAllStudents() throws RemoteException, DataAccessException;

    List<Student> searchStudents(String keyword) throws RemoteException, DataAccessException;

}

#### [NEW] UserService.java (RMI interface)

java

public interface UserService extends Remote {

    boolean registerUser(User user) throws RemoteException, DataAccessException;

    User loginUser(String email, String password) throws RemoteException, DataAccessException;

}

Copy 

DataAccessException.java and 

User.java into common as well.

---

### QRCode-Server (New Project)

#### [NEW] QRCode-Server/pom.xml

Depends on `QRCode-Common` + `mysql-connector` + `jbcrypt`.

#### Copy from original: `dao/`, `db/`

StudentDAO.java, 

UserDAO.java, 

DBConnection.java — **no changes needed**. They still connect directly to MySQL.

#### [NEW] StudentServiceImpl.java

java

public class StudentServiceImpl extends UnicastRemoteObject 

                                implements StudentService {

    private final StudentDAO dao = new StudentDAO();

    public StudentServiceImpl() throws RemoteException {}

    @Override

    public List<Student> getAllStudents() throws RemoteException, DataAccessException {

        return dao.getAllStudents();

    }

    // ... all other methods delegate to dao

}

#### [NEW] UserServiceImpl.java

Same pattern — wraps 

UserDAO.

#### [NEW] RMIServer.java (main class)

java

public class RMIServer {

    public static void main(String[] args) throws Exception {

        Registry registry = LocateRegistry.createRegistry(1099);

        registry.rebind("StudentService", new StudentServiceImpl());

        registry.rebind("UserService", new UserServiceImpl());

        System.out.println("✅ RMI Server running on port 1099...");

    }

}

---

### QRCode-Client (Modified from original)

#### [MODIFY] pom.xml

- **Remove**: `mysql-connector-j` dependency (no DB access on client)
- **Add**: `QRCode-Common` dependency
- Keep: ZXing, webcam, FlatLaf, jbcrypt, jcalendar

#### [NEW] RMIClient.java

Central helper class to connect to server:

java

public class RMIClient {

    private static StudentService studentService;

    private static UserService userService;

    public static void connect(String serverIP) throws Exception {

        Registry reg = LocateRegistry.getRegistry(serverIP, 1099);

        studentService = (StudentService) reg.lookup("StudentService");

        userService = (UserService) reg.lookup("UserService");

    }

    public static StudentService student() { return studentService; }

    public static UserService user() { return userService; }

}

#### [NEW] ServerConfigDialog.java (startup dialog)

A small input dialog on app startup asking for Server IP address.

#### [MODIFY] QRScanner.java

Replace line 56:

java

// BEFORE:

private final StudentDAOInterface studentDAO = new StudentDAO();

// AFTER:

private StudentService studentService = RMIClient.student();

And line 228:

java

// BEFORE:

Student student = studentDAO.getStudentByStudentId(studentId);

// AFTER:

Student student = RMIClient.student().getStudentByStudentId(studentId);

#### [MODIFY] QRGenerator.java, Home.java, Login.java, SignUp.java

Same pattern — replace `new StudentDAO()` / `new UserDAO()` calls with `RMIClient.student()` / `RMIClient.user()`.

#### [DELETE] dao/, db/ packages from Client

Client should NOT have these.

---

## Verification Plan

### Build Order (must follow this order)

1. mvn install  (in QRCode-Common folder)

2. mvn package  (in QRCode-Server folder)

3. mvn package  (in QRCode-Client folder)

### Manual Testing Steps

**On Server Computer:**

1. Start MySQL, verify `student_db` database exists
2. Run: `java -jar QRCode-Server.jar`
3. Confirm console shows: `✅ RMI Server running on port 1099...`
4. Allow port 1099 in Windows Firewall if needed

**On Client Computer:**

1. Run: `java -jar QRCode-Client.jar`
2. Enter the Server Computer's IP address in the startup dialog
3. Test Login — should authenticate via server
4. Test QR Scanner — scan a student QR, result fetched from server DB
5. Test QR Generator — generate and save a QR code
6. Test Add/Edit/Delete students from the admin panel

### Firewall Note

WARNING

On the Server computer, open port **1099** in Windows Firewall: `Control Panel → Windows Defender Firewall → Advanced Settings → Inbound Rules → New Rule → Port 1099`