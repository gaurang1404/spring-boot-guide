# Spring Data JPA Repositories

## Introduction

So far, we have:

- Connected our Spring Boot application to a database.
- Created JPA entities.
- Mapped entities to database tables.

Now we need a way to **perform CRUD (Create, Read, Update, Delete) operations** on those entities.

Writing SQL or DAO classes manually would be repetitive.

Spring Data JPA solves this problem using **Repositories**.

A repository acts as a bridge between your application and the database.

```
Controller
      │
      ▼
Service
      │
      ▼
Repository
      │
      ▼
Hibernate
      │
      ▼
Database
```

Instead of writing SQL, we simply call repository methods.

---

# Repository Hierarchy

Spring Data JPA provides multiple repository interfaces.

Each one builds upon the previous one by adding more functionality.

```
Repository
      │
      ▼
CrudRepository
      │
      ▼
PagingAndSortingRepository
      │
      ▼
JpaRepository
```

---

## Repository

```java
public interface Repository<T, ID> {

}
```

This is a **marker interface**.

It does not contain any methods.

Its purpose is to tell Spring:

> "This interface is a Spring Data Repository."

You rarely extend this directly.

---

## CrudRepository

```java
public interface CrudRepository<T, ID>
```

Adds basic CRUD operations.

Examples:

- Save
- Find
- Delete
- Count
- Exists

If your application only needs basic database operations, this interface is sufficient.

---

## PagingAndSortingRepository

```java
public interface PagingAndSortingRepository<T, ID>
```

Extends `CrudRepository`.

Adds support for:

- Pagination
- Sorting

Useful when displaying large datasets.

Example:

```
Products

Page 1

1-20

↓

Page 2

21-40

↓

Page 3

41-60
```

Instead of loading every record into memory.

---

## JpaRepository

```java
public interface JpaRepository<T, ID>
```

Extends both:

- `CrudRepository`
- `PagingAndSortingRepository`

It also adds many JPA-specific methods.

Examples:

- Batch operations
- Flush
- Save and Flush
- Delete in Batch

This is the repository interface used in **almost every Spring Boot application**.

---

# Which Repository Should You Use?

In most Spring Boot projects, simply extend:

```java
JpaRepository<Entity, PrimaryKeyType>
```

For example:

```java
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```

You automatically get dozens of database operations without writing any implementation.

---

# Creating Our First Repository

Suppose we have this entity:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

}
```

Repository:

```java
@Repository
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```

Notice that there is **no implementation class**.

Spring automatically generates one at runtime.

---

# Why Don't We Write the Implementation?

Normally you would write:

```java
public class UserRepositoryImpl
        implements UserRepository {

}
```

Spring Data JPA does this for us automatically.

When the application starts:

```
Spring Boot

↓

Find Repository Interfaces

↓

Generate Implementation

↓

Register as Spring Bean
```

This is one of the biggest advantages of Spring Data JPA.

---

# Common Repository Methods

## save()

Saves a new entity or updates an existing one.

```java
User user = new User();

user.setName("John");
user.setEmail("john@gmail.com");

userRepository.save(user);
```

If the entity doesn't exist:

```
INSERT
```

If the entity already exists:

```
UPDATE
```

The same method handles both operations.

---

## findById()

Returns an entity by its primary key.

```java
Optional<User> user =
        userRepository.findById(1L);
```

Notice the return type:

```
Optional<User>
```

instead of:

```
User
```

This is because the user may not exist.

---

## Why Optional?

Suppose we search:

```java
userRepository.findById(100L);
```

What if ID 100 doesn't exist?

Returning:

```java
null
```

would easily lead to:

```
NullPointerException
```

Instead, Spring returns:

```java
Optional<User>
```

which forces us to handle the missing value explicitly.

---

# orElse()

Returns the object if present.

Otherwise returns a default value.

```java
User user =
        userRepository
            .findById(1L)
            .orElse(new User());
```

If User 1 exists:

```
Return User
```

Otherwise:

```
Return new User()
```

---

# orElseThrow()

Throws an exception if the object is missing.

```java
User user =
        userRepository
            .findById(1L)
            .orElseThrow();
```

Or with a custom exception:

```java
User user =
        userRepository
            .findById(id)
            .orElseThrow(() ->
                new RuntimeException("User not found"));
```

This is commonly used in service classes.

---

## findAll()

Returns every row from the table.

```java
List<User> users =
        userRepository.findAll();
