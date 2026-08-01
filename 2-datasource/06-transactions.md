# JPA Entity Lifecycle, Persistence Context and Transactions

## Introduction

When working with JPA, entities don't simply exist as ordinary Java objects.

Hibernate keeps track of an entity's lifecycle, determines when changes should be synchronized with the database, and decides when to execute SQL statements.

Understanding the **Persistence Context**, **Entity Lifecycle**, **Transactions**, and the **EntityManager** is essential for understanding how Hibernate works internally.

---

# The Big Picture

Whenever we interact with the database, Hibernate works something like this:

```
@Transactional

        │

Transaction Starts

        │

Persistence Context Created

        │

Entities Become Managed

        │

Changes are Tracked

        │

Transaction Commits

        │

Hibernate Executes SQL

        │

Persistence Context Destroyed
```

Notice that Hibernate doesn't immediately execute SQL whenever you modify an entity.

Instead, it tracks changes and synchronizes them with the database when the transaction completes.

---

# What is a Persistence Context?

A **Persistence Context** is a container managed by Hibernate that stores and tracks entities.

Think of it as Hibernate's **workspace**.

```
Persistence Context

+-------------------------+

User(id=1)

Order(id=15)

Address(id=5)

+-------------------------+
```

Every entity inside this container is called a **Managed Entity**.

Hibernate watches these objects for changes.

---

## Why is it needed?

Suppose we load a user:

```java
User user = entityManager.find(User.class, 1L);

user.setName("John");
```

We never call:

```java
UPDATE users...
```

Yet the database gets updated.

Why?

Because Hibernate is tracking the object inside the Persistence Context.

It knows:

```
Original Name

↓

John

↓

Changed

↓

Generate UPDATE SQL
```

---

# Entity Lifecycle

A JPA entity moves through different states during its lifetime.

```
Transient

↓

Managed (Persistent)

↓

Detached

↓

Removed
```

Let's understand each state.

---

# 1. Transient State

A newly created object.

Hibernate knows nothing about it.

Example:

```java
User user = new User();

user.setName("John");
```

This object:

- Exists only in memory.
- Has never been stored.
- Is **not** inside the Persistence Context.

```
Transient

User

↓

Not Tracked
```

---

# 2. Persistent (Managed) State

Once an entity enters the Persistence Context, it becomes **Managed**.

Example:

```java
User user = entityManager.find(User.class, 1L);
```

or

```java
userRepository.save(user);
```

Now Hibernate tracks every change.

```java
user.setName("Alice");
```

No explicit update call is required.

Hibernate automatically detects the modification.

This feature is known as **Dirty Checking**.

```
Managed Entity

↓

Field Changes

↓

Hibernate Detects Changes

↓

UPDATE Generated
```

---

# 3. Detached State

A detached entity was **previously managed**, but is no longer inside the Persistence Context.

```
Persistence Context

User(id=1)

↓

Transaction Ends

↓

Detached
```

Notice something important:

A detached entity **already has an identifier**.

For example:

```java
User user =
        userRepository.findById(1L)
                .orElseThrow();
```

After the transaction ends:

```
User(id=1)
```

still exists.

But Hibernate is no longer tracking it.

Changing it now does nothing.

```java
user.setName("Bob");
```

No SQL will be generated.

---

## Can a Detached Entity Become Managed Again?

Yes.

Suppose:

```java
userRepository.save(user);
```

Since the entity already has an ID,

Hibernate recognizes that it already exists.

The entity becomes managed again inside the new Persistence Context.

```
Detached

↓

save()

↓

Managed Again
```

---

# 4. Removed State

When we delete an entity:

```java
userRepository.delete(user);
```

Hibernate marks it as **Removed**.

```
Managed

↓

delete()

↓

Removed
```

The actual SQL isn't executed immediately.

Instead,

Hibernate waits until the transaction commits.

Then:

```sql
DELETE FROM users
WHERE id = ?
```

is executed.

---

# Understanding the Lifecycle

```
new User()

↓

Transient

↓

save()

↓

Managed

↓

Transaction Ends

↓

Detached

↓

delete()

↓

Removed
```

---

# What is a Transaction?

A **Transaction** is a group of database operations treated as a single unit of work.

Either:

```
Everything Succeeds
```

or

```
Everything Fails
```

There is no in-between.

---

## Example

Suppose we are transferring money.

```
Withdraw ₹1000

↓

Deposit ₹1000
```

If withdrawal succeeds but deposit fails,

the system becomes inconsistent.

Instead:

```
Start Transaction

↓

Withdraw

↓

Deposit

↓

Commit
```

If anything fails:

```
Rollback

↓

Undo Everything
```

---

# Transaction Boundary

Every transaction has a beginning and an end.

```
Method Starts

↓

Transaction Begins

↓

Business Logic

↓

Transaction Commits

↓

Method Ends
```

Everything inside this area is called the **Transaction Boundary**.

---

