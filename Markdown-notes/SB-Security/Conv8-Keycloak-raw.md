## **Conversation **

### Chat : can you describe how keycloak is integrated with real time actual spring boot application, also usage and all context related to this?

Here is a **real, practical, industry-style explanation** of how **Keycloak is integrated with a Spring Boot application**, how it works in real-time at runtime, and what all parts are involved.

---

# ✅ **1. What Keycloak Actually Does in a Real Application**

Keycloak is an **Identity and Access Management (IAM)** system.

In a real application it becomes responsible for:

| Responsibility      | Description                              |
| ------------------- | ---------------------------------------- |
| Authentication      | Handles login, logout, session mgmt      |
| Authorization       | Role-based or permission-based access    |
| Token Issuance      | Issues JWT/OAuth2 tokens                 |
| User Management     | Users, Roles, Groups                     |
| Identity Federation | Login with Google, GitHub, etc.          |
| SSO                 | Single sign-on across microservices/apps |

**Your Spring Boot app no longer handles login/password authentication.
Keycloak does it. Spring Boot only validates tokens.**

---

# ✅ **2. High-Level Integration Architecture**

```
+---------------+        +-------------------+
| Spring Boot   |        | Keycloak Server   |
| Backend API   |<------>| Authentication    |
| (/products)   | Token  | User mgmt, Tokens |
+-------^-------+        +---------^---------+
        |                          |
        | (Bearer Token)           |
        v                          |
+---------------+        +----------------------------+
| Angular/React | -----> | Keycloak Login UI (Realm) |
| Frontend      |        +----------------------------+
+---------------+
```

---

# ✅ **3. Required Keycloak Setup (Real Scenario)**

### **Step 1: Create a Realm**

Example:
**realm: ecommerce**

A realm is an isolated security domain—like a tenant.

---

### **Step 2: Create a Client**

Example:

```
clientId: product-api
clientType: OpenID Connect
clientAccessType: confidential OR public
```

If backend:

* Often **confidential**
* With client secret

---

### **Step 3: Configure Valid Redirect URIs**

For frontend:

```
http://localhost:4200/*
```

For backend:

* Nothing special; backend does not redirect.

---

### **Step 4: Configure Roles**

Example:

```
ROLE_USER
ROLE_ADMIN
```

You attach these to users or groups.

---

### **Step 5: Create Users**

Example:

```
username: shivendra
password: ******
roles: ROLE_USER
```

---

# ✅ **4. Spring Boot Integration (Real-Time Workflow)**

Spring Boot does NOT authenticate users.
Keycloak authenticates, issues JWT tokens.
Spring Boot **validates the token** and allows or denies access.

---

## ⭐ **A. Add Dependencies**

Use Spring Security + Spring OAuth2 Resource Server.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

---

## ⭐ **B. Configure application.yml**

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/ecommerce
```

Spring Boot now knows:

✔ Where to fetch the public keys
✔ How to validate tokens
✔ How to extract roles

---

## ⭐ **C. Security Configuration**

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
      .authorizeHttpRequests(auth -> auth
        .requestMatchers("/products/public").permitAll()
        .requestMatchers("/products/admin").hasRole("ADMIN")
        .anyRequest().authenticated()
      )
      .oauth2ResourceServer(oauth -> oauth.jwt());

    return http.build();
}
```

---

# 🚀 **5. Full Real-Time Flow When User Accesses Your API**

Let’s take example:

👉 User opens frontend
👉 User clicks **Login**

### **Step 1: Frontend redirects to Keycloak login**

```
https://keycloak/auth?client_id=product-app&redirect_uri=...
```

---

### **Step 2: User logs in on Keycloak**

Keycloak validates:
✔ username
✔ password
✔ MFA (if enabled)
✔ account status
✔ roles

---

### **Step 3: Keycloak issues tokens**

Frontend receives:

* **Access Token (JWT)**
* **Refresh Token**
* **ID Token (optional)**

Access token contains:

```json
{
  "preferred_username": "shivendra",
  "realm_access": {
    "roles": ["ROLE_USER"]
  },
  "exp": 1734872932
}
```

---

### **Step 4: Frontend calls Spring Boot API with token**

```
GET /products

Authorization: Bearer eyJhbGciOiJSUzI1...
```

---

### **Step 5: Spring Boot validates token**

Spring Boot checks:

✔ token signature with Keycloak public key
✔ expiration
✔ issuer
✔ audience
✔ roles

---

### **Step 6: Spring Boot performs authorization**

If endpoint requires:

```
hasRole("ADMIN")
```

Spring Security checks token claims:

```
"roles": ["USER"]
```

If mismatch → `403 Forbidden`.

---

### **Step 7: API returns response**

If valid:

```
200 OK
[
   {"id":1,"product":"Shoes"}
]
```

---

# 🚀 **6. Real-Time Use Cases**

| Use Case              | How Keycloak Helps               |
| --------------------- | -------------------------------- |
| Login / Signup        | Uses Keycloak login UI or API    |
| MFA                   | OTP / TOTP / WebAuthn            |
| Role-based access     | Admin, User, Manager roles       |
| Microservice Security | Same token works across services |
| Logout                | Keycloak ends session everywhere |
| User Federation       | LDAP/AD login                    |
| SSO                   | All apps use same login          |

---

# 🔥 **7. How Registration Works (You Asked Earlier)**

You can register user in two ways:

### ✔ **1. From Keycloak UI (Admin Portal)**

Admin creates users.

### ✔ **2. From Application (Self-Register Flow)**

Your backend calls:

```
POST /admin/realms/{realm}/users
```

Usually through:

* Keycloak Admin REST API
* Or a custom UI connected to Keycloak

Then:

✔ Email verification
✔ Default roles
✔ Required actions (set password, verify email)

---

# 🧠 **8. Why Industry Uses Keycloak with Spring Boot**

### Pros:

✔ No need to write login/signup backend
✔ Secure by design
✔ Provides OAuth2, OpenID Connect, SAML
✔ Token-based, perfect for microservices
✔ Easy integration with Angular/React
✔ Centralized authentication & user mgmt
✔ SSO support

### Cons:

❌ Extra infrastructure
❌ Need DevOps knowledge
❌ Learning curve

---

# 🌟 If You Want, I Can Also Provide:

