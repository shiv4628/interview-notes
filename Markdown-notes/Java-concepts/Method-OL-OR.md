## **Conversation**

### Chat : can you explain method overloading and overriding in detail and various ambiguity scenarios cases also

### Method Overloading and Overriding in Java

#### **1. Method Overloading**

Method overloading occurs when two or more methods in the same class share the same name but differ in parameters (number, type, or order of parameters).

##### **Key Features**

* **Compile-time Polymorphism**: The method to be called is determined at compile time.
* **Rules**:

  * Methods must have the same name.
  * Must differ in the number or type of parameters.
  * Return type can be different, but it doesn't determine overloading.

##### **Example**

```java
class Calculator {
    // Overloaded methods
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

##### **Ambiguity in Overloading**

Ambiguity arises when the compiler cannot determine which method to call.
For instance:

```java
class AmbiguityExample {
    void test(int a, double b) { }
    void test(double a, int b) { }

    public static void main(String[] args) {
        AmbiguityExample obj = new AmbiguityExample();
        obj.test(10, 20); // Compiler error: Ambiguous method call
    }
}
```

Here, both `test(int, double)` and `test(double, int)` are valid, leading to ambiguity.

---

#### **2. Method Overriding**

Method overriding occurs when a subclass provides a specific implementation for a method that is already defined in its parent class.

##### **Key Features**

* **Run-time Polymorphism**: The method to be executed is determined at runtime.
* **Rules**:

  * Methods must have the same name, return type, and parameters.
  * The method in the subclass must have the same or more accessible access modifier.
  * The overriding method cannot throw a broader checked exception than the overridden method.
  * Must involve inheritance.

##### **Example**

```java
class Parent {
    void display() {
        System.out.println("Parent's display method");
    }
}

class Child extends Parent {
    @Override
    void display() {
        System.out.println("Child's display method");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent obj = new Child(); // Dynamic dispatch
        obj.display(); // Calls Child's display method
    }
}
```

---

#### **Ambiguity in Overriding**

Ambiguity can arise in the following cases:

##### **Case 1: Reference type mismatch**

```java
class Parent {
    void display() {
        System.out.println("Parent display");
    }
}

class Child extends Parent {
    void display() {
        System.out.println("Child display");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent obj1 = new Parent();
        Parent obj2 = new Child();

        obj1.display(); // Parent display
        obj2.display(); // Child display (dynamic dispatch)
    }
}
```

##### **Case 2: Covariant Return Types**

Covariant return types allow an overridden method to return a subtype of the return type in the parent class. If there are multiple layers of inheritance, method calls can become confusing.

---

### **Key Differences**

| Feature          | Overloading                             | Overriding                                                   |
| ---------------- | --------------------------------------- | ------------------------------------------------------------ |
| **Definition**   | Same method name, different parameters. | Same method signature, different implementation in subclass. |
| **Polymorphism** | Compile-time                            | Runtime                                                      |
| **Inheritance**  | Not required                            | Required                                                     |
| **Return Type**  | Can differ                              | Must be the same or covariant                                |

If you have specific ambiguity scenarios in mind, let me know, and I can explore them further.


### Chat : "Ambiguity in Overloading" can you give more example in detail

Ambiguity in **method overloading** arises when the compiler cannot decide which overloaded method to invoke due to multiple possibilities that match the call. Let's explore this in detail with examples of different ambiguity scenarios:

---

### **1. Ambiguity Due to Type Conversion (Widening Conversion)**

Java allows implicit type conversion (widening) when calling methods. However, if two or more overloaded methods are eligible due to widening, ambiguity arises.

#### **Example**

```java
class AmbiguityWidening {
    void test(int a) {
        System.out.println("Method with int");
    }

    void test(double a) {
        System.out.println("Method with double");
    }

    public static void main(String[] args) {
        AmbiguityWidening obj = new AmbiguityWidening();
        obj.test(5); // Calls test(int) because 5 is an int
        obj.test(5.5); // Calls test(double) because 5.5 is a double

        obj.test(5L); // Ambiguous: long can be widened to int or double
    }
}
```

Here, the call `obj.test(5L)` (a `long` value) causes ambiguity because:

* `long` can be converted to `int`.
* `long` can be converted to `double`.

---

### **2. Ambiguity Due to Boxing and Widening**

When autoboxing and widening are both applicable, the compiler cannot decide which method to call.

#### **Example**

```java
class AmbiguityBoxing {
    void test(Integer a) {
        System.out.println("Method with Integer");
    }

