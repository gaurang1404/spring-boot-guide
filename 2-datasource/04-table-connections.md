# JPA Entity Relationships, Ownership, and Advanced Mapping

## Introduction

Real-world applications rarely have isolated tables.

For example:

```
A User has one Address.

A User has many Orders.

A Student can enroll in many Courses.

A Course can have many Students.
```

These relationships exist in the database through **foreign keys**, and JPA provides annotations to represent them as Java objects.

In this guide, we'll learn:

- Unidirectional vs Bidirectional relationships
- `@OneToOne`
- `@OneToMany`
- `@ManyToMany`
- `@JoinColumn`
- `@JoinTable`
- Relationship ownership
- `@MapsId`
- A few important Lombok annotations

---

# Database First vs JPA First

Before learning relationships, it's important to understand one thing.

Suppose your entity contains:

```java
@Column(nullable = false)
private String email;
```

Many beginners think this always creates a `NOT NULL` constraint.

**Not necessarily.**

It depends on **who created the database schema**.

---

## JPA First Approach

```
Java Entity

↓

Hibernate

↓

Database Tables
```

Hibernate reads:

```java
@Column(nullable = false)
```

and generates

```sql
email VARCHAR(255) NOT NULL
```

Here, `nullable = false` directly affects the database schema.

---

## Database First Approach (Flyway)

```
Flyway SQL

↓

Database Tables

↓

JPA Entities
```

Example:

```sql
CREATE TABLE users (
    email VARCHAR(255) NOT NULL
);
```

Later:

```java
@Column(nullable = false)
private String email;
```

The annotation **does not modify the database**.

The database has already been created.

---

## So why keep `nullable = false`?

Even in a database-first approach, it is still useful because it:

- Documents the database constraints.
- Makes the entity easier to understand.
- Keeps the entity consistent with the schema.
- Can be used by validation tools and IDEs.

But the **actual enforcement happens in the database**, not because of JPA.

> **Rule of thumb:** If Flyway manages your schema, Flyway is the source of truth, not your JPA annotations.

---

# Understanding Relationships

Suppose we have:

```
User

↓

Address
```

Database:

```
users

id
name

addresses

id
user_id
city
```

The foreign key is:

```
addresses.user_id
```

JPA simply represents this relationship using objects.

---

# Unidirectional vs Bidirectional Relationships

## Unidirectional

Only one class knows about the relationship.

```
User

↓

Address
```

```java
public class User {

    private Address address;

}
```

Address has no idea who owns it.

---

## Bidirectional

Both classes know each other.

```
User

↓

↑

Address
```

```java
public class User {

    private Address address;

}
```

```java
public class Address {

    private User user;

}
```

Now navigation is possible in both directions.

```
user.getAddress()

address.getUser()
```

---

## Which should you choose?

Prefer **unidirectional** unless you actually need navigation in both directions.

Bidirectional relationships are harder to maintain because both sides must stay synchronized.

---

# Relationship Ownership

This is one of the most important JPA concepts.

Many beginners think both sides update the database.

They don't.

Only **one side owns the relationship.**

The owner is responsible for updating the foreign key.

---

Example:

```
users

id

addresses

id
user_id
```

Who owns `user_id`?

The table that **contains the foreign key**.

```
addresses
```

Therefore:

```
Address

↓

@JoinColumn

↓

OWNER
```

The other side is called the **inverse side**.

---

# @OneToOne

Suppose every user has exactly one address.

```
User

↓

Address
```

### Owning Side

```java
@Entity
public class Address {

    @OneToOne
    @JoinColumn(name = "user_id")
    private User user;

}
```

`@JoinColumn` tells Hibernate:

```
The foreign key is

user_id
```

---

### Inverse Side

```java
@Entity
public class User {

    @OneToOne(mappedBy = "user")
    private Address address;

}
```

Notice:

```
mappedBy = "user"
```

Not

```
mappedBy = "user_id"
```

`mappedBy` refers to the **Java field name**, not the database column.

---

# @OneToMany

Suppose:

```
One User

↓

Many Orders
```

Database:

```
orders

user_id
```

The foreign key exists in **Order**.

Therefore:

```
Order

↓

OWNER
```

---

### Many Side

```java
@Entity
public class Order {

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

}
```

---

### One Side

```java
@Entity
public class User {

    @OneToMany(mappedBy = "user")
    private List<Order> orders;

}
```

Again,

```
mappedBy = "user"
```

means:

```
Look at

private User user;
```

inside Order.

---

# What does mappedBy actually mean?

Suppose:

```java
public class Order {

    private User customer;

}
```

Then:

```java
@OneToMany(mappedBy = "customer")
```

Not:

```
mappedBy = "user_id"

mappedBy = "users"

mappedBy = "orders"
```

It always references the **field name**.

---

# @ManyToMany

Suppose:

```
Users

↓

Tags

↑
```

A user can have many tags.

A tag can belong to many users.

Database:

```
users

tags

user_tags
```

The middle table stores the relationship.

---

## @JoinTable

```java
@ManyToMany
@JoinTable(
    name = "user_tags",
    joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "tag_id")
)
private Set<Tag> tags;
```

This annotation usually confuses beginners, so let's break it down.

---

## Imagine These Tables

```
users

+----+
| id |
+----+

tags

+----+
| id |
+----+
```

Since many users can have many tags,

we need another table.

```
user_tags

+---------+--------+
| user_id | tag_id |
+---------+--------+
```

Example:

```
user_tags

+---------+--------+
|    1    |    3   |
|    1    |    5   |
|    2    |    3   |
+---------+--------+
```

Meaning:

```
User 1

↓

Tag 3

Tag 5

User 2

↓

Tag 3
```

---

## Understanding @JoinTable