✔ Sequence diagrams
✔ End-to-end flow diagrams
✔ Real project folder structure
✔ Sample frontend integration
✔ Multi-microservice architecture with Keycloak
✔ Roles & permissions best practices

Just tell me — what depth do you want next?


### Chat : so if we are registering a user on our e-commerce microservice based spring boot application, and we dont want to route to keycloak login/register page, we just have our front end ui and from there user will enter registration inputs like name, username, passwod etc and then internally I want to create user in keylcoak through my spring boot app, can it be possible and is this industry standard?

Short answer: **Yes, it's absolutely possible** AND **yes, it's widely used in industry**—especially for e-commerce, B2C apps, or custom-branded applications where you don’t want the user to see Keycloak’s default login/registration screens.

Below is the **clear explanation, when to use this approach, how it works, what’s standard, and what to avoid**.

---

# ✅ **1. Can you register users in Keycloak *via your Spring Boot microservice* instead of showing Keycloak UI?**

**Yes, you can — and it is a common, industry-standard pattern.**

This approach is typically called:

✔ **Custom Registration Flow**
✔ **Programmatic User Creation in Keycloak**
✔ **Keycloak Admin REST API-based Registration**

Large platforms do this:

* Amazon
* Flipkart
* Myntra
* Meesho
* Any B2C SaaS tool

They never show Keycloak login/register UI — they build their **own UI** and **their backend talks to Keycloak internally**.

---

# ⭐ **2. Where This Approach Fits in Microservice Architecture**

Typical microservice architecture:

```
Frontend (React/Angular)
        |
        v
User-Service (Spring Boot microservice)
        |
        v
Keycloak (User identity store)
```

The **User-Service** acts as the **facade** between the UI and Keycloak.

---

# ⭐ **3. Real Workflow: Custom Registration With Keycloak**

### 🔹 Step 1: User hits your frontend

```
POST /api/register
{
  "name": "Shivendra",
  "email": "shiv@example.com",
  "password": "xyz123"
}
```

### 🔹 Step 2: Your Spring Boot “User-Service” calls Keycloak Admin REST API

It sends a request like:

```
POST /admin/realms/ecommerce/users
```

With:

* username
* email
* attributes
* enabled
* initial password

### 🔹 Step 3: Keycloak creates the user

Stores:

✔ username
✔ hashed password
✔ roles
✔ email verification state

### 🔹 Step 4: Your app returns a response

```
User created successfully
```

No Keycloak page ever shown.

---

# ⭐ **4. Is this an industry-standard approach?**

### 🔥 **YES, especially for B2C and custom apps.**

Use cases where this IS standard:

✔ You want your own branded UI
✔ You want complete control over UX
✔ You need dynamic fields (mobile OTP, address, referral codes)
✔ You don’t want the Keycloak UI to appear anywhere
✔ Your customer should feel your platform, not Keycloak

Most companies use Keycloak as:

**An identity backend → NOT as a customer-facing UI.**

---

# 🚫 **5. When people DO use Keycloak’s default login/registration UI**

This is typically done for:

* Internal enterprise apps
* Admin portals
* Developer tools
* Microservices dashboards
* Low-cost internal authentication setup

Public consumer apps rarely expose Keycloak screens.

---

# ⭐ **6. How to Create Users in Keycloak from Spring Boot (Admin REST API)**

### 1️⃣ Get an Admin Token

Your service uses a service account client to authenticate:

```
POST /realms/master/protocol/openid-connect/token
client_id = admin-cli
client_secret = ****
grant_type = client_credentials
```

### 2️⃣ Call Create User API

```
POST /admin/realms/ecommerce/users
{
  "username": "shivendra",
  "email": "shiv@example.com",
  "enabled": true,
  "firstName": "Shivendra",
  "credentials": [{
       "type": "password",
       "value": "xyz123",
       "temporary": false
  }]
}
```

### 3️⃣ Optionally assign roles

```
POST /admin/realms/ecommerce/users/{id}/role-mappings/realm
```

This is how e-commerce apps do it.

---

# ⭐ **7. Advantages of Custom Registration (Industry View)**

### ✔ Customizable UI

You control design, fields, flow.

### ✔ More secure user-facing endpoints

Keycloak admin endpoints are hidden from the public.

### ✔ Easier to integrate features like:

* OTP verification
* Email verification
* Captcha
* Invite codes
* Referral systems
* Address fields

### ✔ Smoother UX

User never leaves your UI.

---

# ⭐ **8. Any disadvantages?**

Only one:

### ❗You must secure your backend’s communication with Keycloak

Because your microservice must:

* Have a service account
* Use the admin API
* Store a client secret securely (Vault)

But this is normal for microservice architecture.

---

# ⭐ **9. Final Verdict: Should YOU Use This Approach?**

### For **e-commerce**, **yes**, 100% recommended.

You get:

✔ Your own UI
✔ Your own registration flow
✔ All users stored in Keycloak
✔ No exposure of Keycloak UI
✔ Complete flexibility

This is exactly how Keycloak is used in:

* Large retail apps
* Banking portals
* Travel portals
* Multi-tenant SaaS platforms

---

# 👉 If you want, I can also give you:

📌 Spring Boot code for registering a user using Keycloak Admin API
📌 Best architecture for “User-Service” with Keycloak
📌 JWT flow after custom registration
📌 Diagrams for the registration process
📌 Real project folder structure

Just tell me what level of detail you want next.


### Chat : yes please

Great — I’ll give you **everything you need** to implement a *production-grade* custom registration flow with Keycloak in a Spring Boot microservice.

Here’s what I’ll cover:

1. **Architecture Diagram (Custom Registration)**
2. **Exact Step-by-Step Flow (Industry Standard)**
3. **Spring Boot Code – Create User in Keycloak**
4. **Common Configurations (Role Mapping, Email Verification, Password Policy)**
5. **Best Practices Used in Real Companies**
6. **Folder Structure for User-Service**
7. **Sequence Diagram**

---

# 🧱 **1. Architecture Diagram (Real-World)**

