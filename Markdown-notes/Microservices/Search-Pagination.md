## **Conversation**

### Chat : how to implement search feature in a spring boot micro-service e-commerce app

Implementing a search feature in a **Spring Boot microservices-based e-commerce app** involves several steps, depending on the type of search you want to provide (basic, full-text, or advanced with filters). Below is a **step-by-step approach**:

---

## **1. Define Search Requirements**

* **Basic search:** Search by product name, description, or category.
* **Advanced search:** Use filters like price range, brand, and ratings.
* **Full-text search:** Implement search indexing for efficient searching.

---

## **2. Choose a Search Approach**

* **Database Query Search (SQL/JPQL)** – Simple and fast for small datasets.
* **Elasticsearch Search** – Recommended for large datasets and full-text search.
* **Custom Search Algorithm (Lucene, Solr)** – For complex requirements.

---

## **3. Implement Search Using SQL (Basic Approach)**

### **3.1 Define Entity (Product)**

```java
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
    private String category;
    private Double price;

    // Getters and Setters
}
```

### **3.2 Create Repository with Search Query**

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {

    // Case-insensitive search by product name or category
    @Query("SELECT p FROM Product p WHERE LOWER(p.name) LIKE LOWER(CONCAT('%', :keyword, '%')) " +
           "OR LOWER(p.category) LIKE LOWER(CONCAT('%', :keyword, '%'))")
    List<Product> searchProducts(@Param("keyword") String keyword);
}
```

### **3.3 Implement Search Service**

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<Product> searchProducts(String keyword) {
        return productRepository.searchProducts(keyword);
    }
}
```

### **3.4 Expose API in Controller**

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping("/search")
    public ResponseEntity<List<Product>> searchProducts(@RequestParam String keyword) {
        return ResponseEntity.ok(productService.searchProducts(keyword));
    }
}
```

---

## **4. Implement Search Using Elasticsearch (Advanced)**

Elasticsearch is recommended for high-performance, full-text search.

### **4.1 Add Dependencies**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

### **4.2 Configure Elasticsearch**

```yaml
spring:
  elasticsearch:
    uris: http://localhost:9200
```

### **4.3 Create Elasticsearch Entity**

```java
@Document(indexName = "products")
public class Product {

    @Id
    private String id;

    private String name;
    private String description;
    private String category;
    private Double price;
}
```

### **4.4 Elasticsearch Repository**

```java
@Repository
public interface ProductElasticRepository extends ElasticsearchRepository<Product, String> {

    List<Product> findByNameContainingOrCategoryContaining(String name, String category);
}
```

### **4.5 Elasticsearch Search Service**

```java
@Service
public class ProductElasticService {

    @Autowired
    private ProductElasticRepository productElasticRepository;

    public List<Product> search(String keyword) {
        return productElasticRepository.findByNameContainingOrCategoryContaining(keyword, keyword);
    }
}
```

### **4.6 Elasticsearch API Controller**

```java
@RestController
@RequestMapping("/api/products")
public class ProductElasticController {

    @Autowired
    private ProductElasticService productElasticService;

    @GetMapping("/search")
    public ResponseEntity<List<Product>> search(@RequestParam String keyword) {
        return ResponseEntity.ok(productElasticService.search(keyword));
    }
}
```

---

## **5. Enhancements**

### **5.1 Pagination & Sorting**

Modify repository methods to support pagination:

```java
Page<Product> findByNameContaining(String name, Pageable pageable);
```

Example API:

```java
@GetMapping("/search")
public Page<Product> search(@RequestParam String keyword, Pageable pageable) {
    return productRepository.findByNameContaining(keyword, pageable);
}
```

### **5.2 Filtering (Category, Price Range)**

Modify query:

```java
@Query("SELECT p FROM Product p WHERE " +
       "(LOWER(p.name) LIKE LOWER(CONCAT('%', :keyword, '%')) " +
       "OR LOWER(p.category) LIKE LOWER(CONCAT('%', :keyword, '%'))) " +
       "AND p.price BETWEEN :minPrice AND :maxPrice")
List<Product> searchProducts(@Param("keyword") String keyword,
                             @Param("minPrice") Double minPrice,
                             @Param("maxPrice") Double maxPrice);
