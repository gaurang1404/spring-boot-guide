# Database Versioning with Flyway

## Introduction

As applications grow, the database schema changes over time.

Examples:
- Adding a new table
- Adding a new column
- Renaming a column
- Creating indexes
- Adding foreign keys

If developers manually execute SQL scripts, it becomes difficult to keep everyone's database synchronized.

Questions like these start appearing:

- Which SQL scripts have already been executed?
- Has everyone updated their local database?
- What happens when deploying to production?
- How do we ensure migrations run only once?

This is where **Flyway** comes in.

---

# What is Flyway?

**Flyway** is a database migration tool that manages and versions your database schema.

Instead of manually executing SQL scripts, Flyway automatically detects and executes migration files in the correct order.

Think of it as **Git for your database schema**.

```
Application

        │

Flyway

        │

Migration Files (SQL)

        │

Database
```

---

## Why do we need Flyway?

Without Flyway:

```
Developer A
    │
Runs script manually

Developer B
    │
Forgot to run it

Production
    │
Different schema
```

Everyone can end up with different database structures.

With Flyway:

- Every database follows the same migration history.
- SQL scripts run only once.
- Schema changes are version controlled.
- Easy to deploy database changes.
- Team members stay synchronized.

---

# How Flyway Works

Imagine your project has these migration files:

```
V1__initial_migration.sql

V2__add_phone_number.sql

V3__create_orders_table.sql
```

When the application starts (or Flyway is executed):

1. Flyway checks the database.
2. It looks for previously executed migrations.
3. It finds new migration files.
4. Executes them in version order.
5. Records each migration in a history table.

Next time it runs, only new migrations are executed.

---

# Adding Flyway to Spring Boot

## Step 1: Add Dependencies

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>

<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

- `flyway-core` contains the Flyway migration engine.
- `flyway-mysql` provides MySQL-specific database support.

Once these dependencies are added, Spring Boot automatically detects Flyway and runs migrations during application startup.

---

# Creating Migration Files

Flyway expects migration files inside:

```
src
└── main
    └── resources
        └── db
            └── migration
```

Create the first migration:

```
V1__initial_migration.sql
```

The naming convention is important:

```
V1__initial_migration.sql

││
│└── Description
│
└── Version Number
```

Flyway executes migrations in ascending version order.

---

# Example Migration

```sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL
);

CREATE TABLE addresses (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,

    address_line1 VARCHAR(255) NOT NULL,
    address_line2 VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    country VARCHAR(100) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,

    CONSTRAINT fk_address_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
);
```

When Flyway executes this migration, both tables are created.

---

# How Flyway Applies Migrations

When Flyway starts, it performs the following steps:

```
Application Starts
        │
        ▼
Reads Migration Folder
        │
        ▼
Checks Database
        │
        ▼
Reads flyway_schema_history
        │
        ▼
Finds New Migration Files
        │
        ▼
Executes Missing Migrations
        │
        ▼
Updates Migration History
```

Each migration is executed only once.

---

# flyway_schema_history

One of the first things Flyway creates is:

```
flyway_schema_history
```

This table tracks every migration that has been executed.

Example:

| Installed Rank | Version | Description | Success |
|---------------:|---------|-------------|---------|
| 1 | 1 | initial migration | ✔ |

This table contains additional information such as:

- Migration version
- Description
- Script name
- Checksum
- Installation time
- Execution time
- Success status

Before running any migration, Flyway checks this table to determine which scripts have already been applied.

---

# Migration Versioning

Suppose you now want to add a phone number column.

❌ **Do not modify**:

```
V1__initial_migration.sql
```

Instead create:

```
V2__add_phone_number.sql
```

```sql
ALTER TABLE users
ADD phone_number VARCHAR(20);
```

Later:

```
V3__create_orders_table.sql
```

Every schema change should be represented by a **new migration file**.

---

# Why Should You Never Edit an Executed Migration?

After executing a migration, Flyway stores a **checksum** for that file in the `flyway_schema_history` table.

On future runs, Flyway recalculates the checksum and compares it with the stored value.

If they don't match, Flyway assumes the migration has been modified.

Example:

```
Original

V1__initial_migration.sql

↓

Checksum Stored

↓

Later someone edits the file

↓

Checksum Changes

↓

Flyway Validation Error
```

This protects your database from accidental or unauthorized schema changes.

> **Best Practice:** Once a migration has been applied, treat it as immutable. Always create a new migration for any future changes.

