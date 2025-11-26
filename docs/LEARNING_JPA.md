# Learning Guide: Creating JPA Entities from Database Tables

A step-by-step guide for manually creating JPA entities from existing database tables in the HelloJPA project.

---

## 📚 Learning Goal

**Master the process of creating JPA entities manually from existing database tables** - understanding every annotation, mapping, and convention step-by-step (like Eclipse JPA wizard does, but doing it manually to learn deeply).

---

## ✅ Prerequisites

Before starting, ensure you have:

- ✅ **HelloJPA Project Setup:**
  - Java 17, Hibernate 6.4.1, Jakarta Persistence 3.1.0
  - `persistence.xml` configured and tested (connected to UdemyShopDB)
  - `src/main/java/` is EMPTY - ready for learning
  - Database connection: `proit@UdemyShopDB`

- ✅ **Database Ready:**
  - SQL Server 2019 running in Docker
  - UdemyShopDB exists
  - User `proit` has db_owner permissions
  - At least one table exists to map

---

## 🎯 Learning Approach

### Why Manual Entity Creation?

Instead of using auto-generation tools (like Eclipse JPA wizard), we create entities manually to:

1. **Understand JPA annotations deeply** - Know what each annotation does
2. **Learn SQL → Java type mappings** - Master data type conversions
3. **Grasp entity lifecycle** - Understand how JPA manages objects
4. **Build troubleshooting skills** - Debug mapping issues effectively
5. **Develop best practices** - Write clean, maintainable entity code

### Learning Methodology

```
Database Table → Analyze Structure → Design Entity → Write Code → Test → Review
```

1. **Analyze** database table structure (SSMS or SQL queries)
2. **Create** package structure for entities
3. **Write** entity class manually with proper JPA annotations
4. **Test** entity with simple CRUD operations
5. **Review** and understand each annotation and mapping

---

## 📖 Step-by-Step Process

### Phase 1: Choose Your First Table

**Recommendation for Beginners:**
- Start with a **simple table** (3-5 columns, no relationships)
- Learn basic annotations first
- Practice with complex tables later

**How to Choose:**

```sql
-- List all tables in UdemyShopDB
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE'
ORDER BY TABLE_NAME;
```

**Good starter tables:**
- Products, Categories, Tags (simple entities)
- Avoid: OrderItems, CartDetails (have foreign keys)

---

### Phase 2: Analyze Table Structure

**Step 2.1: Get Table Metadata**

Use SSMS or run this query:

```sql
-- Replace 'products' with your table name
SELECT
    c.COLUMN_NAME,
    c.DATA_TYPE,
    c.CHARACTER_MAXIMUM_LENGTH,
    c.NUMERIC_PRECISION,
    c.NUMERIC_SCALE,
    c.IS_NULLABLE,
    c.COLUMN_DEFAULT,
    CASE WHEN pk.COLUMN_NAME IS NOT NULL THEN 'YES' ELSE 'NO' END AS IS_PRIMARY_KEY
FROM
    INFORMATION_SCHEMA.COLUMNS c
LEFT JOIN (
    SELECT ku.COLUMN_NAME
    FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS AS tc
    INNER JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE AS ku
        ON tc.CONSTRAINT_TYPE = 'PRIMARY KEY'
        AND tc.CONSTRAINT_NAME = ku.CONSTRAINT_NAME
        AND ku.TABLE_NAME = 'products'
) pk ON c.COLUMN_NAME = pk.COLUMN_NAME
WHERE
    c.TABLE_NAME = 'products'
ORDER BY
    c.ORDINAL_POSITION;
```

**Step 2.2: Document Your Findings**

Create a mapping plan:

```
Table: products
│
├── id (BIGINT, PRIMARY KEY, IDENTITY)
│   └─→ Java: Long id
│       └─→ Annotations: @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
│
├── name (NVARCHAR(100), NOT NULL)
│   └─→ Java: String name
│       └─→ Annotations: @Column(name = "name", nullable = false, length = 100)
│
├── description (NVARCHAR(500), NULL)
│   └─→ Java: String description
│       └─→ Annotations: @Column(name = "description", length = 500)
│
└── price (DECIMAL(10,2), NULL)
    └─→ Java: BigDecimal price
        └─→ Annotations: @Column(name = "price", precision = 10, scale = 2)
```

