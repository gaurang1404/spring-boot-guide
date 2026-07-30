# Spring Beans and Dependency Injection

## What is it?

One of the biggest features of Spring is that it manages the objects in your application for you.

Instead of creating objects manually using the `new` keyword, Spring creates, configures, stores, and injects them wherever they are needed.

This entire mechanism revolves around five core concepts:

- **Beans** – Objects managed by Spring.
- **Component Scanning** – How Spring finds beans.
- **Stereotype Annotations** – How we tell Spring which classes should become beans.
- **Autowiring (Dependency Injection)** – How Spring connects beans together.
- **Bean Scopes** – How many instances of a bean Spring should create.

These concepts together form the foundation of the Spring Framework and are heavily used in Spring Boot.

---

## Why do we need it?

Imagine an application with hundreds of classes.

Without Spring:

```java
UserRepository repository = new UserRepository();
EmailService emailService = new EmailService();
NotificationService notificationService = new NotificationService(emailService);
UserService userService = new UserService(repository, notificationService);
```

As the project grows:

- Object creation becomes difficult to manage.
- Dependencies become tightly coupled.
- Testing becomes harder.
- Replacing implementations requires code changes everywhere.

Instead, Spring manages all these objects automatically.

```java
@Service
public class UserService {

    private final UserRepository repository;
    private final NotificationService notificationService;

    public UserService(UserRepository repository,
                       NotificationService notificationService) {
        this.repository = repository;
        this.notificationService = notificationService;
    }
}
```

No `new` keyword anywhere.

Spring creates every object and wires them together.

---

# 1. Spring Beans

## What is a Bean?

A **Bean** is simply an object whose lifecycle is managed by the Spring Container.

Instead of creating objects yourself:

```java
EmailService emailService = new EmailService();
```

Spring creates it for you.

---

## How do we create a Bean?

### Method 1 (Most Common) - Using Stereotype Annotations

```java
@Service
public class EmailService {

}
```

Spring automatically creates this object.

---

### Method 2 - Using @Bean

Useful when:

- Creating third-party library objects
- Custom configuration
- You cannot modify the source code

```java
@Configuration
public class AppConfig {

    @Bean
    public EmailService emailService() {
        return new EmailService();
    }

}
```

The returned object becomes a Spring Bean.

---

## Bean Lifecycle (Simplified)

```
Application Starts
        │
        ▼
Component Scan
        │
        ▼
Spring Creates Bean
        │
        ▼
Dependencies Injected
        │
        ▼
Bean Ready to Use
        │
        ▼
Application Stops
        │
        ▼
Bean Destroyed
```

---

# 2. Component Scanning

## What is Component Scanning?

Spring automatically searches for classes that should become beans.

Instead of manually registering every bean, Spring scans packages for specific annotations.

For example:

```
com.example

    UserService.java
    UserRepository.java
    EmailService.java
```

During startup Spring scans these packages.

If it finds:

```java
@Service
public class UserService {
}
```

It automatically creates a bean.

Same for:

```java
@Repository
public class UserRepository {
}
```

and

```java
@Component
public class EmailService {
}
```

No extra configuration required.

---

## How does Spring know where to scan?

When using Spring Boot:

```java
@SpringBootApplication
public class DemoApplication {

}
```

`@SpringBootApplication` includes:

```java
@ComponentScan
```

Spring scans:

- Current package
- All child packages

Example:

```
com.example

    DemoApplication
    service
    repository
    controller
```

Everything inside `com.example` gets scanned.

---

# 3. Stereotype Annotations

Stereotype annotations tell Spring:

> "Create an object of this class and manage it."

---

## @Component

Generic Spring Bean.

```java
@Component
public class EmailService {

}
```

Use when the class doesn't belong to any specific layer.

---

## @Service

Represents business logic.

```java
@Service
public class UserService {

}
```

Semantically clearer than `@Component`.

---

## @Repository

Represents the Data Access Layer.

```java
@Repository
public class UserRepository {

}
```

Also provides automatic exception translation for database exceptions.

---

## @Controller

Used in MVC applications.

Returns views.

```java
@Controller
public class HomeController {

}
```

---

## @RestController

Used for REST APIs.

Returns JSON.

```java
@RestController
public class UserController {

}
```

Equivalent to:

```java
@Controller
@ResponseBody
```

---

## Which one should I use?

| Annotation | Purpose |
|------------|----------|
| @Component | Generic Bean |
| @Service | Business Logic |
| @Repository | Database Layer |
| @Controller | MVC Controller |
| @RestController | REST APIs |

All of them create Spring Beans.

The difference is mainly semantic (and `@Repository` adds exception translation).

---

