# Spring Data Access with JDBC, JPA, Hibernate and Spring Data JPA

## Introduction

Almost every real-world application needs to store and retrieve data from a database.

For example:

- An e-commerce application stores products and orders.
- A banking application stores customer accounts.
- A social media application stores users and posts.

Java applications communicate with databases through several technologies that build upon one another.

```
Database
    ▲
    │
JDBC
    ▲
    │
JPA (Specification)
    ▲
    │
Hibernate (Implementation)
    ▲
    │
Spring Data JPA
```

Understanding the relationship between these technologies is essential before writing database code.

---

# 1. JDBC (Java Database Connectivity)

## What is JDBC?

**JDBC (Java Database Connectivity)** is Java's standard API for communicating with relational databases.

It allows Java applications to:

- Connect to a database
- Execute SQL queries
- Read results
- Insert, update and delete records

JDBC is part of Java itself—it is **not a Spring feature**.

---

## How does JDBC work?

```
Java Application
        │
        ▼
     JDBC API
        │
        ▼
 JDBC Driver (MySQL, PostgreSQL...)
        │
        ▼
     Database
```

The JDBC Driver converts Java calls into database-specific commands.

---

## JDBC Example

```java
Connection connection =
    DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/shop",
        "root",
        "password");

PreparedStatement statement =
    connection.prepareStatement(
        "SELECT * FROM users");

ResultSet result = statement.executeQuery();

while(result.next()) {
    System.out.println(result.getString("name"));
}

connection.close();
```

---

## Problems with JDBC

Although powerful, JDBC requires a lot of boilerplate code.

You have to manually:

- Open connections
- Close connections
- Write SQL
- Convert rows into Java objects
- Handle exceptions
- Manage transactions

For every database operation.

As applications grow, this becomes difficult to maintain.

This led to the creation of ORM frameworks.

---

# 2. JPA (Java Persistence API)

## What is JPA?

**JPA (Java Persistence API)** is a Java specification that defines a standard way to map Java objects to database tables.

Instead of writing SQL for everything, you work with Java objects.

For example,

Instead of writing:

```sql
INSERT INTO users(name,email)
VALUES ('John','john@gmail.com');
```

You simply write:

```java
User user = new User();
user.setName("John");
user.setEmail("john@gmail.com");

entityManager.persist(user);
```

JPA generates the SQL behind the scenes.

---

## Is JPA a framework?

**No.**

JPA is only a **specification (interface/standard).**

It defines:

- How entities should look
- How persistence should work
- Which annotations exist
- Which APIs should be available

But it does **not** contain any implementation.

Think of it like this:

```
JPA
│
├── Rules
├── Interfaces
└── Specifications

(No actual code)
```

Someone still has to implement those rules.

That's where Hibernate comes in.

---

## Benefits of JPA

- Less SQL code
- Maps Java objects to tables
- Database-independent programming
- Standard API
- Easier maintenance
- Supports relationships between tables
- Reduces boilerplate code

---

# 3. Hibernate

## What is Hibernate?

Hibernate is the **most popular implementation of JPA**.

It follows all the rules defined by JPA and provides the actual code that performs the database operations.

```
JPA
    │
Defines Rules
    │
    ▼
Hibernate
Implements those rules
```

Without Hibernate (or another JPA provider), JPA cannot do anything.

---

## Hibernate Responsibilities

Hibernate:

- Converts Java objects into SQL
- Converts SQL results back into Java objects
- Creates SQL queries
- Manages object states
- Supports caching
- Handles lazy loading
- Manages transactions

---

## Example

Suppose we have:

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;

}
```

Saving the object:

```java
entityManager.persist(user);
```

Hibernate automatically generates something similar to:

```sql
INSERT INTO users(id, name)
VALUES (?, ?);
```

You never write this SQL manually.

---

## Why Hibernate became popular

Without Hibernate:

```
Java Object

↓

Write SQL manually

↓

Convert SQL result manually

↓

Handle connections manually
```

With Hibernate:

```
Java Object

↓

Hibernate

↓

SQL Generated Automatically
```

---

# 4. Spring Data JPA

## What is Spring Data JPA?

Spring Data JPA is a Spring module built on top of JPA.

It removes even more boilerplate code by automatically implementing repository classes.

Instead of writing DAO classes manually, you only create an interface.

---

## Without Spring Data JPA

You would typically write:

```java
public class UserRepository {

    public User findById(Long id) {
        // Write query
    }

