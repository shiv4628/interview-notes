# Table of Contents

- [Chat : spring boot batch processing](#spring-boot-batch-processing)
- [Chat : reactive programming in spring boot](#reactive-programming-in-spring-boot)
- [Chat : what is webflux](#what-is-webflux)
- [Chat : what are websockets?](#what-are-websockets)
- [Chat : Chat](#chat)
- [Chat : "ConcurrencyThread-per-request modelEvent loop (Reactive Streams)"](#concurrencythread-per-request-modelevent-loop-reactive-streams)
- [Chat : what is r2dbms?](#what-is-r2dbms)

## **Conversation**

<a id="spring-boot-batch-processing"></a>
### Chat :spring boot batch processing

Spring Boot Batch Processing is a powerful way to handle large volumes of data in a robust and efficient way using the **Spring Batch** framework. It’s ideal for tasks like reading data from a file or database, processing it, and writing the output to another destination (like a file, DB, or API).

Let me break it down for you step by step.

---

## 🔁 What is Spring Batch?

Spring Batch is a lightweight, robust batch processing framework. It provides:

* **Transaction management**
* **Chunk-based processing**
* **Declarative I/O**
* **Job scheduling**
* **Retry/skip support**

---

## ✅ Key Concepts

### 1. **Job**

* A batch job represents the entire process.
* Made up of one or more **Steps**.

### 2. **Step**

* A phase in the job.
* Each step can read-process-write data.
* Follows a **chunk-oriented** processing model.

### 3. **ItemReader**

* Reads input data (e.g., from a CSV, DB, etc.)

### 4. **ItemProcessor**

* Contains business logic for processing data.

### 5. **ItemWriter**

* Writes the processed data (e.g., to DB or file).

---

## 🛠️ Basic Spring Boot Batch Setup

### Step 1: Add Dependency (in `pom.xml` for Maven)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

### Step 2: Configure a Job

```java
@Configuration
@EnableBatchProcessing
public class BatchConfig {

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job sampleJob() {
        return jobBuilderFactory.get("sampleJob")
            .start(sampleStep())
            .build();
    }

    @Bean
    public Step sampleStep() {
        return stepBuilderFactory.get("sampleStep")
            .<String, String>chunk(10)
            .reader(itemReader())
            .processor(itemProcessor())
            .writer(itemWriter())
            .build();
    }

    @Bean
    public ItemReader<String> itemReader() {
        return new ListItemReader<>(List.of("One", "Two", "Three"));
    }

    @Bean
    public ItemProcessor<String, String> itemProcessor() {
        return item -> item.toUpperCase();
    }

    @Bean
    public ItemWriter<String> itemWriter() {
        return items -> items.forEach(System.out::println);
    }
}
```

This simple job reads a list of strings, processes them to uppercase, and prints them out.

---

## ⚙️ Job Launcher

Spring Boot auto-runs the job on application startup. To launch jobs programmatically:

```java
@Autowired
private JobLauncher jobLauncher;

@Autowired
private Job sampleJob;

public void runJob() throws Exception {
    JobParameters params = new JobParametersBuilder()
        .addLong("time", System.currentTimeMillis())
        .toJobParameters();
    jobLauncher.run(sampleJob, params);
}
```

---

## 💡 Real-World Use Cases

* Import data from CSV to database
* Generate reports
* Migrate database records
* Perform scheduled data cleanups

---

Would you like a **real working example** like:

* Reading from a CSV file and writing to a database?
* Reading from a database and writing to another table or file?

Let me know your use case, and I’ll tailor the example for you.


## **Conversation**

<a id="reactive-programming-in-spring-boot"></a>
### Chat : reactive programming in spring boot

Reactive programming in Spring Boot is primarily supported by **Project Reactor**, which provides a reactive streams implementation using **Mono** and **Flux**. This enables building non-blocking, event-driven applications that are highly scalable and efficient.

---

## **Key Concepts**

### 1. **Reactive Streams**

* A standard for handling asynchronous data streams with **backpressure**.
* Components:

  * **Publisher** (Emits data)
  * **Subscriber** (Consumes data)
  * **Subscription** (Manages demand)
  * **Processor** (Both Publisher & Subscriber)

### 2. **Mono & Flux**

* `Mono<T>` → Emits **0 or 1** item.
* `Flux<T>` → Emits **0 to N** items.

---

## **Setting Up Reactive Spring Boot**

1. **Add Dependencies**

   ```xml
   <dependencies>
       <!-- Spring WebFlux -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-webflux</artifactId>
       </dependency>

       <!-- Reactive MongoDB (Optional) -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
       </dependency>
   </dependencies>
   ```

---

## **Creating a Reactive REST API**

### **1. Controller using WebFlux**

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public Mono<User> getUserById(@PathVariable String id) {
        return userService.getUserById(id);
    }

    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.getAllUsers();
    }
}
```

---

### **2. Reactive Service Layer**

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public Mono<User> getUserById(String id) {
        return userRepository.findById(id);
    }

    public Flux<User> getAllUsers() {
        return userRepository.findAll();
    }
}
```

---

### **3. Reactive Repository (Using MongoDB)**

```java
@Repository
public interface UserRepository extends ReactiveCrudRepository<User, String> {
}
```

---

## **Using WebClient for Reactive API Calls**

`WebClient` is the non-blocking alternative to `RestTemplate`.

### **Example: Calling an External API**

```java
@Service
public class ExternalApiService {

    private final WebClient webClient;

    public ExternalApiService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("https://jsonplaceholder.typicode.com").build();
    }

    public Mono<User> fetchUser(String userId) {
        return webClient.get()
                .uri("/users/{id}", userId)
                .retrieve()
                .bodyToMono(User.class);
    }
}
```

---

## **Streaming Data with Server-Sent Events (SSE)**

```java
@GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamData() {
    return Flux.interval(Duration.ofSeconds(1))
               .map(seq -> "Event: " + seq);
}
```

---

## **Benefits of Reactive Programming**

* **Better Performance & Scalability:** Handles a large number of concurrent users efficiently.
* **Non-Blocking I/O:** Doesn't block threads while waiting for data.
* **Backpressure Support:** Prevents overwhelming the system with requests.

---

## **When to Use Reactive Programming?**

✅ **Best for:**

* High-concurrency applications.
* Streaming data, WebSockets, or SSE.
* Microservices communicating via non-blocking APIs.

❌ **Avoid if:**

* Your app is mostly synchronous.
* You're using blocking libraries (e.g., JPA, JDBC).

---

Would you like a specific example, like integrating reactive programming with databases or messaging queues (Kafka, RabbitMQ)? 🚀


<a id="what-is-webflux"></a>
### Chat : what is webflux

### **Spring WebFlux - Reactive Web Framework**

**Spring WebFlux** is a reactive, non-blocking web framework introduced in **Spring 5** as an alternative to Spring MVC. It is designed for handling **asynchronous** and **event-driven** applications, making it ideal for applications that require high concurrency, such as real-time data streaming and microservices.

---

## **Key Features of WebFlux**

* **Non-blocking & Asynchronous**
  Uses **Project Reactor** (`Flux` & `Mono`) to handle requests reactively.
* **Backpressure Handling**
  Prevents overwhelming consumers by managing data flow efficiently.
* **Functional & Annotation-based Programming**
  Supports both traditional `@Controller` and functional-style routing.
* **Works with Reactive Data Stores**
  Compatible with **Reactive MongoDB, Redis, and R2DBC (Reactive SQL)**.
* **WebClient instead of RestTemplate**
  WebFlux provides **WebClient** for making reactive API calls.

---

## **How is WebFlux Different from Spring MVC?**

| Feature     | Spring MVC (Blocking)     | Spring WebFlux (Non-blocking)    |
| ----------- | ------------------------- | -------------------------------- |
| Concurrency | Thread-per-request model  | Event loop (Reactive Streams)    |
| Performance | Good for low concurrency  | High scalability for async tasks |
| Database    | Works with JDBC, JPA      | Works with R2DBC, MongoDB, Redis |
| API Calls   | Uses `RestTemplate`       | Uses `WebClient`                 |
| Streaming   | Uses traditional servlets | Supports SSE & WebSockets        |

---

## **Setting Up WebFlux in Spring Boot**

### **1. Add Dependency**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

---

## **Building a Reactive API with WebFlux**

### **2. Reactive Controller using Mono & Flux**

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public Mono<User> getUserById(@PathVariable String id) {
        return userService.getUserById(id);
    }

    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.getAllUsers();
    }
}
```

---

### **3. Reactive WebClient for API Calls**

```java
@Service
public class ExternalService {

    private final WebClient webClient;

    public ExternalService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("https://jsonplaceholder.typicode.com").build();
    }

    public Mono<String> fetchUserData(String userId) {
        return webClient.get()
                .uri("/users/{id}", userId)
                .retrieve()
                .bodyToMono(String.class);
    }
}
```

---

### **4. Streaming Data with Server-Sent Events (SSE)**

```java
@GetMapping(value = "/events", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamEvents() {
    return Flux.interval(Duration.ofSeconds(1))
               .map(seq -> "Event #" + seq);
}
```

---

## **When Should You Use WebFlux?**

✅ **Use WebFlux when:**

* You need a scalable, high-performance system.
* The application handles **real-time streaming** or **WebSockets**.
* You're working with **non-blocking databases** (MongoDB, Redis, R2DBC).
* The application makes many **concurrent API calls**.

❌ **Avoid WebFlux when:**

* Your app is **synchronous** and uses **JDBC (blocking SQL calls)**.
* You don't need high concurrency or real-time features.

---

## **Final Thoughts**

Spring WebFlux is perfect for **modern, scalable, and high-performance applications** where non-blocking I/O is beneficial. However, it requires a **different mindset** compared to traditional Spring MVC. If your use case fits, it can significantly improve performance and efficiency.

Would you like a deeper dive into **WebSockets, functional routing, or reactive databases** with WebFlux? 🚀


<a id="what-are-websockets"></a>
### Chat : what are websockets?

### **WebSockets - Real-Time Bi-Directional Communication**

**WebSockets** provide a **full-duplex** communication channel over a single **TCP connection**, allowing **real-time** data exchange between a client and a server. Unlike traditional HTTP, which follows a request-response model, WebSockets enable both **clients and servers to send and receive messages independently**.

---

## **Key Features of WebSockets**

✅ **Full-Duplex Communication** – Both client and server can send data anytime.
✅ **Persistent Connection** – No need to repeatedly open and close connections like in HTTP.
✅ **Low Latency** – Ideal for **real-time applications** (chat apps, stock prices, gaming).
✅ **Reduced Network Overhead** – Less overhead compared to polling or long polling.

---

## **How Are WebSockets Different from HTTP?**

| Feature       | HTTP                                 | WebSockets                                   |
| ------------- | ------------------------------------ | -------------------------------------------- |
| Communication | Request-Response                     | Bi-Directional                               |
| Connection    | Opens & Closes per request           | Persistent                                   |
| Latency       | Higher due to re-opening connections | Low latency                                  |
| Use Case      | Static content, APIs                 | Real-time apps (chat, notifications, gaming) |

---

## **How WebSockets Work**

1. **Client initiates a handshake** using an HTTP request.
2. **Server upgrades the connection** to WebSocket protocol (`ws://` or `wss://`).
3. **Bi-Directional Communication** starts between the client and server.
4. **Connection stays open** until explicitly closed.

---

## **WebSockets in Spring Boot (Using WebFlux)**

Spring Boot provides WebSocket support via **Spring WebFlux**.

### **1. Add Dependency**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

---

### **2. Create a WebSocket Handler**

```java
@Component
public class ChatWebSocketHandler implements WebSocketHandler {

    @Override
    public Mono<Void> handle(WebSocketSession session) {
        return session.receive()
                .map(WebSocketMessage::getPayloadAsText)
                .doOnNext(message -> System.out.println("Received: " + message))
                .map(session::textMessage)
                .flatMap(session::send);
    }
}
```

---

### **3. Configure WebSocket Endpoint**

```java
@Configuration
@EnableWebFlux
public class WebSocketConfig implements WebSocketConfigurer {

    private final ChatWebSocketHandler chatHandler;

    public WebSocketConfig(ChatWebSocketHandler chatHandler) {
        this.chatHandler = chatHandler;
    }

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(chatHandler, "/chat").setAllowedOrigins("*");
    }
}
```

---

### **4. Client Example (JavaScript)**

```javascript
const socket = new WebSocket("ws://localhost:8080/chat");

socket.onopen = () => {
    console.log("Connected to WebSocket");
    socket.send("Hello Server!");
};

socket.onmessage = (event) => {
    console.log("Received:", event.data);
};
```

---

## **When to Use WebSockets?**

✅ **Best for:**

* **Chat applications** (e.g., WhatsApp, Slack).
* **Live stock price updates**.
* **Real-time gaming** (multiplayer games).
* **Live notifications** (sports scores, weather alerts).
* **Collaborative tools** (Google Docs, online whiteboards).

❌ **Avoid if:**

* Your use case is simple request-response (REST API is enough).
* You don't need real-time updates.

---

## **Alternatives to WebSockets**

🔹 **Server-Sent Events (SSE)** – One-way updates from server to client (good for real-time dashboards).
🔹 **Long Polling** – HTTP-based real-time alternative but less efficient.

Would you like an example using **Spring MVC instead of WebFlux** or a **chat application demo**? 🚀


<a id="chat"></a>
### Chat : 
" **Best for:** * High-concurrency applications. * Streaming data, WebSockets, or SSE. * Microservices communicating via non-blocking APIs."

can you explain in little more detail?

### Ans : 
Sure! Let’s break it down in more detail.

---

### **1. High-Concurrency Applications**

A **high-concurrency application** is one that needs to handle **many simultaneous users or requests** efficiently. In traditional synchronous applications (like Spring MVC), each request is handled by a separate thread, and if too many users connect at once, the server may run out of threads, causing slowdowns or crashes.

With **Spring WebFlux (Reactive Programming)**:

* It uses a small **fixed number of threads** (Event Loop model).
* Instead of waiting for slow operations (like database queries or API calls), it **continues processing other requests**.
* This results in **better resource utilization and scalability**.

**Example Use Case:**

* A **social media platform** where thousands of users are posting, liking, and commenting at the same time.
* A **food delivery app** handling a large number of concurrent orders and tracking delivery updates.

---

### **2. Streaming Data, WebSockets, or Server-Sent Events (SSE)**

Some applications require **continuous real-time data updates** rather than one-time responses.

#### **WebSockets**

* Allows **two-way communication** between the client and server.
* **Example:** A **live chat app** like WhatsApp or Slack where both users send and receive messages instantly.

#### **Server-Sent Events (SSE)**

* Allows the server to **push updates** to the client **continuously**.
* **Example:** A **live sports score website** where the server sends updates whenever the score changes.

#### **Streaming Data (Flux in WebFlux)**

* Handles **large data sets** efficiently without loading everything into memory at once.
* **Example:** A **real-time stock market application** where stock prices update every second.

---

### **3. Microservices Communicating via Non-Blocking APIs**

**Microservices** are small, independent services that communicate with each other. Traditionally, they use **REST APIs**, but in high-traffic systems, blocking APIs can cause **performance bottlenecks**.

With **WebFlux and Reactive APIs**:

* Microservices **do not block threads** while waiting for responses.
* **WebClient** (instead of RestTemplate) enables **non-blocking communication** between services.
* Improves **response times** and **reduces memory usage**.

**Example Use Case:**

* A **banking system** where different microservices handle **account balances, transactions, and fraud detection**, and they all need to communicate quickly and efficiently.
* An **e-commerce website** where different services (inventory, payments, shipping) talk to each other without slowing down the system.

---

### **When Should You Use WebFlux?**

✅ **Ideal for:**

* **Live updates** (Chat apps, stock market, IoT devices).
* **High traffic applications** (Netflix, Uber, Facebook).
* **Microservices with multiple API calls** (Banking, FinTech, E-commerce).

❌ **Avoid if:**

* You are using **traditional blocking databases like JDBC**.
* Your application is simple and does not need **real-time updates or high concurrency**.

Would you like me to explain with a **real-world example** in more depth? 🚀


<a id="concurrencythread-per-request-modelevent-loop-reactive-streams"></a>
### Chat : "ConcurrencyThread-per-request modelEvent loop (Reactive Streams)"
can you explain these models?

### Ans : 
Sure! Let's go deeper into the two concurrency models:

1. **Thread-per-Request Model (Traditional Blocking Model - Used in Spring MVC)**
2. **Event Loop Model (Reactive Streams - Used in Spring WebFlux)**

---

## **1. Thread-Per-Request Model (Traditional Blocking Model - Spring MVC)**

### **How It Works**

* Each incoming request is assigned a **dedicated thread**.
* The thread waits (blocks) for **I/O operations** (e.g., database queries, API calls).
* Once the request is processed, the thread is **released** for the next request.

### **Example**

Imagine you have a Spring MVC application where users fetch **product details** from a database:

```java
@GetMapping("/product/{id}")
public Product getProduct(@PathVariable String id) {
    return productService.findById(id); // Blocking call (JDBC)
}
```

* The request is handled by a **thread from a thread pool**.
* If the database is slow, the thread **waits** (blocking) until it gets a response.
* Meanwhile, this thread **cannot process other requests**.

### **Limitations**

❌ **Scalability Issues:** If there are too many users, the server **runs out of threads**, leading to slow responses.
❌ **High Memory Usage:** Each thread consumes memory, increasing RAM usage.
❌ **Thread Pool Exhaustion:** If requests take time (e.g., slow database queries), the system slows down as **new requests must wait** for free threads.

### **Example Scenario**

* An **e-commerce site** (Amazon) on **Black Friday** has thousands of users fetching products.
* If all the threads are busy waiting for database responses, **new users have to wait**, causing slowdowns or even crashes.

---

## **2. Event Loop Model (Reactive Streams - Spring WebFlux)**

### **How It Works**

* Instead of creating **one thread per request**, it uses a **small number of threads** (Event Loop).
* A single thread can handle **multiple requests** by not waiting (non-blocking).
* When an operation (like a database query) is needed, the request is **asynchronously queued** and the thread **moves on** to handle other tasks.
* When the response is ready, the thread **resumes processing**.

### **Example (WebFlux)**

```java
@GetMapping("/product/{id}")
public Mono<Product> getProduct(@PathVariable String id) {
    return productService.findById(id); // Non-blocking call (R2DBC)
}
```

* The request is handled by **one thread from the Event Loop**.
* If a database query is needed, the thread **does not wait**. Instead, the operation runs asynchronously.
* When the database responds, the request processing **resumes**.

### **Benefits**

✅ **Handles more requests with fewer threads** → Improves **scalability**.
✅ **Non-blocking** → The system is **faster** even with **slow** I/O operations.
✅ **Better for real-time applications** → Chat apps, stock prices, notifications.

### **Example Scenario**

* A **chat application** like WhatsApp or Slack.
* Instead of creating **one thread per user**, a small **fixed** number of threads **manages all users asynchronously**.
* This makes it possible to handle **millions of users** with **minimal resources**.

---

## **Comparison Table**

| Feature               | **Thread-per-Request Model (Spring MVC)** | **Event Loop Model (Spring WebFlux)** |
| --------------------- | ----------------------------------------- | ------------------------------------- |
| Threads               | One per request                           | Shared, small number of threads       |
| Blocking/Non-blocking | Blocking (waits for response)             | Non-blocking (keeps processing)       |
| Performance           | Slow under high load                      | Highly scalable                       |
| Memory Usage          | High (more threads = more RAM)            | Low (fewer threads)                   |
| Ideal for             | Simple, small applications                | Real-time, high-load apps             |

---

## **When to Choose Which Model?**

✅ **Use Thread-per-Request (Spring MVC) when:**

* Your app has **low to moderate traffic**.
* You use **JDBC databases** (which are blocking).
* You don’t need real-time features.

✅ **Use Event Loop (Spring WebFlux) when:**

* You have **high traffic** and need **better scalability**.
* You are working with **non-blocking databases** (MongoDB, R2DBC).
* Your app requires **real-time data streaming**.

---

## **Final Thoughts**

* **Spring MVC (Thread-per-Request)** is simple but **struggles under heavy load**.
* **Spring WebFlux (Event Loop)** is **highly scalable** but requires a **different mindset**.
* **Use the right model based on your project’s needs**.

Would you like me to show a **real-world example** comparing both approaches? 🚀


<a id="what-is-r2dbms"></a>
### Chat : what is r2dbms?

I think you meant **R2DBC (Reactive Relational Database Connectivity)** instead of **R2DBMS**. Let me explain both concepts:

---

## **1. RDBMS (Relational Database Management System)**

An **RDBMS** is a database that organizes data into **tables** with rows and columns. It follows **ACID (Atomicity, Consistency, Isolation, Durability)** principles and uses **SQL** for querying data.

### **Examples of RDBMS**

✅ **MySQL**
✅ **PostgreSQL**
✅ **Oracle Database**
✅ **SQL Server**

These databases are **blocking** when used with JDBC, meaning they **wait** for queries to finish before proceeding.

---

## **2. R2DBC (Reactive Relational Database Connectivity)**

**R2DBC** is a **reactive API** for working with relational databases in a **non-blocking** manner.
It allows databases like **PostgreSQL, MySQL, and Microsoft SQL Server** to be used **asynchronously** in Spring WebFlux.

### **Why Use R2DBC Instead of JDBC?**

| Feature             | **JDBC (Blocking)**             | **R2DBC (Non-blocking)**       |
| ------------------- | ------------------------------- | ------------------------------ |
| Thread Usage        | One thread per query (blocking) | Uses event loop (non-blocking) |
| Scalability         | Limited by thread pool size     | Highly scalable                |
| Performance         | Slower under high load          | Faster for concurrent queries  |
| Works with WebFlux? | ❌ No                            | ✅ Yes                          |

### **R2DBC Supported Databases**

✅ **PostgreSQL**
✅ **MySQL**
✅ **Microsoft SQL Server**
✅ **H2 Database**

---

## **How to Use R2DBC in Spring Boot WebFlux**

### **1. Add R2DBC Dependency**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
</dependency>
```

### **2. Configure Database Connection**

```yaml
spring:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/mydb
    username: myuser
    password: mypassword
```

### **3. Create Reactive Repository**

```java
@Repository
public interface UserRepository extends ReactiveCrudRepository<User, String> {
    Flux<User> findByName(String name);
}
```

### **4. Service Layer (Using Flux & Mono)**

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public Flux<User> getAllUsers() {
        return userRepository.findAll(); // Returns Flux (multiple users)
    }

    public Mono<User> getUserById(String id) {
        return userRepository.findById(id); // Returns Mono (single user)
    }
}
```

---

## **When to Use R2DBC?**

✅ **Use R2DBC when:**

* You need **high scalability** with Spring WebFlux.
* Your app handles **real-time data streaming**.
* Your database supports **R2DBC drivers**.

❌ **Avoid R2DBC when:**

* Your database **does not support R2DBC**.
* You are using traditional **Spring MVC with JDBC** (blocking).

Would you like a **full example** with a working REST API using R2DBC? 🚀

