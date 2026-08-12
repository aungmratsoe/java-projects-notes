ဒီ code က Java မှာ **Object Serialization** အတွက် သုံးတာပါ။

```java
public class Student implements java.io.Serializable {
    private static final long serialVersionUID = 1L;
}
```

အခု `implements Serializable` နဲ့ `serialVersionUID` ဆိုတာ ဘာလဲဆိုတာ အခြေခံကနေ စပြီး ရှင်းပြမယ်။

---

# 1. Serializable ဆိုတာဘာလဲ?

`Serializable` က **Marker Interface** တစ်ခုပါ။

Marker Interface ဆိုတာ **method မပါတဲ့ interface** ဖြစ်တယ်။

```java
public interface Serializable {
}
```

တွေ့လား?

Method တစ်ခုမှ မရှိဘူး။

ဒါဆို ဘာအတွက်သုံးတာလဲ?

Java ကို

> "ဒီ Object ကို File ထဲသိမ်းလို့ရတယ်"  
> "Network ကနေ ပို့လို့ရတယ်"

ဆိုတာကို အမှတ်အသားပေးတာပါ။

---

## Example

```java
public class Student implements Serializable {

    private int id;
    private String name;

}
```

ဒီ Student Object ကို

- File
    
- Database BLOB
    
- Socket
    
- Cache
    
- Session
    

တို့ထဲ သိမ်းလို့ရပြီ။

---

# 2. Serialization ဆိုတာဘာလဲ?

Serialization ဆိုတာ

Object ➜ Byte Array ပြောင်းခြင်း

ဥပမာ

```java
Student student = new Student();
```

Memory ထဲမှာ

```
Student

id = 1
name = "Aung"
```

ရှိနေတယ်။

Serialization လုပ်လိုက်ရင်

```
Student

↓

0110010101010101000101010101....
```

Byte တွေဖြစ်သွားတယ်။

ဒီ Byte တွေကို

- File
    
- Network
    

ထဲပို့လို့ရတယ်။

---

# 3. Deserialization

ပြန်ဖတ်တဲ့အချိန်

```
0110010101010101....

↓

Student Object
```

ပြန်ဖြစ်လာတယ်။

ဒါကို Deserialization လို့ခေါ်တယ်။

---

# 4. Serializable မလုပ်ရင်

ဥပမာ

```java
public class Student {

}
```

ပြီးတော့

```java
ObjectOutputStream out =
    new ObjectOutputStream(new FileOutputStream("student.dat"));

out.writeObject(student);
```

Run လိုက်ရင်

```
java.io.NotSerializableException
```

ဆိုပြီး Error တက်မယ်။

ဘာကြောင့်လဲ?

Java က

> "ဒီ Object ကို Save လုပ်ခွင့်မပေးထားဘူး"

လို့ ပြောတာ။

---

Serializable ထည့်လိုက်ရင်

```java
public class Student implements Serializable {

}
```

Error မတက်တော့ဘူး။

---

# 5. serialVersionUID ဆိုတာဘာလဲ?

ဒါက အရေးကြီးတယ်။

```java
private static final long serialVersionUID = 1L;
```

ဒီကောင်က

Object Version Number ဖြစ်တယ်။

---

ဥပမာ

Version 1

```java
public class Student implements Serializable {

    int id;
    String name;

}
```

ဒီ Object ကို

```
student.dat
```

ထဲ သိမ်းထားတယ်။

---

နောက်နေ့

Class ကိုပြင်လိုက်တယ်။

```java
public class Student implements Serializable {

    int id;
    String name;
    int age;

}
```

Field အသစ်တိုးလာပြီ။

Java က

```
Old Object

↓

New Class
```

ကို Load လုပ်ရမယ်။

Java ဘယ်လိုသိလဲ?

Version ကြည့်တယ်။

အဲ့ဒါက

```
serialVersionUID
```

ပဲ။

---

# 6. serialVersionUID မရှိရင်

Java က Auto Generate လုပ်တယ်။

ဥပမာ

```
Version

348974389473984
```

Class ကို နည်းနည်းလေး ပြင်လိုက်ရုံနဲ့

```
9834759834759834
```

ဖြစ်သွားနိုင်တယ်။

ပြီးရင်

```
InvalidClassException
```

တက်လာမယ်။

---

ဥပမာ

