
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

## 📊 Data Seeding (Load Data Management)

### Overview

CapyDb offers a complete system to manage seed data separately from schema migrations. This enables:

- ✅ Independent versioning of data and schema
- ✅ Automatic changeset generation from CSV files
- ✅ Intelligent type inference
- ✅ Specific data rollback without affecting schema
- ✅ Label-based filtering (`data-seed`)

### 1. Generate Changelog from CSV

```bash
# Basic syntax
cap carga from-csv --input <file.csv> --table <table> [options]

# Required options:
#   --input, -i <path>    : Path to CSV file
#   --table, -t <name>    : Target table name

# Additional options:
#   --output, -o <path>   : YAML output file (default: db/changelog/carga-updates/YYYYMMDD__carga-<table>.yaml)
#   --author <name>       : Author name (default: CapyDb)
#   --context <context>   : Changeset context (default: common)
#   --add-to-master       : Automatically add to db.changelog-master.yaml
```

**Examples:**

```bash
# Basic generation
cap carga from-csv --input db/carga/Users.csv --table Users

# With all options
cap carga from-csv \
  --input db/carga/Countries.csv \
  --table TabelaAuxiliarCountries \
  --author "Evellyn Fernandes" \
  --output db/changelog/carga-updates/countries.yaml \
  --add-to-master

# Short form
cap carga from-csv -i db/carga/Cities.csv -t Cities

# Batch processing multiple files
for file in db/carga/*.csv; do
  table=$(basename "$file" .csv)
  cap carga from-csv -i "$file" -t "$table" --add-to-master
done
```

**Automatic Type Inference:**

The command analyzes CSV column names and infers types automatically:

| Name Pattern | Inferred Type | Examples |
|--------------|---------------|----------|
| `*Id`, `PublicId` | UUID | `Id`, `UserId`, `PublicId` |
| `Excluido`, `Ativo`, `Is*`, `Has*` | BOOLEAN | `Excluido`, `Ativo`, `IsActive`, `HasPermission` |
| `Data*`, `Date*`, `*Timestamp`, `*At` | TIMESTAMP | `DataCriacao`, `DateCreated`, `CreatedAt`, `UpdatedAt` |
| `*Num`, `*Numero`, `Count`, `Quantidade` | NUMERIC | `CodigoNum`, `Age`, `Count`, `Quantidade` |
| Others | STRING | `Name`, `Email`, `Description` |

**Automatic Dependency Resolution:**

CapyDb automatically detects table dependencies and orders changesets correctly:

- ✅ **Foreign Key Detection**: Columns ending with `Id` (except `Id` and `PublicId`) are considered FKs
- ✅ **Name Resolution**: Removes prefixes like `TabelaAuxiliar` to find referenced tables
- ✅ **Topological Sorting**: Tables without dependencies are loaded first
- ✅ **Circular Dependency Detection**: Detects and reports circular references
- ✅ **Intelligent Matching**: Searches tables by exact name, with prefix, plural, or partial match

**Dependency Example:**
```
ModuloCategoria.csv has column "ModuloId"
  ↓ CapyDb detects FK to "Modulo"
  ↓ Automatically orders:
    1. Modulo.csv           (no dependencies)
    2. ModuloCategoria.csv  (depends on Modulo)
```

**CSV Example:**
```csv
Id,Name,Email,IsActive,CreatedAt,Age
cc53be96-29d4-46ec-882d-042ad26f3aa5,John Doe,john@email.com,true,2024-01-01,30
6771123b-a632-423d-9c8a-f1ec7fd4b438,Mary Smith,mary@email.com,true,2024-01-02,25
```

**Generated YAML:**
```yaml
databaseChangeLog:
  - changeSet:
      id: 20251017-carga-users
      author: CapyDb
      context: common
      labels: data-seed
      changes:
        - loadData:
            tableName: Users
            file: db/carga/Users.csv
            relativeToChangelogFile: false
            separator: ","
            quotchar: '"'
            encoding: UTF-8
            columns:
              - column:
                  name: Id
                  type: UUID
              - column:
                  name: Name
                  type: STRING
              - column:
                  name: Email
                  type: STRING
              - column:
                  name: IsActive
                  type: BOOLEAN
              - column:
                  name: CreatedAt
                  type: TIMESTAMP
              - column:
                  name: Age
                  type: NUMERIC
      rollback:
        - delete:
            tableName: Users
```

### 2. Apply Seed Data

