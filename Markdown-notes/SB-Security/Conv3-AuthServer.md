### Chat : spring boot microservice application auth server, use Jwt

### **Spring Boot Microservices Authentication Server Using JWT**

We will implement a **JWT-based authentication server** using **Spring Boot, Spring Security, and JWT**.

---

### **1. Project Setup**

#### **Dependencies (pom.xml)**

Add the following dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>io.jsonwebtoken</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

---

### **2. Configure User Entity & Repository**

#### **User Entity**

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private String role; // Example: ROLE_USER, ROLE_ADMIN

    // Getters and Setters
}
```

#### **User Repository**

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

---

### **3. JWT Utility Class**

#### **Generate and Validate JWT Token**

```java
@Component
public class JwtUtil {
    private String secretKey = "mySecretKey"; // Use environment variables in production

    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60)) // 1 hour expiry
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
    }

    public String extractUsername(String token) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        return extractUsername(token).equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    private boolean isTokenExpired(String token) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody().getExpiration().before(new Date());
    }
}
```

---

### **4. Implement User Authentication**

#### **UserDetailsService for Authentication**

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        
        return new org.springframework.security.core.userdetails.User(
                user.getUsername(), user.getPassword(), 
                Collections.singleton(new SimpleGrantedAuthority(user.getRole())));
    }
}
```

---

### **5. Authentication Controller (Login & Token Generation)**

```java
@RestController
@RequestMapping("/auth")
public class AuthController {
    @Autowired
    private AuthenticationManager authenticationManager;
    
    @Autowired
    private JwtUtil jwtUtil;
    
    @Autowired
    private CustomUserDetailsService userDetailsService;

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody AuthRequest authRequest) {
        authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(authRequest.getUsername(), authRequest.getPassword()));

        UserDetails userDetails = userDetailsService.loadUserByUsername(authRequest.getUsername());
        String token = jwtUtil.generateToken(userDetails.getUsername());
        
        return ResponseEntity.ok(new AuthResponse(token));
    }
}
```

#### **AuthRequest & AuthResponse DTOs**

```java
public class AuthRequest {
    private String username;
    private String password;
    // Getters and Setters
}

public class AuthResponse {
    private String token;

    public AuthResponse(String token) {
        this.token = token;
    }
    // Getter
}
```

---

