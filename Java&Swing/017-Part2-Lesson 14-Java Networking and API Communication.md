Noted. From now on, I will explain using **modern Java 25 practices**.

Important changes for Java 25 learning:

- Prefer **modern Java APIs** (Java 17–25 style)
    
- Avoid outdated patterns when possible
    
- Use:
    
    - `var` where readability improves
        
    - Records
        
    - Sealed classes
        
    - Pattern matching
        
    - Modern switch expressions
        
    - Virtual Threads (Java 21+)
        
    - Modern `java.time`
        
    - Modern HTTP Client
        
    - Structured concurrency concepts (where applicable)
        
- For Swing + Café POS, we will still use classic OOP/MVC because Swing applications usually require that architecture.
    

Now continue.

---

# Part 2: Advanced Java Knowledge for Swing

# Lesson 14: Java Networking & API Communication

## Building Connected Café POS Applications with Java 25

---

# 1. Why Networking is Important in POS Systems?

A real Café POS system is usually not isolated.

Example:

```
                 Café POS Desktop App

                         |
                         |
                    Internet / LAN

                         |
        ---------------------------------
        |               |               |

   Payment API     Cloud Database    Inventory Server

```

Features:

- Online payment
    
- Cloud backup
    
- Multiple branch synchronization
    
- Customer loyalty system
    
- Remote reporting
    

---

# 2. What is Networking?

Networking means:

> Two or more computers communicate and exchange data.

Basic model:

```
Client

  |
  |
Request

  |
  v

Server

  |
  |
Response

```

Example:

```
POS Client

GET Product List

        |

Server

        |

Return Products

```

---

# 3. Java Networking Packages

Main packages:

```java
java.net
```

and:

```java
java.net.http
```

(Java 11+ modern API)

---

Old:

```
Socket
ServerSocket
URLConnection
```

Modern:

```
HttpClient
HttpRequest
HttpResponse
```

---

# 4. Client-Server Architecture

Café POS example:

```
             Client

       Swing POS Application

                |
                |
             HTTP Request

                |
                v

             Server

        Spring Boot Backend

                |
                |
             Database

                |
                v

              MySQL

```

---

# 5. Socket Programming Concept

Socket:

> Two computers communicate through a connection endpoint.

Example:

```
Computer A

Socket
   |
   |
Network
   |
   |
Socket

Computer B

```

---

# 6. ServerSocket

Server creates a listening port.

Example:

```java
ServerSocket server =
new ServerSocket(5000);

```

Meaning:

```
Waiting at port 5000

```

---

Accept Client:

```java
Socket client =
server.accept();

```

---

Flow:

```
Server

listen()

   |
   |
accept()

   |
   |
Client connected

```

---

# 7. Client Socket

Client:

```java
Socket socket =
new Socket(
"localhost",
5000
);

```

Meaning:

```
Connect:

IP = localhost

Port = 5000

```

---

# 8. Sending Data

OutputStream:

```java
OutputStream out =
socket.getOutputStream();


out.write(
"Order Created"
.getBytes()
);

```

---

Receiving:

```java
InputStream in =
socket.getInputStream();

```

---

# 9. Socket Example

Server:

```java
public class Server {


public static void main(String[] args)
throws Exception{


ServerSocket server =
new ServerSocket(5000);


System.out.println(
"Server Started"
);


Socket socket =
server.accept();


System.out.println(
"Client Connected"
);


}


}

```

---

Client:

```java
public class Client {


public static void main(String[] args)
throws Exception{


Socket socket =
new Socket(
"localhost",
5000
);


System.out.println(
"Connected"
);


}

}

```

---

# 10. Problems with Raw Socket

Manual socket programming:

- Hard to maintain
    
- Need protocol design
    
- Security issues
    
- No standard format
    

Modern applications usually use:

```
HTTP + REST API

```

---

# 11. What is REST API?

REST:

Representational State Transfer

Simple meaning:

> Client and Server communicate using HTTP requests.

Example:

POS wants product:

Request:

```
GET /products/1001

```

Response:

```json
{
"id":1001,
"name":"Coffee",
"price":5000
}

```

---

# 12. HTTP Methods

## GET

Read data:

```
GET /products

```

---

## POST

Create data:

```
POST /orders

```

Example:

```json
{
"product":"Coffee",
"qty":2
}

```

---

## PUT

Update:

```
PUT /products/1001

```

---

## DELETE

Remove:

```
DELETE /products/1001

```

---

# 13. Java 25 HTTP Client

Modern Java uses:

```java
java.net.http.HttpClient
```

---

Create Client:

```java
HttpClient client =
HttpClient.newHttpClient();

```

---

# 14. Sending GET Request

Example:

```java
HttpRequest request =
HttpRequest.newBuilder()

.uri(
URI.create(
"https://api.example.com/products"
)
)

.GET()

.build();


HttpResponse<String> response =
client.send(
request,
HttpResponse.BodyHandlers.ofString()
);


System.out.println(
response.body()
);

```

---

Flow:

```
Java POS

    |
HTTP GET

    |
Server

    |
JSON Response

```

---

# 15. Sending POST Request

Example:

```java
String json =
"""
{
"name":"Coffee",
"price":5000
}
""";


HttpRequest request =
HttpRequest.newBuilder()

.uri(
URI.create(
"https://api.example.com/products"
)
)

.header(
"Content-Type",
"application/json"
)

.POST(
HttpRequest.BodyPublishers.ofString(json)
)

.build();

```

---

# 16. JSON Communication

Real applications don't send Java Objects directly.

They use JSON.

