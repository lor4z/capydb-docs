# CapyDb CLI - Guia Completo

> Ferramenta de linha de comando para gerenciamento de migrations com Liquibase e Entity Framework Core

## 📋 Índice

- [Instalação](#-instalação)
- [Comandos Principais](#-comandos-principais)
- [Migrations](#-migrations)
- [Operações de Banco](#️-operações-de-banco)
- [Utilitários](#️-utilitários)
- [Sistema de Autores](#-sistema-de-autores)
- [Configuração Avançada](#️-configuração-avançada)
- [Troubleshooting](#-troubleshooting)

---

## 🚀 Instalação

### Como Ferramenta Global

```bash
# Compilar e empacotar
dotnet pack src/CapyDb.Cli/CapyDb.Cli.csproj -o nupkg

# Instalar globalmente
dotnet tool install -g capydb.cli --add-source ./nupkg

# Verificar instalação
cap --version
```

### Execução Direta

```bash
# Executar sem instalar
dotnet run --project src/CapyDb.Cli -- [comandos]
```

---

## 📖 Comandos Principais

### Informações Gerais

```bash
# Versão do CapyDb
cap --version

# Ajuda completa
cap --help

# Verificar pré-requisitos
cap doctor

# Despedida (ASCII art)
cap bye
```

---

## 📝 Migrations

### 1. Criar Nova Migration

```bash
# Sintaxe básica
cap migrations add <nome> [opções]

# Opções disponíveis:
#   --no-stubs     : Não criar diretórios específicos de SGBD
#   --author <nome>: Especificar autor manualmente
```

**Exemplos:**

```bash
# Migration simples
cap migrations add criar-usuarios

# Com autor customizado
cap migrations add criar-usuarios --author "João Silva"

# Sem criar stubs para SGBDs específicos
cap migrations add criar-usuarios --no-stubs

# Combinando opções
cap migrations add criar-usuarios --author "Maria Santos" --no-stubs
```

**O que acontece:**
1. ✅ Cria arquivo YAML em `db/changelog/common/`
2. ✅ Atualiza `db.changelog-master.yaml`
3. ✅ Detecta autor automaticamente (ou usa `--author`)
4. ✅ Gera timestamp único para evitar conflitos

**Estrutura gerada:**
```yaml
# db/changelog/common/20250923_014331__criar-usuarios.yaml
databaseChangeLog:
  - changeSet:
      id: 20250923_014331-criar-usuarios
      author: João Silva  # Detectado automaticamente
      context: common
      changes:
        # Suas alterações aqui
```

### 2. Importar Migration do Entity Framework

```bash
# Sintaxe básica
cap migrations import-ef [opções]

# Opções obrigatórias:
#   --assembly <path> : Caminho para DLL do EF Core
#   --name <class>    : Nome da classe Migration
#   --provider <type> : Tipo do provider (sqlserver|postgres|mysql)

# Opções extras:
#   --author <nome>   : Autor customizado
```

**Exemplos:**

```bash
# Import básico do SQL Server
cap migrations import-ef \
  --assembly ./MyApp/bin/Debug/net8.0/MyApp.dll \
  --name CreateUsersTable \
  --provider sqlserver

# Com autor customizado
cap migrations import-ef \
  --assembly ./MyApp.dll \
  --name CreateUsersTable \
  --provider postgres \
  --author "João Silva"

# MySQL
cap migrations import-ef \
  --assembly ./MyApp.dll \
  --name AddProductsTable \
  --provider mysql
```

**O que acontece:**
1. ✅ Carrega assembly .NET especificado
2. ✅ Encontra a classe Migration pelo nome
3. ✅ Executa o método `Up()` em memória
4. ✅ Converte operações EF para YAML Liquibase
5. ✅ Salva em `db/changelog/common/`

**Operações EF Suportadas:**
- ✅ `CreateTable` → `createTable`
- ✅ `AddColumn` → `addColumn`
- ✅ `InsertData` → `insert`
- ✅ `DeleteData` → `delete`
- ⚠️ Outras operações → SQL genérico com comentário

### 3. Merge de Schemas

```bash
# Sintaxe básica
cap migrations mergeschemas [opções]

# Opções:
#   --scope <tipo>        : Escopo do merge (common|mssql|postgres|mysql)
#   --include-merged      : Incluir arquivos já merged anteriormente
#   --delete-old          : Mover arquivos antigos automaticamente
```

**Exemplos:**

```bash
# Merge do escopo comum
cap migrations mergeschemas --scope common

# Incluindo merges antigos
cap migrations mergeschemas --scope common --include-merged

# Com deleção automática dos arquivos antigos
cap migrations mergeschemas --scope postgres --delete-old

# Processo interativo (padrão)
cap migrations mergeschemas --scope mysql
# Pergunta: "Mover arquivos usados para 'db/changelog/deleteSchemas/mysql'? [s/N]"
```

**O que acontece:**
1. ✅ Coleta todos os arquivos .yaml/.yml do escopo
2. ✅ Ordena por timestamp
3. ✅ Consolida em um único arquivo `__merged-<scope>.yaml`
4. ✅ Atualiza `db.changelog-master.yaml`
5. ✅ Opcionalmente move arquivos antigos para `deleteSchemas/`

---

## 🗃️ Operações de Banco

### 1. Gerar Plano de Execução

```bash
# Sintaxe básica
cap plan --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker em vez de Liquibase CLI local
#   --workdir <dir>   : Diretório de trabalho
#   --output <arquivo>: Arquivo de saída para o plano SQL
```

**Exemplos:**

```bash
# Gerar plano básico
cap plan --defaults ./db/changelog/liquibase.properties

# Salvar plano em arquivo
cap plan --defaults ./db/changelog/liquibase.properties --output plan.sql

# Usar Docker
cap plan --defaults ./db/changelog/liquibase.properties --docker

# Com diretório de trabalho específico
cap plan --defaults ./config/liquibase.properties --workdir ./database
```

**Use Cases:**
- 📋 **Code Review**: Anexar `plan.sql` em Pull Requests
- 🔍 **Auditoria**: Revisar mudanças antes de aplicar
- 📊 **Documentação**: Histórico de alterações aplicadas

### 💡 Detecção Automática de Arquivos (Novo na v1.0.7!)

A partir da v1.0.7, o CapyDb possui **busca recursiva inteligente** para `liquibase.properties`:

**Prioridade de busca:**
1. `./db/changelog/liquibase.properties` (recomendado)
2. `./liquibase.properties` (raiz do projeto)
3. `./config/liquibase.properties`
4. `./database/liquibase.properties`
5. `./migrations/liquibase.properties`
6. `./src/*/db/changelog/liquibase.properties` (estruturas de monorepo)
7. `./apps/*/db/changelog/liquibase.properties` (estruturas de monorepo)
8. `./*/db/changelog/liquibase.properties` (qualquer subdiretório)
9. **Busca recursiva geral** em todos os subdiretórios (excluindo node_modules e .git)

**Vantagens:**
- ✅ Funciona automaticamente em monorepos complexos
- ✅ Detecta arquivos em estruturas aninhadas
- ✅ Compatibilidade total com Windows (corrigidos problemas de glob patterns)
- ✅ Não precisa mais especificar `--defaults` em 99% dos casos

```bash
# Agora você pode fazer apenas:
cap apply          # Busca automaticamente!
cap plan           # Em qualquer lugar do projeto!
cap status         # Mesmo em estruturas complexas!

# O --defaults ainda funciona se necessário:
cap apply --defaults ./custom/path/liquibase.properties
```

### 2. Aplicar Migrations

```bash
# Sintaxe básica
cap apply --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
#   --output <arquivo>: Log da execução
```

**Exemplos:**

```bash
# Aplicar todas as migrations pendentes
cap apply --defaults ./db/changelog/liquibase.properties

# Com log detalhado
cap apply --defaults ./db/changelog/liquibase.properties --output apply.log

# Usando Docker
cap apply --defaults ./db/changelog/liquibase.properties --docker
```

### 3. Verificar Status

```bash
# Ver status das migrations
cap status --defaults ./db/changelog/liquibase.properties

# Com Docker
cap status --defaults ./db/changelog/liquibase.properties --docker
```

### 4. Validar Changelog

```bash
# Validar sintaxe e estrutura
cap validate --defaults ./db/changelog/liquibase.properties

# Com output detalhado
cap validate --defaults ./db/changelog/liquibase.properties --output validation.log
```

### 5. Criar Tags

```bash
# Sintaxe básica
cap tag <nome> --defaults <arquivo.properties> [opções]

# Exemplos
cap tag v1.0.0 --defaults ./db/changelog/liquibase.properties
cap tag release-2025-01 --defaults ./db/changelog/liquibase.properties --docker
```

### 6. Remover Tags

```bash
# Sintaxe básica
cap remove-tag <tag> --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
```

**Exemplos:**

```bash
# Remover tag existente
cap remove-tag v1.0.0 --defaults ./db/changelog/liquibase.properties

# Com Docker
cap remove-tag release-2025-01 --defaults ./db/changelog/liquibase.properties --docker

# Com diretório de trabalho específico
cap remove-tag old-tag --defaults ./config/liquibase.properties --workdir ./database
```

**O que acontece:**
1. ✅ Cria changeset temporário com comando SQL UPDATE
2. ✅ Remove a tag da coluna TAG na tabela DATABASECHANGELOG
3. ✅ Limpa arquivo temporário após execução
4. ✅ Valida se a tag existe antes de remover

**Use Cases:**
- 🏷️ **Correção de Tags**: Remover tags criadas por engano
- 🔄 **Re-tagging**: Limpar tags antigas antes de criar novas
- 🧹 **Limpeza**: Manter apenas tags relevantes no histórico

### 7. Rollback

```bash
# Rollback por contagem
cap rollback count <N> --defaults <arquivo.properties> [opções]

# Rollback até tag específica
cap rollback to-tag <tag> --defaults <arquivo.properties> [opções]
```

**Exemplos:**

```bash
# Reverter últimas 3 migrations
cap rollback count 3 --defaults ./db/changelog/liquibase.properties

# Voltar para tag específica
cap rollback to-tag v1.0.0 --defaults ./db/changelog/liquibase.properties

# Com Docker
cap rollback count 1 --defaults ./db/changelog/liquibase.properties --docker
```

---

## 🛠️ Utilitários

### 1. Doctor - Verificação de Pré-requisitos

```bash
# Verificação completa
cap doctor

# Com arquivo de configuração específico
cap doctor --defaults ./db/changelog/liquibase.properties
```

**O que é verificado:**
- ✅ **Java**: Instalação e versão
- ⚠️ **Docker**: Disponibilidade (opcional)
- ⚠️ **Drivers**: Diretório `db/drivers/` e arquivos .jar
- ✅ **Arquivo defaults**: Existência e conteúdo
- ✅ **Conexão DB**: Teste de conectividade

**Exemplo de saída:**
```
🩺 CapyDb Doctor - Verificando pré-requisitos...

✅ Java: java version "17.0.7" 2023-04-18 LTS
⚠️  Docker: Não disponível (opcional)
⚠️  Drivers: Diretório db/drivers não existe
   Crie o diretório e adicione os JARs dos drivers JDBC
✅ Defaults: Arquivo db/changelog/liquibase.properties existe
✅ Banco: Conexão OK

✅ Todos os pré-requisitos estão OK!
```

### 2. Drift Detection - Detecção de Divergências

```bash
# Sintaxe básica
cap drift detect --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
#   --output <arquivo>: Arquivo de relatório (padrão: drift-report.xml)
```

**Exemplos:**

```bash
# Detectar divergências básicas
cap drift detect --defaults ./db/changelog/liquibase.properties

# Com arquivo de saída customizado
cap drift detect --defaults ./db/changelog/liquibase.properties --output drift-analysis.xml

# Usando Docker
cap drift detect --defaults ./db/changelog/liquibase.properties --docker
```

**Use Cases:**
- 🔍 **Schema Validation**: Verificar se DB está sincronizado
- 📊 **Auditoria**: Detectar mudanças manuais não documentadas
- 🚨 **CI/CD**: Validar ambiente antes de deploy

### 3. Squash - Consolidação de Histórico

```bash
# Sintaxe básica
cap squash --tag <tag> --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
```

**Exemplos:**

```bash
# Consolidar até tag específica
cap squash --tag v1.0.0 --defaults ./db/changelog/liquibase.properties

# Com Docker
cap squash --tag release-2025 --defaults ./db/changelog/liquibase.properties --docker
```

**O que acontece:**
1. 🗜️ Aplica todo histórico até a tag especificada
2. 📝 Gera baseline consolidado do schema atual
3. 📦 Move arquivos antigos para `db/changelog/archive/`
4. 🔄 Recria changelog master apontando para baseline

**Estrutura após squash:**
```
db/changelog/
├── archive/
│   └── squash-20250923-014500/
│       ├── common/           # Migrations antigas
│       ├── mssql/
│       └── postgres/
├── common/
│   └── baseline-20250923-014500.yaml  # Schema consolidado
└── db.changelog-master.yaml           # Aponta para baseline
```

### 4. Conversor de INSERTs SQL

```bash
# Sintaxe básica
cap convert-inserts --input <arquivo.sql> --output <arquivo.yaml> [opções]

# Opções:
#   --input <path>    : Arquivo SQL com INSERTs (obrigatório)
#   --output <path>   : Arquivo YAML de saída (obrigatório)
#   --table <nome>    : Nome da tabela (opcional, detectado automaticamente)
#   --author <nome>   : Autor do changeset (opcional)
```

**Exemplos:**

```bash
# Conversão básica
cap convert-inserts --input ./data.sql --output ./changelog.yaml

# Especificar tabela e autor
cap convert-inserts \
  --input ./usuarios.sql \
  --output ./db/changelog/common/seed-usuarios.yaml \
  --table usuarios \
  --author "João Silva"

# Converter múltiplas INSERTs
cap convert-inserts --input ./seed-data.sql --output ./changelog-seed.yaml
```

**Formato de entrada (SQL):**
```sql
INSERT INTO usuarios (id, nome, email, ativo) VALUES (1, 'João Silva', 'joao@email.com', 1);
INSERT INTO usuarios (id, nome, email, ativo) VALUES (2, 'Maria Santos', 'maria@email.com', 1);
INSERT INTO usuarios (id, nome, email, ativo) VALUES (3, 'Pedro Oliveira', 'pedro@email.com', 0);
```

**Formato de saída (YAML):**
```yaml
databaseChangeLog:
  - changeSet:
      id: convert-inserts-20250930-101530
      author: João Silva
      context: common
      changes:
        - insert:
            tableName: usuarios
            columns:
              - column: { name: id, value: 1 }
              - column: { name: nome, value: 'João Silva' }
              - column: { name: email, value: 'joao@email.com' }
              - column: { name: ativo, value: 1 }
        - insert:
            tableName: usuarios
            columns:
              - column: { name: id, value: 2 }
              - column: { name: nome, value: 'Maria Santos' }
              - column: { name: email, value: 'maria@email.com' }
              - column: { name: ativo, value: 1 }
```

**Use Cases:**
- 📊 **Seed Data**: Converter dados iniciais para Liquibase
- 🔄 **Migração**: Importar dados de SQL legado
- 🧪 **Testes**: Criar dados de teste em formato portável
- 📦 **Fixtures**: Preparar dados para diferentes ambientes

**Recursos:**
- ✅ Detecta automaticamente nome da tabela
- ✅ Preserva tipos de dados (strings, números, booleanos, NULL)
- ✅ Suporta múltiplos INSERTs no mesmo arquivo
- ✅ Escapa caracteres especiais corretamente
- ✅ Gera changeSet com ID único baseado em timestamp

---

## 📊 Data Seeding (Carga de Dados)

### Visão Geral

O CapyDb oferece um sistema completo para gerenciar dados iniciais (seed data) separadamente das migrations de schema. Isso permite:

- ✅ Versionamento independente de dados e schema
- ✅ Geração automática de changesets a partir de CSV
- ✅ Inferência inteligente de tipos de dados
- ✅ Rollback específico de dados sem afetar o schema
- ✅ Filtragem por label (`data-seed`)

### 1. Gerar Changelog a partir de CSV

```bash
# Sintaxe básica
cap carga from-csv --input <arquivo.csv> --table <tabela> [opções]

# Opções obrigatórias:
#   --input, -i <path>    : Caminho para arquivo CSV
#   --table, -t <nome>    : Nome da tabela de destino

# Opções adicionais:
#   --output, -o <path>   : Arquivo YAML de saída (padrão: db/changelog/carga-updates/YYYYMMDD__carga-<tabela>.yaml)
#   --author <nome>       : Nome do autor (padrão: CapyDb)
#   --context <contexto>  : Contexto do changeset (padrão: common)
#   --add-to-master       : Adicionar automaticamente ao db.changelog-master.yaml
```

**Exemplos:**

```bash
# Geração básica
cap carga from-csv --input db/carga/Users.csv --table Users

# Com todas as opções
cap carga from-csv \
  --input db/carga/Countries.csv \
  --table TabelaAuxiliarCountries \
  --author "Evellyn Fernandes" \
  --output db/changelog/carga-updates/countries.yaml \
  --add-to-master

# Forma abreviada
cap carga from-csv -i db/carga/Cities.csv -t Cities

# Múltiplos arquivos em lote
for file in db/carga/*.csv; do
  table=$(basename "$file" .csv)
  cap carga from-csv -i "$file" -t "$table" --add-to-master
done
```

**Inferência Automática de Tipos:**

O comando analisa os nomes das colunas do CSV e infere os tipos automaticamente:

| Padrão de Nome | Tipo Inferido | Exemplos |
|----------------|---------------|----------|
| `*Id`, `PublicId` | UUID | `Id`, `UserId`, `PublicId` |
| `Excluido`, `Ativo`, `Is*`, `Has*` | BOOLEAN | `Excluido`, `Ativo`, `IsActive`, `HasPermission` |
| `Data*`, `Date*`, `*Timestamp`, `*At` | TIMESTAMP | `DataCriacao`, `DateCreated`, `CreatedAt`, `UpdatedAt` |
| `*Num`, `*Numero`, `Count`, `Quantidade` | NUMERIC | `CodigoNum`, `Age`, `Count`, `Quantidade` |
| Outros | STRING | `Name`, `Email`, `Description` |

**Resolução Automática de Dependências:**

O CapyDb detecta automaticamente as dependências entre tabelas e ordena os changesets corretamente:

- ✅ **Detecção de Foreign Keys**: Colunas terminadas em `Id` (exceto `Id` e `PublicId`) são consideradas FKs
- ✅ **Resolução de Nomes**: Remove prefixos como `TabelaAuxiliar` para encontrar tabelas referenciadas
- ✅ **Ordenação Topológica**: Tabelas sem dependências são carregadas primeiro
- ✅ **Detecção de Ciclos**: Alerta sobre dependências circulares entre tabelas
- ✅ **Matching Inteligente**: Busca tabelas por nome exato, com prefixo, plural, ou parcial

**Exemplo de Dependências:**
```
ModuloCategoria.csv tem coluna "ModuloId"
  ↓ CapyDb detecta FK para "Modulo"
  ↓ Ordena automaticamente:
    1. Modulo.csv           (sem dependências)
    2. ModuloCategoria.csv  (depende de Modulo)
```

**Exemplo de CSV:**
```csv
Id,Name,Email,IsActive,CreatedAt,Age
cc53be96-29d4-46ec-882d-042ad26f3aa5,João Silva,joao@email.com,true,2024-01-01,30
6771123b-a632-423d-9c8a-f1ec7fd4b438,Maria Santos,maria@email.com,true,2024-01-02,25
```

**YAML Gerado:**
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

### 2. Aplicar Dados de Seed

```bash
# Sintaxe básica
cap carga update --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
#   --output <arquivo>: Log da execução
#   --logs            : Exibir logs detalhados
```

**Exemplos:**

```bash
# Aplicar todos os changesets com label 'data-seed'
cap carga update --defaults ./db/changelog/liquibase.properties

# Com logs detalhados
cap carga update --defaults ./db/changelog/liquibase.properties --logs

# Usando Docker
cap carga update --defaults ./db/changelog/liquibase.properties --docker
```

**O que acontece:**
1. ✅ Liquibase filtra changesets com label `data-seed`
2. ✅ Aplica apenas os changesets de dados ainda não executados
3. ✅ Registra execução na tabela `DATABASECHANGELOG`
4. ✅ Pula changesets de schema (sem label ou com outras labels)

### 3. Remover e Reaplicar Dados (Desenvolvimento)

```bash
# Sintaxe básica
cap carga drop-all --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
#   --logs            : Exibir logs detalhados
```

**Exemplos:**

```bash
# Remover todos os dados e reaplicar
cap carga drop-all --defaults ./db/changelog/liquibase.properties

# Com Docker
cap carga drop-all --defaults ./db/changelog/liquibase.properties --docker
```

**⚠️ ATENÇÃO:**
- Remove TODOS os dados das tabelas referenciadas nos changesets de data-seed
- Útil para desenvolvimento e testes
- **NÃO usar em produção sem backup!**

### 4. Reset Completo (Schema + Dados)

```bash
# Sintaxe básica
cap carga reset --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
#   --logs            : Exibir logs detalhados
```

**Exemplos:**

```bash
# CUIDADO: Apaga TODO o banco e recria
cap carga reset --defaults ./db/changelog/liquibase.properties

# Com confirmação
read -p "Tem certeza que deseja resetar o banco? [s/N] " -n 1 -r
echo
if [[ $REPLY =~ ^[Ss]$ ]]; then
    cap carga reset --defaults ./db/changelog/liquibase.properties
fi
```

**⚠️ PERIGO:**
- Executa `DROP ALL` no banco inteiro
- Recria schema e dados do zero
- **EXTREMAMENTE DESTRUTIVO**
- Requer confirmação manual

### 5. Rollback de Dados

```bash
# Sintaxe básica
cap carga rollback --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
#   --logs            : Exibir logs detalhados
```

**Exemplos:**

```bash
# Reverter último changeset de dados
cap carga rollback --defaults ./db/changelog/liquibase.properties

# Com logs
cap carga rollback --defaults ./db/changelog/liquibase.properties --logs
```

**O que acontece:**
1. ✅ Identifica último changeset com label `data-seed`
2. ✅ Executa bloco `rollback` do changeset
3. ✅ Remove registro da tabela `DATABASECHANGELOG`
4. ✅ Permite reaplicar o changeset posteriormente

### 6. Status de Dados

```bash
# Sintaxe básica
cap carga status --defaults <arquivo.properties> [opções]

# Opções:
#   --docker          : Usar Docker
#   --workdir <dir>   : Diretório de trabalho
```

**Exemplos:**

```bash
# Ver status dos changesets de dados
cap carga status --defaults ./db/changelog/liquibase.properties

# Com Docker
cap carga status --defaults ./db/changelog/liquibase.properties --docker
```

**Saída:**
```
3 changesets have not been applied to sa@jdbc:sqlserver://localhost:1433;...
  db/changelog/carga-updates/20251017__carga-countries.yaml::20251017-carga-countries::Evellyn Fernandes
  db/changelog/carga-updates/20251017__carga-cities.yaml::20251017-carga-cities::Evellyn Fernandes
  db/changelog/carga-updates/20251017__carga-users.yaml::20251017-carga-users::Evellyn Fernandes
```

### Workflow Completo de Data Seeding

```bash
# 1. Preparar CSV
cat > db/carga/Countries.csv << EOF
Id,Name,Code,Population,IsActive
1,Brasil,BR,212000000,true
2,Estados Unidos,US,331000000,true
3,Argentina,AR,45000000,true
EOF

# 2. Gerar changelog
cap carga from-csv \
  --input db/carga/Countries.csv \
  --table Countries \
  --author "Evellyn Fernandes" \
  --add-to-master

# 3. Verificar arquivo gerado
cat db/changelog/carga-updates/20251017__carga-countries.yaml

# 4. Aplicar dados
cap carga update --defaults db/changelog/liquibase.properties

# 5. Verificar status
cap carga status --defaults db/changelog/liquibase.properties

# 6. Se necessário, reverter
cap carga rollback --defaults db/changelog/liquibase.properties
```

### Estrutura de Projeto com Data Seeding

```
projeto/
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

### Resolução Automática de Dependências

O CapyDb CLI v1.2.3+ inclui um sistema inteligente de resolução de dependências que:

**Como Funciona:**

1. **Escaneia todos os changesets** em `db/changelog/carga-updates/`
2. **Lê os cabeçalhos CSV** de cada arquivo referenciado
3. **Detecta Foreign Keys** - colunas terminadas em `Id` (exceto `Id` e `PublicId`)
4. **Resolve nomes de tabelas** - remove prefixos como `TabelaAuxiliar`
5. **Ordena topologicamente** - coloca dependências primeiro
6. **Atualiza db.changelog-carga.yaml** com a ordem correta

**Algoritmo de Matching:**

```
Coluna CSV: "ModuloId"
  ↓ Remove "Id" → "Modulo"
  ↓ Busca por:
    1. Match exato: "Modulo"
    2. Com prefixo: "TabelaAuxiliarModulo"
    3. Plural: "Modulos" ou "TabelaAuxiliarModulos"
    4. Parcial: termina com "Modulo"
  ↓ Encontrado: adiciona dependência
```

**Exemplo Prático:**

```bash
# Você tem estes CSVs:
# - Modulo.csv (sem FKs)
# - ModuloCategoria.csv (tem ModuloId → FK para Modulo)
# - FundamentacoesLegais.csv (tem LeisId → FK para TabelaAuxiliarLeis)

# Ao executar:
cap carga from-csv -i db/carga/ModuloCategoria.csv -t ModuloCategoria --add-to-master
cap carga from-csv -i db/carga/Modulo.csv -t Modulo --add-to-master
cap carga from-csv -i db/carga/FundamentacoesLegais.csv -t FundamentacoesLegais --add-to-master

# O CapyDb reordena automaticamente em db.changelog-carga.yaml:
#   1. Modulo (sem dependências)
#   2. TabelaAuxiliarLeis (sem dependências)
#   3. ModuloCategoria (depende de Modulo)
#   4. FundamentacoesLegais (depende de Leis)

# Resultado: zero erros de FK constraint!
```

**Tratamento de Casos Especiais:**

- **Dependências não encontradas**: Aviso amarelo, mas continua execução
- **Dependências circulares**: Detectadas e reportadas com erro
- **Múltiplas FKs**: Todas são detectadas e consideradas
- **Prefixos customizados**: Sistema tenta múltiplas variações

### Use Cases

**1. Dados de Referência:**
```bash
# Países, estados, cidades
cap carga from-csv -i db/carga/Countries.csv -t Countries --add-to-master
cap carga from-csv -i db/carga/States.csv -t States --add-to-master
cap carga from-csv -i db/carga/Cities.csv -t Cities --add-to-master
cap carga update --defaults db/changelog/liquibase.properties
```

**2. Configurações do Sistema:**
```bash
# Configurações, permissões, roles
cap carga from-csv -i db/carga/SystemConfig.csv -t SystemConfig --add-to-master
cap carga from-csv -i db/carga/Roles.csv -t Roles --add-to-master
cap carga update --defaults db/changelog/liquibase.properties
```

**3. Dados de Teste:**
```bash
# Ambiente de desenvolvimento
cap carga from-csv -i db/carga/TestUsers.csv -t Users --context dev
cap carga update --defaults db/changelog/liquibase-dev.properties
```

---

## 👤 Sistema de Autores

### Detecção Automática (ordem de prioridade)

1. **Parâmetro `--author`** (prioridade máxima)
2. **Variável `CAPY_AUTHOR`** (configuração específica)
3. **Git user.name** (configuração local)
4. **Variáveis do sistema** (`GIT_AUTHOR_NAME`, `USER`, `USERNAME`, `LOGNAME`)
5. **Fallback** (`capydb`)

### Configuração por Ambiente

#### Desenvolvimento Local

```bash
# Configurar Git (detectado automaticamente)
git config user.name "João Silva"

# Ou usar variável específica
export CAPY_AUTHOR="João Silva"  # Linux/Mac
set CAPY_AUTHOR=João Silva       # Windows
```

#### CI/CD Pipelines

**GitHub Actions:**
```yaml
env:
  CAPY_AUTHOR: "${{ github.actor }}"  # Nome do usuário que fez commit
```

**Azure DevOps:**
```yaml
variables:
  CAPY_AUTHOR: "$(Build.RequestedFor)"  # Nome do usuário
```

**Jenkins:**
```groovy
environment {
    CAPY_AUTHOR = "${env.BUILD_USER}"
}
```

### Exemplos Práticos

```bash
# Detecção automática (usa Git)
cap migrations add criar-produtos
# author: João Silva (do Git)

# Autor específico
cap migrations add criar-produtos --author "Maria Santos"
# author: Maria Santos

# Via variável de ambiente
export CAPY_AUTHOR="CI/CD Pipeline"
cap migrations add criar-produtos
# author: CI/CD Pipeline

# Import EF com autor
cap migrations import-ef \
  --assembly MyApp.dll \
  --name CreateProducts \
  --provider sqlserver \
  --author "João Silva"
```

---

## ⚙️ Configuração Avançada

### Arquivo liquibase.properties

**Exemplo completo:**
```properties
# Configuração do banco
url=jdbc:sqlserver://localhost:1433;databaseName=MyApp;trustServerCertificate=true
username=sa
password=MyPassword123

# Configuração do changelog
changeLogFile=db/changelog/db.changelog-master.yaml

# Drivers (se usando CLI local)
classpath=db/drivers/mssql-jdbc-12.4.2.jre11.jar

# Configurações extras
logLevel=INFO
contexts=common,production
labels=!test
```

### Estrutura de Projeto Recomendada

```
projeto/
├── db/
│   ├── changelog/
│   │   ├── common/              # Migrations de schema portáveis
│   │   │   ├── 20250101_120000__initial.yaml
│   │   │   └── 20250102_143000__add-users.yaml
│   │   ├── carga-updates/       # Migrations de data seeding
│   │   │   ├── 20251017__carga-countries.yaml
│   │   │   └── 20251017__carga-cities.yaml
│   │   ├── mssql/               # Específico SQL Server
│   │   ├── postgres/            # Específico PostgreSQL
│   │   ├── mysql/               # Específico MySQL
│   │   ├── archive/             # Arquivos após squash
│   │   ├── deleteSchemas/       # Arquivos após merge
│   │   ├── db.changelog-master.yaml
│   │   └── liquibase.properties
│   ├── carga/                   # Arquivos CSV para data seeding
│   │   ├── Countries.csv
│   │   ├── Cities.csv
│   │   └── Users.csv
│   └── drivers/                 # JARs dos drivers JDBC
│       ├── mssql-jdbc-12.4.2.jre11.jar
│       ├── postgresql-42.7.0.jar
│       └── mysql-connector-j-8.2.0.jar
├── src/
└── README.md
```

### Configuração de Docker

**docker-compose.yml para desenvolvimento:**
```yaml
version: '3.8'
services:
  capy-migrations:
    image: liquibase/liquibase:latest
    volumes:
      - ./db/changelog:/liquibase/changelog
      - ./db/drivers:/liquibase/drivers
    environment:
      - LIQUIBASE_COMMAND_URL=jdbc:sqlserver://sqlserver:1433;databaseName=MyApp
      - LIQUIBASE_COMMAND_USERNAME=sa
      - LIQUIBASE_COMMAND_PASSWORD=MyPassword123
      - LIQUIBASE_COMMAND_CHANGELOG_FILE=changelog/db.changelog-master.yaml
```

**Uso com Docker:**
```bash
# Todos os comandos suportam --docker
cap status --defaults ./db/changelog/liquibase.properties --docker
cap apply --defaults ./db/changelog/liquibase.properties --docker
cap plan --defaults ./db/changelog/liquibase.properties --docker --output plan.sql
```

---

## 🐛 Troubleshooting

### Problemas Comuns

#### 1. Emojis não aparecem no terminal

**Solução automática**: O CapyDb CLI agora configura UTF-8 automaticamente

**Solução manual (Windows):**
```cmd
# No Command Prompt
chcp 65001

# No PowerShell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
```

#### 2. Java não encontrado

```bash
# Verificar se Java está instalado
java -version

# Se não estiver, instalar OpenJDK 8+
# Windows: https://adoptium.net/
# Linux: sudo apt install openjdk-11-jre
# Mac: brew install openjdk@11
```

#### 3. Liquibase não encontrado

**Opção 1: Usar Docker (recomendado)**
```bash
# Instalar Docker Desktop
# Usar --docker em todos os comandos
cap doctor --docker
```

**Opção 2: Instalar Liquibase CLI**
```bash
# Windows (Chocolatey)
choco install liquibase

# Mac (Homebrew)
brew install liquibase

# Linux
# Baixar de https://www.liquibase.org/download
```

#### 4. Drivers JDBC não encontrados

```bash
# Criar diretório
mkdir -p db/drivers

# Baixar drivers necessários:
# SQL Server: https://docs.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server
# PostgreSQL: https://jdbc.postgresql.org/download.html
# MySQL: https://dev.mysql.com/downloads/connector/j/

# Copiar .jar para db/drivers/
cp mssql-jdbc-*.jar db/drivers/
```

#### 5. Erro de conexão com banco

```bash
# Verificar conectividade
cap doctor --defaults ./db/changelog/liquibase.properties

# Verificar configurações no liquibase.properties
# - url correto
# - username/password válidos
# - banco existe
# - firewall permite conexão
```

#### 6. Migration duplicada

```bash
# Verificar conflitos no master
cat db/changelog/db.changelog-master.yaml

# Se necessário, usar merge
cap migrations mergeschemas --scope common
```

### Logs e Debug

#### Habilitar modo debug

```bash
# Variável de ambiente para debug detalhado
export CAPY_DEBUG=true  # Linux/Mac
set CAPY_DEBUG=true     # Windows

# Executar comando
cap plan --defaults ./db/changelog/liquibase.properties
```

**Output com debug:**
```
[DEBUG] Command: liquibase --defaultsFile="./db/changelog/liquibase.properties" updateSQL
[DEBUG] WorkDir: /caminho/do/projeto
[DEBUG] ExitCode: 0

-- Liquibase output aqui --
```

#### Verificar logs do Liquibase

```bash
# Salvar output em arquivo
cap apply --defaults ./db/changelog/liquibase.properties --output apply.log

# Ver logs detalhados
cap validate --defaults ./db/changelog/liquibase.properties --output validation.log
```

---

## 📚 Exemplos de Workflows

### Workflow de Desenvolvimento

```bash
# 1. Criar nova migration
cap migrations add adicionar-tabela-produtos --author "João Silva"

# 2. Editar arquivo gerado
# db/changelog/common/20250923_120000__adicionar-tabela-produtos.yaml

# 3. Verificar plano
cap plan --defaults ./db/changelog/liquibase.properties --output plan.sql

# 4. Aplicar em desenvolvimento
cap apply --defaults ./db/changelog/liquibase.properties

# 5. Verificar status
cap status --defaults ./db/changelog/liquibase.properties

# 6. Criar tag para release
cap tag v1.2.0 --defaults ./db/changelog/liquibase.properties

# 7. Se necessário, remover tag antiga
cap remove-tag v1.1.0 --defaults ./db/changelog/liquibase.properties
```

### Workflow de CI/CD

```bash
# Em build pipeline
export CAPY_AUTHOR="$CI_COMMIT_AUTHOR"

# Verificar pré-requisitos
cap doctor --defaults ./db/changelog/liquibase.properties

# Gerar plano para review
cap plan --defaults ./db/changelog/liquibase.properties --output artifacts/plan.sql

# Em deploy pipeline
cap apply --defaults ./db/changelog/liquibase.properties

# Criar tag de deploy
cap tag "deploy-$(date +%Y%m%d-%H%M%S)" --defaults ./db/changelog/liquibase.properties
```

### Workflow de Migração de EF Core

```bash
# 1. Compilar projeto EF
dotnet build MyApp.sln

# 2. Importar migration específica
cap migrations import-ef \
  --assembly ./MyApp/bin/Debug/net8.0/MyApp.dll \
  --name AddProductsTable \
  --provider sqlserver \
  --author "Maria Santos"

# 3. Verificar resultado
cat db/changelog/common/20250923_*__addproductstable.yaml

# 4. Testar em desenvolvimento
cap apply --defaults ./db/changelog/liquibase.properties
```

### Workflow de Conversão de Dados

```bash
# 1. Exportar dados de tabela existente (exemplo PostgreSQL)
psql -d mydb -c "COPY usuarios TO '/tmp/usuarios.sql' WITH (FORMAT text)"

# 2. Ou criar arquivo SQL manualmente
cat > seed-data.sql << EOF
INSERT INTO usuarios (id, nome, email) VALUES (1, 'Admin', 'admin@email.com');
INSERT INTO usuarios (id, nome, email) VALUES (2, 'User', 'user@email.com');
EOF

# 3. Converter para formato Liquibase
cap convert-inserts \
  --input ./seed-data.sql \
  --output ./db/changelog/common/20250930__seed-usuarios.yaml \
  --author "Sistema"

# 4. Adicionar ao master
# (editar db.changelog-master.yaml)

# 5. Aplicar seed data
cap apply --defaults ./db/changelog/liquibase.properties

# 6. Verificar dados
cap status --defaults ./db/changelog/liquibase.properties
```

---

## 🎯 Melhores Práticas

### Naming Conventions

- **Migrations**: Use kebab-case (`criar-tabela-usuarios`)
- **Tags**: Use semantic versioning (`v1.2.0`) ou timestamp (`deploy-20250923-1200`)
- **Autores**: Nome completo ou username consistente

### Organização de Arquivos

- **Common**: Mudanças portáveis entre SGBDs
- **Específicos**: SQL específico por banco quando necessário
- **Archive**: Manter histórico após squash
- **Drivers**: Versões compatíveis e atualizadas

### Segurança

- **Nunca** commitar senhas em `liquibase.properties`
- Usar variáveis de ambiente para credenciais
- Configurar `.gitignore` adequadamente:

```gitignore
# Credentials
db/changelog/liquibase.properties
*.secret

# Temporary files
*.log
plan*.sql
drift-report*.xml
```

### Performance

- Fazer **squash** periodicamente para manter histórico limpo
- Usar **contexts** e **labels** para ambientes específicos
- Monitorar tamanho do changelog master

---

**Documentação gerada para CapyDb CLI v1.2.3**
*Última atualização: 2025-10-27*

## 🆕 Novidades na v1.2.3

### Data Seeding e Gerenciamento de Dados
- ✅ **Comando `cap carga from-csv`** - Geração automática de changesets a partir de CSV
- ✅ **Inferência Inteligente de Tipos** - Detecta automaticamente UUID, BOOLEAN, TIMESTAMP, NUMERIC, STRING
- ✅ **Resolução Automática de Dependências** - Detecta FKs dos cabeçalhos CSV e ordena tabelas automaticamente
- ✅ **Ordenação Topológica** - Ordenação inteligente previne violações de FK durante carga de dados
- ✅ **Detecção de Dependências Circulares** - Alerta sobre referências circulares entre tabelas
- ✅ **Comandos de Gerenciamento** - update, drop-all, reset, rollback, status para dados
- ✅ **Filtragem por Label** - Usa label `data-seed` para operações específicas
- ✅ **Auto-add to Master** - Opção para adicionar automaticamente ao master changelog

### Melhorias no Pacote
- ✅ **Dependências Embutidas** - CapyDb.Core, Runner e Writers agora fazem parte do CLI
- ✅ **Pacote NuGet Limpo** - Sem dependências externas listadas
- ✅ **Multi-target Completo** - Suporte total para .NET 8.0 e .NET 9.0

### Versões Anteriores (v1.0.7)
- ✅ **Busca Recursiva Aprimorada** - O sistema agora busca `liquibase.properties` em múltiplos locais automaticamente
- ✅ **Suporte Completo a Windows** - Corrigidos problemas com padrões glob no Windows
- ✅ **Suporte a Monorepos** - Funciona perfeitamente com estruturas complexas (`apps/*/`, `src/*/`)
- ✅ **Detecção Inteligente de Assemblies** - Melhor suporte para EF Core em projetos grandes
- ✅ **Multiplataforma** - Testado e validado no Windows, Linux e macOS

---

## 📑 Resumo de Comandos

### Migrations
- `cap migrations add <nome>` - Criar nova migration
- `cap migrations import-ef` - Importar do EF Core
- `cap migrations mergeschemas` - Consolidar migrations

### Operações de Banco
- `cap plan` - Gerar plano SQL
- `cap apply` - Aplicar migrations
- `cap status` - Ver status
- `cap validate` - Validar changelog
- `cap tag <nome>` - Criar tag
- `cap remove-tag <tag>` - Remover tag
- `cap rollback count <N>` - Reverter N migrations
- `cap rollback to-tag <tag>` - Reverter até tag

### Data Seeding (Carga de Dados)
- `cap carga from-csv` - Gerar changelog a partir de CSV
- `cap carga update` - Aplicar dados de seed (label: data-seed)
- `cap carga drop-all` - Remover e reaplicar dados
- `cap carga reset` - Reset completo (schema + dados)
- `cap carga rollback` - Reverter último changeset de dados
- `cap carga status` - Ver status dos dados de seed

### Utilitários
- `cap doctor` - Verificar pré-requisitos
- `cap drift detect` - Detectar divergências
- `cap squash --tag <tag>` - Consolidar histórico
- `cap convert-inserts` - Converter INSERTs SQL
- `cap bye` - Despedida