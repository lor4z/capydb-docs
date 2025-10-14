
# 🚀 CapyDb — Initial Setup

> Step-by-step guide to configure CapyDb for the first time

## 📋 Prerequisites
- ✅ **Java 8+** installed
- ✅ **Database** available (SQL Server, PostgreSQL or MySQL)
- ✅ **CapyDb CLI** installed (`cap --version` should work)

## 🔧 Step 1: Configure `liquibase.properties`

Edit `db/changelog/liquibase.properties` according to your environment:

### SQL Server
```properties
url=jdbc:sqlserver://localhost:1433;databaseName=MyDatabase;trustServerCertificate=true
username=sa
password=YourPassword
changeLogFile=db.changelog-master.yaml
logLevel=INFO
contexts=common
```

### PostgreSQL
```properties
url=jdbc:postgresql://localhost:5432/mydatabase
username=postgres
password=YourPassword
changeLogFile=db.changelog-master.yaml
logLevel=INFO
contexts=common
```

### MySQL
```properties
url=jdbc:mysql://localhost:3306/mydatabase?useSSL=false&allowPublicKeyRetrieval=true
username=root
password=YourPassword
changeLogFile=db.changelog-master.yaml
logLevel=INFO
contexts=common
```

## 🔧 Step 2: Test Your Configuration
```bash
cap doctor --defaults db/changelog/liquibase.properties
```

**Expected output (example):**
```
🩺 CapyDb Doctor - Checking prerequisites...

✅ Java: java version "17.0.7" 2023-04-18 LTS
⚠️  Docker: Not available (optional)
✅ Defaults: db/changelog/liquibase.properties exists
✅ Database: Connection OK

✅ All prerequisites are OK!
```

If connection fails, verify:
1) DB service is running  
2) Credentials are correct  
3) URL (host/port/dbname) is correct  
4) Network/firewall allows access

## 🐳 Alternative: Use Docker
```bash
cap doctor --defaults db/changelog/liquibase.properties --docker
cap status --defaults db/changelog/liquibase.properties --docker
cap apply --defaults db/changelog/liquibase.properties --docker
```

**Requirement:** Docker Desktop installed

## 📝 Step 3: Create your first migration
```bash
cap migrations add my-first-migration
# Inspect the generated file:
cat db/changelog/common/20*__my-first-migration.yaml
```

Edit the file and add your changes, e.g.:
```yaml
databaseChangeLog:
  - changeSet:
      id: 20250923_120000-my-first-migration
      author: Your Name
      context: common
      changes:
        - createTable:
            tableName: users
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: name
                  type: VARCHAR(100)
                  constraints:
                    nullable: false
              - column:
                  name: email
                  type: VARCHAR(255)
                  constraints:
                    nullable: false
                    unique: true
```

## 🔍 Step 4: Test and Apply
```bash
# See what will run (without applying)
cap plan --defaults db/changelog/liquibase.properties --output plan.sql

# Review the plan
cat plan.sql

# Apply
cap apply --defaults db/changelog/liquibase.properties

# Check status
cap status --defaults db/changelog/liquibase.properties
```

## 🎯 Final Structure
```
db/
├─ changelog/
│  ├─ common/
│  │  └─ 20250923_120000__my-first-migration.yaml
│  ├─ db.changelog-master.yaml
│  ├─ liquibase.properties
│  └─ liquibase.properties.example
└─ drivers/          # Optional if not using Docker
```

## 🛠️ Next Steps
### Development
```bash
git config user.name "Your Full Name"
cap migrations add add-products-table
cap migrations add tune-performance-indexes
```

### Production
```bash
cap plan --defaults liquibase.properties --output plan-prod.sql
cat plan-prod.sql
cap apply --defaults liquibase.properties
cap tag v1.0.0 --defaults liquibase.properties
```

### CI/CD
```bash
export CAPY_AUTHOR="CI/CD Pipeline"
cap doctor --defaults liquibase.properties --docker
cap plan --defaults liquibase.properties --docker --output plan-ci.sql
```

## ❗ Security
**Never** commit passwords. Add to `.gitignore`:
```gitignore
db/changelog/liquibase.properties
*.log
plan*.sql
drift-report*.xml
```

Use environment variables in production:
```properties
url=jdbc:sqlserver://localhost:1433;databaseName=${DB_NAME};trustServerCertificate=true
username=${DB_USER}
password=${DB_PASSWORD}
```

**Done! CapyDb is configured and running.**
For the complete reference, see the main guide.
