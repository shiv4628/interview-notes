# Table of Contents

- [Chat : How to generate custom error response in case of java spring application](#how-to-generate-custom-error-response-in-case-of-java-spring-application)
- [Chat : Yes](#yes)

## **Conversation**

<a id="how-to-generate-custom-error-response-in-case-of-java-spring-application"></a>
### Chat : How to generate custom error response in case of java spring application 

In a Java Spring application, you can generate a **custom error response** by using the `@ControllerAdvice` and `@ExceptionHandler` annotations to globally handle exceptions and return a standardized error response. This is useful for REST APIs when you want to return structured JSON error messages.

---

### ✅ Step-by-Step: Creating Custom Error Response

#### 1. **Create a Custom Error Response Class**

```java
public class ErrorResponse {
    private int status;
    private String message;
    private long timestamp;

    public ErrorResponse(int status, String message, long timestamp) {
        this.status = status;
        this.message = message;
        this.timestamp = timestamp;
    }

    // Getters and Setters
}
```

---

#### 2. **Create a Global Exception Handler with `@ControllerAdvice`**

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(value = Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    // Example: Handling specific exception
    @ExceptionHandler(value = IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(IllegalArgumentException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```

---

#### 3. **Throw Exceptions in Your Controller/Service**

```java
@GetMapping("/test")
public ResponseEntity<String> testEndpoint() {
    throw new IllegalArgumentException("Invalid input provided!");
}
```

---

### ✅ Sample Output (JSON)

```json
{
  "status": 400,
  "message": "Invalid input provided!",
  "timestamp": 1716862749321
}
```

---

### 🔄 Optional: Define Custom Exceptions

You can create your own exception class for specific use cases:

```java
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

And then handle it:

```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
    ErrorResponse error = new ErrorResponse(
        HttpStatus.NOT_FOUND.value(),
        ex.getMessage(),
        System.currentTimeMillis()
    );
    return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
}
```

---

Let me know if you'd like to include error codes, sub-errors, or use `@RestControllerAdvice` (which is the same as `@ControllerAdvice` + `@ResponseBody`).


<a id="yes"></a>
### Chat : Yes

Great! Let’s enhance the solution with the following improvements:

### ✅ Goals:

* Use `@RestControllerAdvice` instead of `@ControllerAdvice`
* Include:

  * **HTTP status code**
  * **Custom error code**
  * **Error message**
  * **Timestamp**
  * **Sub-errors (optional field for validation errors or nested issues)**

---

### ✅ 1. Enhanced `ErrorResponse` Class with Sub-errors and Error Code

```java
import java.time.LocalDateTime;
import java.util.List;

public class ErrorResponse {
    private int status;
    private String errorCode;
    private String message;
    private LocalDateTime timestamp;
    private List<String> subErrors;

    public ErrorResponse(int status, String errorCode, String message, LocalDateTime timestamp, List<String> subErrors) {
        this.status = status;
        this.errorCode = errorCode;
        this.message = message;
        this.timestamp = timestamp;
        this.subErrors = subErrors;
    }

    // Getters and Setters
}
```

---

### ✅ 2. Custom Exceptions Example

```java
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

---

### ✅ 3. Global Exception Handler Using `@RestControllerAdvice`

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.List;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            "USER_NOT_FOUND",
            ex.getMessage(),
            LocalDateTime.now(),
            null
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleBadRequest(IllegalArgumentException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "INVALID_ARGUMENT",
            ex.getMessage(),
            LocalDateTime.now(),
            null
        );
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "INTERNAL_ERROR",
            ex.getMessage(),
            LocalDateTime.now(),
            null
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

---

### ✅ 4. Example Controller Usage

```java
@GetMapping("/users/{id}")
public ResponseEntity<String> getUser(@PathVariable String id) {
    if (id.equals("0")) {
        throw new UserNotFoundException("User with ID 0 not found");
    } else if (id.equals("abc")) {
        throw new IllegalArgumentException("ID must be numeric");
    }
    return ResponseEntity.ok("User details for ID " + id);
}
```

---

### ✅ Sample JSON Output

```json
{
  "status": 404,
  "errorCode": "USER_NOT_FOUND",
  "message": "User with ID 0 not found",
  "timestamp": "2025-05-28T15:10:30.345",
  "subErrors": null
}
```

---

Would you like to handle **validation errors (e.g., `@Valid`)** with automatic sub-errors in `subErrors`?

