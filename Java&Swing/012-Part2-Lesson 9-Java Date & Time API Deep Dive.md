# Part 2: Advanced Java Knowledge for Swing

# Lesson 9: Java Date & Time API Deep Dive

## Professional Date/Time Handling for Café POS System

ဒီ Lesson မှာ Java 8 မှာ အသစ်ပါလာတဲ့ **java.time API** ကို လေ့လာပါမယ်။

Café POS System မှာ Date/Time က အလွန်အရေးကြီးပါတယ်။

အသုံးများတဲ့နေရာတွေ:

- Sale Transaction Time
    
- Receipt Date
    
- Daily Sales Report
    
- Monthly Report
    
- Employee Attendance
    
- Product Expiry Date
    
- Database Timestamp
    

---

# 1. Problem with Old Date API

Java အဟောင်းမှာ:

```java
java.util.Date
```

ကို သုံးပါတယ်။

Example:

```java
Date date = new Date();
```

Problem:

- Mutable ဖြစ်တယ်
    
- Thread-safe မဟုတ်
    
- API မလွယ်
    
- Formatting ခက်
    

---

Java 8 မှာ:

```java
java.time
```

API အသစ်ရလာပါတယ်။

---

# 2. Java Time API Architecture

Main Classes:

```id="t5j8ra"
java.time

 |
 |
 ------------------------------
 |          |          |
LocalDate LocalTime LocalDateTime

```

---

## LocalDate

Date only:

```id="6g2h5n"
2026-07-31
```

---

## LocalTime

Time only:

```id="2q1f5k"
10:30:45
```

---

## LocalDateTime

Date + Time:

```id="7j4m6x"
2026-07-31T10:30:45

```

---

# 3. LocalDate

Import:

```java
import java.time.LocalDate;
```

---

Create Current Date:

```java
LocalDate today =
LocalDate.now();

System.out.println(today);
```

Output:

```id="7r9p3s"
2026-07-31
```

---

# 4. Creating Specific Date

Example:

Café opening date:

```java
LocalDate opening =
LocalDate.of(
2025,
1,
1
);

```

Output:

```id="4x8c9q"
2025-01-01
```

---

# 5. Getting Date Information

Example:

```java
LocalDate today =
LocalDate.now();


int year =
today.getYear();


int month =
today.getMonthValue();


int day =
today.getDayOfMonth();

```

Output:

```
Year: 2026
Month: 7
Day: 31
```

---

# 6. Date Calculation

## Add Days

Example:

Expiry date:

```java
LocalDate expiry =
LocalDate.now()
.plusDays(30);

```

Meaning:

Today + 30 days

---

## Add Months

```java
LocalDate reportDate =
today.plusMonths(1);

```

---

## Subtract

```java
today.minusDays(7);

```

---

# 7. Date Comparison

Important for Reports.

Example:

```java
LocalDate saleDate =
LocalDate.of(
2026,
7,
1
);


LocalDate today =
LocalDate.now();


boolean result =
saleDate.isBefore(today);

```

---

Methods:

```java
isBefore()

isAfter()

isEqual()

```

---

# 8. Café POS Example: Daily Report

Requirement:

"Today sales only"

Sale:

```java
class Sale {


private LocalDate date;

private double amount;


}

```

---

Filter:

```java
sales.stream()
.filter(
s -> s.getDate()
.equals(
LocalDate.now()
)
)
.collect(
Collectors.toList()
);

```

---

# 9. LocalTime

Time only:

```java
import java.time.LocalTime;


LocalTime now =
LocalTime.now();

System.out.println(now);

```

Output:

```
10:45:30
```

---

# 10. Creating Time

```java
LocalTime opening =
LocalTime.of(
9,
0
);

```

Output:

```
09:00
```

---

# 11. Time Calculation

Add Hours:

```java
LocalTime close =
opening.plusHours(10);

```

Result:

```
19:00
```

---

Subtract:

```java
time.minusMinutes(30);

```

---

# 12. Comparing Time

Example:

Restaurant open?

```java
LocalTime now =
LocalTime.now();


LocalTime open =
LocalTime.of(9,0);


LocalTime close =
LocalTime.of(21,0);


if(now.isAfter(open)
&& now.isBefore(close)){


System.out.println(
"Open"
);


}

```

---

# 13. LocalDateTime

Most used in POS.

Why?

Transaction needs:

```
Date
+
Time
```

Example:

```
Sale ID: 1001

Date:
2026-07-31

Time:
10:45:30

```

---

Create:

```java
LocalDateTime transactionTime =
LocalDateTime.now();

```

Output:

```
2026-07-31T10:45:30
```

---

# 14. Creating LocalDateTime

```java
LocalDateTime dateTime =
LocalDateTime.of(
2026,
7,
31,
10,
30
);

```

Output:

```
2026-07-31T10:30
```

---

# 15. Formatting Date

Default:

```
2026-07-31
```

Need:

```
31/07/2026
```

Use:

```java
DateTimeFormatter
```

---

Example:

```java
DateTimeFormatter formatter =
DateTimeFormatter.ofPattern(
"dd/MM/yyyy"
);


String result =
today.format(formatter);

```

Output:

```
31/07/2026
```

---