```bash
# Basic syntax
cap carga update --defaults <file.properties> [options]

# Options:
#   --docker          : Use Docker
#   --workdir <dir>   : Working directory
#   --output <file>   : Execution log
#   --logs            : Display detailed logs
```

**Examples:**

```bash
# Apply all changesets with 'data-seed' label
cap carga update --defaults ./db/changelog/liquibase.properties

# With detailed logs
cap carga update --defaults ./db/changelog/liquibase.properties --logs

# Using Docker
cap carga update --defaults ./db/changelog/liquibase.properties --docker
```

### 3. Remove and Reapply Data (Development)

```bash
# Basic syntax
cap carga drop-all --defaults <file.properties> [options]
```

**⚠️ WARNING:**
- Removes ALL data from tables referenced in data-seed changesets
- Useful for development and testing
- **DO NOT use in production without backup!**

### 4. Complete Reset (Schema + Data)

```bash
# Basic syntax
cap carga reset --defaults <file.properties> [options]
```

**⚠️ DANGER:**
- Executes `DROP ALL` on entire database
- Recreates schema and data from scratch
- **EXTREMELY DESTRUCTIVE**
- Requires manual confirmation

### 5. Data Rollback

```bash
# Basic syntax
cap carga rollback --defaults <file.properties> [options]
```

**What happens:**
1. ✅ Identifies last changeset with `data-seed` label
2. ✅ Executes changeset's `rollback` block
3. ✅ Removes record from `DATABASECHANGELOG` table
4. ✅ Allows re-applying the changeset later

### 6. Data Status

```bash
# View status of data changesets
cap carga status --defaults ./db/changelog/liquibase.properties [--docker]
```

### Automatic Dependency Resolution

CapyDb CLI v1.2.3+ includes an intelligent dependency resolution system that:

**How It Works:**

1. **Scans all changesets** in `db/changelog/carga-updates/`
2. **Reads CSV headers** from each referenced file
3. **Detects Foreign Keys** - columns ending with `Id` (except `Id` and `PublicId`)
4. **Resolves table names** - removes prefixes like `TabelaAuxiliar`
5. **Topologically sorts** - places dependencies first
6. **Updates db.changelog-carga.yaml** with correct order

**Matching Algorithm:**

```
CSV Column: "ModuloId"
  ↓ Remove "Id" → "Modulo"
  ↓ Search for:
    1. Exact match: "Modulo"
    2. With prefix: "TabelaAuxiliarModulo"
    3. Plural: "Modulos" or "TabelaAuxiliarModulos"
    4. Partial: ends with "Modulo"
  ↓ Found: add dependency
```

**Practical Example:**

```bash
# You have these CSVs:
# - Modulo.csv (no FKs)
# - ModuloCategoria.csv (has ModuloId → FK to Modulo)
# - FundamentacoesLegais.csv (has LeisId → FK to TabelaAuxiliarLeis)

# When running:
cap carga from-csv -i db/carga/ModuloCategoria.csv -t ModuloCategoria --add-to-master
cap carga from-csv -i db/carga/Modulo.csv -t Modulo --add-to-master
cap carga from-csv -i db/carga/FundamentacoesLegais.csv -t FundamentacoesLegais --add-to-master

# CapyDb automatically reorders in db.changelog-carga.yaml:
#   1. Modulo (no dependencies)
#   2. TabelaAuxiliarLeis (no dependencies)
#   3. ModuloCategoria (depends on Modulo)
#   4. FundamentacoesLegais (depends on Leis)

# Result: zero FK constraint errors!
```

**Special Case Handling:**

- **Dependencies not found**: Yellow warning, but continues execution
- **Circular dependencies**: Detected and reported with error
- **Multiple FKs**: All are detected and considered
- **Custom prefixes**: System tries multiple variations

### Complete Data Seeding Workflow

```bash
# 1. Prepare CSV
cat > db/carga/Countries.csv << EOF
Id,Name,Code,Population,IsActive
1,Brazil,BR,212000000,true
2,United States,US,331000000,true
3,Argentina,AR,45000000,true
EOF

# 2. Generate changelog
cap carga from-csv \
  --input db/carga/Countries.csv \
  --table Countries \
  --author "Evellyn Fernandes" \
  --add-to-master

# 3. Check generated file
cat db/changelog/carga-updates/20251017__carga-countries.yaml

# 4. Apply data
cap carga update --defaults db/changelog/liquibase.properties

# 5. Check status
cap carga status --defaults db/changelog/liquibase.properties

# 6. If needed, rollback
cap carga rollback --defaults db/changelog/liquibase.properties
```