```
                 +-------------------------+
                 |       Frontend UI       |
                 +-----------+-------------+
                             |
                POST /api/auth/register
                             |
                 +-----------v-------------+
                 |     USER-SERVICE        |
                 | (Spring Boot Microserv) |
                 +-----------+-------------+
                             |
                    Keycloak Admin API
                             |
                 +-----------v-------------+
                 |       Keycloak Realm    |
                 +-------------------------+
```

---

# 🚀 **2. How the Custom Registration Flow Works**

### 👉 User enters details on YOUR UI

(name, mobile, email, password, etc.)

### 👉 UI sends request to your backend:

```
POST /api/auth/register
```

### 👉 Your Spring Boot microservice:

1. Validates form input
2. Calls **Keycloak Admin API** using service credentials
3. Creates user in Keycloak realm
4. Assigns default roles (USER)
5. Optionally sends email verification
6. Responds `201 User Created`

---

# 💻 **3. Spring Boot Code – Create User in Keycloak**

### **Step A — Add Dependency**

```xml
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-admin-client</artifactId>
    <version>22.0.1</version>
</dependency>
```

---

### **Step B — Configuration for Keycloak Admin Client**

```java
@Configuration
public class KeycloakConfig {

    @Value("${keycloak.auth-server-url}")
    private String serverUrl;

    @Value("${keycloak.realm}")
    private String realm;

    @Value("${keycloak.client-id}")
    private String clientId;

    @Value("${keycloak.client-secret}")
    private String clientSecret;

    @Bean
    public Keycloak keycloak() {
        return KeycloakBuilder.builder()
                .serverUrl(serverUrl)
                .realm("master") // Admin realm
                .grantType(OAuth2Constants.CLIENT_CREDENTIALS)
                .clientId(clientId)
                .clientSecret(clientSecret)
                .build();
    }
}
```

---

### **Step C — Service Method to Register User**

```java
@Service
public class KeycloakUserService {

    @Autowired
    private Keycloak keycloak;

    @Value("${keycloak.realm}")
    private String realm;

    public ResponseEntity<String> registerUser(RegisterRequest request) {

        // Create user representation
        UserRepresentation user = new UserRepresentation();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setEnabled(true);

        // Set credentials
        CredentialRepresentation credential = new CredentialRepresentation();
        credential.setType(CredentialRepresentation.PASSWORD);
        credential.setValue(request.getPassword());
        credential.setTemporary(false);

        user.setCredentials(Collections.singletonList(credential));

        // Call Keycloak Admin API
        RealmResource realmResource = keycloak.realm(realm);
        UsersResource usersResource = realmResource.users();

        Response response = usersResource.create(user);

        if (response.getStatus() == 201) {
            return ResponseEntity.ok("User registered successfully");
        } else {
            return ResponseEntity.status(response.getStatus())
                                 .body("Failed: " + response.getStatusInfo());
        }
    }
}
```

---

### **Step D — Controller**

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private KeycloakUserService keycloakUserService;

    @PostMapping("/register")
    public ResponseEntity<String> register(@RequestBody RegisterRequest request) {
        return keycloakUserService.registerUser(request);
    }
}
```

### Request DTO

```java
@Data
public class RegisterRequest {
    private String username;
    private String email;
    private String password;
}
```

---

# 🧩 **4. Assigning Default Role (USER)**

```java
public void assignRole(String userId, String roleName) {
    RealmResource realmResource = keycloak.realm(realm);
    RoleRepresentation role = realmResource.roles().get(roleName).toRepresentation();

    realmResource.users()
            .get(userId)
            .roles()
            .realmLevel()
            .add(Collections.singletonList(role));
}
```

Call this after creating the user.

---

# 📧 **5. Enabling Email Verification (Optional but common)**

### In Keycloak admin console:

**Realm → Login → “Verify Email” → ON**

If ON, users receive email automatically.

You can also manually trigger:

```java
realmResource.users().get(userId).sendVerifyEmail();
```

---

# 🔐 **6. Recommended Best Practices (Used by Real Companies)**

### 👍 Best practice 1 — Never store raw client-secret in code

Use Hashicorp Vault, AWS Secrets Manager, or `application-prod.yml` encrypted.

---

### 👍 Best practice 2 — Keep registration inside “User-Service”

Don’t let every service talk to Keycloak.

---

### 👍 Best practice 3 — Add throttling for registration

Avoid bots creating accounts.

---

### 👍 Best practice 4 — Add unique username/email check

Call:

```
GET /admin/realms/{realm}/users?email=a@b.com
```

---

### 👍 Best practice 5 — Add optional multi-step onboarding

Address, mobile, OTP, KYC, etc.

---

# 🗂 **7. Folder Structure for User-Service (Real Project)**

```
user-service/
 ├── controller/
 │    └── AuthController.java
 ├── service/
 │    └── KeycloakUserService.java
 ├── config/
 │    └── KeycloakConfig.java
 ├── dto/
 │    └── RegisterRequest.java
 ├── exception/
 │    └── CustomExceptionHandlers.java
 ├── security/
 │    └── JwtDecoderConfig.java   (if validating tokens)
 └── application.yml
