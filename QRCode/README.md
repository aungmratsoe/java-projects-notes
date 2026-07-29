# 🎓 Student Identity & Encrypted QR Verification System

An end-to-end Desktop Application built with **Java Swing**, **MySQL**, **ZXing**, and **Sarxos Webcam API**. The system manages student records, generates AES-128 encrypted QR codes with dynamic UUID anti-replay tokens, and provides real-time identity verification via live webcam scanning.

## 📌 GitHub Repository Details

- **Repository Name:** `student-qr-verification-system`
    
- **Short Description:**
    
    > A Java Swing & MySQL desktop application for student management, AES-128 encrypted dynamic QR code generation with UUID security tokens, and live webcam verification using ZXing and Sarxos API.
    
- **Topics/Tags:** `java`, `java-swing`, `mysql`, `qr-code`, `zxing`, `webcam-capture`, `flatlaf`, `aes-encryption`, `security-system`, `student-management`
    

## ✨ Key Features

- 🔐 **AES-128 Encrypted QR Payloads:**
    
    - Student payload details are encrypted using **AES-128/ECB/PKCS5Padding**.
        
    - Scanning the QR code with standard mobile camera apps or third-party scanners reveals only unreadable Base64 ciphertext, preventing unauthorized data extraction.
        
- 🔑 **On-Demand UUID Security Tokens (Anti-Replay / Anti-Copy):**
    
    - Unique `UUID` security tokens are generated **strictly when a QR code is generated or regenerated**, rather than during initial student registration.
        
    - Regenerating a student's QR code updates their active token in the database, automatically invalidating any previously printed paper codes or screenshot copies.
        
- 📷 **Real-Time Webcam Scanning & Dual Binarization:**
    
    - Integrated live webcam feed running on a dedicated background thread (~25 FPS).
        
    - Uses dual-pass binarization (`HybridBinarizer` + `GlobalHistogramBinarizer`) via ZXing for maximum barcode recognition accuracy across varying lighting conditions.
        
- 👥 **Student Record Management & Smart Upsert:**
    
    - Full registration and profile management (Student ID, Name, Sex, DOB, Department, Email).
        
    - **Smart Update/Upsert:** Updating checks whether a Student ID exists in the database—if present, it updates the record; if absent, it inserts the student as a new entry.
        
    - Real-time search filtering in `JTable` powered by Swing `DocumentListener`.
        

## 🛠️ Technology Stack & Libraries

|Category|Technology / Library|Description|
|---|---|---|
|**Language**|Java 17+|Core Application Runtime|
|**GUI Framework**|Java Swing + FlatLaf (`com.formdev:flatlaf`)|Clean, modern Look & Feel for desktop applications|
|**Database**|MySQL 8.0+|Relational Database Storage|
|**Database Connector**|MySQL Connector/J (`com.mysql:mysql-connector-j`)|JDBC Driver|
|**QR Engine**|ZXing (`com.google.zxing:core`, `javase`)|QR Code Generation and Decoding|
|**Webcam Driver**|Sarxos Webcam Capture (`com.github.sarxos:webcam-capture`)|Live Camera Stream Integration|
|**Security**|Java Cryptography Extension (JCE)|AES-128 Payload Encryption & Decryption|
|**Calendar UI**|JCalendar (`com.toedter:jcalendar`)|Swing Date Picker|

## 📂 Architecture Overview

The system follows a clean **DAO (Data Access Object)** design pattern and a modular layered architecture:

```
src/
└── com/ams/qrcode/
    ├── dao/          # Database interfaces & implementation (StudentDAO)
    ├── exceptions/   # Custom exception hierarchy (DataAccessException)
    ├── model/        # Data models / POJOs (Student)
    ├── ui/           # Swing UI components (Home, QRGenerator, QRScanner)
    └── utils/        # Utilities (CryptoUtils, QRUtils, DBConnection)
    └── db/           # Db (DBConnection)
```

## 🚀 Getting Started

### Prerequisites

1. **JDK 17 or higher** installed and configured in your `PATH`.
    
2. **MySQL Server 8.0+** running locally or remotely.
    
3. A connected USB or built-in **Webcam**.
    

### 1. Database Setup

Execute the following SQL script in your MySQL Workbench or CLI:

```
CREATE DATABASE IF NOT EXISTS student_db;
USE student_db;

CREATE TABLE IF NOT EXISTS students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    student_id VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    sex VARCHAR(10),
    department VARCHAR(100),
    email VARCHAR(100),
    dob DATE,
    qr_token VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 2. Configure Database Credentials

Update `com/ams/qrcode/utils/DBConnection.java` with your database credentials:

```
private static final String URL = "jdbc:mysql://localhost:3306/student_db";
private static final String USER = "root";
private static final String PASSWORD = "your_mysql_password";
```

### 3. Build & Run

If using **Maven**:

```
# Clean and compile project
mvn clean package

# Run main application
mvn exec:java -Dexec.mainClass="com.ams.qrcode.ui.Home"
```

## 🔄 Verification & Security Workflow

```
[ Student Profile ] ──► [ Save to MySQL DB ] ──► [ Generate QR Code ]
                                                        │
                                                        ▼
                                         [ Generate UUID & Encrypt AES ]
                                                        │
                                                        ▼
[ Access Granted ] ◄── [ Decrypt & Verify Token ] ◄── [ Webcam Scan ]
```

1. **Webcam Capture & Decryption:** The scanner reads the QR image and decrypts the AES payload using internal system keys. If decryption fails (e.g., external QR code scanned), access is immediately rejected.
    
2. **Format Validation:** Extracts `StudentID:` and `Token:` parameters from the decrypted plaintext.
    
3. **Database Token Verification:** Checks `StudentID` against MySQL database records:
    
    - **Token Match:** Scanned token equals database token $\rightarrow$ **Access Granted**.
        
    - **Token Mismatch:** Scanned token differs from active database token $\rightarrow$ **Access Denied (Expired/Regenerated Code)**.
        

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE "null") file for details.