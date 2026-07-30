# Inversion of Control (IoC) and Dependency Injection (DI)

## Introduction

**Inversion of Control (IoC)** and **Dependency Injection (DI)** are the core concepts of the Spring Framework. Almost every feature in Spring is built on top of these principles.

A good understanding of IoC and DI is essential because interviewers often start Spring-related discussions with these topics.

---

# What is Inversion of Control (IoC)?

**Inversion of Control (IoC)** is a design principle in which the responsibility of creating and managing objects is transferred from the application to the Spring Framework.

### Without IoC

Normally, objects create their own dependencies using the `new` keyword.

```java
public class UserService {

    private UserRepository repository = new UserRepository();

    public void getUsers() {
        repository.findAll();
    }
}
```

Here:

- `UserService` is responsible for creating `UserRepository`.
- The classes are **tightly coupled**.
- Replacing `UserRepository` with another implementation requires modifying `UserService`.

---

### With IoC

The Spring IoC Container creates the required objects and provides them to the dependent class.

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }

    public void getUsers() {
        repository.findAll();
    }
}
```

Now:

- `UserService` no longer creates `UserRepository`.
- Spring creates both objects.
- Spring injects `UserRepository` into `UserService`.

This is called **Inversion of Control** because the control of object creation has moved from the application to the Spring Framework.

---

# Why Do We Need IoC?

Without IoC:

- Classes become tightly coupled.
- Difficult to replace implementations.
- Difficult to test.
- More boilerplate code.
- Harder to maintain.

With IoC:

- Loose coupling.
- Better code organization.
- Easier unit testing.
- Easy to replace implementations.
- Better maintainability.

---

# What is the Spring IoC Container?

The **IoC Container** is the part of Spring responsible for:

- Creating objects (Beans)
- Managing their lifecycle
- Injecting dependencies
- Configuring beans
- Destroying beans when the application shuts down

Whenever the application starts, the IoC Container scans the project, creates the required beans, and wires them together.

---

# What is a Dependency?

A **dependency** is simply another object that a class requires to perform its work.

Example:

```java
public class UserService {

    private UserRepository repository;
}
```

Here:

- `UserService` depends on `UserRepository`.
- Therefore, `UserRepository` is called a **dependency** of `UserService`.

---

# What is Dependency Injection (DI)?

**Dependency Injection** is the process by which the Spring IoC Container provides the required dependencies to a class instead of the class creating them itself.

Think of it like this:

```
Without DI

UserService
      |
   creates
      |
UserRepository


With DI

Spring Container
      |
 creates both objects
      |
 injects repository
      |
UserService
```

---

# Relationship Between IoC and DI

Many beginners confuse these two concepts.

**IoC** is the **principle**.

**Dependency Injection** is one of the **ways to achieve IoC**.

Think of it like this:

```
IoC
 │
 └── Dependency Injection
```

Interview Tip:

> IoC tells us **who controls object creation**, while DI tells us **how dependencies are provided**.

---

# Types of Dependency Injection

Spring supports three types of Dependency Injection.

## 1. Constructor Injection (Recommended)

The dependency is provided through the constructor.

```java
@Repository
public class UserRepository {

    public void findAll() {
        System.out.println("Fetching users...");
    }
}
```

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }

    public void displayUsers() {
        repository.findAll();
    }
}
```

### Why Constructor Injection is Recommended

- Dependencies are mandatory.
- Objects become immutable.
- Easier to write unit tests.
- No chance of partially initialized objects.
- Recommended by the Spring team.

---

## 2. Setter Injection

Dependencies are provided using setter methods.

```java
@Service
public class UserService {

    private UserRepository repository;

    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }

    public void displayUsers() {
        repository.findAll();
    }
}
```

### When to Use

- Optional dependencies.
- Rarely used in modern Spring applications.

---

## 3. Field Injection

Dependencies are injected directly into the field.

```java
@Service
public class UserService {

    @Autowired
    private UserRepository repository;

    public void displayUsers() {
        repository.findAll();
    }
}
```

Although simple, field injection is generally discouraged.

---

