# Object-Oriented Programming in Java - Complete Interview Guide 2026

## Table of Contents (Click to Navigate)

### [Part 1: Classes and Objects](#part-1-classes-and-objects)
- [1.1 What is a Class?](#11-what-is-a-class)
- [1.2 What is an Object?](#12-what-is-an-object)
- [1.3 Creating Classes and Objects](#13-creating-classes-and-objects)

### [Part 2: Fields and Methods](#part-2-fields-and-methods)
- [2.1 Instance Variables (Fields)](#21-instance-variables-fields)
- [2.2 Methods](#22-methods)
- [2.3 Method Parameters and Return Types](#23-method-parameters-and-return-types)

### [Part 3: Constructors](#part-3-constructors)
- [3.1 Default Constructor](#31-default-constructor)
- [3.2 Parameterized Constructor](#32-parameterized-constructor)
- [3.3 Constructor Overloading](#33-constructor-overloading)
- [3.4 this() and this Reference](#34-this-and-this-reference)

### [Part 4: Initialization Blocks](#part-4-initialization-blocks)
- [4.1 Instance Initialization Blocks](#41-instance-initialization-blocks)
- [4.2 Static Initialization Blocks](#42-static-initialization-blocks)
- [4.3 Execution Order](#43-execution-order)

### [Part 5: Encapsulation](#part-5-encapsulation)
- [5.1 Data Hiding](#51-data-hiding)
- [5.2 Getters and Setters](#52-getters-and-setters)
- [5.3 Benefits of Encapsulation](#53-benefits-of-encapsulation)

### [Part 6: Inheritance](#part-6-inheritance)
- [6.1 Single Inheritance](#61-single-inheritance)
- [6.2 Multilevel Inheritance](#62-multilevel-inheritance)
- [6.3 Hierarchical Inheritance](#63-hierarchical-inheritance)
- [6.4 super Keyword](#64-super-keyword)
- [6.5 Method Overriding](#65-method-overriding)

### [Part 7: Polymorphism](#part-7-polymorphism)
- [7.1 Compile-Time Polymorphism (Overloading)](#71-compile-time-polymorphism-overloading)
- [7.2 Runtime Polymorphism (Overriding)](#72-runtime-polymorphism-overriding)
- [7.3 Overloading vs Overriding](#73-overloading-vs-overriding)

### [Part 8: Abstraction](#part-8-abstraction)
- [8.1 Abstract Classes](#81-abstract-classes)
- [8.2 Interfaces](#82-interfaces)
- [8.3 Abstract Classes vs Interfaces](#83-abstract-classes-vs-interfaces)

### [Part 9: IS-A vs HAS-A](#part-9-is-a-vs-has-a)
- [9.1 IS-A Relationship (Inheritance)](#91-is-a-relationship-inheritance)
- [9.2 HAS-A Relationship (Composition)](#92-has-a-relationship-composition)
- [9.3 Choosing Between IS-A and HAS-A](#93-choosing-between-is-a-and-has-a)

### [Part 10: Access Modifiers](#part-10-access-modifiers)
- [10.1 Private](#101-private)
- [10.2 Default (Package-Private)](#102-default-package-private)
- [10.3 Protected](#103-protected)
- [10.4 Public](#104-public)

### [Part 11: Non-Access Modifiers](#part-11-non-access-modifiers)
- [11.1 static](#111-static)
- [11.2 final](#112-final)
- [11.3 abstract](#113-abstract)
- [11.4 synchronized](#114-synchronized)
- [11.5 transient](#115-transient)
- [11.6 volatile](#116-volatile)

### [Part 12: Inner and Nested Classes](#part-12-inner-and-nested-classes)
- [12.1 Member Inner Class](#121-member-inner-class)
- [12.2 Static Nested Class](#122-static-nested-class)
- [12.3 Local Inner Class](#123-local-inner-class)
- [12.4 Anonymous Inner Class](#124-anonymous-inner-class)

### [Part 13: Interfaces](#part-13-interfaces)
- [13.1 Interface Basics](#131-interface-basics)
- [13.2 Functional Interfaces](#132-functional-interfaces)
- [13.3 Default Methods](#133-default-methods)
- [13.4 Static Methods in Interfaces](#134-static-methods-in-interfaces)

### [Part 14: Interview Traps & Best Practices](#part-14-interview-traps--best-practices)
- [14.1 Common Interview Traps](#141-common-interview-traps)
- [14.2 Best Practices](#142-best-practices)

### [Part 15: Experienced-Level Q&A (4+ Years)](#part-15-experienced-level-qa-4-years)
- [15.1 Advanced OOP Concepts](#151-advanced-oop-concepts)
- [15.2 Design Patterns & Architecture](#152-design-patterns--architecture)
- [15.3 Real-World Scenarios](#153-real-world-scenarios)

---

## Part 1: Classes and Objects

### 1.1 What is a Class?

A **class** is a blueprint or template for creating objects. It defines the structure (fields) and behavior (methods) that objects will have.

**Memory Tip:** "Class = Blueprint, Object = House built from blueprint"

**Key Characteristics:**
- Logical entity (exists only in code)
- Defines properties and behaviors
- Can have multiple objects

### 1.2 What is an Object?

An **object** is an instance of a class. It has actual values for the fields defined in the class.

**Memory Tip:** "Object = Real Thing with actual values"

**Key Characteristics:**
- Physical entity (occupies memory)
- Has state (field values) and behavior (methods)
- Each object is independent

### 1.3 Creating Classes and Objects

```java
// Class definition
public class Car {
    // Fields (state)
    String color;
    String model;
    
    // Method (behavior)
    void drive() {
        System.out.println("Car is driving");
    }
}

// Creating objects
Car car1 = new Car();  // Object 1
Car car2 = new Car();  // Object 2 (different from car1)
```

**Interview Trap:**
- **Q:** "Are class and object the same?"
- **A:** No. Class is a template; object is an instance of that template.

---

## Part 2: Fields and Methods

### 2.1 Instance Variables (Fields)

Instance variables store the state of an object. Each object has its own copy.

```java
public class Student {
    String name;      // Instance variable
    int age;          // Instance variable
    double gpa;
    
    public static void main(String[] args) {
        Student s1 = new Student();
        s1.name = "Alice";  // s1's own copy
        s1.age = 20;
        
        Student s2 = new Student();
        s2.name = "Bob";    // s2's own copy
        // s1.name and s2.name are independent
    }
}
```

**Memory Tip:** "Instance variables = Per-object storage"

### 2.2 Methods

Methods define the behavior of a class. They perform actions on the object's state.

```java
public class Calculator {
    // Method without parameters, no return
    void showMessage() {
        System.out.println("Hello!");
    }
    
    // Method with parameters, with return
    int add(int a, int b) {
        return a + b;
    }
    
    // Method with multiple parameters
    double multiply(double x, double y) {
        return x * y;
    }
}
```

### 2.3 Method Parameters and Return Types

```java
public class Demo {
    // No parameters, no return
    void greet() {
        System.out.println("Hi");
    }
    
    // Parameters but no return
    void printSum(int a, int b) {
        System.out.println(a + b);
    }
    
    // Return but no parameters
    int getValue() {
        return 42;
    }
    
    // Parameters and return
    String concatenate(String a, String b) {
        return a + b;
    }
}
```

---

## Part 3: Constructors

A **constructor** is a special method that initializes objects. It's called automatically when an object is created.

**Memory Tip:** "Constructor = Initializer called at object birth"

### 3.1 Default Constructor

If you don't define any constructor, Java provides a default no-arg constructor.

```java
public class Person {
    String name;
    
    // Implicit default constructor (if not written)
    // public Person() { }
    
    public static void main(String[] args) {
        Person p = new Person();  // Default constructor called
        System.out.println(p.name);  // null (default value)
    }
}
```

### 3.2 Parameterized Constructor

Initialize objects with specific values:

```java
public class Person {
    String name;
    int age;
    
    // Parameterized constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public static void main(String[] args) {
        Person p = new Person("Alice", 25);
        System.out.println(p.name);  // Alice
        System.out.println(p.age);   // 25
    }
}
```

### 3.3 Constructor Overloading

Multiple constructors with different parameters:

```java
public class Rectangle {
    int length, width;
    
    // Constructor 1: No arguments
    public Rectangle() {
        length = 0;
        width = 0;
    }
    
    // Constructor 2: One argument
    public Rectangle(int side) {
        length = side;
        width = side;
    }
    
    // Constructor 3: Two arguments
    public Rectangle(int l, int w) {
        length = l;
        width = w;
    }
    
    public static void main(String[] args) {
        Rectangle r1 = new Rectangle();           // Calls Constructor 1
        Rectangle r2 = new Rectangle(5);          // Calls Constructor 2
        Rectangle r3 = new Rectangle(5, 10);      // Calls Constructor 3
    }
}
```

### 3.4 this() and this Reference

**this reference:** Points to the current object instance.

**this():** Calls another constructor in the same class.

```java
public class Book {
    String title;
    String author;
    
    // Constructor 1
    public Book() {
        this("Unknown", "Unknown");  // Calls Constructor 2
    }
    
    // Constructor 2
    public Book(String title, String author) {
        this.title = title;
        this.author = author;
    }
    
    public static void main(String[] args) {
        Book b1 = new Book();
        System.out.println(b1.title);  // Unknown
        System.out.println(b1.author); // Unknown
    }
}
```

**Interview Trap:**
- **Q:** "Can this() be used in static methods?"
- **A:** No. `this` refers to an instance; static methods don't have an instance.

---

## Part 4: Initialization Blocks

### 4.1 Instance Initialization Blocks

Code executed every time an object is created, before the constructor.

```java
public class Demo {
    int x;
    
    // Instance initialization block
    {
        x = 10;
        System.out.println("Instance block executed");
    }
    
    public Demo() {
        System.out.println("Constructor executed");
    }
    
    public static void main(String[] args) {
        Demo d = new Demo();
        // Output:
        // Instance block executed
        // Constructor executed
    }
}
```

### 4.2 Static Initialization Blocks

Code executed once when the class is loaded, before any object creation.

```java
public class Config {
    static String path;
    
    // Static initialization block
    static {
        path = "/default/path";
        System.out.println("Static block executed");
    }
    
    public static void main(String[] args) {
        System.out.println("Main executed");
        System.out.println(path);
        // Output:
        // Static block executed
        // Main executed
        // /default/path
    }
}
```

### 4.3 Execution Order

```
1. Static blocks (when class loads) - once per JVM
2. Instance blocks (when object created) - every time
3. Constructor (when object created) - every time
```

**Memory Tip:** "Static = Once per class, Instance = Once per object, Constructor = Last step"

---

## Part 5: Encapsulation

**Encapsulation** = Data Hiding + Controlled Access

Bundle data (variables) and methods together, and hide internal details from the outside world.

### 5.1 Data Hiding

```java
// Bad: Direct access to fields
public class BankAccount {
    public double balance;  // Exposed
}

// Good: Hide with private
public class BankAccount {
    private double balance;  // Hidden
}
```

### 5.2 Getters and Setters

Provide controlled access through methods:

```java
public class BankAccount {
    private double balance;
    
    // Getter
    public double getBalance() {
        return balance;
    }
    
    // Setter with validation
    public void setBalance(double amount) {
        if (amount >= 0) {
            balance = amount;
        } else {
            System.out.println("Invalid amount");
        }
    }
    
    public static void main(String[] args) {
        BankAccount acc = new BankAccount();
        acc.setBalance(1000);          // Valid
        acc.setBalance(-500);          // Invalid
        System.out.println(acc.getBalance());
    }
}
```

### 5.3 Benefits of Encapsulation

✅ **Data Protection:** Control how data is modified  
✅ **Flexibility:** Change internal implementation without affecting external code  
✅ **Validation:** Ensure valid values are set  
✅ **Read-only/Write-only:** Create fields that cannot be modified or read  

**Memory Tip:** "Encapsulation = Private fields + Public getters/setters"

---

## Part 6: Inheritance

**Inheritance** allows one class to acquire properties and methods of another class.

**Memory Tip:** "Inheritance = Child inherits from Parent"

### 6.1 Single Inheritance

One child class extends one parent class:

```java
// Parent class
public class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

// Child class
public class Dog extends Animal {
    void bark() {
        System.out.println("Woof!");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.eat();   // Inherited from Animal
        dog.bark();  // Own method
    }
}
```

### 6.2 Multilevel Inheritance

Grandparent → Parent → Child:

```java
public class Vehicle {
    void drive() {
        System.out.println("Driving");
    }
}

public class Car extends Vehicle {
    void honk() {
        System.out.println("Honk!");
    }
}

public class Tesla extends Car {
    void autoDriver() {
        System.out.println("Auto pilot on");
    }
}

public class Main {
    public static void main(String[] args) {
        Tesla t = new Tesla();
        t.drive();        // From Vehicle
        t.honk();         // From Car
        t.autoDriver();   // Own method
    }
}
```

### 6.3 Hierarchical Inheritance

One parent, multiple children:

```java
public class Shape {
    void draw() {
        System.out.println("Drawing shape");
    }
}

public class Circle extends Shape {
    void calculateArea() {
        System.out.println("Circle area");
    }
}

public class Rectangle extends Shape {
    void calculateArea() {
        System.out.println("Rectangle area");
    }
}
```

### 6.4 super Keyword

Access parent class methods and constructor:

```java
public class Parent {
    void display() {
        System.out.println("Parent display");
    }
}

public class Child extends Parent {
    void display() {
        super.display();  // Call parent's display
        System.out.println("Child display");
    }
}

public class Main {
    public static void main(String[] args) {
        Child c = new Child();
        c.display();
        // Output:
        // Parent display
        // Child display
    }
}
```

**super in Constructor:**

```java
public class Parent {
    String name;
    public Parent(String name) {
        this.name = name;
    }
}

public class Child extends Parent {
    int age;
    public Child(String name, int age) {
        super(name);  // Call parent constructor
        this.age = age;
    }
}
```

### 6.5 Method Overriding

Child class redefines a parent method:

```java
public class Animal {
    void sound() {
        System.out.println("Generic sound");
    }
}

public class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Woof!");
    }
}

public class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Meow!");
    }
}
```

**Interview Trap:**
- **Q:** "Can you override static methods?"
- **A:** No. Static methods are hidden, not overridden. Child class can only hide them.

---

## Part 7: Polymorphism

**Polymorphism** = "Many forms" - Same method name, different behaviors.

### 7.1 Compile-Time Polymorphism (Overloading)

Same method name, different parameters. Decided at compile time.

```java
public class MathOperations {
    // Overload 1: Two integers
    public int add(int a, int b) {
        return a + b;
    }
    
    // Overload 2: Three integers
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    // Overload 3: Two doubles
    public double add(double a, double b) {
        return a + b;
    }
    
    public static void main(String[] args) {
        MathOperations m = new MathOperations();
        System.out.println(m.add(5, 10));         // Calls Overload 1
        System.out.println(m.add(5, 10, 15));     // Calls Overload 2
        System.out.println(m.add(5.5, 10.5));     // Calls Overload 3
    }
}
```

**Overloading Rules:**
- Different number of parameters
- Different types of parameters
- Different order of parameters

### 7.2 Runtime Polymorphism (Overriding)

Same method in parent and child. Decided at runtime based on actual object type.

```java
public class Animal {
    void sound() {
        System.out.println("Some sound");
    }
}

public class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Woof!");
    }
}

public class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Meow!");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal a1 = new Dog();
        Animal a2 = new Cat();
        
        a1.sound();  // Woof! (Runtime decision: actual type is Dog)
        a2.sound();  // Meow! (Runtime decision: actual type is Cat)
    }
}
```

### 7.3 Overloading vs Overriding

| Aspect | Overloading | Overriding |
|--------|-----------|-----------|
| Timing | Compile-time | Runtime |
| Scope | Same class | Parent-child |
| Method name | Same | Same |
| Parameters | Different | Same |
| Return type | Can differ | Must be same/subtype |
| Access modifier | Can differ | Cannot be more restrictive |

**Memory Tip:** "Overloading = Same name different params, Overriding = Child replaces parent"

---

## Part 8: Abstraction

**Abstraction** = Hide complex internal details, show only essential features.

### 8.1 Abstract Classes

Classes with abstract methods (no implementation):

```java
// Abstract class
public abstract class Vehicle {
    abstract void start();  // No implementation
    
    void stop() {
        System.out.println("Vehicle stopped");  // Has implementation
    }
}

// Concrete class
public class Car extends Vehicle {
    @Override
    void start() {
        System.out.println("Car engine starts");
    }
}

public class Main {
    public static void main(String[] args) {
        // Vehicle v = new Vehicle();  // ERROR: Cannot instantiate abstract class
        Vehicle car = new Car();
        car.start();  // Car engine starts
        car.stop();   // Vehicle stopped
    }
}
```

### 8.2 Interfaces

Contract: Classes must implement these methods.

```java
// Interface
public interface Shape {
    double area();
    double perimeter();
}

// Implementation 1
public class Circle implements Shape {
    double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }
}

// Implementation 2
public class Rectangle implements Shape {
    double length, width;
    
    public Rectangle(double l, double w) {
        length = l;
        width = w;
    }
    
    @Override
    public double area() {
        return length * width;
    }
    
    @Override
    public double perimeter() {
        return 2 * (length + width);
    }
}
```

### 8.3 Abstract Classes vs Interfaces

| Feature | Abstract Class | Interface |
|---------|---|---|
| Instantiation | Cannot instantiate | Cannot instantiate |
| Methods | Can have implementation | Only contracts (Java 8+ has default methods) |
| Fields | Any access modifier | public static final |
| Constructor | Can have | Cannot have |
| Multiple inheritance | No | Yes |
| Use case | IS-A relationship | CAN-DO contract |

**Memory Tip:** "Abstract = Extended, Interface = Implemented"

---

## Part 9: IS-A vs HAS-A

### 9.1 IS-A Relationship (Inheritance)

When a child "is a" type of parent:

```java
public class Animal {
    void sleep() {
        System.out.println("Sleeping");
    }
}

public class Dog extends Animal {
    // Dog IS-A Animal
    void bark() {
        System.out.println("Woof!");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.sleep();  // Inherited
        dog.bark();   // Own
    }
}
```

### 9.2 HAS-A Relationship (Composition)

When a class contains another class as a field:

```java
public class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

public class Car {
    Engine engine;  // Car HAS-A Engine
    
    public Car() {
        engine = new Engine();
    }
    
    void drive() {
        engine.start();
        System.out.println("Car driving");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.drive();
    }
}
```

### 9.3 Choosing Between IS-A and HAS-A

- **Use IS-A (Inheritance):** When there's a true hierarchical relationship
- **Use HAS-A (Composition):** When one object is part of another, or for flexibility

**Interview Tip:** Composition is often preferred over inheritance because it's more flexible and avoids tight coupling.

---

## Part 10: Access Modifiers

Control visibility of classes, methods, and fields.

| Modifier | Same Class | Same Package | Child Class | Outside |
|----------|-----------|-------------|-----------|---------|
| `private` | ✅ | ❌ | ❌ | ❌ |
| `default` | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

### 10.1 Private

Accessible only within the same class:

```java
public class BankAccount {
    private double balance;  // Only this class can access
    
    public void deposit(double amount) {
        balance += amount;  // OK: within the same class
    }
    
    public double getBalance() {
        return balance;
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount acc = new BankAccount();
        // acc.balance = 1000;  // ERROR: cannot access private field
        acc.deposit(1000);  // OK: public method
    }
}
```

### 10.2 Default (Package-Private)

Accessible within the same package only (no keyword):

```java
// File: com/example/Parent.java
package com.example;

public class Parent {
    int value = 10;  // Default: accessible in same package
}

// File: com/example/Child.java
package com.example;

public class Child {
    public static void main(String[] args) {
        Parent p = new Parent();
        System.out.println(p.value);  // OK: same package
    }
}

// File: com/other/Outside.java
package com.other;

public class Outside {
    public static void main(String[] args) {
        com.example.Parent p = new com.example.Parent();
        // System.out.println(p.value);  // ERROR: different package
    }
}
```

### 10.3 Protected

Accessible within same package and child classes outside the package:

```java
// File: com/parent/Parent.java
package com.parent;

public class Parent {
    protected int value = 10;
}

// File: com/child/Child.java
package com.child;
import com.parent.Parent;

public class Child extends Parent {
    public void display() {
        System.out.println(value);  // OK: child class
    }
}

// File: com/other/Other.java
package com.other;
import com.parent.Parent;

public class Other {
    public static void main(String[] args) {
        Parent p = new Parent();
        // System.out.println(p.value);  // ERROR: not a child class
    }
}
```

### 10.4 Public

Accessible from everywhere:

```java
public class PublicDemo {
    public int x = 10;  // Accessible everywhere
    
    public void display() {
        System.out.println(x);
    }
}
```

**Interview Trap:**
- **Q:** "What's the default access modifier?"
- **A:** Package-private (no keyword). Variables/methods are accessible in the same package.

---

## Part 11: Non-Access Modifiers

### 11.1 static

Belongs to the class, not instances. Shared by all objects.

```java
public class Counter {
    static int count = 0;  // Shared by all objects
    int id;
    
    public Counter() {
        count++;
        id = count;
    }
    
    public static void main(String[] args) {
        Counter c1 = new Counter();
        Counter c2 = new Counter();
        Counter c3 = new Counter();
        
        System.out.println(Counter.count);  // 3 (shared)
        System.out.println(c1.id);         // 1 (individual)
        System.out.println(c2.id);         // 2 (individual)
    }
}
```

**Static Methods:**

```java
public class MathUtils {
    static int add(int a, int b) {
        return a + b;  // No need to create object
    }
}

public class Main {
    public static void main(String[] args) {
        int result = MathUtils.add(5, 10);  // Call without creating object
        System.out.println(result);  // 15
    }
}
```

**Memory Tip:** "static = Belongs to class, shared by all objects"

### 11.2 final

Cannot be modified after assignment:

**Final Variables:**

```java
public class Constants {
    final int MAX_USERS = 100;  // Cannot change
    
    public void test() {
        MAX_USERS = 200;  // ERROR
    }
}
```

**Final Methods:**

Cannot be overridden:

```java
public class Parent {
    final void importantMethod() {
        System.out.println("Cannot override");
    }
}

public class Child extends Parent {
    void importantMethod() {  // ERROR: cannot override final method
    }
}
```

**Final Classes:**

Cannot be extended:

```java
final public class Immutable {
    // Cannot be extended
}

public class Child extends Immutable {  // ERROR
}
```

### 11.3 abstract

Class or method without full implementation:

```java
public abstract class Shape {
    abstract void draw();  // No implementation
    
    void showInfo() {
        System.out.println("I am a shape");
    }
}
```

### 11.4 synchronized

For thread safety in multi-threaded environments:

```java
public class ThreadSafeCounter {
    int count = 0;
    
    synchronized void increment() {
        count++;  // Only one thread at a time
    }
}
```

### 11.5 transient

Field not serialized (not saved):

```java
public class User {
    String name;
    transient String password;  // Not serialized
}
```

### 11.6 volatile

Variable value always read from main memory (not cached):

```java
public class Flag {
    volatile boolean running = true;  // Always fresh value
}
```

---

## Part 12: Inner and Nested Classes

### 12.1 Member Inner Class

Non-static inner class, has access to outer class members:

```java
public class Outer {
    private int x = 10;
    
    public class Inner {
        void display() {
            System.out.println("Outer's x = " + x);  // Can access outer's private members
        }
    }
    
    public static void main(String[] args) {
        Outer o = new Outer();
        Outer.Inner i = o.new Inner();  // Syntax: outer.new Inner()
        i.display();
    }
}
```

### 12.2 Static Nested Class

Static inner class, does NOT have access to outer instance members:

```java
public class Outer {
    private int x = 10;
    static int y = 20;
    
    static class StaticInner {
        void display() {
            // System.out.println(x);  // ERROR: cannot access non-static outer member
            System.out.println(y);  // OK: static member
        }
    }
    
    public static void main(String[] args) {
        Outer.StaticInner si = new Outer.StaticInner();
        si.display();
    }
}
```

### 12.3 Local Inner Class

Defined inside a method, scope limited to that method:

```java
public class Outer {
    void myMethod() {
        class LocalInner {
            void display() {
                System.out.println("Local inner class");
            }
        }
        
        LocalInner li = new LocalInner();
        li.display();
    }
    
    public static void main(String[] args) {
        Outer o = new Outer();
        o.myMethod();
    }
}
```

### 12.4 Anonymous Inner Class

Class without a name, typically implementing an interface:

```java
// Without anonymous inner class
interface Greeting {
    void greet();
}

class GreetingImpl implements Greeting {
    public void greet() {
        System.out.println("Hello!");
    }
}

// With anonymous inner class (cleaner)
public class Main {
    public static void main(String[] args) {
        Greeting g = new Greeting() {
            @Override
            public void greet() {
                System.out.println("Hello!");
            }
        };
        
        g.greet();
    }
}
```

**Memory Tip:** "Anonymous = Class without name, usually one-time use"

---

## Part 13: Interfaces

### 13.1 Interface Basics

Contract for classes to implement:

```java
public interface Animal {
    void eat();      // Abstract method (no implementation)
    void sleep();
}

public class Dog implements Animal {
    @Override
    public void eat() {
        System.out.println("Dog eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Dog sleeping");
    }
}
```

**Interface Fields:**

```java
public interface Config {
    String APP_NAME = "MyApp";  // Implicitly: public static final
    int MAX_USERS = 100;
}
```

### 13.2 Functional Interfaces

Interface with exactly ONE abstract method. Can be used with lambda expressions:

```java
@FunctionalInterface
public interface Calculator {
    int add(int a, int b);
}

public class Main {
    public static void main(String[] args) {
        // Traditional
        Calculator c1 = new Calculator() {
            public int add(int a, int b) {
                return a + b;
            }
        };
        
        // Lambda (cleaner)
        Calculator c2 = (a, b) -> a + b;
        
        System.out.println(c1.add(5, 10));  // 15
        System.out.println(c2.add(5, 10));  // 15
    }
}
```

### 13.3 Default Methods

Interfaces can have implemented methods (Java 8+):

```java
public interface Vehicle {
    void start();
    
    default void stop() {
        System.out.println("Vehicle stopped");
    }
}

public class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }
    // stop() method inherited (no need to override)
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.start();  // Car started
        car.stop();   // Vehicle stopped
    }
}
```

### 13.4 Static Methods in Interfaces

Static methods in interfaces (Java 8+):

```java
public interface StringUtils {
    static String toUpperCase(String s) {
        return s.toUpperCase();
    }
}

public class Main {
    public static void main(String[] args) {
        String result = StringUtils.toUpperCase("hello");
        System.out.println(result);  // HELLO
    }
}
```

---

## Part 14: Interview Traps & Best Practices

### 14.1 Common Interview Traps

**Trap 1: Multiple Inheritance Diamond Problem**

```java
interface A {
    void display();
}

interface B extends A {
    void display();
}

interface C extends A {
    void display();
}

// Confusing: which display() to use?
interface D extends B, C {  // Java allows multiple interface inheritance
}

// SOLUTION: Interfaces can have default methods
interface A {
    default void display() {
        System.out.println("A");
    }
}

interface B extends A {
    @Override
    default void display() {
        System.out.println("B");
    }
}

interface C extends A {
    @Override
    default void display() {
        System.out.println("C");
    }
}

interface D extends B, C {
    @Override
    default void display() {
        B.super.display();  // Explicitly choose B
    }
}
```

**Trap 2: Calling Parent Constructor**

```java
public class Parent {
    public Parent(int x) {
        System.out.println("Parent constructor");
    }
}

public class Child extends Parent {
    public Child() {
        super(10);  // MUST call parent constructor
    }
}
```

**Trap 3: Object vs Reference**

```java
Animal a = new Dog();  // Reference is Animal type, object is Dog type
a.bark();  // ERROR: Animal doesn't have bark()
// Must cast: ((Dog) a).bark();
```

### 14.2 Best Practices

✅ **Prefer composition over inheritance**  
✅ **Use interface for contracts**  
✅ **Keep classes and methods small and focused**  
✅ **Use encapsulation: private fields, public getters/setters**  
✅ **Use meaningful names**  
✅ **Implement SOLID principles**  
✅ **Avoid deep inheritance hierarchies**  
✅ **Use final for constants**  

---

## Part 15: Experienced-Level Q&A (4+ Years)

### 15.1 Advanced OOP Concepts

**Q1. Explain the difference between method overloading and method overriding. Can you override a static method?**

A: Overloading = Compile-time polymorphism (same class, different parameters). Overriding = Runtime polymorphism (parent-child classes, same method). Static methods CANNOT be overridden; they can only be hidden by child class static methods.

```java
class Parent {
    static void display() {
        System.out.println("Parent static");
    }
}

class Child extends Parent {
    static void display() {
        System.out.println("Child static");  // Method hiding, not overriding
    }
}

Parent p = new Child();
p.display();  // "Parent static" (compile-time decision)
```

**Q2. What is the purpose of abstract classes vs interfaces? When would you use each?**

A: Abstract classes = Shared code + state + constructors (IS-A relationship). Interfaces = Pure contracts (CAN-DO capability). Use abstract classes for related classes sharing implementation; use interfaces for unrelated classes implementing a contract.[web:19][web:20]

**Q3. Explain the concept of "loosely coupled" design. How does it relate to OOP principles?**

A: Loosely coupled = Classes depend on abstractions (interfaces), not concrete implementations. Achieved through dependency injection and programming to interfaces. Improves testability, maintainability, and flexibility.[web:24][web:27]

```java
// Tightly coupled (bad)
public class ReportGenerator {
    private DatabaseConnection db = new DatabaseConnection();
}

// Loosely coupled (good)
public class ReportGenerator {
    private DataSource dataSource;
    
    public ReportGenerator(DataSource ds) {
        this.dataSource = ds;  // Injected dependency
    }
}
```

**Q4. What is the difference between composition and inheritance? When would you prefer composition?**

A: Inheritance = IS-A relationship (child inherits from parent). Composition = HAS-A relationship (one object contains another). Composition is preferred because it's more flexible, avoids tight coupling, and sidesteps multiple inheritance issues.

```java
// Inheritance (tight coupling)
public class SportsCar extends Car { }

// Composition (flexible)
public class SportsCar {
    private Engine engine;
    private Transmission transmission;
}
```

**Q5. How does the Liskov Substitution Principle (LSP) work? Give an example of violating it.**

A: LSP = Derived classes should be substitutable for base classes without breaking the code. Violation example: Square extending Rectangle but throwing exception when width ≠ height.[web:27][web:30]

```java
// Violates LSP
class Rectangle {
    void setWidth(int w) { width = w; }
    void setHeight(int h) { height = h; }
}

class Square extends Rectangle {
    void setWidth(int w) {
        if (w != height) throw new Exception();  // Violates LSP
    }
}
```

### 15.2 Design Patterns & Architecture

**Q6. How would you design a system to enforce that only one instance of a class can exist?**

A: Use the Singleton pattern with private constructor and static getInstance() method.

```java
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
    private DatabaseConnection() { }  // Private constructor
    
    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}

// Thread-safe version (eager initialization)
public class DatabaseConnection {
    private static final DatabaseConnection instance = new DatabaseConnection();
    
    private DatabaseConnection() { }
    
    public static DatabaseConnection getInstance() {
        return instance;
    }
}
```

**Q7. Explain the Builder pattern and when you would use it.**

A: Builder pattern = Create complex objects step-by-step without constructor telescoping. Use when objects have many optional parameters.

```java
public class User {
    private String name;
    private int age;
    private String email;
    private String phone;
    
    private User(UserBuilder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
        this.phone = builder.phone;
    }
    
    public static class UserBuilder {
        private String name;
        private int age;
        private String email;
        private String phone;
        
        public UserBuilder(String name) {
            this.name = name;
        }
        
        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }
        
        public UserBuilder email(String email) {
            this.email = email;
            return this;
        }
        
        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.UserBuilder("Alice")
    .age(25)
    .email("alice@example.com")
    .build();
```

**Q8. What is the difference between UML composition and aggregation?**

A: Composition = Strong ownership (parent deletes child). Aggregation = Weak ownership (parent and child can exist independently). In code, both use object references, but semantics differ.

```java
// Composition (strong)
public class Car {
    private Engine engine;  // Car owns Engine; dies with Car
    
    public Car() {
        this.engine = new Engine();
    }
}

// Aggregation (weak)
public class Company {
    private List<Employee> employees;  // Company has Employees; employees can exist without Company
}
```

### 15.3 Real-World Scenarios

**Q9. You're designing an e-commerce system. How would you structure classes for products, users, and orders using OOP principles?**

A: Use interfaces for contracts, abstract classes for shared behavior, and composition for relationships.

```java
interface PaymentMethod {
    void pay(double amount);
}

interface Shippable {
    void ship();
    void track();
}

abstract class Entity {
    protected int id;
    protected LocalDateTime createdAt;
}

class Product extends Entity {
    private String name;
    private double price;
    private int stock;
}

class User extends Entity implements Shippable {
    private String name;
    private String email;
    
    @Override
    public void ship() { }
    
    @Override
    public void track() { }
}

class Order extends Entity {
    private User customer;
    private List<Product> items;
    private PaymentMethod paymentMethod;
    
    void checkout() {
        // Process order
    }
}
```

**Q10. How would you implement a notification system that supports multiple notification types (Email, SMS, Push) without tightly coupling to specific implementations?**

A: Use Strategy pattern with a Notifier interface.

```java
interface NotificationStrategy {
    void send(String message, String recipient);
}

class EmailNotification implements NotificationStrategy {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending email to " + recipient);
    }
}

class SMSNotification implements NotificationStrategy {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending SMS to " + recipient);
    }
}

class NotificationService {
    private NotificationStrategy strategy;
    
    public void setStrategy(NotificationStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void notify(String message, String recipient) {
        strategy.send(message, recipient);
    }
}

// Usage
NotificationService ns = new NotificationService();
ns.setStrategy(new EmailNotification());
ns.notify("Hello", "user@example.com");

ns.setStrategy(new SMSNotification());
ns.notify("Hello", "+1234567890");
```

**Q11. You have a legacy system with tightly coupled classes. How would you refactor it to be more maintainable?**

A: Introduce interfaces, use dependency injection, extract common logic to base classes, and apply SOLID principles incrementally.

```java
// Before: Tightly coupled
public class ReportGenerator {
    private Database db = new Database();
    private FileWriter fw = new FileWriter();
    
    void generate() {
        db.connect();
        fw.write("Report");
    }
}

// After: Decoupled
interface DataSource {
    List<String> getData();
}

interface OutputWriter {
    void write(String data);
}

public class ReportGenerator {
    private DataSource dataSource;
    private OutputWriter outputWriter;
    
    public ReportGenerator(DataSource ds, OutputWriter ow) {
        this.dataSource = ds;
        this.outputWriter = ow;
    }
    
    void generate() {
        List<String> data = dataSource.getData();
        outputWriter.write(String.join("\n", data));
    }
}
```

---

## Summary Checklist for Interview

- [ ] Understand difference between class and object
- [ ] Know constructors and overloading
- [ ] Understand initialization blocks and execution order
- [ ] Master encapsulation: private, getters/setters
- [ ] Understand inheritance and method overriding
- [ ] Know polymorphism: overloading vs overriding
- [ ] Understand abstraction: abstract classes and interfaces
- [ ] Understand IS-A vs HAS-A relationships
- [ ] Know all access modifiers and their visibility
- [ ] Know non-access modifiers: static, final, abstract, etc.
- [ ] Understand inner classes: member, static, local, anonymous
- [ ] Understand functional interfaces and lambda expressions
- [ ] Understand default and static methods in interfaces
- [ ] Be ready to discuss design patterns
- [ ] Be able to refactor tightly coupled code

---

**Last Updated:** February 2026  
**Prepared for:** OOP Interview Preparation (4+ Years Experience Focus)  
**References:** GeeksforGeeks, Baeldung, Java Documentation
