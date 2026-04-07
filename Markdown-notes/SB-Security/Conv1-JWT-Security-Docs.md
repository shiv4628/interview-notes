# Spring Boot JWT Security Documentation

## Overview
This document provides a concise guide to implementing JWT (JSON Web Tokens) authentication in Spring Boot applications. JWT is used for secure, stateless authentication in RESTful APIs.

## Dependencies
Add these to your `pom.xml` (Maven) or `build.gradle` (Gradle):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

## JWT Token Utility (`JwtTokenUtil.java`)
Utility class for generating and validating JWT tokens using JJWT library.

```java
import io.jsonwebtoken.*;
import org.springframework.stereotype.Component;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtTokenUtil {
    private String secret = "your-secret-key"; // Use a secure key

    public String getUsernameFromToken(String token) {
        return getClaimFromToken(token, Claims::getSubject);
    }

    public Date getExpirationDateFromToken(String token) {
        return getClaimFromToken(token, Claims::getExpiration);
    }

    public <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaimsFromToken(token);
        return claimsResolver.apply(claims);
    }

    private Claims getAllClaimsFromToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }

    private Boolean isTokenExpired(String token) {
        final Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }

    public String generateToken(String username) {
        Map<String, Object> claims = new HashMap<>();
        return doGenerateToken(claims, username);
    }

    private String doGenerateToken(Map<String, Object> claims, String subject) {
        return Jwts.builder().setClaims(claims).setSubject(subject).setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 10)) // 10 hours
                .signWith(SignatureAlgorithm.HS512, secret).compact();
    }

    public Boolean validateToken(String token, String username) {
        final String tokenUsername = getUsernameFromToken(token);
        return (tokenUsername.equals(username) && !isTokenExpired(token));
    }
}
```

## User Details Service (`JwtUserDetailsService.java`)
Custom `UserDetailsService` for loading user data during authentication.

```java
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import java.util.ArrayList;

@Service
public class JwtUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        if ("user".equals(username)) {
            return new User("user", "$2a$10$Dow2Epv.TB9r3zNYYPbR7uF9PmiNzGvWiO3DZvxoBsHn9r.gFAG3C", new ArrayList<>());
            // Password: password (BCrypt encoded)
        } else {
            throw new UsernameNotFoundException("User not found with username: " + username);
        }
    }
}
```

## Authentication Controller (`JwtAuthenticationController.java`)
Endpoint for user login and JWT token generation.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;

@RestController
@CrossOrigin
public class JwtAuthenticationController {
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    @Autowired
    private JwtUserDetailsService userDetailsService;

    @PostMapping("/authenticate")
    public ResponseEntity<?> createAuthenticationToken(@RequestBody JwtRequest authenticationRequest) throws Exception {
        try {
            authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(authenticationRequest.getUsername(), authenticationRequest.getPassword()));
        } catch (AuthenticationException e) {
            throw new Exception("INVALID_CREDENTIALS", e);
        }

        final UserDetails userDetails = userDetailsService.loadUserByUsername(authenticationRequest.getUsername());
        final String token = jwtTokenUtil.generateToken(userDetails.getUsername());
        return ResponseEntity.ok(new JwtResponse(token));
    }
}
```

## Request and Response Models

### `JwtRequest.java`
```java
public class JwtRequest {
    private String username;
    private String password;
    // Getters and setters
}
```

### `JwtResponse.java`
```java
public class JwtResponse {
    private final String token;
    public JwtResponse(String token) {
        this.token = token;
    }
    public String getToken() {
        return token;
    }
}
```

## JWT Concepts

### Claims
Claims are pieces of information stored in the JWT payload. Types:
- **Registered Claims**: Standard claims like `sub` (subject), `iss` (issuer), `exp` (expiration), `iat` (issued at), `aud` (audience).
- **Public Claims**: Custom claims that are collision-resistant.
- **Private Claims**: Custom claims agreed between parties.

Example JWT payload:
```json
{
  "sub": "user123",
  "iss": "auth-server",
  "exp": 1707800000,
  "role": "ADMIN"
}
```

### Parsing
Parsing a JWT involves decoding the token, extracting claims, and verifying the signature. In JJWT:
```java
private Claims getAllClaimsFromToken(String token) {
    return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
}
```
This extracts the payload containing all claims.

### Subject (`sub`)
The subject claim identifies the user/entity the token is issued for. Typically a username, email, or user ID. Extracted using `Claims::getSubject`.

### `getClaimFromToken` Method
Generic method to extract specific claims:
```java
public <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
    final Claims claims = getAllClaimsFromToken(token);
    return claimsResolver.apply(claims);
}
```
Example: `getClaimFromToken(token, Claims::getSubject)` extracts the username.

## ResponseEntity

### `ResponseEntity<T>` vs `ResponseEntity<?>`
- `ResponseEntity<T>`: Specific type (e.g., `ResponseEntity<String>`) for type-safe responses.
- `ResponseEntity<?>`: Wildcard allowing any type, useful for dynamic responses where the exact type varies.

Example:
```java
public ResponseEntity<?> getData(boolean isUser) {
    if (isUser) {
        return ResponseEntity.ok(new User("John", "john@example.com"));
    } else {
        return ResponseEntity.ok("No user found");
    }
}
```

## Testing
Use Postman or curl:
1. POST credentials to `/authenticate` to get JWT token.
2. Include token in `Authorization: Bearer <token>` header for secured endpoints.

## Security Configuration
Configure Spring Security to use JWT for authentication, defining protected endpoints and authentication filters (not shown in detail here).

## Best Practices
- Use strong, unique secret keys.
- Set appropriate token expiration times.
- Implement refresh token mechanisms for long-term sessions.
- Validate tokens on each request.
- Use HTTPS in production.