```

---

# 📍 **8. Sequence Diagram (Registration)**

```
Frontend -> User-Service: POST /register (username, pass)
User-Service -> Keycloak: Create user via Admin API
Keycloak -> User-Service: 201 Created
User-Service -> Keycloak: Assign default role
User-Service -> Frontend: Registration successful
```

---

# 🎁 Want next?

I can also generate:

✅ Login flow with Keycloak (custom login + token retrieval)
✅ How JWT is validated in microservices
✅ Advanced Keycloak setup (realms, clients, roles)
✅ Architecture for multi-microservice authentication
✅ Code for Refresh Token flow
✅ API gateway integration (Keycloak + Gateway)

Just tell me what you’d like next.


## **Conversation **

### Chat : if we are using keycloak for spring sceurity, then how it works if we registr  user in context of user creation in keycloak or realm? 

When you use **Keycloak with Spring Security**, Keycloak becomes the **Identity Provider (IdP)** and **user management system**.
This means **user creation, authentication, roles, groups, tokens — all of it happens in Keycloak**, not in your Spring Boot app.

Let me explain clearly and in context of **user registration** and **how Spring Security interacts with it**.

---

# ✅ **How User Registration Works When Using Keycloak**

There are 2 ways users can be registered:

---

## **1️⃣ Users are created manually inside Keycloak Admin Console**

* You go into the Keycloak Admin UI → Users → Create user
* You set username, email, password, and assign roles.
* Keycloak stores the user internally in its database.

### 🔥 How Spring Boot sees this user?

* Spring Boot does **not** store the user.
* Spring Security also does **not** load from a DB.
* Instead, Spring Security **validates the JWT** issued by Keycloak.
* The roles (realm roles / client roles) inside the JWT decide the authorization.

👉 **Spring Boot only trusts what Keycloak sends in JWT.**
It does *not* have its own user table.

---

## **2️⃣ Users register via your application UI (Self-registration)**

You have a **/register** form.

You have two options:

### **Option A: Your backend calls Keycloak Admin API**

Your Spring Boot service uses Keycloak Admin REST API:

```
POST /admin/realms/{realm}/users
```

You send:

* username
* email
* password
* roles (optional)

Keycloak creates the user.
Spring Boot still does *not* maintain any user in DB.

This is clean and commonly used.

---

### **Option B: Redirect user to Keycloak's own Registration Page**

Keycloak provides built-in pages:

* `/realms/{realm}/protocol/openid-connect/registrations?client_id=your-app`

User registers directly in Keycloak UI.

Your Spring Boot app just receives the JWT after login — no user creation on your side.

---

# 📌 **So what happens in Spring Security?**

When a user logs in via Keycloak:

### **Step 1 — User authenticates with Keycloak**

* Username + password verified in Keycloak (not in Spring).
* Keycloak issues JWT access token and refresh token.

### **Step 2 — User accesses Spring Boot API**

Request:

```
Authorization: Bearer <access_token>
```

### **Step 3 — Spring Security’s Keycloak client verifies the token**

* Signature check
* Issuer check
* Expiry check

### **Step 4 — Spring Security extracts roles**

From JWT claims:

* `realm_access.roles`
* `resource_access.{client}.roles`

Example:

```json
{
  "realm_access": { "roles": ["USER"] },
  "resource_access": {
    "ecommerce-app": { "roles": ["admin"] }
  }
}
```

Spring maps them to authorities like:

```
ROLE_USER
ROLE_admin
```

### **Step 5 — Authorization happens**

Your `@PreAuthorize("hasRole('admin')")` checks these authorities.

---

# 📌 What about your database in Spring Boot?

If using Keycloak:

❌ You do **not** store users in your own DB
❌ You do **not** use UserDetailsService
❌ You do **not** store passwords

Only Keycloak stores:

* users
* hashed passwords
* roles
* groups
* login failures
* session IDs

---

# 🏗️ Example Architecture (Common Setup)

```
Frontend (React / Angular)
            |
            | Redirects to
            v
      Keycloak Login Page
            |
            | JWT token
            v
Spring Boot Microservices
         (secured by Keycloak)
```

User creation path:

```
UI → Spring Boot → Keycloak Admin API → User created
```

OR

```
UI → Keycloak Registration Page → User created
```

Spring Boot **does not manage any user**.

---

# ⭐ Summary

| Feature              | Handled By Keycloak? | Handled By Spring Boot?    |
| -------------------- | -------------------- | -------------------------- |
| User creation        | ✅ Yes                | ❌ No                       |
| Password storage     | ✅ Yes                | ❌ No                       |
| Authentication       | ✅ Yes                | ❌ No                       |
| JWT token generation | ✅ Yes                | ❌ No                       |
| Role management      | ✅ Yes                | ❌ No                       |
| Authorization        | ❌ No                 | ✅ Yes (based on JWT roles) |

---

# 👍 If you want, I can also explain:

✔ Keycloak Realm vs Client roles
✔ How JWT is structured in Keycloak
✔ How to integrate Keycloak with Spring Security (code)
✔ Best practices for user registration flows
✔ Microservice integration with Keycloak

Just tell me!


### Chat : ### **Option A: Your backend calls Keycloak Admin API** so is this industry standard approach, or is it good approach?

Yes — **using Keycloak for user management + JWT-based security is absolutely an industry-standard approach**, and specifically:

# ✅ **Using your backend to call the Keycloak Admin API for user registration IS a good approach — and widely used.**

Let’s break it down so you understand *why* it’s considered standard and reliable.

---

# ⭐ Why this approach is considered industry standard

## **1️⃣ Keycloak is a full-featured Identity Provider (IdP)**

Companies use Keycloak to centralize:

* User creation
* Password policies
* MFA
* Roles & permissions
* SSO (Single Sign-On)
* Social login
* Token handling

**You should not reinvent this in Spring Boot.**
Let the Identity Provider do its job.

---

## **2️⃣ Using Keycloak Admin API for registration is secure & recommended**

### Why?

* You don’t store passwords → Keycloak handles hashing, salting, iterations
* Centralized user management → less duplication
* Standardized API → widely supported
* You can enforce password policies, OTP, verification emails
* It works well in microservices (stateless)

This is the same approach used by:

* **Red Hat SSO**
* **AWS Cognito**
* **Okta**
* **Auth0**
* **Azure AD B2C**

They all use a similar pattern:
**App → Admin API → Identity Provider**

So yes, it's not just standard — it's the default way.

---

# ⭐ Why your microservice should NOT create its own user table

Industry practice discourages storing users separately because:

### 🚫 Duplicated user data

Frontend service DB and Keycloak DB both contain user entries → messy.

### 🚫 Inconsistent security

Your DB’s password rules vs Keycloak password rules → mismatches.

### 🚫 Complicated authentication

If Keycloak manages login but your app stores extra user fields, you need syncing → overhead.

### 🚫 Breaking the “Single Source of Truth” principle

Identity must come from one system only → Keycloak.

---

# ⭐ Why the “Backend → Keycloak Admin API” flow is good

### ✔ Your UI stays clean

The user sees your custom UI, not Keycloak UI.

### ✔ Control over registration logic

You can add:

* mobile number validation
* OTP
* email OTP
* referral code
* business rules

Before calling Keycloak.

### ✔ Good for microservice architecture

The *Auth service* handles user registration, not every microservice.

---

# ⭐ How companies implement this in real life

## **Architecture Example**

```
Client App (Web/Mobile) 
          ↓
Auth Gateway / Auth Microservice (Spring Boot)
          ↓ (calls)
