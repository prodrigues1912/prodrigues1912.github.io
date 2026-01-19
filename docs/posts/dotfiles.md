---
date: 2099-01-01
draft: true
categories:
  - DevOps
  - Terminal
---

# Dotfiles: como organizo minha configuração

Dotfiles são os arquivos de configuração no seu sistema (`.bashrc`, `.gitconfig`, `.vimrc`, etc). Versionar e organizar esses arquivos permite replicar seu ambiente em qualquer máquina.

<!-- more -->

## Por que versionar dotfiles?

- **Backup** - Nunca mais perca suas configurações
- **Portabilidade** - Setup novo em minutos
- **Histórico** - Reverta mudanças que quebraram algo
- **Compartilhamento** - Outros podem aprender com sua config

## Estrutura do repositório

```
dotfiles/
├── fish/
│   └── config.fish
├── git/
│   ├── config
│   └── ignore
├── nvim/
│   └── init.lua
├── tmux/
│   └── tmux.conf
├── starship/
│   └── starship.toml
├── scripts/
│   └── bootstrap.sh
└── README.md
```

Organizo por aplicação, não por localização final.

## GNU Stow para symlinks

[Stow](https://www.gnu.org/software/stow/){:target="_blank"} cria symlinks automaticamente. É a forma mais elegante de gerenciar dotfiles.

### Instalação

```bash
# macOS
brew install stow

# Ubuntu/Debian (incluindo WSL)
sudo apt install stow

# Arch Linux (WSL)
sudo pacman -S stow
```

**Nota:** Stow não funciona nativamente no Windows. Para Windows, use symlinks manuais ou gerencie dotfiles separadamente (veja seção "Dotfiles no Windows" abaixo).

### Como funciona

A estrutura dentro de cada pasta espelha onde os arquivos devem ir em `$HOME`:

```
dotfiles/
├── fish/
│   └── .config/
│       └── fish/
│           └── config.fish    → ~/.config/fish/config.fish
├── git/
│   └── .gitconfig             → ~/.gitconfig
└── nvim/
    └── .config/
        └── nvim/
            └── init.lua       → ~/.config/nvim/init.lua
```

### Comandos

```bash
cd ~/dotfiles

# Criar symlinks para fish
stow fish

# Criar para todos
stow */

# Remover symlinks
stow -D fish

# Simular (dry run)
stow -n fish
```

## Bootstrap script

Script para configurar máquina nova:

```bash
#!/bin/bash
# scripts/bootstrap.sh

set -e

echo "==> Instalando Homebrew..."
if ! command -v brew &> /dev/null; then
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
fi

echo "==> Instalando dependências..."
brew bundle --file=~/dotfiles/Brewfile

echo "==> Configurando Fish como shell padrão..."
if ! grep -q "$(which fish)" /etc/shells; then
    echo "$(which fish)" | sudo tee -a /etc/shells
fi
chsh -s "$(which fish)"

echo "==> Criando symlinks..."
cd ~/dotfiles
stow fish git nvim tmux starship

echo "==> Instalando plugins do Neovim..."
nvim --headless "+Lazy! sync" +qa

echo "==> Pronto!"
```

## Brewfile

Lista de pacotes para instalar:

```ruby
# Brewfile

# Taps
tap "homebrew/bundle"

# CLI tools
brew "fish"
brew "neovim"
brew "tmux"
brew "starship"
brew "stow"
brew "git"
brew "ripgrep"
brew "fd"
brew "fzf"
brew "bat"
brew "eza"
brew "jq"

# Development
brew "uv"
brew "node"
brew "go"

# Apps (casks)
cask "wezterm"
cask "raycast"
cask "1password"
```

Instale tudo com:

```bash
brew bundle --file=~/dotfiles/Brewfile
```

## Configurações exemplo

### Git

```gitconfig
# git/config → ~/.gitconfig

[user]
    name = Paulo Rodrigues
    email = seu@email.com

[init]
    defaultBranch = main

[pull]
    rebase = true

[push]
    autoSetupRemote = true

[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
    lg = log --oneline -10

[core]
    excludesFile = ~/.gitignore
    editor = nvim

[diff]
    tool = difftastic

[merge]
    conflictStyle = diff3
```

### Fish

```fish
# fish/.config/fish/config.fish

# Sem greeting
set -g fish_greeting

# Editor
set -gx EDITOR nvim

# Path
fish_add_path ~/.local/bin

# Aliases
alias ll "eza -la"
alias g git
alias v nvim
alias k kubectl

# Starship prompt
starship init fish | source

# FZF
fzf --fish | source
```

### Starship

```toml
# starship/starship.toml → ~/.config/starship.toml

format = """
$directory\
$git_branch\
$git_status\
$python\
$nodejs\
$line_break\
$character"""

[directory]
truncation_length = 3
truncate_to_repo = true

[git_branch]
symbol = " "

[python]
symbol = " "

[nodejs]
symbol = " "

[character]
success_symbol = "[❯](bold green)"
error_symbol = "[❯](bold red)"
```

## Dicas

### 1. Comece simples

Não copie dotfiles de outros sem entender. Comece com o mínimo e adicione conforme necessário.

### 2. Documente

Adicione comentários explicando configurações não-óbvias:

```bash
# Desabilita bell visual (irritante)
set -g visual-bell off
```

### 3. Secrets separados

Nunca commite tokens ou senhas. Use arquivos locais:

```fish
# config.fish
if test -f ~/.config/fish/local.fish
    source ~/.config/fish/local.fish
end
```

```fish
# local.fish (não versionado)
set -gx GITHUB_TOKEN "ghp_xxx"
```

### 4. Máquinas diferentes

Use condicionais para diferenças entre máquinas:

```fish
# Só no Mac
if test (uname) = "Darwin"
    alias chrome "open -a 'Google Chrome'"
end

# Só em máquina específica
if test (hostname) = "work-laptop"
    set -gx HTTP_PROXY "http://proxy.empresa.com:8080"
end
```

### 5. README

Documente como usar seu repositório:

```markdown
# Dotfiles

Minhas configurações pessoais.

## Setup

1. Clone o repositório
2. Execute o bootstrap
3. Pronto

## Conteúdo

- fish: Shell config
- nvim: Neovim config
- ...
```

## Estrutura alternativa (sem Stow)

Se preferir não usar Stow, symlinks manuais:

```bash
# setup.sh
ln -sf ~/dotfiles/fish/config.fish ~/.config/fish/config.fish
ln -sf ~/dotfiles/git/config ~/.gitconfig
ln -sf ~/dotfiles/nvim ~/.config/nvim
```

Funciona, mas é mais trabalhoso manter.

## Dotfiles no Windows

No Windows, o conceito de dotfiles é diferente. Os principais arquivos de configuração:

### PowerShell Profile

```powershell
# Localização do profile
$PROFILE  # Mostra o caminho

# Geralmente em:
# ~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
```

Exemplo de profile:

```powershell
# Microsoft.PowerShell_profile.ps1

# Aliases
Set-Alias -Name g -Value git
Set-Alias -Name k -Value kubectl
Set-Alias -Name v -Value nvim

# Prompt (usando Starship)
Invoke-Expression (&starship init powershell)

# Funções úteis
function ll { Get-ChildItem -Force }
function mkcd { param($dir) mkdir $dir; cd $dir }
```

### Symlinks no Windows

```powershell
# PowerShell (como Admin)
New-Item -ItemType SymbolicLink -Path "$HOME\.gitconfig" -Target "$HOME\dotfiles\git\config"
New-Item -ItemType SymbolicLink -Path "$HOME\AppData\Local\nvim" -Target "$HOME\dotfiles\nvim"
```

### WSL para o resto

Para Fish, tmux e outras ferramentas Unix, use WSL e gerencie os dotfiles normalmente dentro do ambiente Linux.

## Conclusão

Dotfiles versionados são um investimento que paga dividendos. Na próxima formatação ou máquina nova, você terá seu ambiente pronto em minutos.

Links úteis:

- [Stow](https://www.gnu.org/software/stow/)
- [GitHub ❤ ~/dotfiles](https://dotfiles.github.io/)
- [Awesome dotfiles](https://github.com/webpro/awesome-dotfiles)
