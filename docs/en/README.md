# CapyDb CLI

A command-line tool for managing database migrations with **Liquibase** and **Entity Framework Core**.

## 🚀 What is CapyDb?

CapyDb CLI addresses the challenge of managing database migrations consistently and efficiently, offering:

- ✅ **Automatic configuration detection** - automatically searches for `liquibase.properties`
- ✅ **Migration creation** in Liquibase YAML format
- ✅ **Migration import** from Entity Framework Core
- ✅ **Schema merge and consolidation** automation
- ✅ **Safe execution** with SQL execution plans
- ✅ **Multi-DBMS support** (SQL Server, PostgreSQL, MySQL, Oracle)
- ✅ **Docker integration** and CI/CD pipelines
- ✅ **Drift detection** - identifies undocumented changes
- ✅ **Tag system** - create and remove tags for versioning
- ✅ **Smart rollback** - revert by count or to a specific tag
- ✅ **History squash** - consolidates old migrations
- ✅ **Automatic author detection** via Git/CI/CD
- ✅ **Comprehensive diagnostics** with `cap doctor`
- ✅ **Changelog validation** before execution
- ✅ **INSERTs converter** - converts SQL INSERTs to Liquibase format
- ✅ **Data seeding from CSV** - automatically generates load data changesets
- ✅ **Intelligent type inference** - detects column types from CSV headers
- ✅ **Data management** - update, rollback, and reset seed data independently

## 📦 Installation

### Prerequisites
- .NET 8.0 SDK or higher
- Java 8+ (for Liquibase)

### Global Installation
```bash
# Install via NuGet
dotnet tool install -g capydb.cli

# Verify installation
cap --version  # 1.2.3
```

## 🏁 Getting Started

### 1. Set Up Project
```bash
# Recommended structure
my-project/
├── db/
│   └── changelog/
│       ├── common/
│       ├── db.changelog-master.yaml
│       └── liquibase.properties  # ← CLI auto-detects this!
├── src/
└── Infrastructure/  # Or any project structure
```

### 2. Check Prerequisites
```bash
cap doctor
```

### 3. Create First Migration
```bash
# Create a basic migration
cap migrations add create-users

# With specific author
cap migrations add create-products --author "Your Name"
```

### 4. Import from Entity Framework
```bash
cap migrations import-ef \
  --assembly ./MyApp.dll \
  --name CreateUsersTable \
  --provider sqlserver
```

### 5. Run Migrations (Auto-Detection!)
```bash
# CLI automatically searches in ./db/changelog/liquibase.properties
cap plan      # Generate execution plan
cap apply     # Apply migrations
cap status    # Check database status

# Create tag after deployment
cap tag v1.0.0

# Rollback if needed
cap rollback count 2
cap rollback to-tag v1.0.0
```

### 💡 Automatic Configuration Detection (Enhanced!)

The CLI now features **robust recursive search** for `liquibase.properties`:

**Search Priority:**
1. `./db/changelog/liquibase.properties` (recommended)
2. `./liquibase.properties` (root directory)
3. `./config/liquibase.properties`
4. `./database/liquibase.properties`
5. `./src/*/db/changelog/liquibase.properties` (monorepos!)
6. `./apps/*/db/changelog/liquibase.properties` (monorepos!)
7. **Recursive search in all subdirectories** (excluding node_modules, .git)

**Works perfectly on Windows, Linux, and macOS!**

```bash
# Before (still works):
cap apply --defaults ./db/changelog/liquibase.properties

# Now (even simpler):
cap apply  # Auto-detects in monorepos, nested structures, anywhere!
```

## 📋 Quick Examples

### Recommended Project Structure
```
my-project/
├── db/
│   ├── changelog/
│   │   ├── common/
│   │   │   └── 20250924_120000__create-users.yaml
│   │   ├── db.changelog-master.yaml
│   │   └── liquibase.properties  # ← Auto-detected!
│   └── drivers/
└── src/
```

### Auto-Generated Migration
```yaml
# db/changelog/common/20250924_120000__create-users.yaml
databaseChangeLog:
  - changeSet:
      id: 20250924_120000-create-users
      author: Moisés Drumand  # ← Detected via Git!
      context: common
      changes:
        - createTable:
            tableName: users
            columns:
              - column:
                  name: id
                  type: int
                  constraints:
                    primaryKey: true
```

### Simplified Full Workflow
```bash
# 1. Create migration
cap migrations add create-users

# 2. Review what will be executed
cap plan

# 3. Apply to database
cap apply

# 4. Check status
cap status

# 5. Create version tag
cap tag v1.0.0

# 6. Revert if needed
cap rollback to-tag v1.0.0
```

### Multiple Environments and DBMS
```bash
# Default environment (auto-detected)
cap apply

# PostgreSQL with custom file
cap apply --defaults ./db/changelog/liquibase-postgres.properties

# MySQL with Docker
cap apply --defaults ./db/changelog/liquibase-mysql.properties --docker

# Oracle
cap apply --defaults ./db/changelog/liquibase-oracle.properties
```