---

# Running Flyway Using Maven

By default, Flyway migrations run when the Spring Boot application starts.

However, during development, restarting the application just to apply migrations can become tedious.

Flyway provides a Maven plugin that allows migrations to be run directly from the command line.

---

## Add the Flyway Maven Plugin

Inside the `<plugins>` section of `pom.xml`:

```xml
<plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
    <version>10.15.0</version>

    <configuration>
        <url>jdbc:mysql://localhost:3306/pt_dev</url>
        <user>root</user>
        <password>24142414</password>
    </configuration>
</plugin>
```

Now Flyway can be executed without starting the application.

---

# Useful Flyway Maven Commands

## Migrate

```bash
mvn flyway:migrate
```

Applies all pending migrations.

Most commonly used command.

---

## Validate

```bash
mvn flyway:validate
```

Checks:

- Missing migrations
- Modified migration files
- Checksum mismatches
- Version consistency

It does **not** execute migrations.

---

## Repair

```bash
mvn flyway:repair
```

Repairs Flyway metadata.

Common uses:

- Update checksums after intentional changes (not recommended in production)
- Remove failed migration entries
- Fix metadata inconsistencies

Use this command carefully.

---

## Clean

```bash
mvn flyway:clean
```

Deletes **all database objects managed by Flyway**, including tables, views, and other schema objects.

Because this operation is destructive, Flyway disables it by default.

To enable it:

```xml
<configuration>

    <cleanDisabled>false</cleanDisabled>

</configuration>
```

Only enable `clean` in development environments.

Never enable it in production.

---

# Spring Boot Parent Version Compatibility

While setting up Flyway, an issue was encountered:

- Spring Boot Starter Parent **4.1.0** → Flyway did not work correctly.
- Spring Boot Starter Parent **3.5.5** → Flyway worked as expected.

### Why?

Spring Boot **4.x** is based on a newer generation of the Spring ecosystem (Spring Framework 7 and Jakarta EE 11), and at the time of testing, there were compatibility changes between Spring Boot 4.x, Flyway, and other project dependencies. Since Spring Boot 4.x was still in its early stages, not all libraries and tooling had fully aligned with it.

Spring Boot **3.5.5** is a stable release with mature support for Flyway, Hibernate, and the broader Spring ecosystem, making it a more reliable choice for learning and production projects.

> **Recommendation:** Unless you specifically need Spring Boot 4.x features, use the latest stable 3.x release for better compatibility.

---

# Best Practices

- Treat migration files as immutable after they have been executed.
- Create a new migration file for every schema change.
- Follow the `V<version>__description.sql` naming convention.
- Keep migrations small and focused.
- Use `validate` regularly to detect issues early.
- Use `repair` only when you understand why the metadata needs fixing.
- Never run `clean` against a production database.
- Store migration files in version control along with your application code.

---

# Common Interview Questions

### Q1. What is Flyway?

Flyway is a database migration tool that versions and manages database schema changes using SQL migration files.

---

### Q2. Why is Flyway used?

It ensures all environments (development, testing, production) have the same database schema by applying versioned migrations consistently.

---

### Q3. What is the purpose of the `flyway_schema_history` table?

It records every migration that has been executed, including its version, checksum, execution time, and status. Flyway uses it to determine which migrations still need to run.

---

### Q4. Why shouldn't you edit an executed migration?

Flyway stores a checksum for every executed migration. Editing the file changes the checksum, causing validation to fail. Instead, create a new migration file.

---

### Q5. What is the difference between `migrate`, `validate`, `repair`, and `clean`?

| Command | Purpose |
|---------|---------|
| `migrate` | Applies pending migrations |
| `validate` | Verifies migration history and checksums |
| `repair` | Repairs Flyway metadata and checksums |
| `clean` | Deletes all objects managed by Flyway |

---

## Summary

- Flyway manages and versions database schema changes.
- Database changes are written as versioned SQL migration files.
- Migrations are stored in `src/main/resources/db/migration`.
- Flyway executes each migration only once and records it in `flyway_schema_history`.
- Never modify an already executed migration—create a new version instead.
- The Flyway Maven Plugin lets you manage migrations without restarting the application.
- `migrate`, `validate`, `repair`, and `clean` are the most commonly used Flyway commands.
- Spring Boot 3.x currently provides a more mature and stable Flyway experience than early Spring Boot 4.x releases.