### **6. Security Configuration**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Autowired
    private CustomUserDetailsService userDetailsService;
    
    @Autowired
    private JwtUtil jwtUtil;

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {
        return configuration.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeHttpRequests()
            .requestMatchers("/auth/login").permitAll()
            .anyRequest().authenticated()
            .and()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(new JwtAuthenticationFilter(jwtUtil, userDetailsService), UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

### **7. JWT Authentication Filter**

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final JwtUtil jwtUtil;
    private final CustomUserDetailsService userDetailsService;

    public JwtAuthenticationFilter(JwtUtil jwtUtil, CustomUserDetailsService userDetailsService) {
        this.jwtUtil = jwtUtil;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            chain.doFilter(request, response);
            return;
        }

        String token = header.substring(7);
        String username = jwtUtil.extractUsername(token);
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            if (jwtUtil.validateToken(token, userDetails)) {
                UsernamePasswordAuthenticationToken auth = 
                        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        chain.doFilter(request, response);
    }
}
```

---

### **8. Testing the Authentication Server**

#### **Step 1: Register a User in the Database**

Manually insert a user with a hashed password:

```sql
INSERT INTO users (username, password, role) 
VALUES ('admin', '$2a$10$Dow8j4R5FfU5iImO6NYoRujue/sNf9pDgDJ.2vJ99r8vY8WSVjG6G', 'ROLE_ADMIN');
```

(Use **BCrypt** to hash the password before inserting it.)

#### **Step 2: Login to Get JWT Token**

**POST** `http://localhost:8080/auth/login`

```json
{
  "username": "admin",
  "password": "admin123"
}
```

**Response:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### **Step 3: Use JWT Token to Access Protected Routes**

Include the token in the `Authorization` header when accessing secured endpoints.

```
Authorization: Bearer <your_token>
```

---

### **Next Steps**

✅ Secure other microservices with JWT
✅ Implement role-based access control (RBAC)
✅ Use Redis or database for token blacklisting (logout functionality)

Would you like me to help integrate this with your Angular frontend? 🚀


### Chat : how this is integrated with API gateway?

### **Integrating JWT Authentication with API Gateway in a Spring Boot Microservices Architecture**

When using **Spring Cloud Gateway** as an API Gateway in a **JWT-based authentication system**, the gateway should handle authentication before forwarding requests to microservices.

---

## **1️⃣ Architecture Overview**

* 1️⃣ **Clients (Angular / Postman, etc.)** send login requests to the **Auth Server**.
* 2️⃣ The **Auth Server** generates a **JWT Token** and returns it to the client.
* 3️⃣ The client includes the JWT in the `Authorization` header when calling APIs.
* 4️⃣ **Spring Cloud Gateway** validates the JWT before forwarding requests to microservices.
* 5️⃣ If valid, the request is forwarded; otherwise, it is rejected.

🚀 **Components Involved:**

* **Auth Service** → Handles user authentication & issues JWT tokens.
* **API Gateway (Spring Cloud Gateway)** → Validates JWT before routing requests.
* **Microservices (Product, Order, etc.)** → Only accept requests from the Gateway.

---

## **2️⃣ Setup API Gateway with JWT Authentication**

### **Step 1: Add Dependencies (API Gateway Service)**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.11.5</version>
</dependency>
```

---

### **Step 2: Configure Gateway to Forward Requests**

#### **application.yml (API Gateway Service)**

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: product-service
          uri: http://localhost:8081/
          predicates:
            - Path=/api/products/**
          filters:
            - AuthenticationFilter
          
        - id: order-service
          uri: http://localhost:8082/
          predicates:
            - Path=/api/orders/**
          filters:
            - AuthenticationFilter
```

📌 **Custom filter (`AuthenticationFilter`) will validate JWT before forwarding the request.**

---

### **Step 3: Create JWT Utility in API Gateway**

```java
@Component
public class JwtUtil {
    private String secretKey = "mySecretKey";

    public Claims extractClaims(String token) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody();
    }

    public String extractUsername(String token) {
        return extractClaims(token).getSubject();
    }

    public boolean validateToken(String token) {
        return extractClaims(token).getExpiration().after(new Date());
    }
}
```

---

### **Step 4: Implement Authentication Filter in API Gateway**

```java
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {
    @Autowired
    private JwtUtil jwtUtil;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String authHeader = exchange.getRequest().getHeaders().getFirst(HttpHeaders.AUTHORIZATION);
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        String token = authHeader.substring(7);
        if (!jwtUtil.validateToken(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

✅ **This filter runs before routing requests to microservices, ensuring only valid JWTs are allowed.**

---

## **3️⃣ Secure Microservices (Product & Order)**

Each microservice should:

* Extract JWT from the request
* Validate JWT
* Allow/deny access based on the user's role

### **Product Service - Security Configuration**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeHttpRequests()
            .requestMatchers("/api/products/**").authenticated()
            .and()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        return http.build();
    }
}
```

---

## **4️⃣ Testing the Setup**

### **Step 1: Login & Get JWT Token**

Send a **POST** request to `http://localhost:8080/auth/login`

```json
{
  "username": "admin",
  "password": "admin123"
}
```

#### **Response**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### **Step 2: Access Product Service via API Gateway**

**GET** `http://localhost:8080/api/products`

```
Authorization: Bearer <your_token>
```

✅ If the token is valid, the request is forwarded to `http://localhost:8081/api/products`.
🚫 If the token is missing or invalid, the Gateway returns **401 Unauthorized**.

---

## **5️⃣ Summary**

✔ **Auth Service** → Issues JWT tokens.
✔ **API Gateway** → Validates JWT before forwarding requests.
✔ **Microservices** → Accept requests only if JWT is valid.

Would you like me to integrate role-based access control (RBAC) next? 🚀


### Chat : "/ Use environment variables in production" ??

You're right to ask about that! Instead of hardcoding the JWT secret key (`mySecretKey`) in the application, it's best to store it securely using **environment variables** or a **configuration server**.

### **✅ Use Environment Variables (Recommended for Production)**

Modify the `JwtUtil` class to read the secret key from environment variables:

```java
@Component
public class JwtUtil {
    private final String secretKey;

    public JwtUtil() {
        // Read secret key from environment variables
        this.secretKey = System.getenv("JWT_SECRET_KEY");
    }

    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60)) // 1 hour expiry
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
    }
}
```

Then, set the **environment variable** before running the application:
**Linux/macOS (Terminal)**

```sh
export JWT_SECRET_KEY=mySuperSecretKey
```

**Windows (Command Prompt)**

```sh
set JWT_SECRET_KEY=mySuperSecretKey
```

---

### **✅ Use Spring Boot Configuration (Alternative)**

Instead of environment variables, store the secret key in `application.properties`:

```properties
jwt.secret=mySuperSecretKey
```

Modify `JwtUtil` to read from **Spring Boot properties**:

```java
@Value("${jwt.secret}")
private String secretKey;
```

🔹 **Pros:** More convenient in development
🔹 **Cons:** Not secure for production unless using an external config server

---

### **✅ Use Spring Cloud Config (For Distributed Systems)**

If your microservices are distributed, use **Spring Cloud Config Server** to store and fetch the JWT secret securely.

Would you like a guide on integrating **Spring Cloud Config** for secret management? 🚀


### Chat : Yes

### **Integrating Spring Cloud Config for Secure Secret Management in a Microservices Architecture**

In a **Spring Boot microservices application**, managing sensitive configurations like **JWT secret keys**, **database credentials**, and **API keys** securely is critical. **Spring Cloud Config** provides a centralized configuration server that stores and distributes configurations to all microservices.

---

## **1️⃣ Architecture Overview**

📌 **Spring Cloud Config Server**: Stores secrets in a central location (e.g., Git repository).
📌 **Microservices (Auth, API Gateway, Product, etc.)**: Fetch configuration from the Config Server.
📌 **Secure storage**: JWT secrets and database credentials are stored in a private Git repo or encrypted storage.

---

## **2️⃣ Setting Up Spring Cloud Config Server**

### **Step 1: Create a New Spring Boot Project for Config Server**

Add dependencies in **Config Server's `pom.xml`**:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### **Step 2: Enable Config Server**

Create `ConfigServerApplication.java`:

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

### **Step 3: Configure the Config Server**

Modify `application.properties`:

```properties
server.port=8888
spring.cloud.config.server.git.uri=https://github.com/your-org/config-repo
spring.cloud.config.server.git.clone-on-start=true
```

💡 **Replace `https://github.com/your-org/config-repo`** with your private Git repository containing configuration files.

---

## **3️⃣ Store JWT Secrets in Config Repository**

Create a **Git repository** for configuration files and add a file:

📁 `config-repo/application.yml`

```yaml
jwt:
  secret: mySuperSecretKey
```

📁 `config-repo/auth-service.yml`

```yaml
jwt:
  secret: mySuperSecretKeyForAuthService
```

📁 `config-repo/gateway-service.yml`

```yaml
jwt:
  secret: mySuperSecretKeyForGateway
```

✅ Now, each service can **fetch the JWT secret dynamically** from the Config Server.

---

## **4️⃣ Configure Microservices to Use the Config Server**

### **Step 1: Add Dependencies in Each Microservice (`pom.xml`)**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### **Step 2: Update `bootstrap.yml` in Each Microservice**

Each microservice (Auth, Gateway, Product, etc.) should have a `bootstrap.yml`:

```yaml
spring:
  application:
    name: auth-service
  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true
```

🔹 The `name` property matches the `auth-service.yml` file in the config repo.
🔹 The microservice will **fetch JWT secrets dynamically** at startup.

---

## **5️⃣ Modify JWT Utility Class to Use Config Properties**

Instead of hardcoding the secret key, fetch it from **Spring Cloud Config**.

```java
@Component
@RefreshScope  // Allows dynamic updates without restarting the service
public class JwtUtil {
    @Value("${jwt.secret}")
    private String secretKey;

    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60))
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
    }
}
```

---

## **6️⃣ Refresh Configuration Dynamically Without Restarting**

If you change the secret key in the Git repository, update all services **without restarting**.

### **Step 1: Add Actuator Dependency**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### **Step 2: Enable Refresh Endpoint in `application.properties`**

```properties
management.endpoints.web.exposure.include=refresh
```

### **Step 3: Refresh Configuration**

After updating the config repo, send a **POST** request to:

```
POST http://localhost:8081/actuator/refresh
```

🔹 This will reload the secret **without restarting the service!** 🎉

---

## **7️⃣ Summary**

✔ **Spring Cloud Config Server** stores secrets in Git.
✔ **Microservices fetch JWT secrets dynamically** from the Config Server.
✔ **JWT keys can be updated dynamically** using `/actuator/refresh`.
✔ **No hardcoded secrets** in microservices.

Would you like help with **encrypting secrets** in the config repo for added security? 🔐🚀


### Chat : Yes

### **🔐 Encrypting Secrets in Spring Cloud Config for Added Security**

Storing secrets in a **Git repository** (even a private one) is not secure. **Spring Cloud Config Server** provides a built-in way to encrypt and decrypt secrets **securely** using **Spring Cloud Vault** or **JCE Encryption**.

---

## **1️⃣ Encryption Methods in Spring Cloud Config**

1️⃣ **Spring Cloud Vault** (Recommended for Production)

* Stores secrets in **HashiCorp Vault** (a dedicated secret management tool).
* No secrets are stored in Git—only references to Vault.

2️⃣ **JCE Encryption (Spring Config Encryption)**

* Secrets are **encrypted before committing to Git**.
* Config Server **decrypts values** dynamically at runtime.
* Simpler than Vault but requires a **secure encryption key**.

For now, I'll show **JCE Encryption**, as it's easier to integrate.

---

## **2️⃣ Setting Up JCE Encryption for Spring Cloud Config**

### **Step 1: Add Dependencies in Config Server (`pom.xml`)**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-rsa</artifactId>
</dependency>
```

📌 **`spring-security-rsa`** provides encryption and decryption functionality.

---

### **Step 2: Generate an Encryption Key**

Run the following command to generate an **RSA key pair**:

```sh
openssl genrsa -out config-server-key.pem 2048
openssl rsa -in config-server-key.pem -pubout -out config-server-public-key.pem
```

🔹 **`config-server-key.pem`** → Private Key (Used by Config Server)
🔹 **`config-server-public-key.pem`** → Public Key (Used to encrypt secrets)

---

### **Step 3: Configure Config Server to Use the Key**

In **`application.properties`** of the **Config Server**, set the encryption key:

```properties
encrypt.key-store.location=classpath:config-server-key.pem
encrypt.key-store.password=mySecurePassword
```

📌 The **private key** is stored **securely** and used for decryption.

---

## **3️⃣ Encrypt Secrets Before Storing in Git**

### **Step 1: Start Config Server**

Run the Config Server:

```sh
mvn spring-boot:run
```

### **Step 2: Encrypt the Secret**

Use the `/encrypt` endpoint of the Config Server:

```sh
curl -X POST http://localhost:8888/encrypt -d 'mySuperSecretKey'
```

🔹 Response (Encrypted secret):

```
AQBJKF3o9FBlXpzXv+KGH5…==  
```

---

### **Step 3: Store the Encrypted Secret in Git**

Modify **`config-repo/application.yml`**:

```yaml
jwt:
  secret: "{cipher}AQBJKF3o9FBlXpzXv+KGH5…=="
```

📌 **Spring Cloud Config will decrypt this value dynamically** when microservices request it.

---

## **4️⃣ Microservices Fetch and Use Decrypted Secrets**

✅ **No changes are needed** in microservices!
When a microservice requests `jwt.secret`, **Config Server decrypts it** automatically.

---

## **5️⃣ Dynamic Secret Refresh (Optional)**

To reload updated secrets **without restarting services**, call:

```sh
POST http://localhost:8081/actuator/refresh
```

---

## **🔒 Summary: Secure Secret Management**

✔ **Secrets are encrypted before storing in Git**.
✔ **Config Server decrypts them dynamically**.
✔ **No sensitive data is stored in plaintext**.
✔ **Microservices don't need to change their code**.

Would you like a guide on integrating **HashiCorp Vault** for even higher security? 🚀


### Chat : Yes

### **🔐 Integrating HashiCorp Vault for Secure Secret Management in Spring Boot Microservices**

**HashiCorp Vault** is a production-grade secret management tool that securely stores **API keys, database credentials, JWT secrets**, and more. Instead of storing secrets in **Git (Spring Cloud Config)**, Vault keeps them in a **secure, centralized store**.

---

## **1️⃣ Why Use HashiCorp Vault?**

✔ **Secrets are never stored in Git** – prevents accidental leaks.
✔ **Dynamic secret rotation** – automatically generates short-lived credentials.
✔ **Access control with policies** – limits who can access which secrets.
✔ **Secure encryption and audit logging** – tracks secret access.

---

## **2️⃣ Setting Up HashiCorp Vault Locally**

### **Step 1: Install and Start Vault**

🔹 Download & Install Vault from [here](https://developer.hashicorp.com/vault/docs/install)
🔹 Start Vault in **development mode** (not for production):

```sh
vault server -dev
```

💡 Vault will generate a **root token**, copy it for later use.

---

### **Step 2: Login to Vault**

Open a new terminal and authenticate:

```sh
export VAULT_ADDR='http://127.0.0.1:8200'
vault login root
```

🔹 Replace `root` with the root token from the Vault server logs.

---

### **Step 3: Enable Key-Value Secret Engine**

```sh
vault secrets enable -path=secret kv
```

🔹 This creates a **key-value (KV) storage** at `secret/`.

---

### **Step 4: Store Secrets in Vault**

```sh
vault kv put secret/jwt secret=mySuperSecretKey
```

🔹 This stores `mySuperSecretKey` in `secret/jwt`.

---

## **3️⃣ Configure Spring Boot to Use Vault**

### **Step 1: Add Vault Dependencies**

In each microservice’s `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-vault-config</artifactId>
</dependency>
```

---

### **Step 2: Configure Spring Boot to Fetch Secrets**

In `bootstrap.yml` (each microservice):

```yaml
spring:
  cloud:
    vault:
      uri: http://127.0.0.1:8200
      token: root
      kv:
        enabled: true
```

🔹 Vault will **fetch and inject secrets** into the application.

---

### **Step 3: Fetch Secrets in Microservices**

Modify `application.yml`:

```yaml
jwt:
  secret: ${secret.jwt}
```

🔹 **Spring Boot will replace `${secret.jwt}`** with the value stored in Vault.

---

## **4️⃣ Testing the Integration**

Start your microservice and check if it fetches the secret:

```sh
curl -X GET http://localhost:8081/actuator/env | grep jwt.secret
```

If everything is set up correctly, your application will securely fetch the **JWT secret from Vault** instead of Git.

---

## **5️⃣ Summary**

✔ **Vault securely stores secrets** (not in Git).
✔ **Spring Boot auto-fetches secrets from Vault**.
✔ **Secrets are updated dynamically** without restarting services.

Would you like help with setting up **Vault authentication using Kubernetes or AWS IAM?** 🚀


### Chat : "Now, each service can **fetch the JWT secret dynamically** from the Config Server." why there are multiple ,yml files here above

Great question! The reason there are **multiple `.yml` files** in the **Spring Cloud Config repository** is to manage configurations for multiple microservices **independently** while keeping a common configuration when needed.

---

### **📂 Configuration File Structure in Spring Cloud Config**

Assume you have multiple microservices, such as:

* **Auth Service** (`auth-service`)
* **API Gateway** (`gateway-service`)
* **Product Service** (`product-service`)

Your **config repository** will have:

```
📁 config-repo
│── application.yml        # Common configuration for all services
│── auth-service.yml       # Specific config for Auth Service
│── gateway-service.yml    # Specific config for API Gateway
│── product-service.yml    # Specific config for Product Service
```

---

### **🚀 Why Multiple `.yml` Files?**

1️⃣ **application.yml (Global Config)**

* This contains **common properties** used by all microservices.
* Example (`application.yml`):

  ```yaml
  jwt:
    secret: commonSuperSecretKey
  logging:
    level: INFO
  ```
* All microservices **inherit these properties** unless overridden.

2️⃣ **auth-service.yml (Microservice-Specific Config)**

* Only used by the **Auth Service** (name matches `spring.application.name`).
* Example (`auth-service.yml`):

  ```yaml
  jwt:
    secret: authServiceSecretKey
  auth:
    tokenExpiry: 3600
  ```
* This overrides the **global `application.yml`**.

3️⃣ **gateway-service.yml (API Gateway Config)**

* Example (`gateway-service.yml`):

  ```yaml
  jwt:
    secret: gatewayServiceSecretKey
  security:
    rateLimit: 1000
  ```

---

### **🔄 How Microservices Fetch Their Config?**

Each microservice fetches its config from **Spring Cloud Config Server** using:

```yaml
spring:
  application:
    name: auth-service
  cloud:
    config:
      uri: http://localhost:8888
```

👉 When **Auth Service** starts, it gets **`auth-service.yml` + `application.yml`**.
👉 When **API Gateway** starts, it gets **`gateway-service.yml` + `application.yml`**.

---

### **✅ Key Benefits of This Approach**

✔ **Centralized Configuration Management** (Easy updates).
✔ **Per-Microservice Configuration** (No unnecessary configs).
✔ **Overrides Common Configurations When Needed**.

Would you like to see **an example of fetching configs dynamically at runtime**? 🚀



