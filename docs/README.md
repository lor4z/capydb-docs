# CapyDb CLI 

Uma ferramenta de linha de comando para gerenciamento de migrations de banco de dados com **Liquibase** e **Entity Framework Core**.

## 🚀 O que é o CapyDb?

O CapyDb CLI resolve o problema de gerenciar migrations de banco de dados de forma consistente e eficiente, oferecendo:

- ✅ **Detecção automática de configuração** - busca automaticamente por `liquibase.properties`
- ✅ **Criação de migrations** no formato Liquibase YAML
- ✅ **Importação de migrations** do Entity Framework Core
- ✅ **Merge e consolidação** de schemas automatizada
- ✅ **Execução segura** com planos de execução SQL
- ✅ **Suporte multi-SGBD** (SQL Server, PostgreSQL, MySQL, Oracle)
- ✅ **Integração com Docker** e pipelines CI/CD
- ✅ **Detecção de drift** - identifica mudanças não documentadas
- ✅ **Sistema de tags** - criação e remoção de tags para versionamento
- ✅ **Rollback inteligente** - reverter por contagem ou até uma tag específica
- ✅ **Squash de histórico** - consolida migrations antigas
- ✅ **Detecção automática de autor** via Git/CI/CD
- ✅ **Diagnóstico completo** com `cap doctor`
- ✅ **Validação de changelog** antes da execução
- ✅ **Conversor de INSERTs** - converte INSERTs SQL em formato Liquibase


## 📦 Instalação

### Pré-requisitos
- .NET 8.0 SDK ou superior
- Java 8+ (para Liquibase)

### Instalação Global
```bash
# Instalar via NuGet
dotnet tool install -g capydb.cli

# Verificar instalação
cap --version  # 1.0.7
```

## 🏁 Começando

### 1. Configurar Projeto
```bash
# Estrutura recomendada
meu-projeto/
├── db/
│   └── changelog/
│       ├── common/
│       ├── db.changelog-master.yaml
│       └── liquibase.properties  # ← O CLI busca automaticamente aqui!
├── src/
└── Infrastructure/  # Ou qualquer estrutura de projeto
```

### 2. Verificar Pré-requisitos
```bash
cap doctor
```

### 3. Criar Primeira Migration
```bash
# Criar migration básica
cap migrations add criar-usuarios

# Com autor específico
cap migrations add criar-produtos --author "Seu Nome"
```

### 4. Importar do Entity Framework
```bash
cap migrations import-ef \
  --assembly ./MeuApp.dll \
  --name CreateUsersTable \
  --provider sqlserver
```

### 5. Executar Migrations (Detecção Automática!)
```bash
# O CLI busca automaticamente em ./db/changelog/liquibase.properties
cap plan      # Gerar plano de execução
cap apply     # Aplicar migrations
cap status    # Ver status do banco

# Criar tag após deployment
cap tag v1.0.0

# Rollback se necessário
cap rollback count 2
cap rollback to-tag v1.0.0
```

### 💡 Detecção Automática de Configuração (Aprimorada!)

O CLI agora possui **busca recursiva robusta** para `liquibase.properties`:

**Prioridade de Busca:**
1. `./db/changelog/liquibase.properties` (recomendado)
2. `./liquibase.properties` (diretório raiz)
3. `./config/liquibase.properties`
4. `./database/liquibase.properties`
5. `./src/*/db/changelog/liquibase.properties` (monorepos!)
6. `./apps/*/db/changelog/liquibase.properties` (monorepos!)
7. **Busca recursiva em todos os subdiretórios** (excluindo node_modules, .git)

**Funciona perfeitamente no Windows, Linux e macOS!**

```bash
# Antes (ainda funciona):
cap apply --defaults ./db/changelog/liquibase.properties

# Agora (ainda mais simples):
cap apply  # Detecta automaticamente em monorepos, estruturas aninhadas, qualquer lugar!
```

## 📋 Exemplos Rápidos

### Estrutura de Projeto Recomendada
```
meu-projeto/
├── db/
│   ├── changelog/
│   │   ├── common/
│   │   │   └── 20250924_120000__criar-usuarios.yaml
│   │   ├── db.changelog-master.yaml
│   │   └── liquibase.properties  # ← Detectado automaticamente!
│   └── drivers/
└── src/
```

