# Java Student QR Code Verification System - အလုပ်လုပ်ပုံ ရှင်းလင်းချက်

ဤ Project သည် **Java Swing (GUI)**၊ **MySQL (Database)**၊ **ZXing Library (QR Reader/Writer)** နှင့် **Sarxos Webcam Library** တို့ကို ပေါင်းစပ်၍ ရေးသားထားသော **ကျောင်းသားများ၏ ID နှင့် Access ကို QR Code ဖြင့် စစ်ဆေးသည့် စနစ် (Student QR Identification & Security System)** ဖြစ်ပါသည်။

## 1. System Architecture (စနစ်၏ ဗိသုကာ ပုံစံ)

ဤ Project ကို **DAO (Data Access Object) Pattern** နှင့် **MVC (Model-View-Controller)** သဘောတရားအတိုင်း အလွှာ (Layer) များ ခွဲခြား၍ ရေးသားထားသည်-

1. **Model Layer (`Student.java`)**: ကျောင်းသားတစ်ဦး၏ အချက်အလက်များကို သိမ်းဆည်းပေးသည့် POJO Class ဖြစ်သည်။
    
2. **Database & DAO Layer (`DBConnection.java`, `StudentDAO.java`, `StudentDAOInterface.java`)**: MySQL Database နှင့် တိုက်ရိုက် ချိတ်ဆက်၍ CRUD (Create, Read, Update, Delete) အလုပ်များကို ဆောင်ရွက်သည်။
    
3. **Utility & Exception Layer (`QRUtils.java`, `DataAccessException.java`, `ValidationException.java`)**: QR Code ထုတ်ယူခြင်း/ပုံဖော်ခြင်းနှင့် SQL Error များကို UI သို့ တိုက်ရိုက် မရောက်အောင် ထိန်းချုပ်ပေးသည်။
    
4. **UI Layer (`Home.java`, `QRGenerator.java`, `QRScanner.java`)**: သုံးစွဲသူ မူလစာမျက်နှာ၊ ကျောင်းသား စာရင်းသွင်းခြင်း/QR ထုတ်ပေးခြင်း Panel နှင့် Webcam ဖြင့် Scan ဖတ်သည့် Panel တို့ ဖြစ်ကြသည်။
    

## 2. Database Structure (ဒေတာဘေ့စ် ဖွဲ့စည်းပုံ)

MySQL တွင် `student_db` ဟူသော Database အောက်၌ `students` Table ဖြင့် သိမ်းဆည်းသည်-

- **`id`**: Auto Increment Primary Key
    
- **`student_id`**: ကျောင်းသား ကဒ်နံပါတ် (Unique)
    
- **`name`, `sex`, `department`, `email`, `dob`**: ကျောင်းသား ကိုယ်ရေးအချက်အလက်များ
    
- **`qr_token`**: QR Code အတု/အဟောင်း ရိုက်နှိပ်အသုံးပြုမှုကို တားဆီးရန် အသုံးပြုသော **UUID Security Token**
    
- **`created_at`, `updated_at`**: ထည့်သွင်း/ပြင်ဆင်သည့် အချိန် မှတ်တမ်းများ
    

## 3. အဓိက အလုပ်လုပ်ပုံ Flow (Core Workflow)

### (က) ကျောင်းသားသစ် မှတ်ပုံတင်ခြင်း (Student Registration)

1. User သည် `QRGenerator` UI ရှိ Form တွင် `Student ID`၊ `Name`၊ `DOB`၊ `Sex`၊ `Department`၊ `Email` စသည်တို့ကို ဖြည့်စွက်ပြီး **Register** ခလုတ်ကို နှိပ်သည်။
    
2. `StudentDAO.saveStudent()` မှတစ်ဆင့် MySQL ၏ `students` Table ထဲသို့ `PreparedStatement` ကို အသုံးပြု၍ လုံခြုံစွာ ထည့်သွင်းပေးသည်။
    
3. မကြာသေးမီက ထည့်သွင်းလိုက်သော ကျောင်းသား စာရင်းကို `JTable` တွင် တန်းပြပေးသည်။
    

### (ခ) Security Token ဖြင့် QR Code ထုတ်ယူခြင်း (QR Code Generation)

1. User က `JTable` ထဲမှ ကျောင်းသားတစ်ဦးကို ရွေးချယ်ပြီး **Generate QR Code** ခလုတ်ကို နှိပ်လိုက်သောအခါ-
    
    - စနစ်က **UUID (Universally Unique Identifier)** ဖြင့် မထပ်နိုင်သော **Token အသစ်တစ်လုံး** (`UUID.randomUUID().toString()`) ကို ထုတ်ယူလိုက်သည်။
        
    - ထို Token ကို MySQL DB ရှိ ကျောင်းသား၏ `qr_token` Column တွင် Update သွားလုပ်သည်။
        
2. **QR Payload စာသား တည်ဆောက်ခြင်း**:
    
    ```
    StudentID: STU-1001
    Name: Aung Aung
    Age: 20
    Sex: Male
    Dept: CS
    Email: aung@gmail.com
    Token: e4d9b23a-8f12-4c21-9e12-34abc56789de
    ```
    
