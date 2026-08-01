# JPA Entities and Lombok

## Introduction

After configuring Spring Data JPA and connecting our application to the database, the next step is to tell Hibernate **how Java objects map to database tables**.

This is done using **JPA Entities**.

Instead of manually converting rows into Java objects, Hibernate automatically maps them using annotations.

For example:

```
Database

users
+----+-------+------------------+
| id | name  | email            |
+----+-------+------------------+
| 1  | John  | john@gmail.com   |
| 2  | Alice | alice@gmail.com  |
+----+-------+------------------+

            ⇅

Java

User
---------------
id
name
email
```

Each row in the table becomes a Java object.

---

# What is an Entity?

An **Entity** is a Java class that represents a database table.

- The **class** represents a table.
- Each **object** represents a row.
- Each **field** represents a column.

Example:

```
Table

users

+----+---------+------------------+
| id | name    | email            |
+----+---------+------------------+
| 1  | John    | john@gmail.com   |
| 2  | Alice   | alice@gmail.com  |
+----+---------+------------------+
```

can be represented as:

```java
User john = new User(1L, "John", "john@gmail.com");

User alice = new User(2L, "Alice", "alice@gmail.com");
```

---

# @Entity

Marks a class as a JPA Entity.

```java
@Entity
public class User {

}
```

When Hibernate scans the project, it recognizes this class as a table that should be managed.

Without `@Entity`, Hibernate ignores the class.

---

# @Table

Specifies the database table name.

```java
@Entity
@Table(name = "users")
public class User {

}
```

If omitted, Hibernate uses the class name as the table name (depending on the naming strategy).

---

## Useful Attributes

```java
@Table(
    name = "users"
)
```

Some commonly used attributes include:

| Attribute | Description |
|-----------|-------------|
| `name` | Table name in the database |
| `schema` | Database schema |
| `catalog` | Database catalog |
| `indexes` | Creates indexes on columns |
| `uniqueConstraints` | Defines unique constraints |

Example:

```java
@Table(
    name = "users",
    uniqueConstraints = {
        @UniqueConstraint(columnNames = "email")
    }
)
```

---

# @Id

Every entity requires a **Primary Key**.

`@Id` marks the field that uniquely identifies each row.

```java
@Id
private Long id;
```

Without an `@Id`, Hibernate cannot uniquely identify an entity.

---

# @GeneratedValue

Automatically generates values for the primary key.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Instead of manually assigning IDs,

```java
user.setId(10);
```

the database generates them automatically.

---

## Generation Strategies

### IDENTITY

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

Uses the database's auto-increment feature.

Example:

```
1

2

3

4

5
```

Most commonly used with MySQL.

---

### SEQUENCE

```java
@GeneratedValue(strategy = GenerationType.SEQUENCE)
```

Uses a database sequence.

Commonly used with PostgreSQL and Oracle.

```
Sequence

100

101

102
```

---

### TABLE

```java
@GeneratedValue(strategy = GenerationType.TABLE)
```

Stores the next ID inside a separate table.

Portable across databases but slower.

Rarely used today.

---

### AUTO

```java
@GeneratedValue(strategy = GenerationType.AUTO)
```

Lets Hibernate decide which strategy to use based on the database.

Useful when writing database-independent applications.

---

## Strategy Comparison

| Strategy | Common Database | Notes |
|----------|-----------------|------|
| IDENTITY | MySQL | Uses AUTO_INCREMENT |
| SEQUENCE | PostgreSQL, Oracle | Uses database sequences |
| TABLE | Any | Uses a separate table |
| AUTO | Any | Hibernate chooses automatically |

---

# @Column

Maps a field to a database column.

```java
@Column(name = "email")
private String email;
```

Usually optional, but useful when customizing column properties.

---

## Important Attributes

### name

Specify a different column name.

```java
@Column(name = "email_address")
private String email;
```

---

### nullable

Controls whether the column allows `NULL`.

```java
@Column(nullable = false)
private String email;
```

Equivalent SQL:

```sql
email VARCHAR(255) NOT NULL
```

---

### unique

Creates a unique constraint.

```java
@Column(unique = true)
private String email;
```

Equivalent SQL:

```sql
UNIQUE(email)
```

---

### length

Specifies the maximum length of a string.

```java
@Column(length = 100)
private String name;
```

Equivalent SQL:

```sql
VARCHAR(100)
```

---

### updatable

Determines whether Hibernate can update the column.

```java
@Column(updatable = false)
private String createdBy;
```

Useful for fields that should never change.

---

### insertable

Determines whether the field should be included during INSERT.

```java
@Column(insertable = false)
private LocalDate createdDate;
```

Useful when the database assigns default values.

---

### columnDefinition

Specify the SQL definition manually.