Version A

```
123456
```

Version B

```
987654
```

မတူတော့

```
InvalidClassException
```

ဖြစ်တယ်။

---

# 7. serialVersionUID ကို ကိုယ်တိုင်ရေးရင်

```java
private static final long serialVersionUID = 1L;
```

Java က

Version တူတယ်လို့ ယူဆတယ်။

Class ကို

နည်းနည်းပြင်ထားလည်း

အဟောင်းကို ဖတ်နိုင်တာများတယ် (ပြောင်းလဲမှုအပေါ်မူတည်ပြီး)။

---

# 8. ဘာကြောင့် static?

```java
private static final long serialVersionUID = 1L;
```

Object တိုင်းမှာ Version မလိုဘူး။

Class Version ပဲလိုတယ်။

ဒါကြောင့်

```
static
```

---

# 9. ဘာကြောင့် final?

Version မပြောင်းစေချင်ဘူး။

```
1L
```

ကို Fix လုပ်ထားတာ။

---

# 10. ဘာကြောင့် long?

Java Specification က

```
long
```

ပဲ လက်ခံတယ်။

ဒါကြောင့်

```
1L
```

ရေးရတာ။

---

# 11. private ဘာကြောင့်?

Class အတွင်းမှာပဲ အသုံးပြုဖို့ပါ။

ပြင်ပက Access လုပ်စရာ မလိုဘူး။

---

# 12. `1L` က ဘာကိုဆိုလိုတာလဲ?

```java
1L
```

မှာ `L` က **Long Literal** ဖြစ်တယ်။

```java
1
```

ဆိုရင် `int`

```java
1L
```

ဆိုရင် `long`

ဖြစ်ပါတယ်။

---

# 13. ဘယ်အချိန်မှာ Serializable သုံးလဲ?

အသုံးများတဲ့နေရာတွေကတော့

- File ထဲ Object သိမ်းတဲ့အခါ
    
- Network ကနေ Object ပို့တဲ့အခါ
    
- Socket Programming
    
- RMI (Remote Method Invocation)
    
- HTTP Session
    
- Cache (Ehcache, Hazelcast စသည်)
    
- Java Swing application မှာ Settings သို့မဟုတ် User Preferences ကို Object အနေနဲ့ သိမ်းတဲ့အခါ
    

ဥပမာ Swing Application မှာ

```java
UserSetting setting = new UserSetting();

ObjectOutputStream out =
    new ObjectOutputStream(new FileOutputStream("setting.dat"));

out.writeObject(setting);
```

---

# 14. Flow Diagram

```text
Student Object
        │
        │ implements Serializable
        ▼
ObjectOutputStream
        │
        ▼
Byte Stream
        │
        ▼
student.dat
```

ပြန်ဖတ်တဲ့အခါ

```text
student.dat
      │
      ▼
ObjectInputStream
      │
      ▼
Student Object
```

---

## အကျဉ်းချုပ်

|Code|အဓိပ္ပာယ်|
|---|---|
|`implements Serializable`|ဒီ Class ရဲ့ Object ကို Byte Stream အဖြစ် ပြောင်း (Serialize) လုပ်ခွင့်ပေးသည်|
|`serialVersionUID`|Class ရဲ့ Version ID ဖြစ်ပြီး Serialization/Deserialization လုပ်တဲ့အခါ Class Version ကို စစ်ဆေးဖို့ သုံးသည်|
|`private`|Class အတွင်းမှာပဲ အသုံးပြုရန်|
|`static`|Object တိုင်းအတွက် မဟုတ်ဘဲ Class အတွက် Version တစ်ခုတည်းရှိရန်|
|`final`|Version ID မပြောင်းစေရန်|
|`1L`|`long` အမျိုးအစား Version Number|

**Interview မှာ မေးလေ့ရှိတဲ့ မေးခွန်းတစ်ခုက** "Marker Interface ဆိုတာ ဘာလဲ? `Serializable` က ဘာကြောင့် Marker Interface လို့ခေါ်တာလဲ?" ဆိုတာပါ။ ဒီအကြောင်းကို နားလည်ထားရင် `Serializable` ရဲ့ အလုပ်လုပ်ပုံကို ပိုပြီး ရှင်းရှင်းလင်းလင်း သဘောပေါက်လာပါလိမ့်မယ်။