---
date: 2024-12-08
categories:
  - Python
  - Ferramentas
---

# uv: gerenciador de pacotes Python

[uv](https://docs.astral.sh/uv/) é um gerenciador de pacotes e projetos Python da Astral (mesma do Ruff). Escrito em Rust, substitui pyenv + pip + venv + poetry numa ferramenta só.

<!-- more -->

## Instalação

```bash
# macOS
brew install uv
```

```bash
# Linux/macOS (script)
curl -LsSf https://astral.sh/uv/install.sh | sh
```

```powershell
# Windows
winget install astral-sh.uv
# Ou: irm https://astral.sh/uv/install.ps1 | iex
```

Se instalar via script, adicionar ao PATH:

```bash
# Fish
fish_add_path ~/.local/bin
```

```bash
# Bash/Zsh
export PATH="${HOME}/.local/bin:$PATH"
```

## Comandos essenciais

```bash
# Novo projeto
uv init meu-projeto && cd meu-projeto

# Dependências
uv add requests
uv add pytest --dev

# Sincronizar ambiente
uv sync

# Executar no venv (sem ativar)
uv run python script.py
uv run pytest

# Gerenciar versões Python
uv python install 3.12
uv python list
```

## CLIs globais (substitui pipx)

```bash
uv tool install ruff
```

## Migração

```bash
# De requirements.txt
uv init
uv add -r requirements.txt

# De poetry - uv lê pyproject.toml direto
uv sync
```

## Compatibilidade pip

```bash
# Quando precisar de comandos pip tradicionais
uv pip install pacote
uv pip freeze
```

## Links

- [Documentação](https://docs.astral.sh/uv/)
- [GitHub](https://github.com/astral-sh/uv)