```java
@Column(columnDefinition = "TEXT")
private String description;
```

Generally used only for database-specific requirements.

---

# Complete Entity Example

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(
        nullable = false,
        unique = true,
        length = 255
    )
    private String email;

    @Column(nullable = false)
    private String password;

}
```

Hibernate now knows exactly how this Java class maps to the `users` table.

---

# The Boilerplate Problem

A typical entity quickly becomes repetitive.

```java
public class User {

    private Long id;
    private String name;
    private String email;

    // Getters

    // Setters

    // Constructors

    // Builder

}
```

Most of this code isn't business logic—it's boilerplate.

To reduce this, Java developers commonly use **Project Lombok**.

---

# What is Lombok?

**Project Lombok** is a Java library that automatically generates common boilerplate code during compilation.

Instead of writing getters, setters, constructors, and builders manually, you simply add annotations.

> **Fun Fact:** Lombok gets its name from **Lombok**, an island in Indonesia.

---

# Adding Lombok

Add the dependency:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

---

# Enable Annotation Processing

Lombok works using Java's annotation processing.

In IntelliJ IDEA:

```
Settings

↓

Build, Execution, Deployment

↓

Compiler

↓

Annotation Processors

↓

Enable Annotation Processing
```

Without this setting, Lombok annotations won't generate code inside the IDE.

---

# Common Lombok Annotations

## @Getter

Generates getters for all fields.

```java
@Getter
public class User {

    private String name;

}
```

Generated:

```java
public String getName() {
    return name;
}
```

---

## @Setter

Generates setters for all fields.

```java
@Setter
public class User {

    private String name;

}
```

Generated:

```java
public void setName(String name) {
    this.name = name;
}
```

---

## @NoArgsConstructor

Generates a no-argument constructor.

```java
@NoArgsConstructor
public class User {

}
```

Generated:

```java
public User() {

}
```

JPA requires every entity to have a no-argument constructor (it can be public or protected).

---

## @AllArgsConstructor

Generates a constructor with all fields.

```java
@AllArgsConstructor
public class User {

    private Long id;
    private String name;

}
```

Generated:

```java
public User(Long id, String name) {
    this.id = id;
    this.name = name;
}
```

---

## @Builder

Implements the Builder Pattern automatically.

Instead of:

```java
User user = new User();

user.setName("John");
user.setEmail("john@gmail.com");
user.setPassword("secret");
```

You can write:

```java
User user = User.builder()
        .name("John")
        .email("john@gmail.com")
        .password("secret")
        .build();
```

This is especially useful for classes with many fields.

---

# A Typical Entity with Lombok

```java
@Entity
@Table(name = "users")

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder

public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password;

}
```

With just a few annotations, Lombok generates:

- Getters
- Setters
- No-args constructor
- All-args constructor
- Builder

making the entity much cleaner and easier to read.

---

# Best Practices

- Always annotate JPA entity classes with `@Entity`.
- Explicitly specify table names using `@Table` for clarity.
- Use `GenerationType.IDENTITY` when working with MySQL.
- Use `@Column` only when you need to customize column behavior.
- Prefer Lombok to reduce boilerplate code.
- Always provide a no-argument constructor for JPA entities.
- Use `@Builder` when creating objects with many optional fields.

---

# Common Interview Questions

### Q1. What is a JPA Entity?

A Java class that represents a database table. Each object of the class corresponds to one row in that table.

---

### Q2. What is the purpose of `@Entity`?

It tells JPA/Hibernate that the class should be mapped to a database table and managed as an entity.

---

### Q3. What is the difference between `@Entity` and `@Table`?

- `@Entity` marks the class as a JPA entity.
- `@Table` specifies details about the database table, such as its name and constraints.

---

### Q4. Which `GenerationType` is commonly used with MySQL?

`GenerationType.IDENTITY`, because MySQL commonly uses `AUTO_INCREMENT` for primary keys.

---

### Q5. Why does every JPA entity need a no-argument constructor?

Hibernate uses reflection to create entity instances, so it requires a no-argument constructor (public or protected).

---

### Q6. What is Lombok?

Lombok is a Java library that generates boilerplate code such as getters, setters, constructors, and builders at compile time using annotations.

---

### Q7. Does Lombok replace JPA?

No.

Lombok only generates Java code to reduce boilerplate. JPA is responsible for mapping Java objects to database tables.

---

## Summary

- An **Entity** is a Java class that maps to a database table.
- Each entity instance represents one row in that table.
- `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, and `@Column` are the core annotations used to define this mapping.
- `GenerationType.IDENTITY` is the preferred strategy for MySQL applications.
- **Project Lombok** reduces boilerplate by generating getters, setters, constructors, and builders automatically.
- Combining JPA and Lombok results in clean, readable, and maintainable entity classes.