# Persistence Context and Transactions

A Persistence Context usually exists **only for the duration of a transaction**.

```
Transaction Starts

↓

Persistence Context Created

↓

Entities Managed

↓

Transaction Commits

↓

Persistence Context Destroyed
```

This is why entity lifecycle and transactions are closely related.

---

# @Transactional

Spring provides:

```java
@Transactional
```

to automatically manage transactions.

Example:

```java
@Service
public class UserService {

    @Transactional
    public void updateUser(Long id) {

        User user = userRepository
                .findById(id)
                .orElseThrow();

        user.setName("Alice");

    }

}
```

Notice:

There is no:

```java
userRepository.save(user);
```

Yet Hibernate still updates the database.

Why?

Because:

- Transaction started.
- Persistence Context created.
- Entity became managed.
- Hibernate detected changes.
- Transaction committed.
- SQL executed.

---

## Can @Transactional be used on static methods?

No.

Spring manages transactions using proxies.

Static methods cannot be proxied in the same way as instance methods.

Therefore:

```java
@Transactional
public static void updateUser() {

}
```

will **not** work as expected.

---

# What is EntityManager?

`EntityManager` is the core JPA interface used to interact directly with the Persistence Context.

Spring Data JPA repositories internally use the EntityManager.

```
Repository

↓

EntityManager

↓

Hibernate

↓

Database
```

Although repositories are sufficient for most applications, understanding `EntityManager` helps explain how JPA works under the hood.

---

## Common EntityManager Operations

### Find

```java
User user =
        entityManager.find(User.class, 1L);
```

Loads an entity by its primary key.

---

### Persist

```java
entityManager.persist(user);
```

Makes a transient entity become managed.

---

### Remove

```java
entityManager.remove(user);
```

Marks an entity for deletion.

---

### Contains

```java
entityManager.contains(user);
```

Returns:

```
true
```

if the entity is currently inside the Persistence Context.

Otherwise:

```
false
```

Example:

```java
User user =
        entityManager.find(User.class, 1L);

System.out.println(
        entityManager.contains(user)
);
```

Output:

```
true
```

After the transaction ends:

```java
System.out.println(
        entityManager.contains(user)
);
```

Output:

```
false
```

because the entity is now detached.

---

# Why Doesn't Hibernate Update Immediately?

Suppose:

```java
user.setName("Alice");

user.setEmail("alice@gmail.com");

user.setPassword("12345");
```

If Hibernate executed SQL immediately:

```
UPDATE

UPDATE

UPDATE
```

Three separate queries.

Instead, Hibernate waits until the transaction commits.

```
Transaction

↓

Track Changes

↓

Single UPDATE
```

This improves performance significantly.

---

# Best Practices

- Keep transactions as short as possible.
- Perform business logic inside service methods annotated with `@Transactional`.
- Avoid placing `@Transactional` on controller methods.
- Never rely on changes to detached entities being automatically saved.
- Understand that managed entities are automatically synchronized with the database through dirty checking.
- Use repositories for most CRUD operations and `EntityManager` only when you need lower-level JPA features.

---

# Common Interview Questions

### Q1. What is a Persistence Context?

A Persistence Context is a container managed by Hibernate that stores and tracks managed entities during a transaction.

---

### Q2. What are the four JPA entity lifecycle states?

- Transient
- Managed (Persistent)
- Detached
- Removed

---

### Q3. What is a detached entity?

A detached entity is an entity that was previously managed but is no longer associated with a Persistence Context. It still has an identifier but is no longer tracked for changes.

---

### Q4. What happens when `save()` is called on a detached entity?

The entity becomes managed again within the current Persistence Context, allowing Hibernate to synchronize its changes with the database.

---

### Q5. What is the purpose of `@Transactional`?

`@Transactional` defines a transaction boundary. Spring automatically starts a transaction before the method executes and commits (or rolls back) the transaction when the method finishes.

---

### Q6. Why can't `@Transactional` be applied to static methods?

Because Spring manages transactions using proxy-based AOP, which intercepts instance method calls. Static methods are not invoked through these proxies, so transactional behavior is not applied.

---

### Q7. What does `EntityManager.contains(entity)` do?

It returns `true` if the entity is currently managed by the Persistence Context, otherwise it returns `false`.

---

## Summary

- A **Persistence Context** is Hibernate's workspace where managed entities are tracked.
- Entities move through four lifecycle states: **Transient**, **Managed**, **Detached**, and **Removed**.
- Only **managed entities** are automatically synchronized with the database through dirty checking.
- A **Transaction** groups database operations into a single unit of work—either all operations succeed or all are rolled back.
- A transaction defines the **transaction boundary**, and a Persistence Context is typically created and destroyed within that boundary.
- `@Transactional` simplifies transaction management by automatically starting, committing, or rolling back transactions.
- `EntityManager` is the core JPA interface responsible for interacting with the Persistence Context and managing entity lifecycle operations.