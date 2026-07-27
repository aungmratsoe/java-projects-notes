ဒီ Code က **Java AES Encryption Utility Class** ပါ။

သူ့ရဲ့ ရည်ရွယ်ချက်ကတော့

> **QR Code ထဲမှာ Student Information ကို Plain Text မထည့်ဘဲ Encrypt လုပ်ပြီး သိမ်းဖို့** ဖြစ်ပါတယ်။

ဥပမာ

```
Original Data

ID=1001
NAME=AUNG MRAT
COURSE=JAVA
```

ဒီလိုမထည့်ဘူး။

Encrypt လုပ်ပြီး

```
q9W7mE+K31qNcX4zLxQ+q5TfP6V8oW8v...
```

ဒီလို Base64 String အဖြစ် QR ထဲကိုထည့်တယ်။

Third-party QR Scanner က Scan ရင်

```
q9W7mE+K31qNcX4zLxQ+q5TfP6V8oW8v...
```

ဒီလိုပဲ မြင်မယ်။

ကိုယ့် Application ကပဲ

```
Decrypt()

↓

ID=1001
NAME=AUNG MRAT
COURSE=JAVA
```

ပြန်ဖတ်နိုင်တယ်။

---

# Package

```
package com.ams.qrcode.utils;
```

Package Name သတ်မှတ်တာ။

Project Structure

```
com
 └── ams
      └── qrcode
             └── utils
                    CryptoUtils.java
```

---

# Import

```java
import java.nio.charset.StandardCharsets;
```

String ကို Byte ပြောင်းတဲ့အခါ UTF-8 သုံးမယ်။

ဥပမာ

```
Hello
```

↓

```
48 65 6C 6C 6F
```

---

```java
import java.security.MessageDigest;
```

Hash Algorithm သုံးဖို့။

ဒီနေရာမှာ

```
SHA-256
```

သုံးထားတယ်။

---

```java
import java.util.Arrays;
```

Array ကို Copy လုပ်ဖို့။

---

```java
import java.util.Base64;
```

Base64 Encode / Decode

ဥပမာ

```
ABC
```

↓

```
QUJD
```

---

```java
import javax.crypto.Cipher;
```

AES Encryption Engine

---

```java
import javax.crypto.spec.SecretKeySpec;
```

AES Key Object တည်ဆောက်ဖို့။

---

# Class

```java
public class CryptoUtils {
```

Utility Class

Object မဆောက်ဘဲ

```java
CryptoUtils.encrypt()

CryptoUtils.decrypt()
```

ဒီလိုခေါ်နိုင်တယ်။

---

# SECRET_KEY

```java
private static final String SECRET_KEY =
"AMS_STUDENT_IDENTITY_SYSTEM_KEY_2026";
```

ဒီဟာက

Application ရဲ့ Password ဖြစ်တယ်။

ဥပမာ

```
Secret Password

↓

AMS_STUDENT_IDENTITY_SYSTEM_KEY_2026
```

ဒီ Password မတူရင်

Decrypt မရတော့ဘူး။

---

## private

Class အပြင်က မမြင်နိုင်ဘူး။

---

## static

Object မဆောက်ဘဲ သုံးနိုင်တယ်။

---

## final

ပြောင်းလို့မရတော့ဘူး။

---

# getKey()

```java
private static SecretKeySpec getKey() throws Exception
```

ဒီ Method က

```
String Password

↓

AES Key
```

ပြောင်းပေးတာ။

---

## Step 1

```java
byte[] key =
SECRET_KEY.getBytes(StandardCharsets.UTF_8);
```

Password

```
AMS_STUDENT...
```

↓

Byte Array

```
65
77
83
95
...
```

---

## Step 2

```java
MessageDigest sha =
MessageDigest.getInstance("SHA-256");
```

SHA-256 Object ဆောက်တယ်။

SHA-256 က

```
Input

↓

32 Bytes Hash
```

ထုတ်ပေးတယ်။

---

## Step 3

```java
key = sha.digest(key);
```

ဥပမာ

```
AMS_STUDENT_IDENTITY_SYSTEM_KEY_2026
```

↓

```
E7 A1 2F B9
83 91
....
32 Bytes
```

Hash ဖြစ်သွားတယ်။

---

ဘာလို့ Hash လုပ်တာလဲ?

