---
date: 2025-02-23
categories:
  - Git
  - Ferramentas
---

# Git worktrees

[Git worktrees](https://git-scm.com/docs/git-worktree) permite ter múltiplas branches em diretórios separados, compartilhando o mesmo `.git`. Útil para reviews, hotfixes ou experimentos sem interromper o trabalho atual.

<!-- more -->

## Conceito

Um worktree é um diretório adicional linkado ao mesmo repositório. Cada um pode estar em uma branch diferente.

```bash
# Criar worktree para revisar PR
git worktree add ../meu-repo-review feature/nova-api

# Resultado:
# ~/projetos/meu-repo/         (branch: sua-feature)
# ~/projetos/meu-repo-review/  (branch: feature/nova-api)
```

Ambos compartilham o `.git`: commits e branches ficam sincronizados.

## Comandos

```bash
# Criar worktree de branch existente
git worktree add ../path branch-name

# Criar worktree com nova branch
git worktree add -b nova-branch ../path main

# Listar worktrees
git worktree list

# Remover worktree
git worktree remove ../path

# Forçar remoção (se tiver mudanças)
git worktree remove --force ../path

# Limpar worktrees órfãos
git worktree prune
```

## Exemplo: revisão de PR

```bash
git fetch origin
git worktree add ../projeto-review origin/feature-x

cd ../projeto-review
# revisar, testar

cd ../projeto
git worktree remove ../projeto-review
```

## Exemplo: hotfix

```bash
git worktree add -b hotfix/bug-123 ../projeto-hotfix main

cd ../projeto-hotfix
# corrigir
git commit -am "fix: corrige bug 123"
git push -u origin hotfix/bug-123

cd ../projeto
git worktree remove ../projeto-hotfix
```

## Dicas

- Use sufixos consistentes: `-review`, `-hotfix`, `-experiment`
- Mantenha worktrees no mesmo nível do repo principal
- Remova worktrees quando terminar
- Cada worktree pode ser projeto separado na IDE

## Limitações

Uma branch só pode estar em um worktree por vez. Submodules precisam de atenção extra.

## Links

- [Documentação](https://git-scm.com/docs/git-worktree)