Keycloak Admin API 
          ↓
Keycloak Database
```

Keycloak handles only identity.

Your Spring Boot service controls:

* workflows
* validation
* business logic
* logs
* capture metadata (IP, device, etc.)

---

# ⭐ When companies DON’T use this pattern

Only two scenarios:

### ❌ 1. Public-facing consumer apps needing a branded login UI

Then they use **Keycloak themes** or custom-built login pages.

### ❌ 2. Internal enterprise apps

They redirect to Keycloak login pages (SSO experience).

For both cases → Keycloak is still the identity store.

But the “Backend → Keycloak Admin API” pattern remains.

---

# ⭐ Final Verdict

## 🟢 **Using Keycloak Admin API to create users is absolutely an industry-standard approach — secure, scalable, maintainable, and recommended.**

## 🟢 **Your app should not store its own users if Keycloak manages authentication.**

## 🟢 **Microservice apps commonly follow exactly this pattern.**

---

If you want, I can also show you:

✔ Recommended folder structure
✔ Standard Spring Boot code for calling Keycloak Admin API
✔ User registration flow diagram (industry standard)
✔ How to store additional user attributes (profile info, mobile, etc.)

Tell me, and I’ll prepare it!


### Chat : explain me both the flow registration and login flow in this context

Below is the **complete, clear, industry-standard flow** for both:

✅ **User Registration** (with Keycloak)
✅ **User Login & Access Token flow** (with Keycloak + Spring Security)

I will explain them in **very practical microservice terms**, step-by-step.

---

# 🟦 PART 1 — USER REGISTRATION FLOW (Using Keycloak)

There are **two common flows** for registration:

---

# ✅ **FLOW A — Registration through your own Spring Boot service (Most used)**

(Your frontend does NOT show Keycloak UI)

### **Step-by-step flow**

### **1️⃣ User fills signup form**

Example:

```
POST /auth/register
{
  "username": "shiv",
  "email": "shiv@example.com",
  "password": "Pass@123",
  "mobile": "9876543210"
}
```

### **2️⃣ Your Spring Boot Auth Service validates input**

* password strength
* email format
* mobile format
* any business rules

### **3️⃣ Spring Boot calls the Keycloak Admin API**

```
POST /admin/realms/{realm}/users
```

Payload:

* username
* email
* enabled=true
* attributes (mobile)
* credentials (password)

### **4️⃣ Keycloak creates the user**

* Stores password (BCrypt/Argon2 hashed)
* Assigns default roles/groups
* Email verification (if enabled)
* Sends events/logs internally

### **5️⃣ Spring Boot returns success to frontend**

Example:

```
201 Created
"User registered successfully"
```

---

# 📌 **Result of Registration Flow A**

✔ Your Spring app does **not** store the user
✔ Keycloak becomes the **source of truth**
✔ You have full control of the registration UI
✔ Works great in microservices

---

# 🟦 FLOW B — Registration directly via Keycloak UI (Less common)

Your frontend redirects user to Keycloak’s built-in registration page:

```
/realms/myrealm/protocol/openid-connect/registrations?client_id=my-app
```

User registers → Keycloak creates the user → then redirects back to your app.

Best for:

* Internal enterprise apps (SSO)
* When branding Keycloak UI is easier

---

# 🟦 PART 2 — LOGIN FLOW (OAuth2 / OpenID Connect)

Regardless of how the user was created, login works the same.

Let’s take the most common:

---

# 🔵 **Login Flow with JWT (Authorization Code Flow with PKCE)**

This is the secure OAuth2 flow used by SPA + Spring Boot.

---

## 🚀 Step-by-step Login Flow

### **1️⃣ User clicks "Login"**

Frontend redirects user to Keycloak login page:

```
GET https://keycloak/auth/realms/myrealm/protocol/openid-connect/auth
?client_id=my-app
&redirect_uri=http://frontend/callback
&response_type=code
```

### **2️⃣ User enters username + password in Keycloak**

Authentication happens inside Keycloak:

* check password hash
* check user is enabled
* check MFA
* check user is not locked

### **3️⃣ Keycloak redirects back with Authorization Code**

Example:

```
http://frontend/callback?code=abc123
```

### **4️⃣ Frontend exchanges "code" for tokens**

Frontend → Keycloak:

```
POST /protocol/openid-connect/token
grant_type=authorization_code
code=abc123
client_id=my-app
code_verifier=xxxxx
```

### **5️⃣ Keycloak responds with**

* Access Token (JWT)
* Refresh Token
* ID Token (optional)

Example:

```json
{
  "access_token": "eyJhbGciOi...",
  "refresh_token": "eyJhbGciOi...",
  "expires_in": 3600
}
```

### **6️⃣ Frontend stores tokens (usually in memory)**

(Not in localStorage—unsafe)

---

## 🟢 **Flow continues when calling Spring Boot APIs**

### **7️⃣ Frontend calls your Spring Boot microservice**

```
GET /orders
Authorization: Bearer <access_token>
```

### **8️⃣ Spring Security verifies JWT**

Using:

* public key from Keycloak
* issuer = correct realm
* expiration
* signature

If valid → Continue
If invalid → 401 Unauthorized

### **9️⃣ Spring extracts roles from JWT**

Keycloak adds roles in token:

```
realm_access: { roles: ["USER"] }
resource_access: { "ecom-api": { roles: ["admin"] } }
```

Spring maps to authorities:

```
ROLE_USER
ROLE_admin
```

### **🔟 Spring Boot authorizes via annotations**

```
@PreAuthorize("hasRole('admin')")
```

---

# 🟢 **Refresh Token Flow (After Access token expires)**

When access token expires:

### 1️⃣ Frontend sends refresh token

```
POST /token
grant_type=refresh_token
refresh_token=<token>
```

### 2️⃣ Keycloak issues a new access token

(Fast, no login required)

---

# 🟦 END-TO-END ARCHITECTURE SUMMARY

### **Registration Flow**

```
Frontend → Spring Boot → Keycloak Admin API → User Created
```

### **Login Flow**

```
User → Keycloak Login Page → JWT Token → Spring Boot API
```

---

# 🟩 VISUAL SUMMARY (Simple Diagram)

```
           REGISTRATION FLOW
