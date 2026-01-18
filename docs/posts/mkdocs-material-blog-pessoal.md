---
date: 2025-01-12
categories:
  - Ferramentas
  - Tutorial
---

# MkDocs Material para blog pessoal

[MkDocs](https://www.mkdocs.org/) com tema [Material](https://squidfunk.github.io/mkdocs-material/) é uma opção simples para blogs técnicos.

<!-- more -->

## Setup

```bash
uv init meu-blog && cd meu-blog
uv add mkdocs-material
```

## Estrutura

```
meu-blog/
├── docs/
│   ├── index.md          # página inicial
│   └── posts/
│       └── meu-post.md   # posts do blog
├── mkdocs.yml
└── pyproject.toml
```

## Configuração

```yaml
# mkdocs.yml
site_name: Meu Blog
site_url: https://meublog.com

theme:
  name: material
  features:
    - navigation.instant
    - navigation.tracking
    - search.suggest
    - content.code.copy
  palette:
    - scheme: default
      primary: black
      toggle:
        icon: material/brightness-7
        name: Modo escuro
    - scheme: slate
      primary: black
      toggle:
        icon: material/brightness-4
        name: Modo claro

plugins:
  - search:
      lang: pt
  - blog:
      blog_dir: .              # posts na raiz (não em /blog/)
      post_date_format: long
      archive_name: Arquivo
      categories_name: Categorias

markdown_extensions:
  - pymdownx.highlight:
      anchor_linenums: true
  - pymdownx.inlinehilite
  - pymdownx.superfences
  - admonition
  - toc:
      permalink: true
  - tables
```

## Posts

Cada post precisa de frontmatter com data:

```markdown
---
date: 2025-01-12
categories:
  - Tutorial
---

# Título do Post

Primeiro parágrafo aparece como resumo na listagem.

<!-- more -->

Resto do conteúdo...
```

O `<!-- more -->` controla onde corta o excerpt na página inicial.

## Desenvolvimento local

```bash
uv run mkdocs serve
# Abre em http://localhost:8000
```

## Deploy GitHub Pages

```yaml
# .github/workflows/ci.yml
name: ci
on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync
      - run: uv run mkdocs gh-deploy --force
```

### Domínio customizado

Criar arquivo `docs/CNAME`:

```
meublog.com
```

## Links

- [MkDocs](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [Blog plugin](https://squidfunk.github.io/mkdocs-material/plugins/blog/)