    void test(long a) {
        System.out.println("Method with long");
    }

    public static void main(String[] args) {
        AmbiguityBoxing obj = new AmbiguityBoxing();
        obj.test(5); // Ambiguous: 5 can be autoboxed to Integer or widened to long
    }
}
```

Here, the call `obj.test(5)` is ambiguous because:

* `5` can be autoboxed to `Integer`.
* `5` can also be widened to `long`.

---

### **3. Ambiguity Due to Varargs**

When an overloaded method has a variable argument list (`varargs`), it may cause ambiguity with other methods.

#### **Example**

```java
class AmbiguityVarargs {
    void test(int a) {
        System.out.println("Method with int");
    }

    void test(int... a) {
        System.out.println("Method with varargs");
    }

    public static void main(String[] args) {
        AmbiguityVarargs obj = new AmbiguityVarargs();
        obj.test(5); // Ambiguous: Could match test(int) or test(int...)
    }
}
```

Here, the call `obj.test(5)` is ambiguous because:

* It matches `test(int)` exactly.
* It can also match `test(int...)`.

---

### **4. Ambiguity Due to Method Parameter Order**

When the same types are used in different orders in method signatures, it can create ambiguity.

#### **Example**

```java
class AmbiguityParameterOrder {
    void test(int a, double b) {
        System.out.println("Method with int and double");
    }

    void test(double a, int b) {
        System.out.println("Method with double and int");
    }

    public static void main(String[] args) {
        AmbiguityParameterOrder obj = new AmbiguityParameterOrder();
        obj.test(10, 20); // Ambiguous: Could match both methods
    }
}
```

Here, the call `obj.test(10, 20)` is ambiguous because:

* `10` can be matched to `int` or `double`.
* `20` can also be matched to `int` or `double`.

---

### **5. Ambiguity with Null Values**

When `null` is passed as an argument, and multiple overloaded methods can accept it, ambiguity arises.

#### **Example**

```java
class AmbiguityNull {
    void test(String s) {
        System.out.println("Method with String");
    }

    void test(Object o) {
        System.out.println("Method with Object");
    }

    public static void main(String[] args) {
        AmbiguityNull obj = new AmbiguityNull();
        obj.test(null); // Ambiguous: null can match String or Object
    }
}
```

Here, the call `obj.test(null)` is ambiguous because:

* `null` can match `String` (a subclass of `Object`).
* `null` can also match `Object`.

---

### **6. Ambiguity Due to Mixing Varargs with Other Parameters**

When multiple varargs methods or varargs with other parameters are used, the compiler may fail to resolve the call.

#### **Example**

```java
class AmbiguityMixedVarargs {
    void test(int... a) {
        System.out.println("Method with int varargs");
    }

    void test(String... a) {
        System.out.println("Method with String varargs");
    }

    public static void main(String[] args) {
        AmbiguityMixedVarargs obj = new AmbiguityMixedVarargs();
        obj.test(); // Ambiguous: Could match both int... and String...
    }
}
```

Here, the call `obj.test()` is ambiguous because:

* It could match `test(int...)`.
* It could also match `test(String...)`.

---

### **Resolution of Ambiguities**

To avoid ambiguity:

1. **Use explicit type casting**: Specify the type explicitly to match a specific method.
2. **Avoid method overloading with closely related parameter types**: Design method signatures to have clear distinctions.
3. **Prioritize parameter order clarity**: Avoid mixing parameter orders that can create confusion.
4. **Limit excessive varargs usage**: Use varargs sparingly and carefully in overloaded methods.

Would you like further clarification or examples on any of these cases?


### Chat : " **Use explicit type casting**: "also explain this point

### **Using Explicit Type Casting to Resolve Ambiguity**

Explicit type casting allows you to specify the exact type of a method argument to help the compiler decide which overloaded method to invoke. This is particularly useful in situations where ambiguity arises due to multiple possible matches.

---

### **Example: Resolving Ambiguity with Type Casting**

#### **Without Type Casting**

```java
class ExplicitCastingExample {
    void test(int a) {
        System.out.println("Method with int");
    }