    public void save(User user) {
        // Write persistence logic
    }

}
```

---

## With Spring Data JPA

```java
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```

That's it.

Spring automatically provides implementations for:

- save()
- findById()
- findAll()
- delete()
- count()
- existsById()

and many more.

---

## Technology Stack

```
Your Application

        │

Spring Data JPA

        │

Hibernate

        │

JPA Specification

        │

JDBC

        │

Database Driver

        │

Database
```

Each layer builds upon the previous one.

---

# Setting Up Spring Data JPA with Hibernate

Now that we understand the theory, let's configure our Spring Boot project to connect to a database.

---

## Step 1: Add Spring Data JPA Dependency

For Maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

This dependency includes:

- Spring Data JPA
- Hibernate
- JPA API
- Transaction management

Notice that we do **not** add Hibernate separately—it is included automatically.

---

## Step 2: Add a Database Driver

Spring needs a driver to communicate with a specific database.

### MySQL

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

### PostgreSQL

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

Only add the driver for the database you are using.

---

## Step 3: Configure the Datasource

In `application.yaml`:

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/shop_db
    username: root
    password: password

  jpa:
    hibernate:
      ddl-auto: update

    show-sql: true

    properties:
      hibernate:
        format_sql: true
```

---

## Understanding the Configuration

### datasource.url

Database connection URL.

```yaml
url: jdbc:mysql://localhost:3306/shop_db
```

---

### username

Database username.

```yaml
username: root
```

---

### password

Database password.

```yaml
password: password
```

---

### ddl-auto

Controls how Hibernate handles database tables.

```yaml
ddl-auto: update
```

Common values:

| Value | Meaning |
|--------|----------|
| none | Do nothing |
| validate | Validate schema only |
| update | Update existing tables |
| create | Drop and recreate tables every startup |
| create-drop | Create tables and delete them on shutdown |

For development, `update` is commonly used.

For production, `validate` or `none` is generally preferred.

---

### show-sql

```yaml
show-sql: true
```

Prints generated SQL in the console.

Useful during development.

---

### format_sql

```yaml
format_sql: true
```

Formats SQL for better readability.

---

## What Happens When the Application Starts?

```
Spring Boot Starts
        │
        ▼
Reads application.yaml
        │
        ▼
Creates DataSource
        │
        ▼
Creates Hibernate SessionFactory
        │
        ▼
Scans JPA Entities
        │
        ▼
Creates Repository Beans
        │
        ▼
Application Ready
```

At this point, the application is connected to the database and is ready to perform CRUD operations.

---

## Best Practices

- Use Spring Data JPA instead of writing JDBC code directly for most applications.
- Keep database credentials outside source code using environment variables or configuration files.
- Use `ddl-auto=update` only during development.
- Enable `show-sql` only while debugging or learning.
- Use only one database driver dependency for your chosen database.
- Let Spring Boot auto-configure Hibernate whenever possible.

---

## Common Interview Questions

### Q1. What is the difference between JDBC and JPA?

JDBC is a low-level API that requires writing SQL manually.

JPA is a specification for Object Relational Mapping (ORM), allowing developers to work with Java objects instead of raw SQL.

---

### Q2. Is JPA a framework?

No.

JPA is a specification. It defines the rules but does not provide an implementation.

---

### Q3. What is Hibernate?

Hibernate is the most widely used implementation of the JPA specification. It performs the actual ORM and SQL generation.

---

### Q4. What is Spring Data JPA?

Spring Data JPA is a Spring module that simplifies working with JPA by automatically generating repository implementations and reducing boilerplate code.

---

### Q5. Do we need to add Hibernate separately?

No.

When using `spring-boot-starter-data-jpa`, Hibernate is included automatically as the default JPA provider.

---

### Q6. Why do we still need a database driver if we are using Hibernate?

Hibernate generates SQL, but it still uses **JDBC** underneath to communicate with the database. The JDBC driver is what actually establishes the connection and sends SQL commands to the database.

---

## Summary

- **JDBC** is the low-level Java API for interacting with relational databases.
- **JPA** is a specification that standardizes ORM in Java.
- **Hibernate** is the most popular implementation of the JPA specification.
- **Spring Data JPA** builds on JPA and Hibernate to simplify data access by generating repository implementations automatically.
- To connect a Spring Boot application to a database, you typically:
  1. Add `spring-boot-starter-data-jpa`.
  2. Add the appropriate JDBC driver.
  3. Configure the datasource and JPA properties in `application.yaml`.
- At this stage, the application is ready to work with entities and repositories, which will be covered next.