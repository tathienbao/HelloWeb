# Jakarta EE Course - Complete Setup Guide

A comprehensive guide for **HelloWeb** (Java Servlet) and **HelloJPA** (JPA/Hibernate) projects with SQL Server database integration.

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Quick Start](#quick-start)
5. [Detailed Setup](#detailed-setup)
   - [Step 1: Database Setup](#step-1-database-setup)
   - [Step 2: Database Configuration](#step-2-database-configuration)
   - [Step 3: HelloWeb Setup](#step-3-helloweb-setup)
   - [Step 4: HelloJPA Setup](#step-4-hellojpa-setup)
6. [Testing Your Setup](#testing-your-setup)
7. [Project Structure](#project-structure)
8. [Development Workflow](#development-workflow)
9. [Understanding the Components](#understanding-the-components)
10. [Troubleshooting](#troubleshooting)

---

## Overview

This repository contains two interconnected Jakarta EE learning projects:

- **HelloWeb**: A Java Servlet web application demonstrating basic web development
- **HelloJPA**: A standalone JPA project for learning database persistence with Hibernate

Both projects connect to the same **SQL Server database** (`UdemyShopDB`) running in Docker.

---

## Architecture

```
┌─────────────────┐         ┌──────────────────┐
│    HelloWeb     │         │     HelloJPA     │
│  (Servlet App)  │         │  (JPA Standalone)│
│   Port: 8080    │         │                  │
└────────┬────────┘         └────────┬─────────┘
         │                           │
         │    JDBC Connection        │
         │                           │
         └───────────┬───────────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │   SQL Server 2019     │
         │   (Docker Container)  │
         │   Port: 1433          │
         │                       │
         │  • Database: UdemyShopDB
         │  • Admin: sa / MyPassword123!
         │  • App User: proit / 041192
         └───────────────────────┘
```

---

## Prerequisites

**Required Software:**
- **Java 17** (for HelloJPA) or Java 11+ (for HelloWeb)
- **Maven 3.x**
- **Docker Desktop** (for SQL Server)
- **IntelliJ IDEA Community** (or Eclipse)
- **SQL Server Management Studio (SSMS)** - Windows only (optional but recommended)

**Environment:**
- This guide assumes **WSL2 on Windows** (adapt commands for Linux/Mac)
- SQL Server runs in Docker and is accessible from both WSL and Windows

---

## Quick Start

```bash
# 1. Start SQL Server
docker start sqlserver  # If already created
# OR create new container (see detailed setup)

# 2. Run HelloWeb
cd HelloWeb
mvn clean tomcat7:run
# Access: http://localhost:8080/HelloWeb/hello

# 3. Test HelloJPA connection
cd ../HelloJPA
mvn clean compile
# Ready to write JPA code!
```

---

## Detailed Setup

### Step 1: Database Setup

**1.1. Start SQL Server Container**

```bash
docker run -e "ACCEPT_EULA=Y" \
           -e "MSSQL_SA_PASSWORD=MyPassword123!" \
           -e "MSSQL_PID=Developer" \
           -p 1433:1433 \
           --name sqlserver \
           -v sqlserver_data:/var/opt/mssql \
           -d \
           mcr.microsoft.com/mssql/server:2019-latest
```

**What this does:**
- Creates a SQL Server 2019 container named `sqlserver`
- Sets admin password: `MyPassword123!`
- Exposes port `1433` (SQL Server default)
- Persists data in Docker volume `sqlserver_data`
- Runs in detached mode (`-d`)

**1.2. Verify SQL Server is Running**

```bash
docker ps
# Should show: sqlserver container with status "Up"

docker logs sqlserver
# Should show: "SQL Server is now ready for client connections"
```

**1.3. Docker Management Commands**

```bash
# Check container status
docker ps -a

# Start existing container
docker start sqlserver

# Stop container
docker stop sqlserver

# Restart container
docker restart sqlserver

# View logs
docker logs sqlserver -f  # -f for follow mode

# Remove container (CAUTION: doesn't delete volume data)
docker rm -f sqlserver

# List volumes
docker volume ls

# Remove volume (CAUTION: deletes all data!)
docker volume rm sqlserver_data
```

---

### Step 2: Database Configuration

**2.1. Connect with SQL Server Management Studio (SSMS)**

**Download:** https://aka.ms/ssmsfullsetup

**Connection Settings:**
- **Server:** `127.0.0.1` or `localhost`
- **Authentication:** SQL Server Authentication
- **Login:** `sa`
- **Password:** `MyPassword123!`
- **IMPORTANT:** Go to Options → Connection Properties → Check **"Trust server certificate"**

**2.2. Create Database**

Execute in SSMS:

```sql
-- Create database
CREATE DATABASE UdemyShopDB;
GO

-- Verify
SELECT name FROM sys.databases WHERE name = 'UdemyShopDB';
```

**2.3. Create Application User (proit)**

```sql
USE UdemyShopDB;
GO

-- Create login at server level
CREATE LOGIN proit WITH PASSWORD = '041192';
GO

-- Create user in database
CREATE USER proit FOR LOGIN proit;
GO

-- Grant db_owner role (full permissions on this database)
ALTER ROLE db_owner ADD MEMBER proit;
GO

-- Verify
SELECT name, type_desc FROM sys.database_principals WHERE name = 'proit';
```

**Why separate users?**
- `sa`: Server administrator (for maintenance, creating databases)
- `proit`: Application user (for HelloWeb and HelloJPA to connect)
- **Best Practice:** Applications should NOT use `sa` account

---

### Step 3: HelloWeb Setup

**3.1. Project Overview**

HelloWeb is a Java Servlet application using:
- **Servlet API 4.0** (javax.servlet)
- **Maven Tomcat7 Plugin** (embedded server)
- **Java 11**

**3.2. Verify pom.xml**

Location: `HelloWeb/pom.xml`

```xml
<dependencies>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

**3.3. Run HelloWeb**

```bash
cd HelloWeb
mvn clean tomcat7:run
```

**3.4. Access from Browser**

**From WSL:**
```
http://localhost:8080/HelloWeb/hello
```

**From Windows Host:**
```
http://<WSL-IP>:8080/HelloWeb/hello

# Find WSL IP:
hostname -I | awk '{print $1}'
# Example: http://172.26.88.209:8080/HelloWeb/hello
```

**Expected Output:**
```
Hello from Servlet!
```

**3.5. Stop the Server**

Press `Ctrl+C` in the terminal where Tomcat is running.

---

### Step 4: HelloJPA Setup

**4.1. Project Overview**

HelloJPA is a standalone JPA project using:
- **Hibernate 6.4.1.Final** (JPA implementation)
- **Jakarta Persistence API 3.1.0** (JPA specification)
- **SQL Server JDBC Driver 12.4.2**
- **Java 17**

**4.2. Directory Structure**

```
HelloJPA/
├── pom.xml                           # Maven dependencies
└── src/
    ├── main/
    │   ├── java/                     # YOUR CODE GOES HERE (empty initially)
    │   └── resources/
    │       └── META-INF/
    │           └── persistence.xml   # JPA configuration
    └── test/
        └── java/
```

**4.3. Verify pom.xml**

Location: `HelloJPA/pom.xml`

Key dependencies:
```xml
<dependencies>
    <!-- Hibernate ORM 6.4.1 -->
    <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>6.4.1.Final</version>
    </dependency>

    <!-- SQL Server JDBC Driver -->
    <dependency>
        <groupId>com.microsoft.sqlserver</groupId>
        <artifactId>mssql-jdbc</artifactId>
        <version>12.4.2.jre11</version>
    </dependency>

    <!-- Jakarta Persistence API -->
    <dependency>
        <groupId>jakarta.persistence</groupId>
        <artifactId>jakarta.persistence-api</artifactId>
        <version>3.1.0</version>
    </dependency>
</dependencies>
```

**4.4. Understanding persistence.xml**

Location: `HelloJPA/src/main/resources/META-INF/persistence.xml`

```xml
<persistence-unit name="HelloJPU" transaction-type="RESOURCE_LOCAL">
    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

    <properties>
        <!-- Database Connection -->
        <property name="jakarta.persistence.jdbc.driver"
                  value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
        <property name="jakarta.persistence.jdbc.url"
                  value="jdbc:sqlserver://localhost:1433;databaseName=UdemyShopDB;encrypt=true;trustServerCertificate=true"/>
        <property name="jakarta.persistence.jdbc.user"
                  value="proit"/>
        <property name="jakarta.persistence.jdbc.password"
                  value="041192"/>

        <!-- Hibernate Settings -->
        <property name="hibernate.dialect"
                  value="org.hibernate.dialect.SQLServerDialect"/>
        <property name="hibernate.hbm2ddl.auto"
                  value="validate"/>  <!-- Won't modify database schema -->
        <property name="hibernate.show_sql"
                  value="true"/>      <!-- Print SQL to console -->
        <property name="hibernate.format_sql"
                  value="true"/>      <!-- Format SQL for readability -->
    </properties>
</persistence-unit>
```

**Key Configuration Explained:**

| Property | Value | Purpose |
|----------|-------|---------|
| `persistence-unit name` | HelloJPU | Reference name in Java code |
| `jdbc.url` | jdbc:sqlserver://localhost:1433;databaseName=UdemyShopDB | Connection string |
| `jdbc.user` | proit | Database user (NOT sa) |
| `jdbc.password` | 041192 | User password |
| `hibernate.hbm2ddl.auto` | validate | Only validate schema, don't modify database |
| `hibernate.show_sql` | true | Log SQL statements (for learning) |

**4.5. Compile HelloJPA**

```bash
cd HelloJPA
mvn clean compile
```

**Expected Output:**
```
[INFO] BUILD SUCCESS
```

---

## Testing Your Setup

### Test 1: SQL Server Connectivity

**From WSL terminal:**

```bash
# Install sqlcmd (if not installed)
# For Ubuntu/Debian:
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
curl https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list
sudo apt-get update
sudo ACCEPT_EULA=Y apt-get install -y mssql-tools unixodbc-dev

# Test connection
/opt/mssql-tools/bin/sqlcmd -S localhost -U proit -P 041192 -d UdemyShopDB -Q "SELECT DB_NAME() AS CurrentDatabase;"
```

**Expected Output:**
```
CurrentDatabase
---------------
UdemyShopDB

(1 rows affected)
```

### Test 2: HelloWeb Application

```bash
cd HelloWeb
mvn clean tomcat7:run
```

Open browser: `http://localhost:8080/HelloWeb/hello`

**Expected:** "Hello from Servlet!" message

### Test 3: HelloJPA Connection

Create a test file:

```bash
cd HelloJPA
mkdir -p src/main/java/test
```

Create `src/main/java/test/TestConnection.java`:

```java
package test;

import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;

public class TestConnection {
    public static void main(String[] args) {
        EntityManagerFactory emf = null;
        try {
            System.out.println("Connecting to UdemyShopDB...");
            emf = Persistence.createEntityManagerFactory("HelloJPU");
            System.out.println("✅ CONNECTION SUCCESSFUL!");
            System.out.println("   - Database: UdemyShopDB");
            System.out.println("   - User: proit");
        } catch (Exception e) {
            System.err.println("❌ CONNECTION FAILED!");
            e.printStackTrace();
        } finally {
            if (emf != null) emf.close();
        }
    }
}
```

Run:

```bash
mvn exec:java -Dexec.mainClass="test.TestConnection"
```

**Expected Output:**
```
✅ CONNECTION SUCCESSFUL!
   - Database: UdemyShopDB
   - User: proit
```

**Clean up test file:**
```bash
rm -rf src/main/java/test
```

---

## Project Structure

### HelloWeb Structure

```
HelloWeb/
├── pom.xml                                    # Maven config
├── README.md                                  # This file
├── docker-compose.yml                         # SQL Server config
├── example-jpa-pom.xml                        # Reference for HelloJPA setup
└── src/
    └── main/
        ├── java/
        │   └── com/example/servlet/
        │       └── Hello.java                 # Example servlet
        └── webapp/
            └── WEB-INF/
                └── web.xml                    # Servlet configuration
```

### HelloJPA Structure

```
HelloJPA/
├── pom.xml                                    # Maven config (Java 17, Hibernate 6)
└── src/
    ├── main/
    │   ├── java/                              # EMPTY - write your entities here
    │   │   └── (your packages here)
    │   │       ├── entity/                    # JPA entities (@Entity classes)
    │   │       └── Main.java                  # Entry point for CRUD operations
    │   └── resources/
    │       └── META-INF/
    │           └── persistence.xml            # JPA config (connected to UdemyShopDB)
    └── test/
        └── java/                              # Unit tests
```

---

## Development Workflow

### Working with HelloJPA (Learning JPA)

**Step 1: Create Entity Package**

```bash
cd HelloJPA/src/main/java
mkdir -p org/example/entity
```

**Step 2: Create Entity Class**

Create `org/example/entity/Product.java`:

```java
package org.example.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name", nullable = false)
    private String name;

    @Column(name = "price")
    private Double price;

    // Constructors
    public Product() {}

    public Product(String name, Double price) {
        this.name = name;
        this.price = price;
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public Double getPrice() { return price; }
    public void setPrice(Double price) { this.price = price; }
}
```

**Step 3: Create Main Class for CRUD**

Create `org/example/Main.java`:

```java
package org.example;

import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;
import org.example.entity.Product;

public class Main {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("HelloJPU");
        EntityManager em = emf.createEntityManager();

        try {
            // CREATE
            em.getTransaction().begin();
            Product product = new Product("Laptop", 999.99);
            em.persist(product);
            em.getTransaction().commit();
            System.out.println("Product created: " + product.getId());

            // READ
            Product found = em.find(Product.class, product.getId());
            System.out.println("Product found: " + found.getName());

            // UPDATE
            em.getTransaction().begin();
            found.setPrice(899.99);
            em.getTransaction().commit();
            System.out.println("Product updated");

            // DELETE
            em.getTransaction().begin();
            em.remove(found);
            em.getTransaction().commit();
            System.out.println("Product deleted");

        } finally {
            em.close();
            emf.close();
        }
    }
}
```

**Step 4: Create Database Table**

In SSMS:

```sql
USE UdemyShopDB;
GO

CREATE TABLE products (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(255) NOT NULL,
    price FLOAT
);
GO
```

**Step 5: Run Your Application**

```bash
cd HelloJPA
mvn clean compile exec:java -Dexec.mainClass="org.example.Main"
```

---

## Understanding the Components

### Why Two Projects?

| Project | Purpose | Learning Focus |
|---------|---------|----------------|
| HelloWeb | Web layer | Servlets, HTTP, web requests/responses |
| HelloJPA | Data layer | Database operations, ORM, JPA concepts |

**In real applications:** These layers would be combined, but separating them helps understand each technology independently.

### Why Docker for SQL Server?

**Advantages:**
- ✅ Consistent environment across machines
- ✅ Easy to start/stop/reset
- ✅ No installation on host system
- ✅ Data persists in Docker volumes
- ✅ Isolated from other SQL Server instances

**Alternative:** Install SQL Server directly on Windows

### Why Java 17 for HelloJPA?

- Hibernate 6.x requires Java 11+
- Java 17 is an LTS (Long Term Support) version
- Modern features and better performance
- HelloWeb uses Java 11 for compatibility with older servlet containers

### Hibernate DDL Auto Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `validate` | Only validates schema | **Production** (current setup) |
| `update` | Updates schema automatically | Development (be careful!) |
| `create` | Creates schema, drops existing | Testing |
| `create-drop` | Creates schema, drops on exit | Unit tests |
| `none` | No schema operations | Manual schema management |

**Current Setup:** `validate` - Safe for existing databases, won't modify schema

---

## Troubleshooting

### Problem: "Cannot connect to SQL Server"

**Solution 1: Check Docker container**
```bash
docker ps
# If not running:
docker start sqlserver
docker logs sqlserver
```

**Solution 2: Check port availability**
```bash
lsof -i :1433
# If another service is using port 1433, stop it or change Docker port
```

**Solution 3: Verify credentials**
```bash
/opt/mssql-tools/bin/sqlcmd -S localhost -U proit -P 041192 -Q "SELECT @@VERSION"
```

### Problem: "Login failed for user 'proit'"

**Cause:** User not created or wrong password

**Solution:** Recreate user in SSMS:
```sql
USE UdemyShopDB;
DROP USER IF EXISTS proit;
DROP LOGIN IF EXISTS proit;

CREATE LOGIN proit WITH PASSWORD = '041192';
CREATE USER proit FOR LOGIN proit;
ALTER ROLE db_owner ADD MEMBER proit;
```

### Problem: "Database 'UdemyShopDB' does not exist"

**Solution:** Create database:
```sql
CREATE DATABASE UdemyShopDB;
```

### Problem: Maven build fails - "Could not resolve dependencies"

**Solution:** Clear Maven cache and rebuild:
```bash
rm -rf ~/.m2/repository
mvn clean install -U
```

### Problem: WSL IP address changed

**Solution:** Find new IP:
```bash
hostname -I | awk '{print $1}'
# Use this IP in Windows browser
```

### Problem: "Port 8080 already in use"

**Solution:** Kill process or change port:
```bash
# Find process
lsof -i :8080

# Kill process
kill -9 <PID>

# Or change port in pom.xml:
<configuration>
    <port>8081</port>  <!-- Change to 8081 -->
</configuration>
```

### Problem: Hibernate validation error - "Table not found"

**Cause:** Table doesn't exist in database

**Solution:** Create table in SSMS or change `hibernate.hbm2ddl.auto` to `update`:
```xml
<property name="hibernate.hbm2ddl.auto" value="update"/>
```

### Problem: "ClassNotFoundException: com.microsoft.sqlserver.jdbc.SQLServerDriver"

**Cause:** Maven didn't download JDBC driver

**Solution:**
```bash
cd HelloJPA
mvn dependency:purge-local-repository -DreResolve=false
mvn clean install
```

---

## Additional Resources

**Documentation:**
- Jakarta EE: https://jakarta.ee/
- Hibernate: https://hibernate.org/orm/documentation/
- SQL Server Docker: https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker

**Learning Path:**
1. ✅ Setup complete (you are here)
2. Learn JPA basics: Entities, EntityManager, JPQL
3. Learn Servlet basics: Request/Response, Sessions
4. Integrate JPA with Servlets
5. Build a complete CRUD web application

---

## Quick Reference

### Most Common Commands

```bash
# Start SQL Server
docker start sqlserver

# Run HelloWeb
cd HelloWeb && mvn clean tomcat7:run

# Compile HelloJPA
cd HelloJPA && mvn clean compile

# Test database connection
/opt/mssql-tools/bin/sqlcmd -S localhost -U proit -P 041192 -d UdemyShopDB -Q "SELECT 1"

# Check WSL IP
hostname -I | awk '{print $1}'
```

### Connection Details Quick Reference

| Component | Value |
|-----------|-------|
| **SQL Server Host** | localhost |
| **SQL Server Port** | 1433 |
| **Database Name** | UdemyShopDB |
| **Admin User** | sa |
| **Admin Password** | MyPassword123! |
| **App User** | proit |
| **App Password** | 041192 |
| **Persistence Unit** | HelloJPU |
| **HelloWeb URL** | http://localhost:8080/HelloWeb/hello |

---

**Last Updated:** November 26, 2025
**Author:** Learning Jakarta EE
**Purpose:** Educational project for understanding Servlets, JPA, and Hibernate