    void test(double a) {
        System.out.println("Method with double");
    }

    public static void main(String[] args) {
        ExplicitCastingExample obj = new ExplicitCastingExample();

        obj.test(5);       // Calls test(int) (exact match for int)
        obj.test(5.5);     // Calls test(double) (exact match for double)

        obj.test(5L);      // Ambiguous: long can be widened to int or double
    }
}
```

#### **With Type Casting**

```java
class ExplicitCastingExample {
    void test(int a) {
        System.out.println("Method with int");
    }

    void test(double a) {
        System.out.println("Method with double");
    }

    public static void main(String[] args) {
        ExplicitCastingExample obj = new ExplicitCastingExample();

        obj.test((int) 5L);    // Resolves to test(int)
        obj.test((double) 5L); // Resolves to test(double)
    }
}
```

### **How It Works**

* **Before Type Casting**: The compiler tries to determine the best match. If multiple methods are valid (e.g., `int` and `double`), it results in ambiguity.
* **After Type Casting**: By explicitly casting `5L` to either `int` or `double`, you remove ambiguity by forcing the argument to match one specific method.

---

### **Why Is This Important?**

In many real-world applications, you might have overloaded methods with closely related parameter types. Ambiguity can result in compile-time errors, which can be avoided with explicit casting.

---

### **Example: Ambiguity with Autoboxing and Type Casting**

#### **Without Type Casting**

```java
class ExplicitCastingAutoboxing {
    void test(Integer a) {
        System.out.println("Method with Integer");
    }

    void test(long a) {
        System.out.println("Method with long");
    }

    public static void main(String[] args) {
        ExplicitCastingAutoboxing obj = new ExplicitCastingAutoboxing();

        obj.test(10);  // Ambiguous: 10 can be autoboxed to Integer or widened to long
    }
}
```

#### **With Type Casting**

```java
class ExplicitCastingAutoboxing {
    void test(Integer a) {
        System.out.println("Method with Integer");
    }

    void test(long a) {
        System.out.println("Method with long");
    }

    public static void main(String[] args) {
        ExplicitCastingAutoboxing obj = new ExplicitCastingAutoboxing();

        obj.test((Integer) 10); // Resolves to test(Integer)
        obj.test((long) 10);    // Resolves to test(long)
    }
}
```

---

### **When to Use Explicit Casting**

1. **Ambiguity due to widening**:

   * E.g., between `int`, `long`, and `double`.
2. **Ambiguity due to autoboxing**:

   * E.g., between primitive types and their wrapper classes.
3. **Ambiguity with null**:

   * When `null` can match multiple method signatures.

#### **Example: Resolving Ambiguity with Null**

```java
class ExplicitCastingNull {
    void test(String s) {
        System.out.println("Method with String");
    }

    void test(Object o) {
        System.out.println("Method with Object");
    }

    public static void main(String[] args) {
        ExplicitCastingNull obj = new ExplicitCastingNull();

        obj.test((String) null); // Resolves to test(String)
        obj.test((Object) null); // Resolves to test(Object)
    }
}
```

---

### **Advantages of Explicit Casting**

* **Clarity**: Makes the intended method call explicit and removes ambiguity.
* **Control**: Provides more control over which method is invoked.
* **Error Avoidance**: Prevents compile-time errors caused by ambiguous method calls.

If you'd like, I can explain more specific examples or edge cases where explicit casting plays a critical role.


### Chat : what are the rules of method overloading in case of exception and access modifiers

In Java, method overloading primarily focuses on method signatures (method name and parameters) and is unrelated to return type, access modifiers, or exceptions. However, there are rules and considerations regarding **access modifiers** and **exceptions** in method overloading:

---

### **Rules Regarding Access Modifiers in Method Overloading**

Access modifiers define the visibility of a method. Overloaded methods can have different access modifiers, as access modifiers are not considered when determining whether methods are overloaded.

#### **Key Points**

1. **Allowed**: You can change the access modifier in overloaded methods.

   * Each method can have its own access modifier.
   * The access modifier of one overloaded method does not affect another.

#### **Example**

```java
class AccessModifierExample {
    // Public access modifier
    public void test(int a) {
        System.out.println("Public method with int");
    }

    // Private access modifier
    private void test(String a) {
        System.out.println("Private method with String");
    }

