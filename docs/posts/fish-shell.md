---
date: 2025-02-09
categories:
  - Ferramentas
  - Terminal
---

# Fish shell: configuração mínima

[Fish](https://fishshell.com/) é um shell com autosuggestions e syntax highlighting nativos. Não é POSIX-compliant, mas na prática isso raramente é problema.

<!-- more -->

## Instalação

```bash
# macOS
brew install fish

# Ubuntu/Debian (incluindo WSL)
sudo apt install fish

# Arch Linux
sudo pacman -S fish
```

Definir como shell padrão:

```bash
echo $(which fish) | sudo tee -a /etc/shells
chsh -s $(which fish)
```

**Nota:** Fish não roda nativamente no Windows. Uso via WSL.

## Configuração básica

```bash
# ~/.config/fish/config.fish

# Sem greeting
set -g fish_greeting

# Editor
set -gx EDITOR nvim

# Path
fish_add_path ~/.local/bin

# Abbreviations (expandem ao digitar, melhor que aliases)
abbr -a g git
abbr -a k kubectl
abbr -a gs "git status"
abbr -a gc "git commit"
abbr -a gp "git push"

# Aliases (para comandos que não quero expandir)
alias ll "ls -la"
```

## Configuração via browser

```bash
fish_config
```

Abre interface web para configurar cores, prompt, funções e abbreviations.

## Funções

Funções ficam em `~/.config/fish/functions/`:

```bash
# ~/.config/fish/functions/mkcd.fish
function mkcd
    mkdir -p $argv[1] && cd $argv[1]
end
```

```bash
# ~/.config/fish/functions/serve.fish
function serve
    python -m http.server $argv[1]
end
```

## Links

- [Fish Shell](https://fishshell.com/)