---

### Phase 3: SQL Type → Java Type Mapping Reference

| SQL Server Type | Java Type | JPA Annotation | Notes |
|-----------------|-----------|----------------|-------|
| **BIGINT** | `Long` | `@Column` | Use wrapper class for nullable |
| **INT** | `Integer` | `@Column` | Use wrapper class for nullable |
| **SMALLINT** | `Short` | `@Column` | |
| **TINYINT** | `Byte` | `@Column` | |
| **BIT** | `Boolean` | `@Column` | |
| **DECIMAL(p,s)** | `BigDecimal` | `@Column(precision=p, scale=s)` | For money/precise values |
| **FLOAT** | `Double` | `@Column` | |
| **REAL** | `Float` | `@Column` | |
| **NVARCHAR(n)** | `String` | `@Column(length=n)` | Unicode strings |
| **VARCHAR(n)** | `String` | `@Column(length=n)` | ASCII strings |
| **NCHAR(n)** | `String` | `@Column(length=n)` | Fixed length |
| **TEXT** | `String` | `@Lob` | Large text |
| **DATE** | `LocalDate` | `@Column` | Java 8+ date |
| **DATETIME** | `LocalDateTime` | `@Column` | Java 8+ timestamp |
| **DATETIME2** | `LocalDateTime` | `@Column` | Higher precision |
| **TIME** | `LocalTime` | `@Column` | Java 8+ time |
| **TIMESTAMP** | `Timestamp` | `@Column` | Legacy, avoid if possible |
| **VARBINARY** | `byte[]` | `@Lob` | Binary data |
| **UNIQUEIDENTIFIER** | `UUID` | `@Column` | GUIDs |

**⚠️ Important Rules:**

1. **Use wrapper classes** (Long, Integer, Boolean) for nullable columns - prevents NullPointerException
2. **Use BigDecimal** for money/currency - never use Float/Double
3. **Use Java 8+ date/time types** (LocalDate, LocalDateTime) - not java.util.Date
4. **Match nullability** - nullable in DB = nullable Java field type

---

### Phase 4: Create Entity Class

**Step 4.1: Create Package Structure**

```bash
cd HelloJPA/src/main/java
mkdir -p org/example/entity
```

**Step 4.2: Create Entity Class**

File: `org/example/entity/Product.java`

```java
package org.example.entity;

import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity                                    // 1. Marks this class as JPA entity
@Table(name = "products")                  // 2. Maps to "products" table
public class Product {

    // PRIMARY KEY
    @Id                                    // 3. Primary key field
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // 4. Auto-increment
    @Column(name = "id")                   // 5. Maps to "id" column
    private Long id;

    // REGULAR COLUMNS
    @Column(name = "name", nullable = false, length = 100)
    private String name;

    @Column(name = "description", length = 500)
    private String description;

    @Column(name = "price", precision = 10, scale = 2)
    private BigDecimal price;

    // CONSTRUCTORS
    public Product() {                      // 6. NO-ARG constructor (REQUIRED by JPA)
    }

    public Product(String name, String description, BigDecimal price) {
        this.name = name;
        this.description = description;
        this.price = price;
    }

    // GETTERS AND SETTERS (REQUIRED)
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }

    // OPTIONAL BUT RECOMMENDED
    @Override
    public String toString() {
        return "Product{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", description='" + description + '\'' +
                ", price=" + price +
                '}';
    }
}
```

**Annotation Explanations:**

| Annotation | Purpose | Required? |
|------------|---------|-----------|
| `@Entity` | Marks class as JPA entity | ✅ Yes |
| `@Table(name = "...")` | Maps to database table | ⚠️ Recommended (explicit is better) |
| `@Id` | Marks primary key field | ✅ Yes |
| `@GeneratedValue` | Auto-increment strategy | Only for auto-generated keys |
| `@Column(name = "...")` | Maps to database column | ⚠️ Recommended (explicit is better) |