    // Protected access modifier
    protected void test(double a) {
        System.out.println("Protected method with double");
    }
}

public class Main {
    public static void main(String[] args) {
        AccessModifierExample obj = new AccessModifierExample();
        obj.test(10);    // Public method
        obj.test(10.5);  // Protected method (accessible within the same package or subclass)
        // obj.test("Java"); // Cannot be accessed because it's private
    }
}
```

---

### **Rules Regarding Exceptions in Method Overloading**

Exceptions thrown by methods are not considered part of the method signature in Java. This means that overloaded methods can throw:

1. **No exceptions**.
2. **The same exceptions**.
3. **Different exceptions**.

#### **Key Points**

* The exception types (checked or unchecked) thrown by the methods do not impact method overloading.
* Each overloaded method can have its own exception declaration.
* If a method is overloaded, the exception handling is entirely dependent on the individual overloaded method.

#### **Example**

```java
class ExceptionExample {
    void process(int a) throws Exception {
        System.out.println("Method with int, throws Exception");
    }

    void process(String a) throws NullPointerException {
        System.out.println("Method with String, throws NullPointerException");
    }

    void process(double a) {
        System.out.println("Method with double, throws no exceptions");
    }
}

public class Main {
    public static void main(String[] args) {
        ExceptionExample obj = new ExceptionExample();

        try {
            obj.process(10); // Throws Exception
        } catch (Exception e) {
            System.out.println("Caught Exception");
        }

        obj.process(10.5); // No exception handling required

        try {
            obj.process("Java"); // Throws NullPointerException
        } catch (NullPointerException e) {
            System.out.println("Caught NullPointerException");
        }
    }
}
```

---

### **Key Differences from Method Overriding**

To avoid confusion, here's how access modifiers and exceptions behave in **method overriding** (not overloading):

* **Access Modifiers in Overriding**:

  * The overriding method **cannot reduce the visibility** of the parent method.
  * For example, if the parent method is `public`, the overriding method must also be `public`.

* **Exceptions in Overriding**:

  * The overriding method **cannot throw broader checked exceptions** than the overridden method.
  * For example, if the parent method throws `IOException`, the overriding method cannot throw `Exception`.

In contrast, for **overloading**, there are **no such restrictions** on access modifiers or exceptions.

---

### **Summary Table**

| Feature              | Method Overloading        | Method Overriding                        |
| -------------------- | ------------------------- | ---------------------------------------- |
| **Access Modifiers** | Can vary between methods. | Cannot reduce visibility.                |
| **Exceptions**       | Can vary between methods. | Cannot throw broader checked exceptions. |

If you want examples of specific scenarios or edge cases, let me know!


### Chat : "For example, if the parent method is `public`, the overriding method must also be `public`. "so if the parent method is private can we keep child method as public

If the parent method is **private**, it is not visible to the child class. As a result:

* A **private** method in the parent class cannot be overridden in the child class because it is not inherited.
* If a method with the same name and signature is defined in the child class, it is considered a completely **new method** and not an overriding of the parent class method.
* The child class can define this new method with **any access modifier**, including `public`.

---

### **Example: Parent Method is Private**

```java
class Parent {
    private void display() {
        System.out.println("Parent's private method");
    }
}

class Child extends Parent {
    @Override
    public void display() { // This is a new method, not an override.
        System.out.println("Child's public method");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent parent = new Parent();
        // parent.display(); // Error: display() is private in Parent

        Child child = new Child();
        child.display(); // Calls Child's public method
    }
}
```

---

### **Key Points**

1. **Private methods are not inherited**: Since private methods are confined to the class they are defined in, they cannot be overridden.
2. **Child method is independent**: The method in the child class with the same name and signature is unrelated to the parent method.
3. **Access modifier in the child**: The new method in the child class can have a broader access modifier (like `public`), as it is not bound by the access modifier of the parent method.

---

### **What Happens Internally?**

When a method in the child class has the same name and parameters as a private method in the parent:

* The private method in the parent class remains hidden.
* The new method in the child class becomes part of the child class's definition, entirely independent of the parent class's private method.

Let me know if you'd like further clarification or deeper examples!


### Chat : what is parent is default or protected

If a **parent method** is marked as **default** (package-private) or **protected**, the rules for overriding in the child class are as follows:

---

### **1. Parent Method is `default` (Package-Private)**

#### **Rules**:

* A `default` method in the parent class is accessible only within the **same package**.
* If the child class is in the **same package** as the parent:

  * The `default` method can be **overridden** in the child class.
  * The child class can use the same or a **broader access modifier** (`protected` or `public`) for the overriding method.
* If the child class is in a **different package**:

  * The `default` method is **not accessible** in the child class and hence **cannot be overridden**.
  * If a method with the same name and parameters is defined in the child class, it will be a **new method**, not an override.

---

#### **Example: Parent Method is `default`**

**Case 1: Parent and Child in the Same Package**

```java
package samepackage;

