
# CapyDb CLI — Complete Guide

> Command-line tool to manage database migrations with Liquibase and Entity Framework Core

## 📋 Table of Contents
- [Installation](#installation)
- [Core Commands](#core-commands)
- [Migrations](#migrations)
- [Database Operations](#database-operations)
- [Utilities](#utilities)
- [Author System](#author-system)
- [Advanced Configuration](#advanced-configuration)
- [Troubleshooting](#troubleshooting)

---

## 🚀 Installation

### As a Global .NET Tool
```bash
# Build and pack
dotnet pack src/CapyDb.Cli/CapyDb.Cli.csproj -o nupkg

# Install globally
dotnet tool install -g capydb.cli --add-source ./nupkg

# Check installation
cap --version
```

### Run Directly (without installing)
```bash
dotnet run --project src/CapyDb.Cli -- [commands]
```

---

## 📖 Core Commands

### General Info
```bash
# CapyDb version
cap --version

# Full help
cap --help

# Check prerequisites
cap doctor

# Farewell (ASCII art)
cap bye
```

---

## 📝 Migrations

### 1) Create a New Migration
```bash
# Basic syntax
cap migrations add <name> [options]

# Options:
#   --no-stubs       : Do not create vendor-specific stub folders
#   --author <name>  : Set author explicitly
```

**Examples:**
```bash
cap migrations add create-users
cap migrations add create-users --author "John Doe"
cap migrations add create-users --no-stubs
cap migrations add create-users --author "Mary Smith" --no-stubs
```

**What happens:**
1. ✅ Creates a YAML file under `db/changelog/common/`
2. ✅ Updates `db.changelog-master.yaml`
3. ✅ Detects author automatically (or uses `--author`)
4. ✅ Generates a unique timestamp to avoid conflicts

**Generated structure:**
```yaml
# db/changelog/common/20250923_014331__create-users.yaml
databaseChangeLog:
  - changeSet:
      id: 20250923_014331-create-users
      author: John Doe  # auto-detected
      context: common
      changes:
        # Your changes here
```

### 2) Import a Migration from Entity Framework
```bash
cap migrations import-ef   --assembly ./MyApp/bin/Debug/net8.0/MyApp.dll   --name CreateUsersTable   --provider sqlserver|postgres|mysql   [--author "Your Name"]
```

**What happens:**
1. ✅ Loads the specified .NET assembly
2. ✅ Finds the Migration class by name
3. ✅ Executes `Up()` in-memory
4. ✅ Converts EF operations to Liquibase YAML
5. ✅ Saves under `db/changelog/common/`

**Supported EF operations:**
- ✅ `CreateTable` → `createTable`
- ✅ `AddColumn` → `addColumn`
- ✅ `InsertData` → `insert`
- ✅ `DeleteData` → `delete`
- ⚠️ Others → generic SQL with comment

### 3) Schema Merge
```bash
cap migrations mergeschemas --scope common|mssql|postgres|mysql [--include-merged] [--delete-old]
```

---

## 🗃️ Database Operations

### 1) Generate Execution Plan
```bash
cap plan --defaults ./db/changelog/liquibase.properties [--docker] [--workdir DIR] [--output plan.sql]
```

**Use cases:** code review, audit, documentation.

### 2) Apply Migrations
```bash
cap apply --defaults ./db/changelog/liquibase.properties [--docker] [--workdir DIR] [--output apply.log]
```

### 3) Check Status
```bash
cap status --defaults ./db/changelog/liquibase.properties [--docker]
```

### 4) Validate Changelog
```bash
cap validate --defaults ./db/changelog/liquibase.properties [--output validation.log]
```

### 5) Tags
```bash
# create
cap tag v1.0.0 --defaults ./db/changelog/liquibase.properties
# remove
cap remove-tag v1.0.0 --defaults ./db/changelog/liquibase.properties
```

### 6) Rollback
```bash
cap rollback count 3 --defaults ./db/changelog/liquibase.properties
cap rollback to-tag v1.0.0 --defaults ./db/changelog/liquibase.properties
```

---

## 🛠️ Utilities

### Doctor — Prerequisite Check
```bash
cap doctor [--defaults ./db/changelog/liquibase.properties]
```

### Drift Detection
```bash
cap drift detect --defaults ./db/changelog/liquibase.properties [--docker] [--workdir DIR] [--output drift-report.xml]
```

### Squash — History Consolidation
```bash
cap squash --tag v1.0.0 --defaults ./db/changelog/liquibase.properties [--docker]
```

### INSERTs Converter
```bash
cap convert-inserts --input ./data.sql --output ./db/changelog/common/seed.yaml [--table users] [--author "System"]
```

---

## 👤 Author System

Priority order:
1) `--author` param  
2) `CAPY_AUTHOR` env var  
3) `git config user.name`  
4) OS env vars (`USER`, `USERNAME`, etc.)  
5) Fallback: `capydb`

---

## ⚙️ Advanced Configuration

### Example `liquibase.properties`
```properties
url=jdbc:sqlserver://localhost:1433;databaseName=MyApp;trustServerCertificate=true
username=sa
password=MyPassword123
changeLogFile=db/changelog/db.changelog-master.yaml
classpath=db/drivers/mssql-jdbc-12.4.2.jre11.jar
logLevel=INFO
contexts=common,production
labels=!test
```

### Suggested Project Layout
```
db/
  changelog/
    common/
    mssql/
    postgres/
    mysql/
    archive/
    deleteSchemas/
    db.changelog-master.yaml
    liquibase.properties
  drivers/
```

---

## 🐛 Troubleshooting

**Liquibase via Docker**
```bash
cap doctor --docker
```

**JDBC drivers missing**
```
mkdir -p db/drivers
# Put the vendor JARs here
```

**Connection issues**
- Check URL/credentials/network/firewall
- Run `cap doctor --defaults ./db/changelog/liquibase.properties`

---

## 📚 Example Workflows

### Dev
```bash
cap migrations add add-products --author "John Doe"
cap plan --defaults ./db/changelog/liquibase.properties --output plan.sql
cap apply --defaults ./db/changelog/liquibase.properties
cap status --defaults ./db/changelog/liquibase.properties
cap tag v1.2.0 --defaults ./db/changelog/liquibase.properties
```

### CI/CD
```bash
export CAPY_AUTHOR="$CI_COMMIT_AUTHOR"
cap doctor --defaults ./db/changelog/liquibase.properties
cap plan --defaults ./db/changelog/liquibase.properties --output artifacts/plan.sql
cap apply --defaults ./db/changelog/liquibase.properties
cap tag "deploy-$(date +%Y%m%d-%H%M%S)" --defaults ./db/changelog/liquibase.properties
```

### EF Core Migration
```bash
dotnet build MyApp.sln
cap migrations import-ef   --assembly ./MyApp/bin/Debug/net8.0/MyApp.dll   --name AddProductsTable   --provider sqlserver   --author "Mary Smith"
cat db/changelog/common/20250923_*__addproductstable.yaml
cap apply --defaults ./db/changelog/liquibase.properties
```

---

## 🎯 Best Practices
- Use **kebab-case** for migration names
- Never commit credentials; use env vars
- Periodically **squash** history
- Use **contexts/labels** per environment
