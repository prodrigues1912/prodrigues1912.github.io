---
date: 2025-03-10
categories:
  - DevOps
  - Docker
---

# Docker no Mac sem Docker Desktop

[Colima](https://github.com/abiosoft/colima) é uma alternativa open source ao Docker Desktop. Gratuito, leve e funciona igual.

<!-- more -->

## Instalação

```bash
brew install colima docker docker-compose

colima start
docker ps  # verificar se está funcionando
```

Por padrão: 2 CPUs, 2GB RAM, 60GB disco.

## Configuração

```bash
# Mais recursos
colima start --cpu 4 --memory 8 --disk 100

# Editar config permanente
colima start --edit
```

### Docker socket

Colima configura automaticamente. Se precisar definir manualmente:

```bash
# Fish
set -gx DOCKER_HOST "unix://$HOME/.colima/default/docker.sock"
```

```bash
# Bash/Zsh
export DOCKER_HOST="unix://${HOME}/.colima/default/docker.sock"
```

## Uso diário

```bash
colima start
colima stop
colima status
```

Colima não inicia com o Mac. Iniciar manualmente quando precisar.

## docker compose

```bash
docker compose up -d
docker compose logs -f
docker compose down
```

## Kubernetes (opcional)

```bash
colima start --kubernetes
```

Cluster single-node acessível via `kubectl`.

## Troubleshooting

```bash
# Verificar status
colima status

# Reiniciar
colima restart

# Verificar socket
ls -la ~/.colima/default/docker.sock

# Deletar VM e começar do zero
colima delete

# Limpar imagens/containers
docker system prune -a
```

### Múltiplos profiles

```bash
colima start --profile work
export DOCKER_HOST="unix://${HOME}/.colima/work/docker.sock"
```

## Links

- [Colima](https://github.com/abiosoft/colima){:target="_blank"}
- [Lima](https://github.com/lima-vm/lima){:target="_blank"}