**GenerationType Strategies:**

| Strategy | Description | SQL Server |
|----------|-------------|------------|
| `IDENTITY` | Database auto-increment | ✅ Use for IDENTITY columns |
| `SEQUENCE` | Database sequence | ❌ Not supported in SQL Server |
| `TABLE` | Separate table for IDs | ⚠️ Avoid (performance) |
| `AUTO` | JPA chooses strategy | ⚠️ Not recommended (unpredictable) |

---

### Phase 5: Create Main Class for Testing

File: `org/example/Main.java`

```java
package org.example;

import jakarta.persistence.*;
import org.example.entity.Product;
import java.math.BigDecimal;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        // Get EntityManagerFactory from persistence.xml
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("HelloJPU");
        EntityManager em = emf.createEntityManager();

        try {
            // TEST 1: CREATE - Insert new entity
            System.out.println("\n=== TEST 1: CREATE ===");
            em.getTransaction().begin();

            Product laptop = new Product("Laptop", "Dell XPS 13", new BigDecimal("1299.99"));
            em.persist(laptop);  // Save to database

            em.getTransaction().commit();
            System.out.println("✅ Created: " + laptop);

            // TEST 2: READ - Find by ID
            System.out.println("\n=== TEST 2: READ ===");
            Product found = em.find(Product.class, laptop.getId());
            System.out.println("✅ Found: " + found);

            // TEST 3: READ ALL - Query all entities
            System.out.println("\n=== TEST 3: READ ALL ===");
            List<Product> products = em.createQuery("SELECT p FROM Product p", Product.class)
                    .getResultList();
            products.forEach(p -> System.out.println("  - " + p));

            // TEST 4: UPDATE - Modify entity
            System.out.println("\n=== TEST 4: UPDATE ===");
            em.getTransaction().begin();

            found.setPrice(new BigDecimal("1199.99"));
            em.merge(found);  // Update in database

            em.getTransaction().commit();
            System.out.println("✅ Updated: " + found);

            // TEST 5: DELETE - Remove entity
            System.out.println("\n=== TEST 5: DELETE ===");
            em.getTransaction().begin();

            em.remove(found);  // Delete from database

            em.getTransaction().commit();
            System.out.println("✅ Deleted product ID: " + found.getId());

            System.out.println("\n=== ALL TESTS PASSED ===");

        } catch (Exception e) {
            if (em.getTransaction().isActive()) {
                em.getTransaction().rollback();
            }
            System.err.println("❌ Error: " + e.getMessage());
            e.printStackTrace();
        } finally {
            em.close();
            emf.close();
        }
    }
}
```

**Key CRUD Operations:**

| Operation | JPA Method | Requires Transaction? |
|-----------|------------|----------------------|
| **Create** | `em.persist(entity)` | ✅ Yes |
| **Read** | `em.find(Class, id)` | ❌ No |
| **Update** | `em.merge(entity)` | ✅ Yes |
| **Delete** | `em.remove(entity)` | ✅ Yes |
| **Query** | `em.createQuery(jpql)` | ❌ No |

---

### Phase 6: Run and Test

**Step 6.1: Compile**

```bash
cd HelloJPA
mvn clean compile
```

**Step 6.2: Run Main**

```bash
mvn exec:java -Dexec.mainClass="org.example.Main"
```

**Step 6.3: Check Hibernate SQL Logs**

You should see SQL statements in console:

```sql
Hibernate: insert into products (description,name,price) values (?,?,?)
Hibernate: select p1_0.id,p1_0.description,p1_0.name,p1_0.price from products p1_0 where p1_0.id=?
Hibernate: select p1_0.id,p1_0.description,p1_0.name,p1_0.price from products p1_0
Hibernate: update products set description=?,name=?,price=? where id=?
Hibernate: delete from products where id=?
```

**Step 6.4: Verify in Database**

Open SSMS and check:

```sql
USE UdemyShopDB;
SELECT * FROM products;
```

---

## ✅ Code Review Checklist

Before considering your entity complete, check:

### Entity Class Checklist

- [ ] Has `@Entity` annotation at class level
- [ ] Has `@Table(name = "table_name")` matching database table
- [ ] Has `@Id` annotation on primary key field
- [ ] Uses correct `@GeneratedValue` strategy for auto-increment
- [ ] Has **no-arg constructor** (required by JPA)
- [ ] Has parametrized constructor for convenience
- [ ] All fields have **getters and setters**
- [ ] Uses **wrapper classes** (Long, Integer) for nullable numeric fields
- [ ] Uses `BigDecimal` for money/currency fields
- [ ] Column names in `@Column(name = "...")` match database exactly
- [ ] Nullable constraints match database (`nullable = true/false`)
- [ ] String lengths match database (`length = n`)
- [ ] Has `toString()` method for debugging

### Main Class Checklist

- [ ] Gets `EntityManagerFactory` with correct persistence unit name (`"HelloJPU"`)
- [ ] Creates `EntityManager` from factory
- [ ] Uses `try-finally` to close resources properly
- [ ] Wraps **write operations** (persist, merge, remove) in transactions
- [ ] Calls `begin()` before and `commit()` after write operations
- [ ] Handles exceptions and rolls back on error
- [ ] Closes `EntityManager` and `EntityManagerFactory` in finally block

### Testing Checklist

- [ ] Code compiles without errors
- [ ] Application connects to database successfully
- [ ] CREATE operation inserts data correctly
- [ ] READ operation retrieves correct data
- [ ] UPDATE operation modifies data correctly
- [ ] DELETE operation removes data correctly
- [ ] Hibernate SQL logs appear in console
- [ ] Data appears correctly in SSMS query results

---

## ❌ Common Mistakes & Solutions

### Mistake #1: Forgetting @Entity

```java
// ❌ WRONG
public class Product {
    @Id
    private Long id;
}
```

```java
// ✅ CORRECT
@Entity
@Table(name = "products")
public class Product {
    @Id
    private Long id;
}
```

**Error:** `java.lang.IllegalArgumentException: Unknown entity`

---

### Mistake #2: Using Primitive Types for Nullable Columns

```java
// ❌ WRONG (if column is nullable)
@Column(name = "age")
private int age;  // NullPointerException if NULL in DB!
```

```java
// ✅ CORRECT
@Column(name = "age")
private Integer age;  // Can handle NULL values
```

---

### Mistake #3: Missing No-Arg Constructor

```java
// ❌ WRONG
public class Product {
    public Product(String name) {
        this.name = name;
    }
    // Missing no-arg constructor!
}
```

```java
// ✅ CORRECT
public class Product {
    public Product() {}  // REQUIRED by JPA

    public Product(String name) {
        this.name = name;
    }
}
```

**Error:** `org.hibernate.InstantiationException: No default constructor for entity`

---

### Mistake #4: Forgetting Getters/Setters

```java
// ❌ WRONG
@Entity
public class Product {
    @Id
    private Long id;
    private String name;
    // No getters/setters!
}
```

```java
// ✅ CORRECT
@Entity
public class Product {
    @Id
    private Long id;
    private String name;

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

---

### Mistake #5: Column Name Mismatch

```java
// ❌ WRONG (if DB column is "product_name")
@Column(name = "name")  // Mismatch!
private String name;
```

```java
// ✅ CORRECT
@Column(name = "product_name")  // Matches DB exactly
private String name;
```

**Error:** `SQLSyntaxErrorException: Invalid column name 'name'`

---

### Mistake #6: Forgetting Transactions

```java
// ❌ WRONG
em.persist(product);  // No transaction!
```

```java
// ✅ CORRECT
em.getTransaction().begin();
em.persist(product);
em.getTransaction().commit();
```

**Error:** Changes not saved to database

---

### Mistake #7: Using Float/Double for Money

```java
// ❌ WRONG
@Column(name = "price")
private Double price;  // Precision loss!
```

```java
// ✅ CORRECT
@Column(name = "price", precision = 10, scale = 2)
private BigDecimal price;  // Exact precision
```

---

## 📚 Key JPA Annotations Reference

### Class-Level Annotations

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@Entity` | Marks class as JPA entity | `@Entity` |
| `@Table(name = "...")` | Maps to database table | `@Table(name = "products")` |