### Migration Gerada Automaticamente
```yaml
# db/changelog/common/20250924_120000__criar-usuarios.yaml
databaseChangeLog:
  - changeSet:
      id: 20250924_120000-criar-usuarios
      author: Evellyn Fernandes  # ← Detectado via Git!
      context: common
      changes:
        - createTable:
            tableName: usuarios
            columns:
              - column:
                  name: id
                  type: int
                  constraints:
                    primaryKey: true
```

### Fluxo Completo Simplificado
```bash
# 1. Criar migration
cap migrations add criar-usuarios

# 2. Revisar o que vai ser executado
cap plan

# 3. Aplicar no banco
cap apply

# 4. Verificar status
cap status

# 5. Criar tag de versão
cap tag v1.0.0

# 6. Se precisar reverter
cap rollback to-tag v1.0.0
```

### Múltiplos Ambientes e SGBDs
```bash
# Ambiente padrão (detecta automaticamente)
cap apply

# PostgreSQL com arquivo customizado
cap apply --defaults ./db/changelog/liquibase-postgres.properties

# MySQL com Docker
cap apply --defaults ./db/changelog/liquibase-mysql.properties --docker

# Oracle
cap apply --defaults ./db/changelog/liquibase-oracle.properties
```

### Conversão de INSERTs SQL
```bash
# Converter arquivo SQL com INSERTs para formato Liquibase
cap convert-inserts --input ./data.sql --output ./changelog.yaml

# Especificar nome da tabela
cap convert-inserts --input ./data.sql --table usuarios --output ./changelog.yaml
```

## 🧪 Testes

O projeto inclui testes de integração automatizados usando Jest e Prisma.

```bash
# Executar testes de integração
cd tests/integration
npm install
npm test

# Testes com diferentes SGBDs
npm test -- --testMatch="**/migration.test.ts"
```

## 📚 Documentação

Para documentação completa, visite: [Documentação](https://docusaurus.io/pt-BR/docs)

## 🔧 Comandos Principais

| Comando | Descrição |
|---------|-----------|
| `cap doctor` | Verificar pré-requisitos e conectividade |
| `cap migrations add <nome>` | Criar nova migration com autor automático |
| `cap migrations import-ef` | Importar migrations do EF Core |
| `cap migrations mergeschemas` | Consolidar múltiplas migrations |
| `cap plan` | Gerar plano SQL de execução |
| `cap apply` | Aplicar migrations no banco |
| `cap status` | Ver status e migrations pendentes |
| `cap validate` | Validar sintaxe do changelog |
| `cap drift detect` | Detectar mudanças não documentadas |
| `cap tag <nome>` | Criar tag para versionamento |
| `cap remove-tag <tag>` | Remover tag existente |
| `cap rollback count <N>` | Reverter N migrations |
| `cap rollback to-tag <tag>` | Reverter até uma tag específica |
| `cap squash --tag <tag>` | Consolidar histórico até tag |
| `cap bye` | Despedida com ASCII art 🦫 |

## 💬 Contato

- 📧 **E-mail**: lora@gmail.com
- 💼 **LinkedIn**: [Evellyn Fernandes](https://www.linkedin.com/in/evellynloraine)
- 🐱 **GitHub**: [lor4z](https://github.com/lor4z)

## 📄 Licença

Este projeto está sob licença Apache 2.0.

## 🔗 Links Úteis

- **NuGet Package**: https://www.nuget.org/packages/capydb.cli/
- **GitHub Repository**: https://github.com/lor4z/capybara-db
- **Versão Atual**: 1.0.9

## 🆕 Novidades na v1.0.9

- ✅ **Busca de arquivos aprimorada no Windows** - Corrigidos problemas com padrões glob
- ✅ **Busca recursiva robusta** - Encontra liquibase.properties em qualquer lugar
- ✅ **Suporte a monorepos** - Funciona com estruturas de projeto complexas
- ✅ **Detecção de assemblies melhorada** - Melhor integração com EF Core
- ✅ **Compatibilidade multiplataforma** - Testado no Windows, Linux, macOS

---

**Desenvolvido com ❤️ para facilitar o gerenciamento de migrations de banco de dados.**