```

Use:

```java
public List<Product> search(String keyword, Double minPrice, Double maxPrice) {
    return productRepository.searchProducts(keyword, minPrice, maxPrice);
}
```

---

## **6. Deploy & Test**

* Test API using **Postman**:

  ```
  GET /api/products/search?keyword=laptop
  ```
* Use Elasticsearch **Kibana** for monitoring.

---

## **Conclusion**

| Approach                  | Best for                         | Pros                      | Cons                      |
| ------------------------- | -------------------------------- | ------------------------- | ------------------------- |
| **SQL Search**            | Small to medium datasets         | Simple, easy to implement | Slow for large datasets   |
| **Elasticsearch**         | Large datasets, full-text search | Fast, scalable            | Requires additional setup |
| **Custom (Lucene, Solr)** | Complex needs                    | Highly customizable       | Harder to maintain        |

For **basic search**, **SQL queries** work well. For **large-scale e-commerce**, use **Elasticsearch** for faster performance.

Do you want help with setting up Elasticsearch or additional filtering features? 🚀


### Chat : Yes

Great! Let's set up **Elasticsearch** for your Spring Boot microservices-based e-commerce app. 🚀

---

# **1. Install and Configure Elasticsearch**

### **1.1 Install Elasticsearch (Locally)**

1. **Download and Install**:

   * [Download Elasticsearch](https://www.elastic.co/downloads/elasticsearch)
   * Extract and run:

     ```sh
     bin/elasticsearch
     ```
   * Default port: **9200**

2. **Verify Installation**
   Run in the browser or use **cURL**:

   ```sh
   curl -X GET "http://localhost:9200/"
   ```

   If Elasticsearch is running, it will return a JSON response.

---

# **2. Add Elasticsearch to Spring Boot**

### **2.1 Add Dependencies (Spring Boot + Elasticsearch)**

In `pom.xml`, add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

---

# **3. Configure Elasticsearch in Spring Boot**

### **3.1 Application Properties**

Add these settings in `application.yml`:

```yaml
spring:
  elasticsearch:
    uris: http://localhost:9200
  data:
    elasticsearch:
      repositories:
        enabled: true
```

---

# **4. Define Elasticsearch Entity**

Elasticsearch uses **documents** instead of relational tables.

### **4.1 Create a Product Entity**

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;

@Document(indexName = "products") // Index name in Elasticsearch
public class Product {

    @Id
    private String id;
    private String name;
    private String description;
    private String category;
    private Double price;
    
    // Constructors, Getters & Setters
}
```

---

# **5. Implement Elasticsearch Repository**

Spring Boot provides **Spring Data Elasticsearch** to simplify queries.

### **5.1 Create Repository**

```java
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface ProductElasticRepository extends ElasticsearchRepository<Product, String> {

    List<Product> findByNameContainingOrCategoryContaining(String name, String category);
}
```

---

# **6. Implement Search Service**

### **6.1 Create Product Search Service**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class ProductElasticService {

    @Autowired
    private ProductElasticRepository productElasticRepository;

    public List<Product> search(String keyword) {
        return productElasticRepository.findByNameContainingOrCategoryContaining(keyword, keyword);
    }
}
```

---

# **7. Create Search API in Controller**

### **7.1 Expose Search Endpoint**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductElasticController {

    @Autowired
    private ProductElasticService productElasticService;

    @GetMapping("/search")
    public ResponseEntity<List<Product>> search(@RequestParam String keyword) {
        return ResponseEntity.ok(productElasticService.search(keyword));
    }
}
```

---

# **8. Adding Filtering (Price Range, Sorting)**

Now, let's enhance the search with **filters** and **sorting**.

### **8.1 Modify Repository for Price Range**

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.annotations.Query;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

public interface ProductElasticRepository extends ElasticsearchRepository<Product, String> {

    @Query("{\"bool\": {\"must\": [ {\"multi_match\": {\"query\": \"?0\", \"fields\": [\"name\", \"category\"]}} ], \"filter\": [{\"range\": {\"price\": {\"gte\": ?1, \"lte\": ?2}}}]}}")
    Page<Product> searchWithFilter(String keyword, Double minPrice, Double maxPrice, Pageable pageable);
}
```

---

### **8.2 Modify Service to Handle Filters**

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class ProductElasticService {

    @Autowired
    private ProductElasticRepository productElasticRepository;

    public Page<Product> searchWithFilters(String keyword, Double minPrice, Double maxPrice, int page, int size) {
        return productElasticRepository.searchWithFilter(keyword, minPrice, maxPrice, PageRequest.of(page, size));
    }
}
```

---

### **8.3 Update Controller for Filtering**

```java
import org.springframework.data.domain.Page;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/products")
public class ProductElasticController {

    @Autowired
    private ProductElasticService productElasticService;

    @GetMapping("/search")
    public ResponseEntity<Page<Product>> search(
            @RequestParam String keyword,
            @RequestParam(required = false, defaultValue = "0") Double minPrice,
            @RequestParam(required = false, defaultValue = "100000") Double maxPrice,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {

        return ResponseEntity.ok(productElasticService.searchWithFilters(keyword, minPrice, maxPrice, page, size));
    }
}
```

