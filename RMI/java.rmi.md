`java.rmi` package ထဲက အဓိက method/class တွေကို ရှင်းပြပေးပါမယ်။

## java.rmi Package Structure

`java.rmi` package ထဲမှာ interface, class အများကြီး ပါပေမယ့် **အသုံးအများဆုံး** တွေကတော့:

- **`Remote`** interface (marker interface)
- **`Naming`** class (client-side lookup)
- **`RemoteException`** class
- Sub-package: **`java.rmi.registry`** (Registry, LocateRegistry)
- Sub-package: **`java.rmi.server`** (UnicastRemoteObject)

---

## 1. `java.rmi.Remote` Interface

Remote object ဖြစ်ဖို့ implement လုပ်ရမယ့် **marker interface** ပါ (method မပါဘူး) — RMI နဲ့ export လုပ်မယ့် interface **အားလုံး** ဒီကို extend လုပ်ရပါတယ်။

```java
public interface StudentService extends Remote {
    // Remote method တစ်ခုချင်းစီက RemoteException throw လုပ်ရမယ်
    List<Student> getAllStudents() throws RemoteException;
    void addStudent(Student s) throws RemoteException;
}
```

---

## 2. `java.rmi.Naming` Class (Static Methods)

Client side ကနေ remote object ကို lookup/bind လုပ်ဖို့ static method **၅ ခု** ပါတယ်:

### `Naming.lookup(String url)`

Server ကနေ remote object reference ကို ရယူတယ် (Client side):

```java
StudentService service = (StudentService) Naming.lookup("rmi://192.168.1.10:1099/StudentService");
```

### `Naming.bind(String name, Remote obj)`

Remote object ကို registry ထဲ **အသစ်** register လုပ်တယ် (Server side) — name ရှိပြီးသားဆိုရင် `AlreadyBoundException` throw:

```java
StudentServiceImpl impl = new StudentServiceImpl();
Naming.bind("StudentService", impl);
```

### `Naming.rebind(String name, Remote obj)`

Register လုပ်တယ်, ရှိပြီးသား name ရှိရင် **overwrite** လုပ်တယ် (bind ထက် ပိုသုံးလေ့ရှိတယ် - server restart ရင် error မတက်အောင်):

```java
Naming.rebind("StudentService", impl);
```

### `Naming.unbind(String name)`

Registry ကနေ binding ကို ဖြုတ်တယ်:

```java
Naming.unbind("StudentService");
```

### `Naming.list(String url)`

Registry ထဲမှာ bind ထားတဲ့ service name အားလုံးကို string array အနေနဲ့ ပြန်ပေးတယ်:

```java
String[] services = Naming.list("rmi://192.168.1.10:1099");
```

---

## 3. `java.rmi.RemoteException` Class

Network-related error (connection fail, server unreachable, marshalling error) ဖြစ်ရင် throw ဖြစ်တဲ့ checked exception ပါ — Remote interface ရဲ့ method **အားလုံး** ဒါကို throw declare လုပ်ရပါတယ်။

```java
try {
    service.addStudent(student);
} catch (RemoteException ex) {
    // Network/connection error handle
    JOptionPane.showMessageDialog(this, "Server connection lost: " + ex.getMessage());
}
```

**Common subclasses**: `ConnectException`, `ConnectIOException`, `NoSuchObjectException`, `ServerException`, `UnmarshalException`

---

## 4. `java.rmi.registry` Sub-package

### `LocateRegistry` Class (Server side - Registry ဖန်တီး/ချိတ်ဆက်ဖို့)

**`createRegistry(int port)`** — Server side မှာ registry အသစ် စတင်တယ်:

```java
Registry registry = LocateRegistry.createRegistry(1099);
```

**`getRegistry(String host, int port)`** — Registry ရှိပြီးသားကို connect လုပ်တယ်:

```java
Registry registry = LocateRegistry.getRegistry("192.168.1.10", 1099);
```

### `Registry` Interface (bind/lookup alternative to Naming class)

```java
registry.rebind("StudentService", impl);
StudentService service = (StudentService) registry.lookup("StudentService");
```

---

## 5. `java.rmi.server` Sub-package

### `UnicastRemoteObject` Class (Server side implementation base class)

Remote object implementation class က ဒါကို **extend** လုပ်ရတယ် (network stub ကို auto export လုပ်ပေးတယ်):

```java
public class StudentServiceImpl extends UnicastRemoteObject implements StudentService {
    
    public StudentServiceImpl() throws RemoteException {
        super(); // parent constructor RemoteException throw လို့ ဒီလိုပဲ ရေးရမယ်
    }
    
    @Override
    public List<Student> getAllStudents() throws RemoteException {
        return studentDao.getAll();
    }
    
    @Override
    public void addStudent(Student s) throws RemoteException {
        studentDao.insert(s);
    }
}
```

**`exportObject(Remote obj, int port)`** — static method, `extends UnicastRemoteObject` မလုပ်ချင်ရင် manual export:

```java
StudentServiceImpl impl = new StudentServiceImpl();
StudentService stub = (StudentService) UnicastRemoteObject.exportObject(impl, 0);
```

---

## Full Example — Server Side Setup

```java
// StudentServiceImpl.java
public class StudentServiceImpl extends UnicastRemoteObject implements StudentService {
    public StudentServiceImpl() throws RemoteException { super(); }
    
    public List<Student> getAllStudents() throws RemoteException {
        return studentDao.getAll();
    }
}

// Server.java (main class)
public class Server {
    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.createRegistry(1099);
            StudentServiceImpl impl = new StudentServiceImpl();
            registry.rebind("StudentService", impl);
            System.out.println("Server started...");
        } catch (RemoteException ex) {
            ex.printStackTrace();
        }
    }
}
```

## Full Example — Client Side Lookup

```java
public class Client {
    public static void main(String[] args) {
        try {
            StudentService service = (StudentService) Naming.lookup("rmi://localhost:1099/StudentService");
            List<Student> students = service.getAllStudents();
            students.forEach(System.out::println);
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```

---

## Quick Summary Table

|Class/Interface|ရည်ရွယ်ချက်|Side|
|---|---|---|
|`Remote`|Marker interface (implement လုပ်ရမယ်)|Both|
|`Naming`|URL-based lookup/bind/rebind/unbind/list|Client (lookup), Server (bind)|
|`RemoteException`|Network error checked exception|Both|
|`LocateRegistry`|Registry create/get|Server (create), Client (getRegistry)|
|`Registry`|bind/rebind/lookup interface|Both|
|`UnicastRemoteObject`|Remote object base class (auto export)|Server|

---

QRCode project ရဲ့ `StudentService` interface design (method signature တွေ) concrete အနေနဲ့ ရေးပေးရမလား၊ ဒါမှမဟုတ် `java.rmi.server` ထဲက **`RMISocketFactory`** (custom port/SSL configuration) အကြောင်း ထပ်လေ့လာချင်ပါသလား?