---------------------------------------------
Frontend → Auth Microservice → Keycloak (Admin API)


           LOGIN FLOW
---------------------------------------------
Frontend → Keycloak Login Page → JWT Tokens
Frontend → Spring Boot Microservice (Authorization)
```

---

# 🟣 Want advanced diagrams?

I can generate:

✔ Sequence diagram
✔ Component diagram
✔ Microservice architecture diagram with Keycloak
✔ High-level BRD-level diagram
✔ Class flow for Spring Security + Keycloak
✔ Token contents example

Just tell me!

### Chat : explain in more detail the registration flow

Below is a **very detailed, step-by-step, industry-level explanation** of the **Registration Flow** when using **Spring Boot + Keycloak Admin API**.
This is exactly how most companies implement it in real microservice systems.

I’ll cover:

* **Actors**
* **Sequence of requests**
* **Validations**
* **Keycloak operations**
* **Spring Boot code-level responsibilities**
* **Best practices**
* **Optional flows (OTP, email verification)**

---

# 🟦 **ACTORS**

1. **User (Client App)** – Web / Mobile
2. **API Gateway / Auth Service (Spring Boot)** – Your backend
3. **Keycloak Admin API** – Identity server
4. **Keycloak Database (internal)**

---

# 🟩 **FULL REGISTRATION FLOW — Detailed Walkthrough**

---

# **1️⃣ User submits signup form**

Frontend sends:

```
POST /auth/register
Content-Type: application/json
```

Payload:

```json
{
  "username": "shiv123",
  "email": "shiv@example.com",
  "password": "Pass@123",
  "mobile": "98765XXXXX",
  "firstName": "Shiv",
  "lastName": "Mishra"
}
```

---

# **2️⃣ Spring Boot receives request and performs validations**

### Validation types:

✔ Email format
✔ Password strength (optional)
✔ Mobile number format
✔ Username rules
✔ Check if email already exists (call to Keycloak)
✔ Check if username already exists (call to Keycloak)
✔ Business rules (ex: age verification)

### API example:

Spring Boot calls Keycloak:

```
GET /admin/realms/myrealm/users?email=shiv@example.com
```

If user exists → return error:

```
409 Conflict
"Email already registered"
```

---

# **3️⃣ Prepare Keycloak user creation payload**

Spring Boot prepares:

```json
{
  "username": "shiv123",
  "firstName": "Shiv",
  "lastName": "Mishra",
  "email": "shiv@example.com",
  "enabled": true,
  "credentials": [
    {
      "type": "password",
      "value": "Pass@123",
      "temporary": false
    }
  ],
  "attributes": {
    "mobile": "98765XXXXX",
    "referral_code": "REF1234"
  }
}
```

---

# **4️⃣ Spring Boot calls Keycloak Admin API to create user**

```
POST /admin/realms/myrealm/users
Authorization: Bearer <admin-service-token>
```

→ This call creates the **User object** in Keycloak **without roles yet**.

Response:

```
201 Created
Location: /admin/realms/myrealm/users/<uuid>
```

---

# **5️⃣ Spring Boot assigns default roles (optional but common)**

Many companies add:

* `ROLE_USER`
* `ROLE_CUSTOMER`
* `APP_USER`

You call:

```
POST /admin/realms/myrealm/users/{id}/role-mappings/realm
```

Payload:

```json
[
  {
    "id": "role-id",
    "name": "USER"
  }
]
```

---

# **6️⃣ Keycloak stores user internally**

Inside its database, Keycloak stores:

* username
* email
* firstName, lastName
* attributes (mobile, etc.)
* hashed password (PBKDF2, bcrypt, or Argon2)
* roles
* groups
* creation timestamp
* requiredActions (ex: VERIFY_EMAIL)

---

# **7️⃣ (Optional) Keycloak triggers built-in actions**

If enabled in realm settings:

### ✔ Email Verification

Keycloak automatically sends a verification email.

### ✔ Set Temporary Password (if password.temporary=true)

User must set new password on first login.

### ✔ Admin Events

Audit logs for:

* User created
* Credentials updated
* Role assigned

---

# **8️⃣ Spring Boot returns final response**

```
201 CREATED
{
  "message": "User registered successfully. Please verify your email.",
  "userId": "0c20c232-8bd0-47f8-8d75-accd272"
}
```

---

# 🟪 **EXTENDED REALISTIC REGISTRATION FLOW (More Detail)**

This is how a production-grade flow looks.

---

# **🔵 Registration Flow Timeline (Detailed)**

```
[Frontend]
   |
   |   1. /auth/register (with form data)
   v
[Spring Boot Auth Service]
   |
   |   2. Input validation
   |   3. Check existing user in Keycloak
   |   4. Create user in Keycloak
   |   5. Assign roles in Keycloak
   |   6. (optional) Trigger email verification
   v
[Keycloak Admin API]
   |
   |   Stores user + password + attributes
   v
[Keycloak Database]
   |
   |   7. Response back to Spring Boot
   v
[Spring Boot]
   |
   |   8. return success to frontend
   v