---

# **9. Testing the Search API**

### **9.1 Test Basic Search**

Use **Postman** or **cURL**:

```sh
curl -X GET "http://localhost:8080/api/products/search?keyword=laptop"
```

### **9.2 Test Search with Price Filters**

```sh
curl -X GET "http://localhost:8080/api/products/search?keyword=laptop&minPrice=500&maxPrice=1500&page=0&size=10"
```

---

# **10. Deploy Elasticsearch in Docker (Optional)**

If you want to run **Elasticsearch in Docker**, use:

```sh
docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.0.0
```

---

# **Summary**

✅ **Basic Search** – Find products by name/category
✅ **Full-Text Search** – Faster search using Elasticsearch
✅ **Filters** – Filter by price range
✅ **Pagination** – Fetch results in pages

Would you like me to help with **adding sorting (e.g., price low to high)** or any other enhancements? 🚀

## **Conversation**

### Chat : Implement a basic ecommerce spring boot microservice application with filtering and search functionality 

Here’s a step-by-step guide to implement a basic e-commerce Spring Boot microservice application with filtering and search functionality:

---

### 1. **Project Setup**

* Use **Spring Initializr** to generate a Spring Boot project with dependencies:

  * **Spring Web**
  * **Spring Data JPA**
  * **H2 Database** (or MySQL)
  * **Spring Boot DevTools**
  * **Lombok**
  * **Spring Boot Actuator**

---

### 2. **Define the Project Structure**

```
src/main/java/com/ecommerce
    ├── controller/
    │       ├── ProductController.java
    ├── model/
    │       ├── Product.java
    ├── repository/
    │       ├── ProductRepository.java
    ├── service/
    │       ├── ProductService.java
    ├── EcommerceApplication.java
```

---

### 3. **Entity Class**

Define the `Product` entity with fields for search and filtering.

```java
package com.ecommerce.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String category;
    private String brand;
    private Double price;
}
```

---

### 4. **Repository**

Extend Spring Data JPA `JpaRepository` for database operations.

```java
package com.ecommerce.repository;

import com.ecommerce.model.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface ProductRepository extends JpaRepository<Product, Long> {
    
    // Search by name containing a keyword (case insensitive)
    List<Product> findByNameContainingIgnoreCase(String keyword);

    // Filter by category and price range
    @Query("SELECT p FROM Product p WHERE p.category = :category AND p.price BETWEEN :minPrice AND :maxPrice")
    List<Product> filterByCategoryAndPriceRange(
            @Param("category") String category,
            @Param("minPrice") Double minPrice,
            @Param("maxPrice") Double maxPrice);
}
```

---

### 5. **Service**

Encapsulate the business logic in the `ProductService` class.

```java
package com.ecommerce.service;

import com.ecommerce.model.Product;
import com.ecommerce.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }

    public List<Product> searchProducts(String keyword) {
        return productRepository.findByNameContainingIgnoreCase(keyword);
    }

    public List<Product> filterProducts(String category, Double minPrice, Double maxPrice) {
        return productRepository.filterByCategoryAndPriceRange(category, minPrice, maxPrice);
    }
}
```

---

### 6. **Controller**

Expose APIs to handle search and filtering.

```java
package com.ecommerce.controller;

import com.ecommerce.model.Product;
import com.ecommerce.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }

    @GetMapping("/search")
    public List<Product> searchProducts(@RequestParam String keyword) {
        return productService.searchProducts(keyword);
    }

    @GetMapping("/filter")
    public List<Product> filterProducts(
            @RequestParam String category,
            @RequestParam Double minPrice,
            @RequestParam Double maxPrice) {
        return productService.filterProducts(category, minPrice, maxPrice);
    }
}
```

---

### 7. **Application Properties**

Configure the H2 database or your preferred database.

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
spring.h2.console.enabled=true
```

---

### 8. **Sample Data Initialization**

Add some sample data using a `data.sql` file in `src/main/resources`.

```sql
INSERT INTO PRODUCT (NAME, CATEGORY, BRAND, PRICE) VALUES 
('Laptop', 'Electronics', 'Dell', 800.00),
('Smartphone', 'Electronics', 'Samsung', 500.00),
('Chair', 'Furniture', 'Ikea', 150.00),
('Table', 'Furniture', 'Ikea', 250.00);
```

---

### 9. **Testing the APIs**

Use **Postman** or any API testing tool to test these endpoints:

* `GET /api/products` - Fetch all products.
* `GET /api/products/search?keyword=laptop` - Search products by name.
* `GET /api/products/filter?category=Furniture&minPrice=100&maxPrice=200` - Filter products by category and price range.

---

### 10. **Extend Functionality**

You can extend this application by:

* Adding pagination and sorting.
* Implementing user authentication and roles.
* Integrating a front-end UI using React/Angular.
* Using a cloud database like AWS RDS or Google Cloud SQL for production.

Let me know if you want code for additional features!


### Chat : implement paging sorting and filtering implement user authentication and roles using jwt

Here’s an extended implementation of your e-commerce Spring Boot microservice to include **pagination, sorting, filtering**, and **user authentication and roles using JWT**.

---

## 1. **Enhance Pagination, Sorting, and Filtering**

### Update the Repository

Add a method to use **pagination** and **sorting** for filtering.

```java
package com.ecommerce.repository;

