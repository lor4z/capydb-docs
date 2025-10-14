
# CapyDb Docs — Bilingual + Theme Toggle (works out of the box)

## What you get
- 🇧🇷 **pt-BR** and 🇬🇧 **English** versions (folder-based under `docs/` and `docs/en/`)
- 🔦 Built-in **light/dark** theme toggle
- 🔎 Search per language, working table of contents (anchors)
- 🐳 Docker Compose that installs the i18n plugin automatically and serves

## How to run
```bash
docker compose up
```
Open: http://localhost:8000  
Use the language switcher in the header. Toggle light/dark in the header as well.

## Structure
```
.
├─ docs/
│  ├─ README.md                 # pt-BR
│  ├─ CAPY-CLI-GUIDE.md        # pt-BR
│  ├─ SETUP-INICIAL.md         # pt-BR
│  ├─ DEVELOPMENT.md           # pt-BR
│  └─ en/
│     ├─ README.md             # en
│     ├─ CAPY-CLI-GUIDE.md     # en
│     ├─ SETUP-INICIAL.md      # en
│     └─ DEVELOPMENT.md        # en
├─ mkdocs.yml
└─ docker-compose.yml
```

## Notes
- Keep filenames identical between `docs/` and `docs/en/` so i18n matches pages.
- Edit the menu in `mkdocs.yml` > `nav`. The translations for menu labels are under `plugins.i18n.nav_translations`.
- Anchors (index links) use the heading text. Example: a section `## My Section` is reachable as `[My Section](#my-section)` on the same page.