# Constructor Injection vs Setter Injection vs Field Injection

| Feature | Constructor | Setter | Field |
|----------|------------|---------|--------|
| Recommended | ✅ Yes | Sometimes | ❌ No |
| Immutable Dependencies | ✅ | ❌ | ❌ |
| Easy to Test | ✅ | ✅ | ❌ |
| Optional Dependencies | ❌ | ✅ | ❌ |
| Spring Recommended | ✅ | Sometimes | ❌ |

---

# How Does Spring Perform Dependency Injection?

Suppose we have:

```java
@Repository
public class UserRepository {

}
```

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

### Step 1

Spring starts the application.

---

### Step 2

Spring scans the project for components.

It finds:

- `@Repository`
- `@Service`
- `@Controller`
- `@Component`

---

### Step 3

Spring creates a bean of `UserRepository`.

---

### Step 4

Spring notices that `UserService` requires a `UserRepository`.

---

### Step 5

Instead of calling:

```java
new UserRepository()
```

Spring injects the existing bean into the constructor.

---

### Step 6

The application is now ready with all dependencies wired together.

---

# Real-Life Analogy

Imagine buying a car.

### Without IoC

You buy the car.

Then you build:

- Engine
- Wheels
- Seats
- Battery

yourself before driving.

---

### With IoC

The manufacturer assembles the car for you.

You simply receive a fully assembled car and start driving.

Spring acts like the manufacturer—it assembles all the required objects before handing them to your application.

---

# Advantages of IoC and DI

- Loose coupling
- Easier maintenance
- Better code reusability
- Easier unit testing
- Better scalability
- Cleaner architecture
- Objects have a single responsibility
- Easy to swap implementations

---

# Common Misconceptions

### IoC and DI are the same.

❌ Incorrect.

IoC is the design principle.

DI is one technique used to implement IoC.

---

### Spring creates objects only when needed.

Not always.

By default, **singleton beans are created eagerly during application startup**. Prototype beans are created only when requested.

---

### Constructor Injection requires `@Autowired`.

Since Spring Framework 4.3, if a class has **only one constructor**, `@Autowired` is optional.

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

This is the preferred modern style.

---

# Best Practices

- Prefer Constructor Injection.
- Avoid Field Injection.
- Keep dependencies minimal.
- Program against interfaces instead of concrete classes.
- Make dependencies `final` whenever possible.
- Use Setter Injection only for optional dependencies.

---

# Interview Keywords

- Inversion of Control (IoC)
- Dependency Injection (DI)
- IoC Container
- Spring Container
- ApplicationContext
- Bean
- Loose Coupling
- Constructor Injection
- Setter Injection
- Field Injection
- Object Lifecycle
- Dependency Resolution
- Wiring
- `@Autowired`
- `@Service`
- `@Repository`
- `@Component`

---

# Common Interview Questions

### What is Inversion of Control (IoC)?

IoC is a design principle where the responsibility of creating and managing objects is transferred from the application to the Spring IoC Container.

---

### What is Dependency Injection?

Dependency Injection is the mechanism by which the Spring IoC Container provides the required dependencies to a class instead of the class creating them itself.

---

### What is the relationship between IoC and DI?

IoC is the principle of delegating object creation to a container, while Dependency Injection is one technique used by Spring to implement that principle.

---

### Which type of Dependency Injection is recommended?

Constructor Injection.

It creates immutable objects, makes dependencies mandatory, improves testability, and is the approach recommended by the Spring team.

---

### Why is Field Injection discouraged?

- Difficult to unit test.
- Hidden dependencies.
- Cannot create immutable classes.
- Encourages poor design.

---

# Summary

- **IoC** is the principle of handing over object creation and lifecycle management to the Spring Framework.
- **Dependency Injection** is the mechanism Spring uses to provide required objects (dependencies) to other objects.
- The **Spring IoC Container** creates, manages, and wires beans together.
- **Constructor Injection** is the preferred way to inject dependencies because it is safer, cleaner, and easier to test.
- IoC and DI together promote **loose coupling**, making applications more maintainable, scalable, and testable.