3. **QR Code Image ထုတ်လုပ်ခြင်း**:
    
    - **`QRUtils.generateQRCode()`**: UI ပေါ်တွင် ပြသရန် 230x230 pixel ပုံရိပ် ထုတ်ပေးသည်။
        
    - **`QRUtils.saveQRCodeToFile()`**: Print ထုတ်ရန် သို့မဟုတ် သိမ်းဆည်းရန်အတွက် 350x350 pixel high-resolution PNG ပုံရိပ်အဖြစ် `qrcodes/STU-1001.png` သို့ သိမ်းဆည်းသည်။
        
4. **Regenerate Logic (QR အသစ် ပြန်ထုတ်ခြင်း)**:
    
    - အကယ်၍ QR Code ကို ဒုတိယအကြိမ် ပြန်လည် ထုတ်ယူပါက Confirm Dialog ပြသည်။ OK ပေးပါက Token အသစ် ပြန်လည် ထုတ်ပေးမည်ဖြစ်ရာ **ယခင် ထုတ်ထားဖူးသော QR Code အဟောင်းသည် အလိုအလျောက် ပယ်ဖျက် (Invalid) ဖြစ်သွားမည် ဖြစ်သည်။**
        

### (ဂ) Webcam ဖြင့် Live Scan ဖတ်ပြီး Identity စစ်ဆေးခြင်း (QR Code Scanning)

1. `QRScanner` UI ပွင့်လာပါက **Sarxos Webcam Library** ကို အသုံးပြု၍ Background Thread (`ExecutorService`) ထဲတွင် Webcam ကို စတင် ဖွင့်လှစ်သည်။
    
2. **Frame Parsing Loop (~25 FPS)**:
    
    - Webcam မှ ရရှိလာသော ပုံရိပ်တိုင်းကို UI ရှိ `lblCam` ပေါ်တွင် Display တိုက်ရိုက် ပြသပေးသည်။
        
    - Image ကို **ZXing MultiFormatReader** သို့ ပေးပို့ decode လုပ်သည်။
        
    - Scan ဖတ်သည့် နေရာတွင် ကြည်လင်ပြတ်သားမှု ကောင်းမွန်စေရန် **HybridBinarizer** နှင့် အလင်းပြန်မှု လျှော့ချပေးသော **GlobalHistogramBinarizer** နည်းလမ်း (၂) မျိုးလုံးဖြင့် Dual-Binarization စစ်ဆေးသည်။
        
3. **Data Extract & Database Verification**:
    
    - Scan ဖတ်မိသော Payload စာသားမှ `StudentID:` နှင့် `Token:` ကို ခွဲထုတ်ယူသည်။
        
    - `StudentDAO.getStudentByStudentId(studentId)` ဖြင့် DB ထဲမှ ထိုကျောင်းသား၏ Record ကို ဆွဲထုတ်သည်။
        
4. **Security Check 結果များ**:
    
    - **`StudentID` သို့မဟုတ် `Token` မပါပါက/မမှန်ပါက**: _"Invalid Code: This is not a student ID QR code."_
        
    - **DB တွင် ကျောင်းသား မရှိပါက**: _"STUDENT NOT FOUND IN DATABASE!"_
        
    - **Scanned Token == DB Token**: **Access Granted!** (ကျောင်းသား အချက်အလက်များ တက်လာမည်)
        
    - **Scanned Token != DB Token**: **Access Denied!** (_"EXPIRED OR REGENERATED QR CODE!"_ - QR အသစ် ပြန်ထုတ်ထားသောကြောင့် QR အဟောင်း မိနေခြင်း ဖြစ်သည်)
        

## 4. Key Security & Architecture Features (အရေးပါသော နည်းပညာအချက်များ)

|Feature|နည်းပညာ/ပါဝင်သည့် Class|အကျိုးကျေးဇူး|
|---|---|---|
|**SQL Injection Protection**|`PreparedStatement` (`StudentDAO.java`)|SQL Injection Attack များမှ ကာကွယ်ပေးထားခြင်း။|
|**Anti-Replay / Anti-Copy QR**|`UUID QrToken` (`Student.java`, `QRScanner.java`)|QR Code မိတ္တူကူးထားခြင်း သို့မဟုတ် QR ဟောင်းသုံးခြင်းကို တားဆီးပေးခြင်း။|
|**Decoupled Architecture**|Interface + DAO Pattern|UI Code နှင့် Database Logic သီးခြားစီ ခွဲထားသဖြင့် ပြင်ဆင်ရ လွယ်ကူခြင်း။|
|**Custom Exception Handling**|`DataAccessException`|SQL Error များကို User ဆီ တိုက်ရိုက် မရောက်စေဘဲ Friendly Error Message အဖြစ် ပြောင်းပေးခြင်း။|
|**High Performance Scanning**|Background Thread Executor|UI Freeze မဖြစ်ဘဲ Webcam Loop ကို သီးသန့် Thread ဖြင့် အကောင်အထည်ဖော်ထားခြင်း။|

## ၅။ စနစ် အသုံးပြုမှု အစဉ်လိုက် (Summary Chart)

```
[ Register Student ] ──► [ DB Saves Record ] ──► [ Generate QR Code ] ──► [ Token Generated & Saved to DB ]
                                                                                   │
                                                                                   ▼
[ Access Granted ] ◄── [ Match Token with DB ] ◄── [ Scan via Webcam ] ◄── [ Save/Print QR PNG ]
```