class Parent {
    void display() { // Default method
        System.out.println("Parent's default method");
    }
}

public class Child extends Parent {
    @Override
    public void display() { // Overriding with broader access modifier
        System.out.println("Child's public method");
    }

    public static void main(String[] args) {
        Parent parent = new Parent();
        parent.display(); // Calls Parent's method

        Child child = new Child();
        child.display(); // Calls Child's method
    }
}
```

**Case 2: Parent and Child in Different Packages**

```java
// In package "parentpkg"
package parentpkg;

public class Parent {
    void display() { // Default method
        System.out.println("Parent's default method");
    }
}

// In package "childpkg"
package childpkg;

import parentpkg.Parent;

public class Child extends Parent {
    @Override
    public void display() { // This will NOT override Parent's method
        System.out.println("Child's public method");
    }

    public static void main(String[] args) {
        Child child = new Child();
        child.display(); // Calls Child's method (new, not overriding)
    }
}
```

In this case, `display()` in `Parent` is not accessible in `Child` because it's default (package-private), so the `display()` in `Child` is a **new method**.

---

### **2. Parent Method is `protected`**

#### **Rules**:

* A `protected` method in the parent class is accessible:

  * In the **same package**.
  * In a **different package**, but only through **inheritance**.
* A `protected` method can be **overridden** in the child class, regardless of whether the child is in the same or a different package.
* The child class can use the same or a **broader access modifier** (`public`) for the overriding method but cannot use a **narrower modifier** (like `default` or `private`).

---

#### **Example: Parent Method is `protected`**

**Case 1: Parent and Child in the Same Package**

```java
package samepackage;

class Parent {
    protected void display() {
        System.out.println("Parent's protected method");
    }
}

public class Child extends Parent {
    @Override
    public void display() { // Overriding with broader access modifier
        System.out.println("Child's public method");
    }

    public static void main(String[] args) {
        Parent parent = new Parent();
        parent.display(); // Calls Parent's method

        Child child = new Child();
        child.display(); // Calls Child's method
    }
}
```

**Case 2: Parent and Child in Different Packages**

```java
// In package "parentpkg"
package parentpkg;

public class Parent {
    protected void display() {
        System.out.println("Parent's protected method");
    }
}

// In package "childpkg"
package childpkg;

import parentpkg.Parent;

public class Child extends Parent {
    @Override
    public void display() { // Overriding with public access
        System.out.println("Child's public method");
    }

    public static void main(String[] args) {
        Child child = new Child();
        child.display(); // Calls Child's method
    }
}
```

---

### **Summary Table**

| **Parent Modifier** | **Visibility to Child in Same Package** | **Visibility to Child in Different Package** | **Overridable?**      | **Child's Access Modifier**             |
| ------------------- | --------------------------------------- | -------------------------------------------- | --------------------- | --------------------------------------- |
| **private**         | Not accessible                          | Not accessible                               | No                    | Independent method                      |
| **default**         | Accessible                              | Not accessible                               | Yes (in same package) | Same or broader (`protected`, `public`) |
| **protected**       | Accessible                              | Accessible (via inheritance)                 | Yes                   | Same or broader (`public`)              |
| **public**          | Accessible                              | Accessible                                   | Yes                   | Same or broader (`public`)              |

Let me know if you'd like further clarification or more examples!


## **Conversation**

### Chat : Method overloading concept 

**Method Overloading** is a concept in object-oriented programming (OOP), particularly in Java, where multiple methods in the same class have the same name but different parameter lists. It allows methods to perform similar functionalities with variations in input types or number of arguments.

### Key Features of Method Overloading:

1. **Same Name:** All methods share the same name.
2. **Different Parameters:** Differentiation is based on:

   * Number of parameters.
   * Type of parameters.
   * Order of parameters.
3. **Compile-Time Polymorphism:** Method overloading is resolved during compile time, also called **static binding**.

---

### Example of Method Overloading in Java:

```java
public class Calculator {

