
# Docs Starter — MkDocs Material (com Docker)

Um pacote mínimo para você rodar suas documentações **localmente** mesmo que seu Git seja privado.

## ✅ O que vem pronto
- **MkDocs + Material** já configurado
- **Docker Compose** para rodar sem instalar Python
- Estrutura `docs/` com exemplos
- Script para **copiar** seus `.md` de um repositório privado local
- Pipeline exemplo do **Azure DevOps** para publicar (opcional)

---

## 1) Rodar localmente com Docker (recomendado)
Pré‑requisito: [Docker Desktop]

```bash
# na pasta do projeto (onde está este README.md)
docker compose up
```

Acesse: http://localhost:8000 (auto reload).  
O MkDocs vai ler `./docs` e `./mkdocs.yml`.

> Dica: se seus Markdown já estão em outro repo/pasta, use o script abaixo para copiar para `./docs` ou monte a pasta com um volume no compose.

---

## 2) Copiar seus .md de um repositório privado local
Se você já tem os `.md` no seu monorepo (ex.: `C:\dev\sisprevmais-monorepo\docs`), rode:

```powershell
# PowerShell
.\scripts\copy-docs-from-repo.ps1 -Source "C:\dev\sisprevmais-monorepo\docs"
```

Isso **sincroniza** (copia por cima) os arquivos para `./docs`.

> O script NÃO faz `git clone` nem pede token; ele apenas copia de uma pasta local já clonada/privada.

---

## 3) Rodar sem Docker (via Python)
Pré‑requisito: Python 3.11+

```bash
pip install -r requirements.txt
mkdocs serve -a 0.0.0.0:8000
```

---

## 4) Publicar (opcional) — Azure DevOps Pipeline
Exemplo mínimo para buildar o site estático como artefato:

```yaml
# .azure-pipelines/mkdocs-ci.yml
trigger: none

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.11'
- script: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
    mkdocs build --strict
  displayName: 'Build MkDocs'
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: 'site'
    ArtifactName: 'mkdocs-site'
    publishLocation: 'Container'
```

Você pode depois publicar o conteúdo do artefato `site/` em qualquer host estático (Azure Static Web Apps, Storage + CDN, Nginx interno, etc.).

---

## 5) Personalizar a navegação
- Edite `mkdocs.yml` (seções `nav` e `theme`).
- Acrescente seus `.md` em `docs/` (ou subpastas).
- Se quiser extrair docs direto do monorepo, você pode:
  - **(A)** Copiar com o script acima, ou
  - **(B)** Montar a pasta do monorepo no Docker Compose (ver comentário dentro do `docker-compose.yml`).

---

## 6) Estrutura
```
docs-starter-mkdocs-material/
├─ docs/
│  ├─ index.md
│  └─ guia.md
├─ scripts/
│  └─ copy-docs-from-repo.ps1
├─ docker-compose.yml
├─ mkdocs.yml
├─ requirements.txt
└─ README.md
```

---

## 7) Dicas
- Se seus `.md` estão espalhados pelo monorepo, considere centralizar em uma pasta `docs/`.
- Para docs multilíngue, use subpastas `pt-br/`, `en/`, etc.
- Para coleções grandes, você pode explorar plugins como `search` (já incluído) e hierarquia via `nav`.

Boa documentação! 🦫
