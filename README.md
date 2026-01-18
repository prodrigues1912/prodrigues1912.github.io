# paulorodrigues.blog

Blog pessoal. Notas e tutoriais sobre tecnologia e outros interesses.

## Desenvolvimento

Requisitos: Python 3.13+ e [uv](https://docs.astral.sh/uv/)

```bash
uv sync                 # instalar dependências
uv run mkdocs serve     # rodar servidor local
```

O site estará disponível em http://localhost:8000

## Tecnologias

- [MkDocs](https://www.mkdocs.org/) + [Material](https://squidfunk.github.io/mkdocs-material/)
- [GitHub Pages](https://pages.github.com/)
- [uv](https://docs.astral.sh/uv/)

## Deploy

Automático via GitHub Actions ao fazer push na `main`.