AES-128 Key က

```
16 Bytes
```

ပဲ လက်ခံတယ်။

ဒါပေမယ့်

```
AMS_STUDENT_IDENTITY_SYSTEM_KEY_2026
```

က

```
37 Characters
```

လောက်ရှိတယ်။

တိုက်ရိုက်မသုံးနိုင်ဘူး။

အရင်

```
SHA-256

↓

32 Bytes
```

လုပ်တယ်။

---

## Step 4

```
key = Arrays.copyOf(key,16);
```

SHA-256 က

```
32 Bytes
```

ထုတ်တယ်။

AES-128 က

```
16 Bytes
```

လိုတယ်။

ဒါကြောင့်

```
First 16 Bytes
```

ပဲ ယူတယ်။

```
32 Bytes

1111111111111111
2222222222222222

↓

1111111111111111
```

---

## Step 5

```java
return new SecretKeySpec(key,"AES");
```

AES Key Object ပြန်ပေးတယ်။

---

# encrypt()

```java
public static String encrypt(String strToEncrypt)
```

Input

```
Student Data
```

↓

Encrypted Base64

ပြန်ထုတ်တယ်။

---

## Step 1

```java
SecretKeySpec secretKey = getKey();
```

AES Key ရယူတယ်။

---

## Step 2

```java
Cipher cipher =
Cipher.getInstance("AES/ECB/PKCS5Padding");
```

ဒီလိုဖတ်ရတယ်။

```
AES

/

ECB

/

PKCS5Padding
```

### AES

Encryption Algorithm

---

### ECB

Encryption Mode

(Production မှာတော့ **ECB ကို မသုံးသင့်ပါဘူး**။ Pattern leakage ဖြစ်နိုင်လို့ `AES/GCM/NoPadding` သို့မဟုတ် `AES/CBC/PKCS5Padding` ကို ပိုအကြံပြုကြပါတယ်။ ဒါပေမယ့် သင့်လို Local Student QR Project အတွက် သင်ယူရေးအနေနဲ့ နားလည်ဖို့တော့ လွယ်ပါတယ်။)

---

### PKCS5Padding

AES က

```
16 Bytes
```

အုပ်စုလိုက် Encrypt လုပ်တယ်။

ဥပမာ

```
HELLO
```

က

```
5 Bytes
```

ပဲရှိတယ်။

ဒါကြောင့်

```
HELLOXXXXXXXXXXX
```

လို Padding ဖြည့်တယ်။

Decrypt လုပ်တဲ့အချိန်

Padding ကိုပြန်ဖယ်တယ်။

---

## Step 3

```java
cipher.init(
Cipher.ENCRYPT_MODE,
secretKey
);
```

Encrypt Mode နဲ့ Key ထည့်ပေးတယ်။

---

## Step 4

```java
byte[] encryptedBytes =
cipher.doFinal(
strToEncrypt.getBytes(StandardCharsets.UTF_8)
);
```

ဒီ Line က တကယ် Encrypt လုပ်တဲ့နေရာ။

```
HELLO
```

↓

```
8F A2 C4 ...
```

Binary Data ထွက်လာတယ်။

---

## Step 5

```java
return Base64
.getEncoder()
.encodeToString(encryptedBytes);
```

Encrypted Binary ကို

Base64 ပြောင်းတယ်။

ဘာလို့?

Binary Data က

```
�ƿ��@�
```

လို ဖြစ်နေတတ်တယ်။

QR Code ထဲမှာ ထည့်ရခက်တယ်။

ဒါကြောင့်

```
Base64

↓

rT76M8Pke9Qa...
```

ပြောင်းတယ်။

---

# decrypt()

```java
public static String decrypt(String strToDecrypt)
```

Encrypt ရဲ့ ပြောင်းပြန်လုပ်ငန်း။

---

## Step 1

```java
SecretKeySpec secretKey = getKey();
```

Key ယူတယ်။

---

## Step 2

```java
Cipher cipher =
Cipher.getInstance(
"AES/ECB/PKCS5Padding");
```

Encrypt နဲ့ တူရမယ်။

---

## Step 3

```java
cipher.init(
Cipher.DECRYPT_MODE,
secretKey
);
```

Decrypt Mode ပြောင်းတယ်။

---

## Step 4

