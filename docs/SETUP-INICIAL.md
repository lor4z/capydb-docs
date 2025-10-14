# 🚀 Setup Inicial do CapyDb

> Guia passo-a-passo para configurar o CapyDb pela primeira vez

## 📋 Pré-requisitos

Antes de começar, você precisa ter:

- ✅ **Java 8+** instalado
- ✅ **Banco de dados** (SQL Server, PostgreSQL ou MySQL)
- ✅ **CapyDb CLI** instalado (`cap --version` deve funcionar)

## 🔧 Passo 1: Configurar liquibase.properties

### Opção A: Usar arquivo criado automaticamente

O CapyDb já criou um arquivo de exemplo em `db/changelog/liquibase.properties`.

**Edite este arquivo** e ajuste as configurações:

```properties
# ALTERE ESTAS CONFIGURAÇÕES PARA SEU AMBIENTE:

# URL do seu banco (ajuste servidor, porta e nome do banco)
url=jdbc:sqlserver://localhost:1433;databaseName=MeuBancoDados;trustServerCertificate=true

# Suas credenciais
username=sa
password=SuaSenhaAqui
```

### Opção B: Configurações por tipo de banco

#### 🔷 SQL Server
```properties
url=jdbc:sqlserver://localhost:1433;databaseName=MeuBanco;trustServerCertificate=true
username=sa
password=MinhaSenh@123
changeLogFile=db.changelog-master.yaml
logLevel=INFO
contexts=common
```

#### 🐘 PostgreSQL
```properties
url=jdbc:postgresql://localhost:5432/meubanco
username=postgres
password=MinhaSenh@123
changeLogFile=db.changelog-master.yaml
logLevel=INFO
contexts=common
```

#### 🐬 MySQL
```properties
url=jdbc:mysql://localhost:3306/meubanco?useSSL=false&allowPublicKeyRetrieval=true
username=root
password=MinhaSenh@123
changeLogFile=db.changelog-master.yaml
logLevel=INFO
contexts=common
```

## 🔧 Passo 2: Testar Configuração

```bash
# Verificar se tudo está configurado corretamente
cap doctor --defaults db/changelog/liquibase.properties
```

**Resultado esperado:**
```
🩺 CapyDb Doctor - Verificando pré-requisitos...

✅ Java: java version "17.0.7" 2023-04-18 LTS
⚠️  Docker: Não disponível (opcional)
✅ Defaults: Arquivo db/changelog/liquibase.properties existe
✅ Banco: Conexão OK

✅ Todos os pré-requisitos estão OK!
```

### Se der erro de conexão:

1. **Verifique se o banco está rodando**
2. **Confirme as credenciais** (username/password)
3. **Teste a URL** (servidor, porta, nome do banco)
4. **Verifique firewall/rede**

## 🐳 Opção Alternativa: Usar Docker

Se você não quiser instalar Liquibase/drivers localmente:

```bash
# Todos os comandos funcionam com --docker
cap doctor --defaults db/changelog/liquibase.properties --docker
cap status --defaults db/changelog/liquibase.properties --docker
cap apply --defaults db/changelog/liquibase.properties --docker
```

**Pré-requisito**: Docker Desktop instalado

## 📝 Passo 3: Criar sua primeira migration

```bash
# Criar nova migration
cap migrations add minha-primeira-migration

# Ver o que foi criado
cat db/changelog/common/20*__minha-primeira-migration.yaml
```

**Edite o arquivo** e adicione suas mudanças:

```yaml
databaseChangeLog:
  - changeSet:
      id: 20250923_120000-minha-primeira-migration
      author: Seu Nome
      context: common
      changes:
        - createTable:
            tableName: usuarios
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: nome
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

## 🔍 Passo 4: Testar a migration

```bash
# Ver o que será executado (sem aplicar)
cap plan --defaults db/changelog/liquibase.properties --output plano.sql

# Ver o arquivo gerado
cat plano.sql

# Se estiver tudo OK, aplicar
cap apply --defaults db/changelog/liquibase.properties

# Verificar status
cap status --defaults db/changelog/liquibase.properties
```

## 🎯 Estrutura Final

Após o setup, você terá:

```
db/
├── changelog/
│   ├── common/
│   │   └── 20250923_120000__minha-primeira-migration.yaml
│   ├── db.changelog-master.yaml        # ✅ Já existe
│   ├── liquibase.properties           # ✅ Configurado
│   └── liquibase.properties.example   # 📝 Referência
└── drivers/                            # 📦 Opcional (se não usar Docker)
```

## 🛠️ Próximos Passos

### Para Desenvolvimento:
```bash
# Configurar autor automático
git config user.name "Seu Nome Completo"

# Criar migrations conforme necessário
cap migrations add adicionar-tabela-produtos
cap migrations add ajustar-indices-performance
```

### Para Produção:
```bash
# Sempre gerar plano primeiro
cap plan --defaults liquibase.properties --output plano-producao.sql

# Revisar plano antes de aplicar
cat plano-producao.sql

# Aplicar em produção
cap apply --defaults liquibase.properties

# Criar tag de versão
cap tag v1.0.0 --defaults liquibase.properties
```

### Para CI/CD:
```bash
# Em pipelines, usar variável de ambiente para autor
export CAPY_AUTHOR="CI/CD Pipeline"

# Validar antes de deploy
cap doctor --defaults liquibase.properties --docker
cap plan --defaults liquibase.properties --docker --output plano-ci.sql
```

## ❗ Segurança

### NUNCA commitar senhas!

Adicione ao `.gitignore`:
```gitignore
# Configurações com credenciais
db/changelog/liquibase.properties

# Logs e arquivos temporários
*.log
plano*.sql
drift-report*.xml
```

### Use variáveis de ambiente em produção:
```bash
# Em vez de senha no arquivo
url=jdbc:sqlserver://localhost:1433;databaseName=${DB_NAME};trustServerCertificate=true
username=${DB_USER}
password=${DB_PASSWORD}
```

## 🆘 Problemas Comuns

### Java não encontrado
```bash
# Verificar instalação
java -version

# Se não tiver, instalar (Windows)
# Baixar de: https://adoptium.net/
```

### Banco não conecta
```bash
# Testar conexão manual
# SQL Server: verificar se SQL Server está rodando
# PostgreSQL: verificar se serviço postgres está ativo
# MySQL: verificar se mysqld está rodando
```

### Drivers não encontrados
```bash
# Opção 1: Usar Docker (recomendado)
cap status --defaults liquibase.properties --docker

# Opção 2: Baixar drivers
# SQL Server: https://docs.microsoft.com/en-us/sql/connect/jdbc/
# PostgreSQL: https://jdbc.postgresql.org/download.html
# MySQL: https://dev.mysql.com/downloads/connector/j/
```

---

**🎉 Pronto! Agora você está com o CapyDb configurado e funcionando!**

Para referência completa, consulte: `CAPY-CLI-GUIDE.md`