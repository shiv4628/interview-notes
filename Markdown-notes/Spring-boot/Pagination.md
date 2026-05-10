# Table of Contents

- [Chat : Pagination and sorting in spring boot application](#pagination-and-sorting-in-spring-boot-application)
- [Chat : Yes I want some more advanced example such as integration with a front end and adding filters](#yes-i-want-some-more-advanced-example-such-as-integration-with-a-front-end-and-adding-filters)
- [Chat : How can we use custom queries](#how-can-we-use-custom-queries)
- [Chat : Chat](#chat)
- [Chat : Chat](#chat-1)
- [Chat : Chat](#chat-2)
- [Chat : so basically which one is recommended?](#so-basically-which-one-is-recommended)
- [Chat : yes please](#yes-please)
- [Chat : can we go with hybrid approach also?](#can-we-go-with-hybrid-approach-also)
- [Chat : how to implement in spring boot with hybrid approach?](#how-to-implement-in-spring-boot-with-hybrid-approach)
- [Chat : how can we implement search functionality in spring boot app using mongoDB, also include pagination and filtering](#how-can-we-implement-search-functionality-in-spring-boot-app-using-mongodb-also-include-pagination-and-filtering)
- [Chat : what is MongoTemplate ?](#what-is-mongotemplate)

## **Conversation**

<a id="pagination-and-sorting-in-spring-boot-application"></a>
### Chat : Pagination and sorting in spring boot application 

Pagination and sorting are commonly used in Spring Boot applications to handle large datasets effectively. You can implement them using **Spring Data JPA** and the `Pageable` and `Sort` interfaces. Here's how you can do it:

---

### 1. **Add Dependencies**

Ensure you have the necessary dependencies in your `pom.xml` (for Maven):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

---

### 2. **Entity Class**

Define your entity class (e.g., `Product`):

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private Double price;

    // Getters and Setters
}
```

---

### 3. **Repository Interface**

Extend `JpaRepository` to enable pagination and sorting:

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

---

### 4. **Service Layer**

Create a service to implement pagination and sorting logic:

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Page<Product> getProducts(int page, int size, String sortBy, String sortDir) {
        Sort sort = sortDir.equalsIgnoreCase("asc") ? Sort.by(sortBy).ascending() : Sort.by(sortBy).descending();
        Pageable pageable = PageRequest.of(page, size, sort);
        return productRepository.findAll(pageable);
    }
}
```

---

### 5. **Controller Layer**

Create a REST controller to expose the API:

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public Page<Product> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "asc") String sortDir) {
        return productService.getProducts(page, size, sortBy, sortDir);
    }
}
```

---

### 6. **Test the API**

You can test the API using tools like **Postman** or **cURL**.

Example Request:

```
GET /products?page=0&size=5&sortBy=price&sortDir=desc
```

---

### 7. **Response**

The response includes both the data and metadata about pagination:

```json
{
  "content": [
    { "id": 1, "name": "Product A", "price": 50.0 },
    { "id": 2, "name": "Product B", "price": 40.0 }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 5
  },
  "totalPages": 2,
  "totalElements": 10,
  "last": false,
  "first": true,
  "sort": {
    "sorted": true,
    "unsorted": false,
    "empty": false
  }
}
```

---

### Key Notes:

* **Sorting**: `Sort.by()` allows multi-field sorting if needed, e.g., `Sort.by("price", "name").ascending()`.
* **Pagination**: `PageRequest.of(page, size)` is the standard method for creating a pageable instance.
* **Custom Queries**: You can use custom JPQL queries with pagination by using `@Query` and `Pageable`.

Would you like a more advanced example, such as integrating with a frontend or adding custom filters?


<a id="yes-i-want-some-more-advanced-example-such-as-integration-with-a-front-end-and-adding-filters"></a>
### Chat : Yes I want some more advanced example such as integration with a front end and adding filters

To build a more advanced Spring Boot application with **pagination**, **sorting**, and **filters**, and integrate it with a **frontend**, here's how you can proceed:

---

### 1. **Enhance the Backend**

#### A. **Custom Filters**

Modify the repository or create a custom implementation for filtering data dynamically.

**Entity Class (e.g., Product):**

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private Double price;
    private String category;

    // Getters and Setters
}
```

**Custom Specification for Filtering:**
Use the `Specification` interface from **Spring Data JPA**.

```java
import org.springframework.data.jpa.domain.Specification;

public class ProductSpecification {
    public static Specification<Product> hasCategory(String category) {
        return (root, query, criteriaBuilder) -> 
            category == null ? null : criteriaBuilder.equal(root.get("category"), category);
    }

    public static Specification<Product> hasNameLike(String name) {
        return (root, query, criteriaBuilder) -> 
            name == null ? null : criteriaBuilder.like(root.get("name"), "%" + name + "%");
    }

    public static Specification<Product> hasPriceBetween(Double minPrice, Double maxPrice) {
        return (root, query, criteriaBuilder) -> {
            if (minPrice == null && maxPrice == null) return null;
            if (minPrice != null && maxPrice != null) {
                return criteriaBuilder.between(root.get("price"), minPrice, maxPrice);
            }
            if (minPrice != null) {
                return criteriaBuilder.greaterThanOrEqualTo(root.get("price"), minPrice);
            }
            return criteriaBuilder.lessThanOrEqualTo(root.get("price"), maxPrice);
        };
    }
}
```

**Repository with Specifications:**

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

public interface ProductRepository extends JpaRepository<Product, Long>, JpaSpecificationExecutor<Product> {
}
```

#### B. **Service with Combined Filters**

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Page<Product> getFilteredProducts(
            String name, String category, Double minPrice, Double maxPrice,
            int page, int size, String sortBy, String sortDir) {

        Sort sort = sortDir.equalsIgnoreCase("asc") ? Sort.by(sortBy).ascending() : Sort.by(sortBy).descending();
        Pageable pageable = PageRequest.of(page, size, sort);

        return productRepository.findAll(
            Specification.where(ProductSpecification.hasNameLike(name))
                          .and(ProductSpecification.hasCategory(category))
                          .and(ProductSpecification.hasPriceBetween(minPrice, maxPrice)),
            pageable
        );
    }
}
```

#### C. **Controller with Query Parameters**

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public Page<Product> getFilteredProducts(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "asc") String sortDir) {
        return productService.getFilteredProducts(name, category, minPrice, maxPrice, page, size, sortBy, sortDir);
    }
}
```

---

### 2. **Frontend Integration**

#### A. **Frontend Framework**

Use **React.js** (or any modern framework like Angular or Vue.js) for the frontend.

**Install Axios for API Calls:**

```bash
npm install axios
```

#### B. **React Component Example**

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const ProductList = () => {
    const [products, setProducts] = useState([]);
    const [page, setPage] = useState(0);
    const [size, setSize] = useState(10);
    const [sortBy, setSortBy] = useState('id');
    const [sortDir, setSortDir] = useState('asc');
    const [filters, setFilters] = useState({ name: '', category: '', minPrice: '', maxPrice: '' });

    useEffect(() => {
        fetchProducts();
    }, [page, size, sortBy, sortDir, filters]);

    const fetchProducts = async () => {
        const { name, category, minPrice, maxPrice } = filters;
        const response = await axios.get('http://localhost:8080/products', {
            params: {
                name,
                category,
                minPrice,
                maxPrice,
                page,
                size,
                sortBy,
                sortDir,
            },
        });
        setProducts(response.data.content);
    };

    return (
        <div>
            <h1>Product List</h1>

            {/* Filters */}
            <div>
                <input
                    type="text"
                    placeholder="Name"
                    value={filters.name}
                    onChange={(e) => setFilters({ ...filters, name: e.target.value })}
                />
                <input
                    type="text"
                    placeholder="Category"
                    value={filters.category}
                    onChange={(e) => setFilters({ ...filters, category: e.target.value })}
                />
                <input
                    type="number"
                    placeholder="Min Price"
                    value={filters.minPrice}
                    onChange={(e) => setFilters({ ...filters, minPrice: e.target.value })}
                />
                <input
                    type="number"
                    placeholder="Max Price"
                    value={filters.maxPrice}
                    onChange={(e) => setFilters({ ...filters, maxPrice: e.target.value })}
                />
            </div>

            {/* Table */}
            <table>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Category</th>
                        <th>Price</th>
                    </tr>
                </thead>
                <tbody>
                    {products.map((product) => (
                        <tr key={product.id}>
                            <td>{product.name}</td>
                            <td>{product.category}</td>
                            <td>{product.price}</td>
                        </tr>
                    ))}
                </tbody>
            </table>

            {/* Pagination Controls */}
            <div>
                <button onClick={() => setPage(page - 1)} disabled={page === 0}>
                    Previous
                </button>
                <button onClick={() => setPage(page + 1)}>Next</button>
            </div>
        </div>
    );
};

export default ProductList;
```

---

### 3. **Running the Application**

* **Backend**: Run the Spring Boot application (`ProductController` endpoint is exposed on `http://localhost:8080/products`).
* **Frontend**: Run the React app (`http://localhost:3000`).

---

### 4. **Future Enhancements**

* **JWT Authentication** for secure APIs.
* **Caching** (e.g., Redis) to optimize frequently accessed data.
* **UI Improvements** using a library like **Material-UI** or **Bootstrap**.
* Export filtered results to **CSV/Excel**.

Would you like details about implementing any of these additional features?


<a id="how-can-we-use-custom-queries"></a>
### Chat : How can we use custom queries 

In **Spring Data JPA**, you can define **custom queries** using the `@Query` annotation or by implementing custom repository methods. Here's a detailed guide to using custom queries in your Spring Boot application:

---

### **1. Using the `@Query` Annotation**

The `@Query` annotation allows you to define custom JPQL or native SQL queries in your repository.

#### Example: Define Custom Queries for Filtering

**Repository Interface:**

```java
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface ProductRepository extends JpaRepository<Product, Long> {

    // Custom JPQL query
    @Query("SELECT p FROM Product p WHERE p.category = :category AND p.price BETWEEN :minPrice AND :maxPrice")
    List<Product> findByCategoryAndPriceRange(@Param("category") String category,
                                              @Param("minPrice") Double minPrice,
                                              @Param("maxPrice") Double maxPrice);

    // Custom native SQL query
    @Query(value = "SELECT * FROM product WHERE category = :category AND price BETWEEN :minPrice AND :maxPrice",
           nativeQuery = true)
    List<Product> findByCategoryAndPriceRangeNative(@Param("category") String category,
                                                    @Param("minPrice") Double minPrice,
                                                    @Param("maxPrice") Double maxPrice);
}
```

**Service Layer:**

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<Product> getProductsByCategoryAndPrice(String category, Double minPrice, Double maxPrice) {
        return productRepository.findByCategoryAndPriceRange(category, minPrice, maxPrice);
    }
}
```

**Controller Layer:**

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping("/filter")
    public List<Product> filterProducts(@RequestParam String category,
                                         @RequestParam Double minPrice,
                                         @RequestParam Double maxPrice) {
        return productService.getProductsByCategoryAndPrice(category, minPrice, maxPrice);
    }
}
```

---

### **2. Using Dynamic Queries with `EntityManager`**

For more complex queries, you can use the `EntityManager` to build dynamic JPQL.

**Custom Repository Implementation:**

1. **Create a Custom Interface:**

```java
public interface ProductCustomRepository {
    List<Product> findProductsByDynamicFilters(String name, String category, Double minPrice, Double maxPrice);
}
```

2. **Implement the Interface:**

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import jakarta.persistence.TypedQuery;
import org.springframework.stereotype.Repository;

@Repository
public class ProductCustomRepositoryImpl implements ProductCustomRepository {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<Product> findProductsByDynamicFilters(String name, String category, Double minPrice, Double maxPrice) {
        StringBuilder query = new StringBuilder("SELECT p FROM Product p WHERE 1=1");

        if (name != null && !name.isEmpty()) {
            query.append(" AND p.name LIKE :name");
        }
        if (category != null && !category.isEmpty()) {
            query.append(" AND p.category = :category");
        }
        if (minPrice != null) {
            query.append(" AND p.price >= :minPrice");
        }
        if (maxPrice != null) {
            query.append(" AND p.price <= :maxPrice");
        }

        TypedQuery<Product> typedQuery = entityManager.createQuery(query.toString(), Product.class);

        if (name != null && !name.isEmpty()) {
            typedQuery.setParameter("name", "%" + name + "%");
        }
        if (category != null && !category.isEmpty()) {
            typedQuery.setParameter("category", category);
        }
        if (minPrice != null) {
            typedQuery.setParameter("minPrice", minPrice);
        }
        if (maxPrice != null) {
            typedQuery.setParameter("maxPrice", maxPrice);
        }

        return typedQuery.getResultList();
    }
}
```

3. **Integrate with the Main Repository:**

```java
public interface ProductRepository extends JpaRepository<Product, Long>, ProductCustomRepository {
}
```

---

### **3. Using Projections in Custom Queries**

You can return specific fields instead of entire entities using projections.

**Define a Projection Interface:**

```java
public interface ProductProjection {
    String getName();
    Double getPrice();
}
```

**Custom Query with Projection:**

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Query("SELECT p.name AS name, p.price AS price FROM Product p WHERE p.category = :category")
    List<ProductProjection> findProductsByCategory(@Param("category") String category);
}
```

**Usage:**

```java
List<ProductProjection> products = productRepository.findProductsByCategory("Electronics");
products.forEach(p -> {
    System.out.println("Name: " + p.getName());
    System.out.println("Price: " + p.getPrice());
});
```

---

### **4. Pagination with Custom Queries**

You can combine custom queries with pagination.

**Custom Query with Pagination:**

```java
@Query("SELECT p FROM Product p WHERE p.category = :category")
Page<Product> findByCategory(@Param("category") String category, Pageable pageable);
```

**Usage in Service:**

```java
Page<Product> products = productRepository.findByCategory("Electronics", PageRequest.of(0, 10, Sort.by("price").descending()));
```

---

### **5. Native Queries for Complex Scenarios**

For advanced queries that can't be expressed in JPQL, use native SQL.

**Native SQL Example:**

```java
@Query(value = "SELECT * FROM product WHERE MATCH(name, description) AGAINST(:keyword IN NATURAL LANGUAGE MODE)", nativeQuery = true)
List<Product> searchProductsByKeyword(@Param("keyword") String keyword);
```

---

### Key Points:

* Use `@Query` for custom JPQL or SQL queries.
* Use `EntityManager` for dynamic or complex queries.
* Combine custom queries with pagination using `Pageable`.
* Use projections to optimize query results.
* For complex filtering, prefer `Specification` or custom repositories.

Would you like to explore any of these examples further?


<a id="chat"></a>
### Chat :
public static Specification<Product> hasCategory(String category) { return (root, query, criteriaBuilder) -> category == null ? null : criteriaBuilder.equal(root.get("category"), category); }

can you explain this line in more detail ?

### Ans :

This line is part of the **Specification** pattern used in **Spring Data JPA** to create dynamic queries in a flexible and reusable way. Let’s break it down in detail:

---

### **Code Overview:**

```java
public static Specification<Product> hasCategory(String category) {
    return (root, query, criteriaBuilder) -> 
        category == null ? null : criteriaBuilder.equal(root.get("category"), category);
}
```

---

### **What Is Happening Here?**

1. **`public static Specification<Product>`:**

   * This defines a static method that returns a **`Specification`** object for the `Product` entity.
   * A `Specification` is a functional interface that builds a query dynamically.

2. **`hasCategory(String category)`:**

   * The method name (`hasCategory`) describes its purpose: to filter products based on the `category` field.
   * It takes a parameter `category` that specifies the category to filter by.

3. **`return (root, query, criteriaBuilder) ->`:**

   * This is a **lambda expression** implementing the `toPredicate` method of the `Specification` interface.
   * The `toPredicate` method generates a query condition (a `Predicate`) based on the inputs:

     * **`root`**: Represents the entity being queried (`Product` in this case). You use it to navigate the entity's fields.
     * **`query`**: Represents the `CriteriaQuery` itself. It is used for query customization (e.g., adding `GROUP BY`).
     * **`criteriaBuilder`**: A helper object to construct the actual query conditions (`WHERE` clauses).

4. **`category == null ? null :`:**

   * This checks if the `category` parameter is `null`. If it is:

     * The method returns `null`, meaning no filtering is applied for `category`.
   * If `category` is not `null`, it moves to the next part of the expression.

5. **`criteriaBuilder.equal(root.get("category"), category)`:**

   * **`criteriaBuilder.equal`**: This creates a `Predicate` that corresponds to a SQL `=` condition.

     * Example: `SELECT * FROM product WHERE category = 'Electronics'`.
   * **`root.get("category")`**: Refers to the `category` field in the `Product` entity. This maps to the column in the database.

---

### **What Query Does It Build?**

When you call this method with a `category` value, it creates a query condition that filters products with the given category.

#### Example 1: `hasCategory("Electronics")`

* Generates: `WHERE category = 'Electronics'`.

#### Example 2: `hasCategory(null)`

* Generates: No filtering condition (because it returns `null`).

---

### **How Is It Used?**

This `Specification` is combined with others to build complex queries dynamically.

#### Single Condition:

```java
Specification<Product> spec = ProductSpecification.hasCategory("Electronics");
List<Product> products = productRepository.findAll(spec);
```

#### Combined with Other Filters:

```java
Specification<Product> spec = Specification.where(ProductSpecification.hasCategory("Electronics"))
                                           .and(ProductSpecification.hasPriceBetween(100.0, 500.0));

List<Product> products = productRepository.findAll(spec);
```

This generates a query like:

```sql
SELECT * FROM product 
WHERE category = 'Electronics' 
AND price BETWEEN 100.0 AND 500.0;
```

---

### **Key Benefits:**

1. **Dynamic Querying:**

   * You can build complex queries at runtime by combining specifications.

2. **Null-Safe Filtering:**

   * If the input (`category`) is `null`, no condition is added, making it safe for optional filters.

3. **Reusable Logic:**

   * You can define these specifications once and reuse them across your application.

---

Would you like me to show how to combine this with another filter or explain the `criteriaBuilder` object further?


<a id="chat-1"></a>
### Chat : 
" 2\. **Frontend Integration** "

can you implement using angular

### Ans : 
Here's how you can implement the **frontend** using Angular for integrating **pagination**, **sorting**, and **filters** with the Spring Boot backend.

---

### **1. Set Up Angular Project**

Create a new Angular project:

```bash
ng new product-frontend
cd product-frontend
```

Install required dependencies (e.g., **Bootstrap** for styling):

```bash
npm install bootstrap
```

Add Bootstrap to `angular.json`:

```json
"styles": [
  "src/styles.css",
  "node_modules/bootstrap/dist/css/bootstrap.min.css"
],
"scripts": []
```

---

### **2. Create a Service for Backend API Calls**

Generate a service:

```bash
ng generate service services/product
```

**`product.service.ts`:**

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class ProductService {
  private apiUrl = 'http://localhost:8080/products'; // Update with your backend URL

  constructor(private http: HttpClient) {}

  getProducts(filters: any, page: number, size: number, sortBy: string, sortDir: string): Observable<any> {
    let params = new HttpParams()
      .set('page', page)
      .set('size', size)
      .set('sortBy', sortBy)
      .set('sortDir', sortDir);

    if (filters.name) params = params.set('name', filters.name);
    if (filters.category) params = params.set('category', filters.category);
    if (filters.minPrice) params = params.set('minPrice', filters.minPrice);
    if (filters.maxPrice) params = params.set('maxPrice', filters.maxPrice);

    return this.http.get<any>(this.apiUrl, { params });
  }
}
```

---

### **3. Create a Component for Product Listing**

Generate a component:

```bash
ng generate component components/product-list
```

**`product-list.component.ts`:**

```typescript
import { Component, OnInit } from '@angular/core';
import { ProductService } from '../../services/product.service';

@Component({
  selector: 'app-product-list',
  templateUrl: './product-list.component.html',
  styleUrls: ['./product-list.component.css']
})
export class ProductListComponent implements OnInit {
  products: any[] = [];
  filters = { name: '', category: '', minPrice: '', maxPrice: '' };
  page = 0;
  size = 10;
  sortBy = 'id';
  sortDir = 'asc';
  totalPages = 0;

  constructor(private productService: ProductService) {}

  ngOnInit(): void {
    this.fetchProducts();
  }

  fetchProducts(): void {
    this.productService.getProducts(this.filters, this.page, this.size, this.sortBy, this.sortDir).subscribe({
      next: (data) => {
        this.products = data.content;
        this.totalPages = data.totalPages;
      },
      error: (err) => {
        console.error('Error fetching products:', err);
      },
    });
  }

  updateFilter(key: string, value: string): void {
    this.filters[key] = value;
    this.page = 0; // Reset to the first page when filters change
    this.fetchProducts();
  }

  changePage(newPage: number): void {
    this.page = newPage;
    this.fetchProducts();
  }

  sort(field: string): void {
    if (this.sortBy === field) {
      this.sortDir = this.sortDir === 'asc' ? 'desc' : 'asc';
    } else {
      this.sortBy = field;
      this.sortDir = 'asc';
    }
    this.fetchProducts();
  }
}
```

---

**`product-list.component.html`:**

```html
<div class="container mt-4">
  <h1>Product List</h1>

  <!-- Filters -->
  <div class="mb-3">
    <div class="row">
      <div class="col-md-3">
        <input
          type="text"
          class="form-control"
          placeholder="Name"
          (input)="updateFilter('name', $event.target.value)"
        />
      </div>
      <div class="col-md-3">
        <input
          type="text"
          class="form-control"
          placeholder="Category"
          (input)="updateFilter('category', $event.target.value)"
        />
      </div>
      <div class="col-md-3">
        <input
          type="number"
          class="form-control"
          placeholder="Min Price"
          (input)="updateFilter('minPrice', $event.target.value)"
        />
      </div>
      <div class="col-md-3">
        <input
          type="number"
          class="form-control"
          placeholder="Max Price"
          (input)="updateFilter('maxPrice', $event.target.value)"
        />
      </div>
    </div>
  </div>

  <!-- Product Table -->
  <table class="table table-striped">
    <thead>
      <tr>
        <th (click)="sort('name')">Name</th>
        <th (click)="sort('category')">Category</th>
        <th (click)="sort('price')">Price</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let product of products">
        <td>{{ product.name }}</td>
        <td>{{ product.category }}</td>
        <td>{{ product.price }}</td>
      </tr>
    </tbody>
  </table>

  <!-- Pagination -->
  <nav>
    <ul class="pagination">
      <li class="page-item" [class.disabled]="page === 0">
        <button class="page-link" (click)="changePage(page - 1)">Previous</button>
      </li>
      <li class="page-item" *ngFor="let i of [].constructor(totalPages); let idx = index" [class.active]="idx === page">
        <button class="page-link" (click)="changePage(idx)">{{ idx + 1 }}</button>
      </li>
      <li class="page-item" [class.disabled]="page === totalPages - 1">
        <button class="page-link" (click)="changePage(page + 1)">Next</button>
      </li>
    </ul>
  </nav>
</div>
```

---

### **4. Add the Component to a Route**

Update `app.module.ts`:

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { ProductListComponent } from './components/product-list/product-list.component';

@NgModule({
  declarations: [
    AppComponent,
    ProductListComponent,
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Add a route in `app-routing.module.ts`:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { ProductListComponent } from './components/product-list/product-list.component';

const routes: Routes = [
  { path: '', component: ProductListComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

---

### **5. Run the Application**

1. Start the Angular development server:

   ```bash
   ng serve
   ```
2. Visit `http://localhost:4200` in your browser.
3. Ensure your Spring Boot backend is running (`http://localhost:8080/products`).

---

### **Features Implemented**

1. Filtering by `name`, `category`, `minPrice`, `maxPrice`.
2. Sorting by any column (`name`, `category`, `price`).
3. Pagination with dynamic navigation.

Let me know if you'd like additional enhancements, such as better styling, JWT authentication, or exporting data!


<a id="chat-2"></a>
### Chat : 
" C. **Controller with Query Parameters** "

can we convert this to post request then send with request body and not with query paramters?

### Ans :

Yes! Instead of sending filter parameters as **query parameters**, we can use a **POST request with a request body** to send the filtering, sorting, and pagination options.

---

## **1. Modify Spring Boot Backend to Accept POST Requests with a Request Body**

### **Update Controller**

Modify the controller to accept filtering criteria as a request body.

```java
import org.springframework.data.domain.*;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/products")
@CrossOrigin(origins = "http://localhost:4200")
public class ProductController {

    private final ProductRepository productRepository;

    public ProductController(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @PostMapping("/search")
    public Page<Product> searchProducts(@RequestBody ProductFilterRequest filterRequest) {
        Specification<Product> spec = Specification.where(null);

        if (filterRequest.getName() != null) {
            spec = spec.and(ProductSpecification.hasName(filterRequest.getName()));
        }
        if (filterRequest.getCategory() != null) {
            spec = spec.and(ProductSpecification.hasCategory(filterRequest.getCategory()));
        }
        if (filterRequest.getMinPrice() != null && filterRequest.getMaxPrice() != null) {
            spec = spec.and(ProductSpecification.hasPriceBetween(filterRequest.getMinPrice(), filterRequest.getMaxPrice()));
        }

        Pageable pageable = PageRequest.of(filterRequest.getPage(), filterRequest.getSize(),
                Sort.by(Sort.Direction.fromString(filterRequest.getSortDir()), filterRequest.getSortBy()));

        return productRepository.findAll(spec, pageable);
    }
}
```

---

### **Create a DTO for the Request Body**

This DTO (`ProductFilterRequest`) will hold the filtering, sorting, and pagination details.

```java
public class ProductFilterRequest {
    private String name;
    private String category;
    private Double minPrice;
    private Double maxPrice;
    private int page;
    private int size;
    private String sortBy;
    private String sortDir;

    // Getters and Setters
}
```

---

## **2. Modify Angular Service to Use a POST Request**

Update the Angular service to send filters in the request body instead of using query parameters.

### **Update `product.service.ts`**

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class ProductService {
  private apiUrl = 'http://localhost:8080/products/search'; // Updated endpoint

  constructor(private http: HttpClient) {}

  getProducts(filters: any, page: number, size: number, sortBy: string, sortDir: string): Observable<any> {
    const requestBody = {
      name: filters.name || null,
      category: filters.category || null,
      minPrice: filters.minPrice || null,
      maxPrice: filters.maxPrice || null,
      page: page,
      size: size,
      sortBy: sortBy,
      sortDir: sortDir
    };

    return this.http.post<any>(this.apiUrl, requestBody);
  }
}
```

---

## **3. Update Angular Component to Use the New API**

No major changes are needed in the `product-list.component.ts`, except that it now works with a POST request.

### **`product-list.component.ts`**

```typescript
import { Component, OnInit } from '@angular/core';
import { ProductService } from '../../services/product.service';

@Component({
  selector: 'app-product-list',
  templateUrl: './product-list.component.html',
  styleUrls: ['./product-list.component.css']
})
export class ProductListComponent implements OnInit {
  products: any[] = [];
  filters = { name: '', category: '', minPrice: '', maxPrice: '' };
  page = 0;
  size = 10;
  sortBy = 'id';
  sortDir = 'asc';
  totalPages = 0;

  constructor(private productService: ProductService) {}

  ngOnInit(): void {
    this.fetchProducts();
  }

  fetchProducts(): void {
    this.productService.getProducts(this.filters, this.page, this.size, this.sortBy, this.sortDir).subscribe({
      next: (data) => {
        this.products = data.content;
        this.totalPages = data.totalPages;
      },
      error: (err) => {
        console.error('Error fetching products:', err);
      },
    });
  }

  updateFilter(key: string, value: string): void {
    this.filters[key] = value;
    this.page = 0;
    this.fetchProducts();
  }

  changePage(newPage: number): void {
    this.page = newPage;
    this.fetchProducts();
  }

  sort(field: string): void {
    if (this.sortBy === field) {
      this.sortDir = this.sortDir === 'asc' ? 'desc' : 'asc';
    } else {
      this.sortBy = field;
      this.sortDir = 'asc';
    }
    this.fetchProducts();
  }
}
```

---

## **4. Update Angular Component Template**

No changes are required in `product-list.component.html`. It remains the same.

---

## **5. Summary of Changes**

### **Backend (Spring Boot)**

✅ Converted `GET` request to a `POST` request in the controller.
✅ Created a `ProductFilterRequest` DTO for request body.
✅ Modified repository to use `Specification<Product>`.

### **Frontend (Angular)**

✅ Modified `product.service.ts` to use `POST` instead of `GET`.
✅ Updated `product-list.component.ts` to work with the new API.

---

### **Final Notes**

* This approach makes the API **cleaner** because we avoid long query parameters.
* It supports **complex filtering** with multiple criteria.
* It allows **sending large filter data** efficiently.

---

Let me know if you want further improvements like caching, infinite scrolling, or JWT authentication! 🚀


<a id="so-basically-which-one-is-recommended"></a>
### Chat : so basically which one is recommended?

### **GET vs. POST for Pagination, Sorting, and Filtering**

Both **GET** and **POST** have their advantages and are used in different scenarios. Let’s compare them and decide which one is better suited for your **Spring Boot + Angular** application.

---

### **✅ When to Use `GET` (Query Parameters)**

**Best for:**

* Simple filtering (e.g., `?name=phone&category=electronics&page=1`)
* Read-only requests that **do not modify data**
* Bookmarkable URLs (users can share URLs with filters applied)
* Caching by browsers and CDNs (better performance in some cases)

**Cons:**

* Limited by URL length (not good for large/complex filters)
* Exposes parameters in the URL (not secure for sensitive data)
* Not ideal for complex request bodies (nested filters, ranges, etc.)

---

### **✅ When to Use `POST` (Request Body)**

**Best for:**

* Complex filtering (e.g., searching with multiple criteria like `minPrice`, `maxPrice`, `tags`, `brand`, etc.)
* When sending **large data** (avoids URL length limitations)
* When security is a concern (parameters are not exposed in the URL)
* Better structuring of request data (cleaner and more readable JSON)

**Cons:**

* Cannot be bookmarked easily (not user-friendly for sharing links)
* Not cached by browsers/CDNs by default
* Slightly more overhead in terms of request processing

---

### **🔹 Which One Should You Use?**

* If your filters are **simple** (like `name`, `category`, `page`, `sort`), then **GET** is fine.
* If your filters are **complex** (like multiple price ranges, multiple categories, nested filters, etc.), then **POST** is the better choice.
* If you need **SEO-friendly URLs** (e.g., `?category=phones&page=1`), use **GET**.
* If you want **better security** and a **cleaner API**, use **POST**.

---

### **📌 Recommended Approach for Your Spring Boot + Angular App**

Since your app has **pagination, sorting, and filtering**, I recommend:

* **Use `GET` for simple use cases** (if filters are few, like name/category/sort).
* **Use `POST` for advanced filtering** (if filters involve multiple conditions, ranges, or complex logic).

➡️ **Hybrid Approach:**
You can support **both** and let the frontend decide!

* Use `GET` for direct pagination (`/products?page=1&size=10`).
* Use `POST` when a user applies **complex filters** (like price range, tags, etc.).

Let me know if you want help implementing both approaches! 🚀


<a id="yes-please"></a>
### Chat : yes please

Great! Let's implement a **hybrid approach** where your Spring Boot backend supports **both GET and POST** for fetching products:

---

## ✅ **1. Spring Boot Backend: Support Both GET and POST**

### 🔹 `GET /products` — Simple Query Parameters

```java
@GetMapping
public Page<Product> getProducts(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) String category,
        @RequestParam(required = false) Double minPrice,
        @RequestParam(required = false) Double maxPrice,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "asc") String sortDir
) {
    Specification<Product> spec = Specification.where(ProductSpecification.hasName(name))
            .and(ProductSpecification.hasCategory(category))
            .and(ProductSpecification.hasPriceBetween(minPrice, maxPrice));

    Pageable pageable = PageRequest.of(page, size,
            Sort.by(Sort.Direction.fromString(sortDir), sortBy));

    return productRepository.findAll(spec, pageable);
}
```

---

### 🔹 `POST /products/search` — Complex Filters in Request Body

**DTO Class**:

```java
public class ProductFilterRequest {
    private String name;
    private String category;
    private Double minPrice;
    private Double maxPrice;
    private int page = 0;
    private int size = 10;
    private String sortBy = "id";
    private String sortDir = "asc";

    // Getters and Setters
}
```

**POST Endpoint**:

```java
@PostMapping("/search")
public Page<Product> searchProducts(@RequestBody ProductFilterRequest filterRequest) {
    Specification<Product> spec = Specification.where(ProductSpecification.hasName(filterRequest.getName()))
            .and(ProductSpecification.hasCategory(filterRequest.getCategory()))
            .and(ProductSpecification.hasPriceBetween(filterRequest.getMinPrice(), filterRequest.getMaxPrice()));

    Pageable pageable = PageRequest.of(filterRequest.getPage(), filterRequest.getSize(),
            Sort.by(Sort.Direction.fromString(filterRequest.getSortDir()), filterRequest.getSortBy()));

    return productRepository.findAll(spec, pageable);
}
```

---

## ✅ **2. Angular Frontend: Allow Switching Between GET and POST**

### 🔹 Angular Service (`product.service.ts`)

```ts
getProductsGET(filters: any, page: number, size: number, sortBy: string, sortDir: string): Observable<any> {
  let params = new HttpParams()
    .set('page', page)
    .set('size', size)
    .set('sortBy', sortBy)
    .set('sortDir', sortDir);

  if (filters.name) params = params.set('name', filters.name);
  if (filters.category) params = params.set('category', filters.category);
  if (filters.minPrice) params = params.set('minPrice', filters.minPrice);
  if (filters.maxPrice) params = params.set('maxPrice', filters.maxPrice);

  return this.http.get(`${this.apiUrl}`, { params });
}

getProductsPOST(filters: any, page: number, size: number, sortBy: string, sortDir: string): Observable<any> {
  const body = {
    name: filters.name || null,
    category: filters.category || null,
    minPrice: filters.minPrice || null,
    maxPrice: filters.maxPrice || null,
    page,
    size,
    sortBy,
    sortDir
  };
  return this.http.post(`${this.apiUrl}/search`, body);
}
```

---

### 🔹 Angular Component: Toggle Mode

In `product-list.component.ts`, you can toggle between GET and POST with a flag:

```ts
mode: 'get' | 'post' = 'get'; // 'post' for complex filters

fetchProducts(): void {
  const serviceCall = this.mode === 'get'
    ? this.productService.getProductsGET(this.filters, this.page, this.size, this.sortBy, this.sortDir)
    : this.productService.getProductsPOST(this.filters, this.page, this.size, this.sortBy, this.sortDir);

  serviceCall.subscribe({
    next: (data) => {
      this.products = data.content;
      this.totalPages = data.totalPages;
    },
    error: (err) => console.error('Error fetching products:', err),
  });
}
```

---

### 🔹 Allow Switching in the Template (Optional for Demo)

```html
<div class="mb-3">
  <label><input type="radio" [(ngModel)]="mode" value="get"> Use GET</label>
  <label class="ms-3"><input type="radio" [(ngModel)]="mode" value="post"> Use POST</label>
</div>
```

> Now your Angular app can **toggle between GET and POST** mode, and your backend supports both.

---

## ✅ Summary

| Feature         | GET       | POST               |
| --------------- | --------- | ------------------ |
| Simplicity      | ✅ Easy    | ❌ Slightly verbose |
| Bookmarkable    | ✅ Yes     | ❌ No               |
| Caching         | ✅ Yes     | ❌ No               |
| Complex Filters | ❌ Hard    | ✅ Easy             |
| Secure Params   | ❌ Exposed | ✅ Hidden in body   |

**Recommended Practice**:

* Use `GET` when filters are simple or you want shareable URLs.
* Use `POST` for complex use cases — especially in admin panels or advanced search forms.

Would you like a GitHub-ready project example for this full-stack setup?


<a id="can-we-go-with-hybrid-approach-also"></a>
### Chat : can we go with hybrid approach also?

Yes, absolutely — a **hybrid approach** is not only possible but also a **recommended architectural practice** when your application needs to support both:

* **Simple, URL-based searches** (good for user-facing, SEO-friendly, shareable pages), and
* **Complex, flexible searches** (good for internal dashboards, advanced filters, or admin panels).

---

## ✅ What Does "Hybrid Approach" Mean?

It means your **Spring Boot backend** supports both:

1. **GET `/products`** for:

   * Lightweight filtering via query params (name, category, etc.)
   * Shareable/searchable/bookmarkable URLs

2. **POST `/products/search`** for:

   * Complex filters (e.g. multiple tags, nested objects, ranges)
   * Clean JSON body
   * More flexible structure (better for frontends with dynamic filter forms)

---

## 🔧 How to Structure a Hybrid Spring Boot API

You already have:

### ✅ `GET /products` — for simple filtering:

```java
@GetMapping
public Page<Product> getProducts(...) {
    // Use query params
}
```

### ✅ `POST /products/search` — for complex filtering:

```java
@PostMapping("/search")
public Page<Product> searchProducts(@RequestBody ProductFilterRequest filterRequest) {
    // Use request body
}
```

### 💡 Bonus: You can also make POST `/products` return all with pagination (no filters)

---

## 🤖 How Angular Can Use the Hybrid Approach

### In Angular Service:

```ts
getProductsGET(filters: any, page: number, size: number, sortBy: string, sortDir: string): Observable<any> {
  // query param-based call
}

getProductsPOST(filters: any, page: number, size: number, sortBy: string, sortDir: string): Observable<any> {
  // JSON body-based call
}
```

### In Component:

```ts
mode: 'get' | 'post' = 'get';  // Toggle mode based on form logic or user preference

fetchProducts(): void {
  const request = this.mode === 'get'
    ? this.productService.getProductsGET(...)
    : this.productService.getProductsPOST(...);

  request.subscribe((data) => {
    this.products = data.content;
    this.totalPages = data.totalPages;
  });
}
```

---

## 🧠 When to Use Which in Real Applications

| Scenario                                                                    | Use GET?      | Use POST?                      |
| --------------------------------------------------------------------------- | ------------- | ------------------------------ |
| Basic product search page                                                   | ✅ Yes         | ❌ Not needed                   |
| Advanced product filters (price range, multiple tags, complex AND/OR logic) | ❌ Limited     | ✅ Yes                          |
| Internal admin dashboard with lots of filters                               | ❌ Too verbose | ✅ Clean body                   |
| Public-facing URLs that users can bookmark/share                            | ✅ Ideal       | ❌ No (body isn't shown in URL) |
| SEO-friendly product search                                                 | ✅             | ❌                              |
| Need to hide filter structure from URL (e.g., sensitive data)               | ❌             | ✅                              |

---

## ✅ Recommendation

### Use **GET by default**, and **POST for advanced/optional search**:

* Implement both endpoints in backend.
* Angular can choose based on UI complexity.
* Keep the logic DRY by reusing Specification logic or service methods internally.

---

### Want me to give you a fully working GitHub-ready sample project with:

* Spring Boot (GET + POST hybrid)
* Angular (toggle GET/POST from UI)
* Simple frontend filters
* Paging + sorting

Let me know — I can generate or share code structure for that.


<a id="how-to-implement-in-spring-boot-with-hybrid-approach"></a>
### Chat : how to implement in spring boot with hybrid approach?

To implement a **hybrid approach in Spring Boot** (supporting both GET and POST for pagination, sorting, and filtering), follow these steps:

---

## ✅ Step 1: Create the Entity

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String category;
    private Double price;

    // Getters and Setters
}
```

---

## ✅ Step 2: Create a Repository

```java
public interface ProductRepository extends JpaRepository<Product, Long>, JpaSpecificationExecutor<Product> {
}
```

Use `JpaSpecificationExecutor` for dynamic query building.

---

## ✅ Step 3: Create Specifications for Filtering

```java
public class ProductSpecification {

    public static Specification<Product> hasName(String name) {
        return (root, query, cb) -> 
            name == null ? null : cb.like(cb.lower(root.get("name")), "%" + name.toLowerCase() + "%");
    }

    public static Specification<Product> hasCategory(String category) {
        return (root, query, cb) -> 
            category == null ? null : cb.equal(cb.lower(root.get("category")), category.toLowerCase());
    }

    public static Specification<Product> hasPriceBetween(Double min, Double max) {
        return (root, query, cb) -> {
            if (min != null && max != null) return cb.between(root.get("price"), min, max);
            if (min != null) return cb.greaterThanOrEqualTo(root.get("price"), min);
            if (max != null) return cb.lessThanOrEqualTo(root.get("price"), max);
            return null;
        };
    }
}
```

---

## ✅ Step 4: Create a Filter DTO for POST

```java
public class ProductFilterRequest {
    private String name;
    private String category;
    private Double minPrice;
    private Double maxPrice;
    private int page = 0;
    private int size = 10;
    private String sortBy = "id";
    private String sortDir = "asc";

    // Getters and Setters
}
```

---

## ✅ Step 5: Create Controller with Hybrid Endpoints

```java
@RestController
@RequestMapping("/products")
@CrossOrigin(origins = "http://localhost:4200") // optional for Angular
public class ProductController {

    private final ProductRepository productRepository;

    public ProductController(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    // ✅ GET endpoint for simple filter via query parameters
    @GetMapping
    public Page<Product> getProducts(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "asc") String sortDir
    ) {
        Specification<Product> spec = Specification.where(ProductSpecification.hasName(name))
                .and(ProductSpecification.hasCategory(category))
                .and(ProductSpecification.hasPriceBetween(minPrice, maxPrice));

        Pageable pageable = PageRequest.of(page, size,
                Sort.by(Sort.Direction.fromString(sortDir), sortBy));

        return productRepository.findAll(spec, pageable);
    }

    // ✅ POST endpoint for advanced filters using request body
    @PostMapping("/search")
    public Page<Product> searchProducts(@RequestBody ProductFilterRequest filterRequest) {
        Specification<Product> spec = Specification.where(ProductSpecification.hasName(filterRequest.getName()))
                .and(ProductSpecification.hasCategory(filterRequest.getCategory()))
                .and(ProductSpecification.hasPriceBetween(filterRequest.getMinPrice(), filterRequest.getMaxPrice()));

        Pageable pageable = PageRequest.of(filterRequest.getPage(), filterRequest.getSize(),
                Sort.by(Sort.Direction.fromString(filterRequest.getSortDir()), filterRequest.getSortBy()));

        return productRepository.findAll(spec, pageable);
    }
}
```

---

## ✅ Testing the Hybrid Endpoints

### 🔹 1. Simple GET Call:

```
GET http://localhost:8080/products?name=phone&category=electronics&minPrice=100&maxPrice=1000&page=0&size=10&sortBy=price&sortDir=desc
```

### 🔹 2. Complex POST Call:

```
POST http://localhost:8080/products/search
Content-Type: application/json

{
  "name": "phone",
  "category": "electronics",
  "minPrice": 100,
  "maxPrice": 1000,
  "page": 0,
  "size": 10,
  "sortBy": "price",
  "sortDir": "desc"
}
```

---

## 🧠 Why This Works Well

* You keep **GET** for user-facing apps where URLs are shareable.
* You allow **POST** for admin dashboards or advanced filters.
* The logic is **shared via `Specification`**, avoiding code duplication.

---

Would you like me to:

* Add **Angular integration**
* Add **unit tests** for this hybrid controller
* Add **Swagger/OpenAPI** to document both endpoints

Let me know what you'd like next!


## **Conversation**

<a id="how-can-we-implement-search-functionality-in-spring-boot-app-using-mongodb-also-include-pagination-and-filtering"></a>
### Chat : how can we implement search functionality in spring boot app using mongoDB, also include pagination and filtering

To implement search functionality in a **Spring Boot** application using **MongoDB**, along with pagination and filtering, follow these steps:

---

## **1. Add Dependencies**

Make sure your `pom.xml` has the required dependencies:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

---

## **2. Create an Entity**

Define a **MongoDB Document**:

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "products")
public class Product {
    
    @Id
    private String id;
    private String name;
    private String category;
    private double price;
    
    // Constructors
    public Product() {}
    
    public Product(String name, String category, double price) {
        this.name = name;
        this.category = category;
        this.price = price;
    }

    // Getters and Setters
}
```

---

## **3. Create a Repository**

Use `MongoRepository` for CRUD operations:

```java
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductRepository extends MongoRepository<Product, String> {
}
```

---

## **4. Implement the Search with Pagination and Filtering**

Use `Pageable` for pagination and `Criteria` for filtering:

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ProductService {

    private final MongoTemplate mongoTemplate;

    public ProductService(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    public Page<Product> searchProducts(String name, String category, Double minPrice, Double maxPrice, int page, int size) {
        Query query = new Query();

        // Adding Filters
        if (name != null && !name.isEmpty()) {
            query.addCriteria(Criteria.where("name").regex(name, "i")); // Case-insensitive search
        }
        if (category != null && !category.isEmpty()) {
            query.addCriteria(Criteria.where("category").is(category));
        }
        if (minPrice != null && maxPrice != null) {
            query.addCriteria(Criteria.where("price").gte(minPrice).lte(maxPrice));
        }

        // Pagination
        Pageable pageable = PageRequest.of(page, size);
        query.with(pageable);

        // Execute Query
        List<Product> products = mongoTemplate.find(query, Product.class);
        long total = mongoTemplate.count(query, Product.class);

        return new org.springframework.data.domain.PageImpl<>(products, pageable, total);
    }
}
```

---

## **5. Expose an API Endpoint**

Create a **REST Controller**:

```java
import org.springframework.data.domain.Page;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping("/search")
    public Page<Product> searchProducts(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {

        return productService.searchProducts(name, category, minPrice, maxPrice, page, size);
    }
}
```

---

## **6. Test the API**

### **Example Request**

```
GET http://localhost:8080/products/search?name=phone&category=electronics&minPrice=100&maxPrice=1000&page=0&size=5
```

### **Example Response**

```json
{
    "content": [
        {
            "id": "65a4f14df",
            "name": "iPhone 13",
            "category": "electronics",
            "price": 799.99
        },
        {
            "id": "65a4f14df",
            "name": "Samsung Galaxy S22",
            "category": "electronics",
            "price": 699.99
        }
    ],
    "pageable": {
        "pageNumber": 0,
        "pageSize": 5
    },
    "totalElements": 2
}
```

---

### **Summary**

✅ Implemented **MongoDB search** with filters (`name`, `category`, `price`).
✅ Used **pagination** with `Pageable`.
✅ Exposed an **API endpoint** to fetch results.

Would you like me to add sorting as well? 🚀


<a id="what-is-mongotemplate"></a>
### Chat : what is MongoTemplate ?

### **What is `MongoTemplate` in Spring Boot?**

`MongoTemplate` is a **helper class** provided by **Spring Data MongoDB** that simplifies interactions with MongoDB. It provides a lower-level API than `MongoRepository`, giving you **more control over queries**, data aggregation, and transactions.

---

## **Key Features of `MongoTemplate`**

* **Direct access to MongoDB queries** using `Query`, `Criteria`, etc.
* Supports **custom filtering, sorting, and pagination**.
* Provides **CRUD operations** (`insert`, `update`, `find`, `remove`, etc.).
* Supports **aggregation pipelines** (like `groupBy`, `sum`, `count`).
* Can execute **native MongoDB queries**.

---

## **Example Usage of `MongoTemplate`**

### **1. Basic Query Example**

```java
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class ProductService {

    private final MongoTemplate mongoTemplate;

    public ProductService(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    public List<Product> findProductsByCategory(String category) {
        Query query = new Query();
        query.addCriteria(Criteria.where("category").is(category));
        return mongoTemplate.find(query, Product.class);
    }
}
```

Here:

* `Query` is used to **filter** documents.
* `Criteria.where("category").is(category)` matches a field.
* `mongoTemplate.find()` executes the query.

---

### **2. Pagination and Sorting**

```java
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.mongodb.core.query.Sort;

public List<Product> getPagedProducts(int page, int size) {
    Query query = new Query();
    Pageable pageable = PageRequest.of(page, size);
    
    query.with(pageable);
    query.with(Sort.by(Sort.Direction.ASC, "price"));

    return mongoTemplate.find(query, Product.class);
}
```

* `PageRequest.of(page, size)`: Implements pagination.
* `Sort.by(Sort.Direction.ASC, "price")`: Sorts results.

---

### **3. Insert a Document**

```java
public void addProduct(Product product) {
    mongoTemplate.insert(product);
}
```

---

### **4. Update a Document**

```java
import org.springframework.data.mongodb.core.query.Update;

public void updateProductPrice(String id, double newPrice) {
    Query query = new Query(Criteria.where("id").is(id));
    Update update = new Update().set("price", newPrice);

    mongoTemplate.updateFirst(query, update, Product.class);
}
```

* `updateFirst()` updates the first matching document.

---

### **5. Delete a Document**

```java
public void deleteProduct(String id) {
    Query query = new Query(Criteria.where("id").is(id));
    mongoTemplate.remove(query, Product.class);
}
```

---

## **When to Use `MongoTemplate` vs `MongoRepository`?**

| Feature                      | `MongoRepository` | `MongoTemplate` |
| ---------------------------- | ----------------- | --------------- |
| **CRUD operations**          | ✅ Easy to use     | ✅ More control  |
| **Custom Queries**           | ❌ Limited         | ✅ Flexible      |
| **Aggregation Queries**      | ❌ No              | ✅ Yes           |
| **Complex Filtering**        | ❌ No              | ✅ Yes           |
| **Performance Optimization** | ❌ No              | ✅ Yes           |

* Use `MongoRepository` for **basic CRUD**.
* Use `MongoTemplate` when you need **custom queries, filtering, and aggregations**.

---

Would you like an example with **aggregation pipelines** as well? 🚀