# 4. Autowiring (Dependency Injection)

## What is Autowiring?

Autowiring means Spring automatically provides the required dependencies.

Suppose:

```java
@Service
public class EmailService {

}
```

Another class needs it.

Without Spring:

```java
public class UserService {

    private EmailService emailService =
            new EmailService();

}
```

With Spring:

```java
@Service
public class UserService {

    private final EmailService emailService;

    public UserService(EmailService emailService) {
        this.emailService = emailService;
    }

}
```

Spring automatically injects the existing bean.

---

## What if multiple beans exist?

Example:

```java
@Service
public class StripePaymentService implements PaymentService {
}
```

```java
@Service
public class PaypalPaymentService implements PaymentService {
}
```

Spring doesn't know which one to inject.

Solution:

```java
@Service
public class OrderService {

    public OrderService(
        @Qualifier("stripePaymentService")
        PaymentService paymentService) {

    }

}
```

Or:

```java
@Primary
@Service
public class StripePaymentService implements PaymentService {
}
```

---

# 5. Bean Scopes

Bean Scope determines how many objects Spring creates.

---

## Singleton (Default)

One object for the entire application.

```java
@Service
@Scope("singleton")
public class EmailService {

}
```

```
UserService --------\
                     \
                      --> Same EmailService Object
                     /
OrderService -------/
```

Most commonly used.

---

## Prototype

A new object every time it is requested.

```java
@Component
@Scope("prototype")
public class ReportGenerator {

}
```

```
Request 1 -> Object A

Request 2 -> Object B

Request 3 -> Object C
```

---

## Request Scope

One object per HTTP request.

```java
@Component
@RequestScope
public class UserRequest {

}
```

---

## Session Scope

One object per user session.

```java
@Component
@SessionScope
public class ShoppingCart {

}
```

---

## Application Scope

One object for the entire ServletContext.

```java
@Component
@ApplicationScope
public class GlobalStatistics {

}
```

---

## Scope Comparison

| Scope | Number of Objects |
|--------|-------------------|
| Singleton | One for the application |
| Prototype | New every request from the container |
| Request | One per HTTP request |
| Session | One per HTTP session |
| Application | One per ServletContext |

---

# Putting Everything Together

```java
@Repository
public class UserRepository {

}
```

```java
@Service
public class EmailService {

}
```

```java
@Service
public class UserService {

    private final UserRepository repository;
    private final EmailService emailService;

    public UserService(UserRepository repository,
                       EmailService emailService) {

        this.repository = repository;
        this.emailService = emailService;
    }
}
```

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

### What happens when the application starts?

1. Spring performs component scanning.
2. Finds all stereotype annotations.
3. Creates beans.
4. Stores them in the IoC Container.
5. Automatically injects dependencies.
6. Beans are ready to use throughout the application.

---

## Best Practices

- Prefer **constructor injection** over field injection.
- Use the appropriate stereotype annotation (`@Service`, `@Repository`, etc.) instead of `@Component` everywhere.
- Keep the main application class in the root package so component scanning covers the entire project.
- Use `@Bean` only when annotation-based bean creation isn't possible (e.g., third-party classes).
- Use `@Qualifier` or `@Primary` when multiple implementations of the same interface exist.
- Stick with the default **Singleton** scope unless another scope is required.

---

## Common Interview Questions

### Q1. What is a Spring Bean?

A Java object managed by the Spring IoC Container.

---

### Q2. What is Component Scanning?

It is the process by which Spring automatically discovers classes annotated with stereotype annotations and registers them as beans.

---

### Q3. What is the difference between `@Component` and `@Service`?

Both create Spring Beans. `@Service` is a specialized form of `@Component` used to indicate business logic, improving readability and intent.

---

### Q4. Why is constructor injection preferred?

It makes dependencies mandatory, supports immutable (`final`) fields, improves testability, and is the recommended approach in modern Spring.

---

### Q5. What happens if multiple beans of the same type exist?

Spring throws a `NoUniqueBeanDefinitionException` unless you specify which bean to inject using `@Qualifier` or mark one bean as `@Primary`.

---

### Q6. What is the default bean scope?

`Singleton`.

---

## Summary

- A **Bean** is a Java object managed by the Spring Container.
- **Component Scanning** automatically discovers bean classes.
- **Stereotype Annotations** (`@Component`, `@Service`, `@Repository`, etc.) tell Spring which classes to manage.
- **Autowiring** automatically injects dependencies between beans.
- **Constructor Injection** is the recommended form of dependency injection.
- **Bean Scopes** control how many instances of a bean Spring creates.
- Together, these concepts form the backbone of Spring's Dependency Injection (DI) mechanism.