# 16. Common Date Formats

|Pattern|Meaning|
|---|---|
|yyyy|Year|
|MM|Month|
|dd|Day|
|HH|Hour|
|mm|Minute|
|ss|Second|

---

Example:

```java
"yyyy-MM-dd HH:mm:ss"

```

Output:

```
2026-07-31 10:30:45
```

---

# 17. Parsing String to Date

User Input:

```
31/07/2026
```

Convert:

```java
DateTimeFormatter formatter =
DateTimeFormatter.ofPattern(
"dd/MM/yyyy"
);


LocalDate date =
LocalDate.parse(
"31/07/2026",
formatter
);

```

---

# 18. Café POS Transaction Model

Professional Model:

```java
public class Sale {


private int id;

private double total;


private LocalDateTime createdAt;


}

```

---

Create Sale:

```java
Sale sale =
new Sale();


sale.setCreatedAt(
LocalDateTime.now()
);

```

---

Database:

```
sales table

id
total
created_at

```

---

# 19. Database Timestamp Mapping

MySQL:

```sql
created_at TIMESTAMP
```

Java:

```java
LocalDateTime

```

---

JDBC:

```java
preparedStatement.setObject(
1,
LocalDateTime.now()
);

```

---

Read:

```java
LocalDateTime time =
resultSet.getObject(
"created_at",
LocalDateTime.class
);

```

---

# 20. Sales Report Example

Requirement:

"Last 7 days sales"

Today:

```java
LocalDate today =
LocalDate.now();

```

Start:

```java
LocalDate start =
today.minusDays(7);

```

Filter:

```java
sales.stream()
.filter(
s ->
s.getDate()
.isAfter(start)
)
.collect(
Collectors.toList()
);

```

---

# 21. Duration

Calculate time difference.

Example:

Order preparation time:

```java
LocalTime start =
LocalTime.of(10,0);


LocalTime end =
LocalTime.of(10,15);


Duration duration =
Duration.between(
start,
end
);

```

Result:

```
15 minutes
```

---

# 22. Period

Date difference:

Example:

Employee worked:

```
2026-01-01
to
2026-07-31
```

Use:

```java
Period period =
Period.between(
startDate,
endDate
);

```

Result:

```
6 months
```

---

# 23. Instant

For machine timestamp:

```java
Instant now =
Instant.now();

```

Used for:

- Logging
    
- Distributed systems
    
- Server timestamps
    

---

# 24. Time Zone

LocalDateTime:

```
2026-07-31 10:00
```

Problem:

Different countries.

Use:

```java
ZonedDateTime

```

Example:

```java
ZonedDateTime now =
ZonedDateTime.now(
ZoneId.of(
"Asia/Yangon"
)
);

```

---

# 25. Café POS Date Architecture

Complete flow:

```
Customer Order

      |
      v

Sale Transaction

      |
      v

LocalDateTime.now()

      |
      v

MySQL TIMESTAMP

      |
      v

Daily Report

      |
      v

Stream API

```

---

# 26. Best Practices

## Rule 1

Avoid:

```java
Date
Calendar

```

New projects use:

```java
LocalDate
LocalDateTime

```

---

## Rule 2

Store timestamps consistently.

Database:

```
UTC
```

Display:

```
Local timezone
```

---

## Rule 3

Use formatter at UI boundary only.

Good:

```
Database
 |
LocalDateTime
 |
Swing UI
 |
String Format

```

---

# 27. Interview Questions

## Q1: Difference between LocalDate and LocalDateTime?

LocalDate:

```
Date only
```

LocalDateTime:

```
Date + Time
```

---

## Q2: Why Java 8 Date API?

Because old Date API was:

- Mutable
    
- Hard to use
    
- Not thread safe
    

---

## Q3: How to compare dates?

Use:

```java
isBefore()

isAfter()

isEqual()

```

---

## Q4: How to format date?

Use:

```java
DateTimeFormatter

```

---

# Practice Project

Upgrade Café POS Sale System:

Create:

```java
Sale

id
customerName
totalAmount
createdAt
```

Use:

```java
LocalDateTime

DateTimeFormatter

Stream API

```

Implement:

```
getTodaySales()

getWeeklySales()

getMonthlySales()

calculateTotalRevenue()

```

---

# Lesson 9 Summary

ဒီနေ့သင်ယူခဲ့တာ:

✅ Java Time API  
✅ LocalDate  
✅ LocalTime  
✅ LocalDateTime  
✅ Date Formatting  
✅ Date Parsing  
✅ Date Comparison  
✅ Duration  
✅ Period  
✅ Database Timestamp Mapping  
✅ POS Report Examples

---

# Next Lesson

# Lesson 10: Java Generics Deep Dive

## Writing Reusable Professional Code

သင်ယူမည့်အရာ:

- Why Generics?
    
- Generic Class
    
- Generic Method
    
- Type Safety
    
- Wildcards (`?`)
    
- `extends`
    
- `super`
    
- Generic Repository Pattern
    
- Café POS DAO Design
    

Example:

```java
Repository<Product>

Repository<Customer>

Repository<Order>

```

ပြီးရင် Java Backend Architecture ပိုင်းကို ဆက်သွားပါမယ်။