```java
Base64.getDecoder()
.decode(strToDecrypt.trim())
```

Base64

↓

Original Binary

---

## Step 5

```java
cipher.doFinal(...)
```

Decrypt လုပ်တယ်။

```
rT76M8P...

↓

HELLO
```

---

## Step 6

```java
return new String(
decryptedBytes,
StandardCharsets.UTF_8
);
```

Byte Array

↓

String

ပြောင်းတယ်။

---

# Catch Block

```java
catch(Exception e){
    return null;
}
```

ဘာတွေ Error တက်နိုင်လဲ?

ဥပမာ

QR Scanner နဲ့ Scan လုပ်လိုက်တဲ့ QR က

```
HELLO
```

Plain Text ဖြစ်နေတယ်။

ဒါကို

```java
decrypt()
```

ခေါ်လိုက်ရင်

```
Invalid Base64
```

ဖြစ်နိုင်တယ်။

ဒါမှမဟုတ်

Wrong Key နဲ့ Decrypt လုပ်ရင်

```
BadPaddingException

IllegalBlockSizeException
```

တက်နိုင်တယ်။

ဒီအချိန်

```
null
```

ပြန်ပေးတယ်။

ဒါကြောင့်

```java
String result = CryptoUtils.decrypt(data);

if(result == null){
    JOptionPane.showMessageDialog(
        null,
        "Invalid QR Code"
    );
}
```

ဒီလို စစ်လို့ရတယ်။

---

# Code Flow တစ်ခုလုံး

```
Student Information

        │
        ▼
encrypt()

        │
        ▼
SECRET_KEY

        │
        ▼
SHA-256

        │
        ▼
First 16 Bytes

        │
        ▼
AES-128 Key

        │
        ▼
AES Encrypt

        │
        ▼
Encrypted Bytes

        │
        ▼
Base64 Encode

        │
        ▼
QR Code
```

Scan ပြန်ဖတ်တဲ့အချိန်

```
QR Code

      │
      ▼
Base64 Decode

      │
      ▼
AES Decrypt

      │
      ▼
Original Student Data
```

---

# ဒီ Code ရဲ့ အားသာချက်

- ✅ QR Code ထဲမှာ Plain Text မသိမ်းထားလို့ အချက်အလက်ကို တိုက်ရိုက်မဖတ်နိုင်ဘူး။
- ✅ Encrypt/Decrypt API က ရိုးရှင်းပြီး `CryptoUtils.encrypt()`၊ `CryptoUtils.decrypt()` နဲ့ အသုံးပြုရလွယ်တယ်။
- ✅ Secret Key ကို တူညီစွာသုံးတဲ့ Application ကပဲ Data ကိုပြန်ဖတ်နိုင်တယ်။

# သတိထားရမယ့် အချက်များ

- ⚠️ `SECRET_KEY` ကို Code ထဲမှာ Hard-code လုပ်ထားတာကြောင့် Application ကို Reverse Engineer လုပ်နိုင်သူက Key ကို ရှာတွေ့နိုင်ပါတယ်။
- ⚠️ `AES/ECB/PKCS5Padding` သည် သင်ယူရေး သို့မဟုတ် ရိုးရှင်းတဲ့ Project များအတွက် အဆင်ပြေပေမယ့် Security ပိုလိုအပ်တဲ့ Production System တွေမှာ မသုံးသင့်ပါ။
- ⚠️ Production အတွက် `AES/GCM/NoPadding` (Authenticated Encryption) သို့မဟုတ် အနည်းဆုံး `AES/CBC/PKCS5Padding` + Random IV ကို အသုံးပြုတာက ပိုလုံခြုံပါတယ်။

**အကြံပြုချက်:** Student Management System အတွက် Local QR Verification Project ဆိုရင် ဒီ Code က AES Encryption, Hashing, Base64, Secret Key, Cipher API တွေကို နားလည်ဖို့ အလွန်ကောင်းတဲ့ နမူနာတစ်ခုဖြစ်ပါတယ်။ Security ပိုမြင့်တဲ့ Version တစ်ခု တည်ဆောက်ချင်ရင်တော့ နောက်တစ်ဆင့်အနေနဲ့ **AES-GCM**, **Random IV**, **Key Management**, **Digital Signature** တွေကို ဆက်လေ့လာသင့်ပါတယ်။