### Field-Level Annotations

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@Id` | Primary key | `@Id` |
| `@GeneratedValue` | Auto-increment | `@GeneratedValue(strategy = GenerationType.IDENTITY)` |
| `@Column` | Column mapping | `@Column(name = "name", nullable = false, length = 100)` |
| `@Transient` | Don't persist | `@Transient` |
| `@Lob` | Large object (TEXT/BLOB) | `@Lob` |
| `@Enumerated` | Enum mapping | `@Enumerated(EnumType.STRING)` |

### @Column Attributes

| Attribute | Type | Purpose | Example |
|-----------|------|---------|---------|
| `name` | String | Column name | `name = "product_name"` |
| `nullable` | boolean | Can be NULL? | `nullable = false` |
| `length` | int | String length | `length = 255` |
| `precision` | int | Decimal digits total | `precision = 10` |
| `scale` | int | Decimal digits after point | `scale = 2` |
| `unique` | boolean | Unique constraint | `unique = true` |
| `insertable` | boolean | Include in INSERT | `insertable = true` |
| `updatable` | boolean | Include in UPDATE | `updatable = true` |

---

## 🎓 Practice Exercises

### Exercise 1: Simple Entity (Beginner)

Create a `Category` entity:

**Database Table:**
```sql
CREATE TABLE categories (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(50) NOT NULL,
    description NVARCHAR(200)
);
```

**Your Task:** Create the JPA entity class

---

### Exercise 2: More Data Types (Intermediate)

Create a `Customer` entity with various types:

**Database Table:**
```sql
CREATE TABLE customers (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    email NVARCHAR(100) NOT NULL,
    first_name NVARCHAR(50) NOT NULL,
    last_name NVARCHAR(50) NOT NULL,
    birth_date DATE,
    is_active BIT DEFAULT 1,
    created_at DATETIME2 DEFAULT GETDATE()
);
```

**Your Task:** Map all fields with correct Java types

---

### Exercise 3: Full CRUD Application (Advanced)

Build a complete product management system:

1. Create `Product` entity
2. Implement ProductDAO with methods:
   - `save(Product)` - Create/Update
   - `findById(Long)` - Read
   - `findAll()` - Read all
   - `delete(Long)` - Delete
3. Create console menu interface
4. Test all operations

---

## 🚀 Next Steps

After mastering basic entity creation:

1. **Learn Relationships:**
   - `@OneToMany` / `@ManyToOne`
   - `@ManyToMany`
   - `@OneToOne`
   - Cascade operations
   - Fetch strategies (LAZY vs EAGER)

2. **Advanced Queries:**
   - JPQL (Java Persistence Query Language)
   - Criteria API
   - Native SQL queries
   - Named queries

3. **Entity Lifecycle:**
   - `@PrePersist`, `@PostPersist`
   - `@PreUpdate`, `@PostUpdate`
   - `@PreRemove`, `@PostRemove`

4. **Performance Optimization:**
   - Batch operations
   - Second-level cache
   - Query optimization
   - N+1 problem solutions

---

## 📖 Additional Resources

### Official Documentation
- **Jakarta Persistence:** https://jakarta.ee/specifications/persistence/
- **Hibernate ORM:** https://hibernate.org/orm/documentation/

### Books
- "Java Persistence with Hibernate" by Christian Bauer
- "Pro JPA 2" by Mike Keith and Merrick Schincariol

### Online Tutorials
- Baeldung JPA tutorials
- Hibernate official guides
- Thorben Janssen's blog

---

**Last Updated:** November 26, 2025
**For:** JakartaCourse/HelloJPA Project
**Prerequisite:** Complete project setup in main README.md