### Project Structure with Data Seeding

```
project/
├── db/
│   ├── changelog/
│   │   ├── common/              # Schema migrations
│   │   │   ├── 20250101_120000__create-tables.yaml
│   │   │   └── 20250102_143000__add-columns.yaml
│   │   ├── carga-updates/       # Data seed migrations
│   │   │   ├── 20251017__carga-countries.yaml
│   │   │   ├── 20251017__carga-cities.yaml
│   │   │   └── 20251017__carga-users.yaml
│   │   ├── db.changelog-master.yaml
│   │   └── liquibase.properties
│   ├── carga/                   # CSV source files
│   │   ├── Countries.csv
│   │   ├── Cities.csv
│   │   └── Users.csv
│   └── drivers/
└── src/
```

### Use Cases

**1. Reference Data:**
```bash
# Countries, states, cities
cap carga from-csv -i db/carga/Countries.csv -t Countries --add-to-master
cap carga from-csv -i db/carga/States.csv -t States --add-to-master
cap carga from-csv -i db/carga/Cities.csv -t Cities --add-to-master
cap carga update --defaults db/changelog/liquibase.properties
```

**2. System Configuration:**
```bash
# Settings, permissions, roles
cap carga from-csv -i db/carga/SystemConfig.csv -t SystemConfig --add-to-master
cap carga from-csv -i db/carga/Roles.csv -t Roles --add-to-master
cap carga update --defaults db/changelog/liquibase.properties
```

**3. Test Data:**
```bash
# Development environment
cap carga from-csv -i db/carga/TestUsers.csv -t Users --context dev
cap carga update --defaults db/changelog/liquibase-dev.properties
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

---

## 🆕 What's New in v1.2.3

**Generated documentation for CapyDb CLI v1.2.3**
*Last updated: 2025-10-27*

### Data Seeding and Data Management
- ✅ **`cap carga from-csv` Command** - Automatic changeset generation from CSV files
- ✅ **Intelligent Type Inference** - Automatically detects UUID, BOOLEAN, TIMESTAMP, NUMERIC, STRING
- ✅ **Automatic Dependency Resolution** - Detects FKs from CSV headers and automatically orders tables
- ✅ **Topological Sorting** - Smart ordering prevents FK constraint violations during data load
- ✅ **Circular Dependency Detection** - Warns about circular references between tables
- ✅ **Management Commands** - update, drop-all, reset, rollback, status for seed data
- ✅ **Label-based Filtering** - Uses `data-seed` label for specific operations
- ✅ **Auto-add to Master** - Option to automatically include in master changelog

### Package Improvements
- ✅ **Embedded Dependencies** - CapyDb.Core, Runner, and Writers are now part of the CLI
- ✅ **Clean NuGet Package** - No external dependencies listed
- ✅ **Full Multi-target Support** - Complete support for .NET 8.0 and .NET 9.0

### Previous Features (v1.0.7)
- ✅ **Enhanced Recursive Search** - System now searches for `liquibase.properties` in multiple locations automatically
- ✅ **Full Windows Support** - Fixed glob pattern issues on Windows
- ✅ **Monorepo Support** - Works perfectly with complex structures (`apps/*/`, `src/*/`)
- ✅ **Intelligent Assembly Detection** - Better support for EF Core in large projects
- ✅ **Cross-platform** - Tested and validated on Windows, Linux, and macOS

---

## 📑 Command Summary

### Migrations
- `cap migrations add <name>` - Create new migration
- `cap migrations import-ef` - Import from EF Core
- `cap migrations mergeschemas` - Consolidate migrations

### Database Operations
- `cap plan` - Generate SQL plan
- `cap apply` - Apply migrations
- `cap status` - View status
- `cap validate` - Validate changelog
- `cap tag <name>` - Create tag
- `cap remove-tag <tag>` - Remove tag
- `cap rollback count <N>` - Revert N migrations
- `cap rollback to-tag <tag>` - Revert to tag

### Data Seeding (Load Data)
- `cap carga from-csv` - Generate changelog from CSV
- `cap carga update` - Apply seed data (label: data-seed)
- `cap carga drop-all` - Remove and reapply data
- `cap carga reset` - Complete reset (schema + data)
- `cap carga rollback` - Rollback last data changeset
- `cap carga status` - View seed data status

### Utilities
- `cap doctor` - Check prerequisites
- `cap drift detect` - Detect divergences
- `cap squash --tag <tag>` - Consolidate history
- `cap convert-inserts` - Convert SQL INSERTs
- `cap bye` - Farewell