### Converting SQL INSERTs
```bash
# Convert SQL INSERTs file to Liquibase format
cap convert-inserts --input ./data.sql --output ./changelog.yaml

# Specify table name
cap convert-inserts --input ./data.sql --table users --output ./changelog.yaml
```

### Data Seeding from CSV
```bash
# Generate data-seed changelog from CSV
cap carga from-csv --input db/carga/Users.csv --table Users

# With author and auto-add to master changelog
cap carga from-csv \
  --input db/carga/Countries.csv \
  --table TabelaAuxiliarCountries \
  --author "Your Name" \
  --add-to-master

# Custom output location
cap carga from-csv \
  --input db/carga/Cities.csv \
  --table Cities \
  --output db/changelog/custom/cities.yaml

# Automatic dependency resolution and ordering!
# When using --add-to-master, CapyDb automatically:
# ✅ Detects foreign keys from CSV column names (ending with "Id")
# ✅ Resolves table dependencies
# ✅ Orders changesets topologically (dependencies first)
# ✅ Prevents FK constraint violations during load

# Apply data seeds
cap carga update --defaults db/changelog/liquibase.properties

# Rollback data seeds
cap carga rollback --defaults db/changelog/liquibase.properties

# Check data seed status
cap carga status --defaults db/changelog/liquibase.properties
```

## 🧪 Tests

The project includes automated integration tests using Jest and Prisma.

```bash
# Run integration tests
cd tests/integration
npm install
npm test

# Tests with different DBMS
npm test -- --testMatch="**/migration.test.ts"
```

## 📚 Documentation

For complete documentation, visit: [Documentation](https://lor4z.github.io/capydb-docs)

## 🔧 Main Commands

| Command | Description |
|---------|-------------|
| `cap doctor` | Check prerequisites and connectivity |
| `cap migrations add <name>` | Create new migration with auto-detected author |
| `cap migrations import-ef` | Import migrations from EF Core |
| `cap migrations mergeschemas` | Consolidate multiple migrations |
| `cap plan` | Generate SQL execution plan |
| `cap apply` | Apply migrations to database |
| `cap status` | View database status and pending migrations |
| `cap validate` | Validate changelog syntax |
| `cap drift detect` | Detect undocumented changes |
| `cap tag <name>` | Create tag for versioning |
| `cap remove-tag <tag>` | Remove existing tag |
| `cap rollback count <N>` | Revert N migrations |
| `cap rollback to-tag <tag>` | Revert to a specific tag |
| `cap squash --tag <tag>` | Consolidate history up to a tag |
| `cap carga from-csv` | Generate data-seed changelog from CSV |
| `cap carga update` | Apply data seeds (label: data-seed) |
| `cap carga drop-all` | Remove all data and reapply |
| `cap carga reset` | Drop entire database and reapply all |
| `cap carga rollback` | Rollback last data changeset |
| `cap carga status` | Check data seed status |
| `cap bye` | Farewell with ASCII art 🦫 |

## 💬 Contact

- 📧 **Email**: evellynloraine@gmail.com
- 💼 **LinkedIn**: [Evellyn Fernandes](https://www.linkedin.com/in/evellynloraine)
- 🐱 **GitHub**: [lor4z](https://github.com/lor4z)
- 📚 **Documentation:** [CapyDb Docs](https://lor4z.github.io/capydb-docs)

## 📄 License

This project is licensed under Apache 2.0.

## 🔗 Useful Links

- **NuGet Package**: https://www.nuget.org/packages/capydb.cli/
- **GitHub Repository**: https://github.com/lor4z/capybara-db
- **Current Version**: 1.2.3

## 🆕 What's New in v1.2.3

### Data Seeding Features
- ✅ **CSV to Changelog Generator** - `cap carga from-csv` command
- ✅ **Intelligent Type Inference** - Automatically detects UUID, BOOLEAN, TIMESTAMP, NUMERIC, STRING
- ✅ **Automatic Dependency Resolution** - Detects FKs from CSV headers and orders tables automatically
- ✅ **Topological Sorting** - Smart ordering prevents FK constraint violations during data load
- ✅ **Circular Dependency Detection** - Warns about circular references between tables
- ✅ **Data Management Commands** - update, drop-all, reset, rollback, status for seed data
- ✅ **Auto-add to Master** - Option to automatically include in db.changelog-master.yaml
- ✅ **Label-based Filtering** - Uses `data-seed` label for selective operations

### Package Improvements
- ✅ **Embedded Dependencies** - CapyDb.Core, Runner, and Writers are now embedded
- ✅ **No External Dependencies** - Cleaner NuGet package with all DLLs included
- ✅ **Multi-target Support** - Full support for .NET 8.0 and .NET 9.0

### Previous Features (v1.0.7)
- ✅ **Enhanced file search on Windows** - Fixed glob pattern issues
- ✅ **Robust recursive search** - Finds liquibase.properties anywhere
- ✅ **Monorepo support** - Works with complex project structures
- ✅ **Improved assembly detection** - Better EF Core integration
- ✅ **Cross-platform compatibility** - Tested on Windows, Linux, macOS

---

**Developed with ❤️ to simplify database migration management.**