```java
@JoinTable(

    name = "user_tags",

    joinColumns =
        @JoinColumn(name = "user_id"),

    inverseJoinColumns =
        @JoinColumn(name = "tag_id")

)
```

### name

```
name = "user_tags"
```

This is the **join table**.

```
users

↓

user_tags

↓

tags
```

---

### joinColumns

```
joinColumns =
    @JoinColumn(name = "user_id")
```

This refers to **this entity**.

Suppose we are inside:

```java
class User {

}
```

Then

```
user_id
```

points to the User table.

Think:

> "Which column stores **my** ID?"

---

### inverseJoinColumns

```
inverseJoinColumns =
    @JoinColumn(name = "tag_id")
```

This refers to the **other entity**.

Think:

> "Which column stores the other entity's ID?"

---

A simple way to remember:

```
Current Entity

↓

joinColumns

Other Entity

↓

inverseJoinColumns
```

---

# Fetch Strategies

JPA decides **when** related objects should be loaded.

---

## EAGER

```java
@ManyToMany(fetch = FetchType.EAGER)
```

Loads everything immediately.

```
Load User

↓

Load Tags

↓

Load Address

↓

Load Orders
```

Useful for small relationships but can become expensive.

---

## LAZY

```java
@ManyToMany(fetch = FetchType.LAZY)
```

Loads related data only when needed.

```
Load User

↓

Later...

↓

user.getTags()

↓

Now load Tags
```

Recommended for most collections.

---

# @MapsId

Suppose:

```
User

↓

Passport
```

Both share the same primary key.

```
users

id = 5

passport

id = 5
```

Instead of:

```
passport_id

user_id
```

they both use the same ID.

---

Example:

```java
@Entity
public class Passport {

    @Id
    private Long id;

    @OneToOne
    @MapsId
    @JoinColumn(name = "id")
    private User user;

}
```

`@MapsId` tells Hibernate:

> "Use the primary key of User as the primary key of Passport."

This is commonly used in shared primary-key one-to-one relationships.

---

# Useful Lombok Annotations

---

## @Builder

Creates objects using the Builder Pattern.

```java
User user = User.builder()
        .name("John")
        .email("john@gmail.com")
        .build();
```

---

## @Builder.Default

Normally,

```java
@Builder
class User {

    private List<String> roles = new ArrayList<>();

}
```

The builder ignores this default value.

Instead use:

```java
@Builder.Default
private List<String> roles = new ArrayList<>();
```

Now every object created using the builder starts with an empty list unless another value is provided.

---

## @ToString

Automatically generates:

```java
System.out.println(user);
```

Output:

```
User(id=1, name=John, email=john@gmail.com)
```

---

## Why can @ToString be dangerous?

Suppose:

```
User

↓

Orders

↑

User
```

A bidirectional relationship.

Printing:

```
User

↓

Orders

↓

User

↓

Orders

↓

User
```

never ends.

Eventually causing a:

```
StackOverflowError
```

---

## @ToString.Exclude

Exclude relationship fields.

```java
@ToString.Exclude
@OneToMany(mappedBy = "user")
private List<Order> orders;
```

Now printing a User doesn't recursively print Orders (and back to User again).

This is highly recommended for bidirectional relationships.

---

# Best Practices

- Let Flyway define your schema when following a database-first approach.
- Keep JPA annotations consistent with the database even if they don't create the schema.
- Prefer unidirectional relationships unless navigation in both directions is required.
- Remember: **the side with the foreign key owns the relationship.**
- `mappedBy` always refers to the **Java field name**, not the database column.
- Use `FetchType.LAZY` for most collections (`@OneToMany`, `@ManyToMany`) to avoid unnecessary data loading.
- Use `@ToString.Exclude` on relationship fields in bidirectional mappings.
- Use `@Builder.Default` whenever a builder should preserve field default values.

---

# Common Interview Questions

### Q1. Does `@Column(nullable = false)` create a `NOT NULL` constraint?

Only if Hibernate is generating the schema (JPA-first approach). If Flyway or another migration tool creates the database, it is mainly documentation and metadata—the database constraint must already exist in the migration.

---

### Q2. What is the difference between unidirectional and bidirectional relationships?

A unidirectional relationship can be navigated from one entity to another, while a bidirectional relationship can be navigated from both entities.

---

### Q3. What does `mappedBy` mean?

`mappedBy` identifies the inverse side of a relationship and points to the **field name** in the owning entity that manages the relationship.

---

### Q4. Which side owns a relationship?

The owning side is the entity that contains the foreign key (or defines the join table). Only the owning side updates the relationship in the database.

---

### Q5. What is the purpose of `@JoinTable`?

`@JoinTable` defines the intermediate table used to represent a many-to-many relationship, including which columns reference each participating entity.

---

### Q6. What does `@MapsId` do?

`@MapsId` maps a relationship so that the child entity shares the same primary key as its parent.

---

### Q7. Why use `@Builder.Default`?

Because the Lombok builder ignores normal field initializers. `@Builder.Default` ensures those default values are preserved when using the builder.

---

## Summary

- In a **database-first** approach, Flyway defines the schema, while JPA annotations primarily document and map it.
- Choose **unidirectional** relationships by default and use bidirectional mappings only when needed.
- The entity containing the **foreign key** is usually the **owning side** of the relationship.
- `mappedBy` always references a **Java field name**, not a database column.
- `@JoinColumn` maps foreign keys, while `@JoinTable` maps join tables for many-to-many relationships.
- `FetchType.LAZY` is generally preferred for collections to improve performance.
- `@MapsId` is used when two entities share the same primary key.
- `@Builder.Default` and `@ToString.Exclude` are important Lombok annotations that prevent common pitfalls when working with entities.