import com.ecommerce.model.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

public interface ProductRepository extends JpaRepository<Product, Long>, JpaSpecificationExecutor<Product> {
}
```

---

### Create a `Specification` for Dynamic Filtering

Use Spring Data JPA's `Specification` to support dynamic filtering.

```java
package com.ecommerce.specification;

import com.ecommerce.model.Product;
import org.springframework.data.jpa.domain.Specification;

public class ProductSpecification {

    public static Specification<Product> categoryEquals(String category) {
        return (root, query, criteriaBuilder) ->
                category == null ? null : criteriaBuilder.equal(root.get("category"), category);
    }

    public static Specification<Product> priceBetween(Double minPrice, Double maxPrice) {
        return (root, query, criteriaBuilder) -> {
            if (minPrice == null || maxPrice == null) return null;
            return criteriaBuilder.between(root.get("price"), minPrice, maxPrice);
        };
    }

    public static Specification<Product> nameContains(String keyword) {
        return (root, query, criteriaBuilder) ->
                keyword == null ? null : criteriaBuilder.like(criteriaBuilder.lower(root.get("name")), "%" + keyword.toLowerCase() + "%");
    }
}
```

---

### Update the Service

Integrate pagination, sorting, and filtering into the service layer.

```java
package com.ecommerce.service;

import com.ecommerce.model.Product;
import com.ecommerce.repository.ProductRepository;
import com.ecommerce.specification.ProductSpecification;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Page<Product> getProducts(String category, Double minPrice, Double maxPrice, String keyword, int page, int size, String sortField, String sortDirection) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(Sort.Direction.fromString(sortDirection), sortField));
        Specification<Product> spec = Specification
                .where(ProductSpecification.categoryEquals(category))
                .and(ProductSpecification.priceBetween(minPrice, maxPrice))
                .and(ProductSpecification.nameContains(keyword));
        return productRepository.findAll(spec, pageable);
    }
}
```

---

### Update the Controller

Add a new endpoint for fetching paginated, sorted, and filtered results.

```java
package com.ecommerce.controller;

import com.ecommerce.model.Product;
import com.ecommerce.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public Page<Product> getProducts(
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice,
            @RequestParam(required = false) String keyword,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortField,
            @RequestParam(defaultValue = "ASC") String sortDirection) {
        return productService.getProducts(category, minPrice, maxPrice, keyword, page, size, sortField, sortDirection);
    }
}
```

---

## 2. **Implement User Authentication and Roles Using JWT**

### Add Dependencies

Add the required dependencies for JWT in `pom.xml`.

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

---

### Define User Entity

```java
package com.ecommerce.model;

import jakarta.persistence.*;
import lombok.Data;

import java.util.Set;

@Entity
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;

    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "role")
    private Set<String> roles;
}
```

---

### Add JWT Utility

```java
package com.ecommerce.util;

import io.jsonwebtoken.*;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.Set;

@Component
public class JwtUtil {

    private final String SECRET_KEY = "your_secret_key";

    public String generateToken(String username, Set<String> roles) {
        return Jwts.builder()
                .setSubject(username)
                .claim("roles", roles)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 10)) // 10 hours validity
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                .compact();
    }

    public String extractUsername(String token) {
        return Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}
```

---

### Implement Security Configuration

```java
package com.ecommerce.config;

import com.ecommerce.util.JwtUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Autowired
    private JwtUtil jwtUtil;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .antMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated();
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(HttpSecurity http, PasswordEncoder passwordEncoder)
            throws Exception {
        return http.getSharedObject(AuthenticationManagerBuilder.class)
                .inMemoryAuthentication()
                .withUser("admin")
                .password(passwordEncoder.encode("password"))
                .roles("ADMIN")
                .and()
                .withUser("user")
                .password(passwordEncoder.encode("password"))
                .roles("USER")
                .and()
                .and()
                .build();
    }
}
```

---

Let me know if you’d like me to expand on any section or provide additional features!