[Frontend]
```

---

# 🟦 OPTIONAL EXTENSIONS (Many companies use these)

## **1️⃣ OTP-based mobile verification**

Your Spring Boot flow becomes:

```
/auth/register → validate → send OTP → validate OTP → create user in Keycloak
```

## **2️⃣ Capture additional user profile data**

Store in Keycloak attributes:

* address
* profile_image_url
* gender
* dob

Keycloak attributes are flexible key/value pairs.

## **3️⃣ After registration, automatically login user**

Your backend:

1. Creates user in Keycloak
2. Calls Keycloak token endpoint with the credentials
3. Returns JWT to frontend

---

# 🟧 DECISION: What goes in Keycloak vs Your DB?

### Stored in Keycloak:

* Identity info
* Password
* Email
* Roles
* Status
* MFA data
* Login failures

### Stored in your database:

* Order history
* Cart data
* Addresses
* Business-specific details

Identity stays in Keycloak.
Business domain data stays in your services.

---

# 🟩 Summary in one sentence

**User registration = Your Spring Boot service validates the request → calls Keycloak Admin API → creates the user → assigns roles → Keycloak stores identity → Spring Boot returns success.**

---

# If you want, I can also provide:

✔ A complete **sequence diagram**
✔ A full **Spring Boot code sample**
✔ A **Postman collection**
✔ A **microservice architecture diagram showing Auth Service + Keycloak**

Just tell me!


## **Conversation **

### Chat : I have a doubt in context of spring boot security keycloak integration, that if we let keycloack to manage user then we it means we dont have User as entity, or we are not storing this user to DB then, how we fetching this user for authentication if the keycloak is managing user and users are there in keycloak realm?

This is a **very important and common confusion** when integrating Spring Boot with Keycloak. Let’s break it down slowly and clearly.

---

## 🔑 Core Idea (Understand This First)

When you use **Keycloak**, you are **delegating authentication (and optionally user management)** to it.

👉 That means:

* **Keycloak stores users**
* **Keycloak authenticates users**
* **Spring Boot does NOT fetch user from DB for authentication**

---

## 🧠 Then what happens during login?

### Step-by-step flow:

1. User logs in (via UI or API)
2. Request goes to **Keycloak**, NOT your Spring Boot app
3. Keycloak:

   * Verifies username/password from its own DB (realm)
   * If valid → generates **JWT token**
4. That JWT is sent to your Spring Boot app
5. Spring Boot:

   * **DOES NOT query DB**
   * Just **validates JWT signature**
   * Extracts user info from token

---

## 📦 So where is the "User" in Spring Boot?

👉 Important: You usually **do NOT need a User entity for authentication**

Instead, Spring Boot gets user details from JWT:

```json
{
  "sub": "12345",
  "preferred_username": "john",
  "email": "john@example.com",
  "realm_access": {
    "roles": ["USER"]
  }
}
```

Spring Security converts this into:

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
```

---

## 🔍 Then how do we "fetch user"?

### ✅ Option 1: Use JWT directly (Most common)

```java
Jwt jwt = (Jwt) authentication.getPrincipal();
String username = jwt.getClaim("preferred_username");
```

👉 No DB call needed

---

### ✅ Option 2: Create your own User table (Hybrid approach)

Even though Keycloak manages authentication, you may still:

* Store **business-related user data** in your DB:

  * Address
  * Orders
  * Preferences
  * Profile info

Example:

```java
User user = userRepository.findByKeycloakId(jwt.getSubject());
```

👉 Here:

* `keycloakId` = `sub` from JWT

---

### ✅ Option 3: Sync users from Keycloak (Advanced)

* When user registers → you create user in:

  * Keycloak
  * Your DB
* Or sync via Keycloak Admin API

---

## ❗ Important Clarification

👉 **Authentication vs User Data**

| Responsibility            | Who handles it     |
| ------------------------- | ------------------ |
| Login (username/password) | Keycloak           |
| Token generation          | Keycloak           |
| Token validation          | Spring Boot        |
| Business user data        | Your DB (optional) |

---

## 💡 Real-world Example (E-commerce)

Let’s say user logs in:

* Keycloak handles login
* Spring Boot receives JWT

Now:

* To place order → you use `userId` from JWT
* To fetch address → query your DB

---

## ⚠️ Common Mistake

❌ "If Keycloak manages users, we don't need users anywhere else"

✔️ Correct thinking:

* You **don't need user for authentication**
* But you **still need user for business logic**

---

## 🧩 Simple Analogy

Think of Keycloak like:

* 🔐 **Security Guard (Authentication)**

And your DB like:

* 📦 **Customer Data Storage**

Security guard verifies identity
Warehouse stores customer details

---

## 🧠 Final Answer to Your Question

> "Are we not storing users in DB? How are we fetching users?"

👉 Answer:

* You **do NOT fetch users for authentication**
* Spring Boot **trusts JWT from Keycloak**
* You only fetch user from DB **if you need business data**

---

If you want, next I can show you:

✅ Full **Spring Boot + Keycloak + DB user sync flow**
✅ How to design `User` entity with Keycloak ID
✅ Real code example for extracting user from JWT

Just tell me 👍


### Chat : yes please explain all these

Great — now let’s go **deep and practical**, exactly how it works in real projects with **Spring Boot + Keycloak**.

I’ll explain in **3 layers**:

---

# 🧩 1. Overall Architecture (Real Industry Setup)

```
Frontend (Angular)
        ↓
Spring Boot (Resource Server)
        ↓
Keycloak (Auth Server)
        ↓
Your Database (Business Data)
```

---

## 🔑 Responsibilities clearly separated

| Layer       | Responsibility                    |
| ----------- | --------------------------------- |
| Keycloak    | Authentication + user credentials |
| Spring Boot | Authorization + APIs              |
| DB          | Business user data                |

---

# 🧱 2. Designing Your User Entity (IMPORTANT)

Even with Keycloak, you **should still create a User table**

👉 But NOT for password/login

👉 Only for business data

---

## ✅ Example User Entity

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // 🔑 IMPORTANT: Link with Keycloak
    private String keycloakId;   // = sub from JWT

    private String username;
    private String email;

    // Business fields
    private String address;
    private String phoneNumber;

}
```

---

## 🔗 Why `keycloakId`?

Because Keycloak gives:

```json
"sub": "f84c1a2b-xxxx-xxxx"
```

👉 This is your **unique identifier**

---

# 🔄 3. Complete Flow (End-to-End)

Let’s simulate **User Registration + Login + API Call**

---

## 🧑‍💻 STEP 1: User Registration (Custom UI)

👉 You said:

> "We don’t want Keycloak UI"

Yes, this is common.

---

### Flow:

1. User fills form in Angular
2. Request goes to Spring Boot
3. Spring Boot calls **Keycloak Admin API**

---

### 🔧 Create user in Keycloak

```java
Keycloak keycloak = KeycloakBuilder.builder()
    .serverUrl("http://localhost:8080")
    .realm("master")
    .clientId("admin-cli")
    .username("admin")
    .password("admin")
    .build();

UserRepresentation user = new UserRepresentation();
user.setUsername("john");
user.setEmail("john@example.com");
user.setEnabled(true);

Response response = keycloak.realm("my-realm").users().create(user);
```

---

### 🔑 Extract Keycloak ID

```java
String userId = response.getLocation().getPath().replaceAll(".*/([^/]+)$", "$1");
```

---

### 💾 Save in your DB

```java
User appUser = new User();
appUser.setKeycloakId(userId);
appUser.setUsername("john");
appUser.setEmail("john@example.com");