    // Method to add two integers
    public int add(int a, int b) {
        return a + b;
    }

    // Method to add three integers
    public int add(int a, int b, int c) {
        return a + b + c;
    }

    // Method to add two double values
    public double add(double a, double b) {
        return a + b;
    }

    public static void main(String[] args) {
        Calculator calc = new Calculator();

        System.out.println(calc.add(10, 20));         // Calls add(int, int)
        System.out.println(calc.add(10, 20, 30));    // Calls add(int, int, int)
        System.out.println(calc.add(10.5, 20.5));    // Calls add(double, double)
    }
}
```

---

### Output:

```
30
60
31.0
```

### Advantages of Method Overloading:

1. **Improves Code Readability:** Methods with similar names and different parameters simplify understanding.
2. **Reusability:** No need to define new method names for similar functionality.
3. **Flexibility:** Enables code to handle various input types or counts.

### Rules to Remember:

1. Method overloading must differ in the parameter list.
2. Return type alone does not differentiate overloaded methods.
3. You can overload methods within the same class or between parent and child classes.


### Chat : ?

In the provided code, the `sum` method is overloaded with three different parameter types: `Integer`, `Object`, and `String`. When calling `sum(null)`:

### Key Points to Consider:

1. `null` is a valid value that can match any reference type, so it could match `Object`, `String`, or `Integer` (which is a wrapper class for `int`).
2. **Most Specific Match Rule:** When multiple methods can be invoked, Java selects the most specific method. A `String` is more specific than `Object`, and `Integer` is more specific than both `Object` and `String`.
3. If there is ambiguity (e.g., two methods are equally specific), a **compilation error** occurs.

---

### What Happens in This Code?

* `null` could match any of the three parameter types (`Object`, `String`, `Integer`).
* Since both `String` and `Integer` are subclasses of `Object`, the most specific match will be chosen.
* However, `Integer` and `String` are unrelated classes, and neither is more specific than the other for `null`.

---

### Output:

The code results in a **compilation error** due to ambiguity. The Java compiler cannot decide whether to invoke `sum(Integer)` or `sum(String)`.

---

### Solution:

To resolve the ambiguity, explicitly cast `null` to the desired type. For example:

```java
new SumofNumber().sum((String) null); // Calls the String version
new SumofNumber().sum((Integer) null); // Calls the Integer version
```


### Chat : Method overriding thows rules 

In Java, when you **override** a method, there are specific rules related to the use of `throws` (exceptions). Here's a breakdown of those rules:

---

### **1. You can throw fewer or narrower checked exceptions**

```java
class Parent {
    void show() throws IOException {}
}

class Child extends Parent {
    @Override
    void show() throws FileNotFoundException {} // OK: narrower exception
}
```

* `FileNotFoundException` is a subclass of `IOException`, so it's allowed.

---

### **2. You cannot throw broader or new checked exceptions**

```java
class Parent {
    void show() throws IOException {}
}

class Child extends Parent {
    @Override
    void show() throws Exception {} // ERROR: broader exception
}
```

* `Exception` is broader than `IOException`, so this will cause a compile-time error.

---

### **3. You can throw any unchecked (runtime) exceptions**

```java
class Parent {
    void show() {}
}

class Child extends Parent {
    @Override
    void show() throws ArithmeticException {} // OK: unchecked exception
}
```

* Runtime exceptions like `NullPointerException`, `ArithmeticException`, etc., are allowed freely.

---

### **4. If superclass method does not throw a checked exception, the overriding method cannot either**

```java
class Parent {
    void display() {}
}

class Child extends Parent {
    @Override
    void display() throws IOException {} // ERROR: checked exception not allowed
}
```

* You cannot introduce new checked exceptions.

---

### Summary Table:

| Superclass throws   | Subclass override can throw  |
| ------------------- | ---------------------------- |
| No exception        | Only unchecked exceptions    |
| Checked exception A | Same, subclass of A, or none |
| Unchecked exception | Any (no restriction)         |

Let me know if you want an example involving interfaces or abstract classes too.