Java Object:

```java
class Product {


int id;

String name;


}

```

JSON:

```json
{
"id":1,
"name":"Coffee"
}

```

---

# 17. JSON Libraries

Popular:

## Jackson

Used by Spring.

Example:

```java
ObjectMapper mapper =
new ObjectMapper();


String json =
mapper.writeValueAsString(product);

```

---

Object from JSON:

```java
Product product =
mapper.readValue(
json,
Product.class
);

```

---

# 18. Café POS API Example

Product API:

Request:

```
GET

/api/products

```

Response:

```json
[
{
"id":1,
"name":"Coffee",
"price":5000
},
{
"id":2,
"name":"Cake",
"price":7000
}
]

```

---

Swing Flow:

```
Button Click

      |

Service

      |

HTTP Client

      |

REST API

      |

JSON

      |

Product List

      |

JTable Update

```

---

# 19. Asynchronous HTTP Request

Java 25 supports asynchronous style:

```java
client.sendAsync(
request,
HttpResponse.BodyHandlers.ofString()
)

.thenApply(
HttpResponse::body
)

.thenAccept(
System.out::println
);

```

---

Benefit:

UI မပိတ်ပါ။

---

# 20. Swing + HTTP Best Architecture

Wrong:

```
Button

 |
 |
HTTP Request

 |
 |
Wait 10 seconds

 |
 |
Update UI

```

UI Freeze.

---

Correct:

```
Swing EDT

 |
 |
Service Layer

 |
 |
HttpClient Async

 |
 |
Response

 |
 |
SwingUtilities.invokeLater()

 |
 |
Update JTable

```

---

# 21. Error Handling

Network problems:

- No Internet
    
- Server down
    
- Timeout
    
- Invalid response
    

Example:

```java
try {


response =
client.send(
request,
handler
);


}
catch(IOException e){


throw new NetworkException(
"Server unavailable"
);


}

```

---

Custom Exception:

```java
public class NetworkException
extends RuntimeException {


public NetworkException(
String message
){

super(message);

}

}

```

---

# 22. Timeout Handling

Example:

```java
HttpClient client =
HttpClient.newBuilder()

.connectTimeout(
Duration.ofSeconds(5)
)

.build();

```

---

Meaning:

```
If server doesn't respond

within 5 seconds

throw error

```

---

# 23. Authentication Concept

Real APIs need security.

Example:

Header:

```
Authorization:
Bearer TOKEN

```

Java:

```java
.header(
"Authorization",
"Bearer abc123"
)

```

---

# 24. Café POS Cloud Architecture

Professional:

```
                 Swing POS

                    |

              Java HttpClient

                    |

              REST API Server

                    |

          --------------------

          |                  |

       MySQL              Redis


```

---

# 25. Local Network POS

Restaurant with multiple terminals:

```
Cashier PC

     |

 LAN

     |

Server PC

     |

MySQL

```

---

# 26. Java 25 Virtual Threads (Introduction)

Java 21+ feature.

Traditional:

```
1000 Tasks

1000 Platform Threads

```

Problem:

Memory.

Virtual Threads:

```
1000 Tasks

Lightweight Threads

```

---

Example:

```java
Thread.startVirtualThread(
() -> {

processOrder();

}
);

```

---

Useful for:

- HTTP requests
    
- Database calls
    
- I/O operations
    

---

# 27. Virtual Thread + POS Example

Many branches:

```
Branch 1
Branch 2
Branch 3
...
Branch 1000


        |

Virtual Threads


        |

Sync Orders

```

---

# 28. Networking Best Practices

## Use:

✅ HttpClient

✅ JSON

✅ REST API

✅ Async calls

✅ Timeout

✅ Exception handling

Avoid:

❌ Raw socket for business apps

❌ Blocking UI thread

❌ Hard-coded URLs

---

# 29. Interview Questions

## Q1: Difference between Socket and HTTP?

Socket:

- Low-level communication
    

HTTP:

- Application protocol built on TCP
    

---

## Q2: Why use HttpClient?

Modern:

- Async support
    
- HTTP/2
    
- Better API
    

---

## Q3: Why JSON?

Because:

- Language independent
    
- Human readable
    
- Easy transfer
    

---

## Q4: Why async request in Swing?

To prevent UI freezing.

---

# Practice Project

Create Café POS Remote Product Service:

Build:

```
ProductClient

OrderClient

PaymentClient

NetworkException

ApiResponse<T>

```

Features:

1. Fetch products from API
    
2. Create order through API
    
3. Handle timeout
    
4. Update Swing JTable asynchronously
    

Use:

- Java 25 HttpClient
    
- JSON
    
- CompletableFuture
    
- Virtual Threads
    

---

# Lesson 14 Summary

Today we learned:

✅ Client-Server Architecture  
✅ Socket Concept  
✅ REST API  
✅ HTTP Methods  
✅ Java HttpClient  
✅ GET Request  
✅ POST Request  
✅ JSON Communication  
✅ Async HTTP  
✅ Network Exception Handling  
✅ Swing + API Architecture  
✅ Virtual Threads Introduction

---

# Next Lesson

# Lesson 15: Modern Java 25 Features Deep Dive

## Writing Professional Java 25 Code

Topics:

- var
    
- Records
    
- Sealed Classes
    
- Pattern Matching
    
- Switch Expressions
    
- Text Blocks
    
- Virtual Threads
    
- Modern Collections
    
- Java 25 Best Practices
    

ပြီးရင် Java 25 Style နဲ့ Café POS Code ကို Modernize လုပ်ပါမယ်။