userRepository.save(appUser);
```

---

## 🔐 STEP 2: Login Flow

1. Angular sends credentials → Keycloak `/token` endpoint
2. Keycloak returns:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

---

## 🚀 STEP 3: API Call to Spring Boot

```
GET /orders
Authorization: Bearer <token>
```

---

# 🧠 4. Spring Boot JWT Handling

Spring Boot acts as:

👉 **OAuth2 Resource Server**

---

## 🔧 Config

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2
            .jwt()
        );

    return http.build();
}
```

---

## 🔍 What happens internally?

Spring Boot:

1. Validates JWT using Keycloak public key
2. Extracts claims
3. Creates Authentication object

---

# 📥 5. Extract User in Controller

```java
@GetMapping("/profile")
public String getUser(Authentication authentication) {

    Jwt jwt = (Jwt) authentication.getPrincipal();

    String keycloakId = jwt.getSubject();   // 🔑 main link
    String username = jwt.getClaim("preferred_username");

    return username;
}
```

---

# 🔗 6. Fetch User from DB

```java
User user = userRepository.findByKeycloakId(keycloakId);
```

👉 Now you have:

* Authenticated user ✔️
* Business data ✔️

---

# 🔄 7. Full Flow Summary

```
[Registration]
Angular → Spring Boot → Keycloak
                     → DB

[Login]
Angular → Keycloak → JWT

[API Call]
Angular → Spring Boot → Validate JWT → Extract user → Fetch DB data
```

---

# ⚙️ 8. Advanced: Sync Strategies

### ✅ Option A: On Registration (Best)

* Create user in:

  * Keycloak
  * Your DB

---

### ✅ Option B: Lazy Creation

```java
User user = userRepository.findByKeycloakId(id);

if (user == null) {
    user = new User();
    user.setKeycloakId(id);
    user.setUsername(jwt.getClaim("preferred_username"));
    userRepository.save(user);
}
```

---

### ✅ Option C: Event-based sync (Advanced)

* Use Keycloak events
* Kafka / webhook
* Sync users automatically

---

# ⚠️ 9. Important Things to Remember

### ❌ Do NOT:

* Store passwords in your DB
* Authenticate users yourself

---

### ✅ Always:

* Trust Keycloak for auth
* Use JWT for identity
* Use DB for business logic

---

# 🧠 Final Mental Model

👉 Think like this:

* **Keycloak = Identity Provider (Who are you?)**
* **Spring Boot = Resource Server (What can you do?)**
* **DB = Business System (What do you own?)**

---

# 🚀 If you want next level

I can next show you:

✅ Role-based authorization (from Keycloak → Spring Boot)
✅ How roles are mapped (`ROLE_USER`, `ROLE_ADMIN`)
✅ Full working project structure (Angular + Spring Boot + Keycloak)
✅ Token structure deep dive

Just tell me 👍

## **Conversation **

### Chat : Any disadvantages? Only one: ❗You must secure your backend’s communication with Keycloak Because your microservice must: Have a service account Use the admin API Store a client secret securely (Vault) But this is normal for microservice architecture. so here, what is exactly service account?

Great question — this is a **very important concept** when integrating your Spring Boot microservice with Keycloak.

Let’s break it down simply and practically 👇

---

# 🔐 What is a *Service Account* in Keycloak?

A **service account** is a special account that represents a **machine (your backend service)** instead of a human user.

👉 Think of it like this:

* 👤 Normal user → logs in via UI (username/password)
* 🤖 Service account → used by your **Spring Boot microservice** to talk to Keycloak **programmatically**

---

# 🧠 Why do you need a Service Account?

In your case:

You said:

> "I want to create users in Keycloak from my Spring Boot backend"

To do that, your backend must call:
👉 Keycloak **Admin REST API**

But Keycloak will ask:
❓ “Who are you? Why should I allow this?”

➡️ That’s where **service account authentication** comes in.

---

# 🔄 How it works (Flow)

### Step 1: Create a client in Keycloak

Example:

```
client-id: ecommerce-backend
```

---

### Step 2: Enable Service Account

In Keycloak:

```
Client → Settings → Service Accounts Enabled = ON
```

➡️ This automatically creates:
👉 `service-account-ecommerce-backend`

---

### Step 3: Assign Roles

You give this service account permissions like:

* manage-users
* view-users
* realm-admin (careful with this)

---

### Step 4: Your Spring Boot app authenticates

Your backend sends:

```
POST /realms/{realm}/protocol/openid-connect/token
```

With:

```text
grant_type=client_credentials
client_id=ecommerce-backend
client_secret=xyz
```

---

### Step 5: Keycloak returns Access Token

Now your backend can call:

```
POST /admin/realms/{realm}/users
```

Using that token ✅

---

# 🔑 So what exactly is the Service Account?

👉 In one line:

> A **service account is an identity for your backend service**, used to securely authenticate with Keycloak without a user.

---

# 🧩 Real-world analogy

Think of:

* 👤 User → customer logging into your e-commerce app
* 🤖 Service account → your backend acting like an **admin robot** talking to Keycloak

---

# ⚠️ Important responsibility (this is what your note was about)

Because you're using a service account:

### You MUST secure:

* 🔐 `client_secret` (never hardcode in code)
* 🔐 Store it in:

  * HashiCorp Vault ✅ (best practice)
  * Environment variables
  * Kubernetes secrets

---

# 🚨 Common mistake (important)

Don’t confuse:

* ❌ User authentication (Authorization Code flow)
* ❌ Service account (Client Credentials flow)

They are used for completely different purposes.

---

# 🧠 When do we use Service Accounts?

Use it when your backend needs to:

* Create users
* Reset passwords
* Assign roles
* Manage realms
* Call protected APIs internally

---

# 💡 In your architecture

In your **Spring Boot microservice + Angular app**:

* Angular → authenticates users
* Spring Boot → uses **service account** to:

  * create user in Keycloak
  * manage roles
  * sync user data

---

If you want next step, I can show you:

👉 Full **Spring Boot code** to call Keycloak Admin API using service account
👉 With **RestTemplate / WebClient + token handling + Vault integration**

Just tell me 👍