```

Equivalent SQL:

```sql
SELECT * FROM users;
```

---

## delete()

Deletes an entity.

```java
userRepository.delete(user);
```

Equivalent SQL:

```sql
DELETE
FROM users
WHERE id = ?;
```

---

## deleteById()

Deletes using the primary key.

```java
userRepository.deleteById(1L);
```

---

## existsById()

Checks whether an entity exists.

```java
boolean exists =
        userRepository.existsById(1L);
```

Returns:

```
true
```

or

```
false
```

Useful before updates or deletes.

---

## count()

Returns the total number of rows.

```java
long count =
        userRepository.count();
```

Equivalent SQL:

```sql
SELECT COUNT(*)
FROM users;
```

---

# Most Frequently Used Methods

| Method | Purpose |
|---------|----------|
| `save()` | Insert or update an entity |
| `findById()` | Find entity using primary key |
| `findAll()` | Retrieve all records |
| `delete()` | Delete an entity |
| `deleteById()` | Delete using ID |
| `existsById()` | Check if entity exists |
| `count()` | Count total rows |

These methods cover the majority of CRUD operations in typical applications.

---

# Understanding show-sql

Earlier, we added the following configuration:

```yaml
spring:
  jpa:
    show-sql: true
```

## What does it do?

It tells Hibernate to print every SQL statement it executes.

For example:

```java
userRepository.findAll();
```

Console:

```sql
select
    u.id,
    u.name,
    u.email
from
    users u;
```

Another example:

```java
userRepository.save(user);
```

Console:

```sql
insert into users
(name, email)
values (?, ?);
```

This is extremely useful while learning Spring Data JPA because it lets you see the SQL generated by Hibernate.

---

## Should it be enabled in production?

Generally, **no**.

Reasons:

- Produces unnecessary console logs.
- Can impact performance slightly.
- May expose sensitive SQL statements.

It is mainly intended for development and debugging.

---

# Putting Everything Together

Entity:

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

}
```

Repository:

```java
@Repository
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```

Using the repository:

```java
User user = User.builder()
        .name("John")
        .build();

userRepository.save(user);

User savedUser = userRepository
        .findById(user.getId())
        .orElseThrow();

List<User> users =
        userRepository.findAll();
```

Notice that we never wrote:

- SQL
- JDBC code
- DAO implementation
- Repository implementation

Spring Data JPA handled everything.

---

# Best Practices

- Prefer extending `JpaRepository` for most Spring Boot applications.
- Return `Optional<T>` when an entity may not exist.
- Use `orElseThrow()` in service classes when a missing entity should be treated as an error.
- Use `show-sql: true` only during development.
- Keep repository interfaces focused on data access logic only.
- Place business logic inside service classes, not repositories.

---

# Common Interview Questions

### Q1. What is a Repository in Spring Data JPA?

A Repository is a Spring Data interface that provides an abstraction for performing database operations on entities without writing implementation code.

---

### Q2. What is the difference between `CrudRepository` and `JpaRepository`?

`CrudRepository` provides basic CRUD operations.

`JpaRepository` extends `CrudRepository` and `PagingAndSortingRepository`, adding JPA-specific features such as batch operations, flushing, and additional convenience methods.

---

### Q3. Why does `findById()` return `Optional<T>`?

Because the requested entity may not exist. `Optional` encourages explicit handling of missing values and helps avoid `NullPointerException`.

---

### Q4. What is the difference between `orElse()` and `orElseThrow()`?

- `orElse()` returns a default value if the entity is absent.
- `orElseThrow()` throws an exception if the entity is absent.

---

### Q5. Do we need to implement repository interfaces?

No.

Spring Data JPA automatically generates the implementation at runtime.

---

### Q6. What does `spring.jpa.show-sql=true` do?

It prints the SQL generated by Hibernate to the console, making it easier to understand and debug database operations.

---

## Summary

- Repositories provide a simple abstraction over database operations.
- `Repository` is a marker interface.
- `CrudRepository` adds basic CRUD methods.
- `PagingAndSortingRepository` adds pagination and sorting support.
- `JpaRepository` combines these features and adds JPA-specific functionality, making it the preferred choice in most Spring Boot applications.
- Spring automatically generates repository implementations at runtime.
- `Optional`, `orElse()`, and `orElseThrow()` provide a safer way to handle missing entities.
- Enabling `show-sql` helps visualize the SQL generated by Hibernate during development.