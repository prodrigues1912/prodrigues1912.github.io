---
date: 2025-05-11
categories:
  - DevOps
  - Terraform
---

# Terraform: referência

Comandos, sintaxe e padrões para Terraform com exemplos para AWS, GCP e Azure.

<!-- more -->

## Instalação

```bash
# macOS
brew install terraform

# Linux
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Windows
winget install HashiCorp.Terraform
```

## Comandos essenciais

```bash
terraform init          # Inicializa, baixa providers
terraform plan          # Mostra mudanças
terraform apply         # Aplica mudanças
terraform destroy       # Remove tudo

terraform fmt           # Formata código
terraform validate      # Valida sintaxe
terraform output        # Mostra outputs
terraform state list    # Lista recursos no state
```

### Opções úteis

```bash
terraform plan -out=plan.tfplan    # Salva plano
terraform apply plan.tfplan        # Aplica plano salvo
terraform apply -auto-approve      # Sem confirmação
terraform apply -target=aws_instance.web  # Só um recurso
terraform destroy -target=aws_instance.web
```

## Estrutura de projeto

```
projeto/
├── main.tf           # Recursos principais
├── variables.tf      # Declaração de variáveis
├── outputs.tf        # Outputs
├── providers.tf      # Configuração de providers
├── terraform.tfvars  # Valores das variáveis (não commitar)
└── versions.tf       # Versões de providers
```

## Providers

=== "AWS"

    ```hcl
    terraform {
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
        }
      }
    }

    provider "aws" {
      region = "us-east-1"

      default_tags {
        tags = {
          ManagedBy = "terraform"
        }
      }
    }
    ```

=== "GCP"

    ```hcl
    terraform {
      required_providers {
        google = {
          source  = "hashicorp/google"
          version = "~> 5.0"
        }
      }
    }

    provider "google" {
      project = "meu-projeto"
      region  = "us-central1"
    }
    ```

=== "Azure"

    ```hcl
    terraform {
      required_providers {
        azurerm = {
          source  = "hashicorp/azurerm"
          version = "~> 3.0"
        }
      }
    }

    provider "azurerm" {
      features {}
      subscription_id = "xxx-xxx-xxx"
    }
    ```

## Variáveis

### Declaração

```hcl
variable "environment" {
  type        = string
  description = "Ambiente (dev, staging, prod)"
  default     = "dev"
}

variable "instance_count" {
  type    = number
  default = 1
}

variable "enable_monitoring" {
  type    = bool
  default = true
}

variable "allowed_cidrs" {
  type    = list(string)
  default = ["10.0.0.0/8"]
}

variable "tags" {
  type = map(string)
  default = {
    Team = "platform"
  }
}

variable "config" {
  type = object({
    name     = string
    size     = number
    enabled  = optional(bool, true)
  })
}
```

### Validação

```hcl
variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Deve ser dev, staging ou prod."
  }
}

variable "instance_type" {
  type = string
  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Deve ser da família t3."
  }
}
```

### Passando valores

```bash
# terraform.tfvars (automático)
environment = "prod"
instance_count = 3

# Linha de comando
terraform apply -var="environment=prod"
terraform apply -var-file="prod.tfvars"

# Variável de ambiente
export TF_VAR_environment=prod
```

## Locals

```hcl
locals {
  name_prefix = "${var.project}-${var.environment}"

  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
  }

  is_prod = var.environment == "prod"
}

resource "aws_instance" "web" {
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web"
  })
}
```

## Recursos básicos

=== "AWS"

    ```hcl
    # VPC
    resource "aws_vpc" "main" {
      cidr_block           = "10.0.0.0/16"
      enable_dns_hostnames = true
      tags = { Name = "main" }
    }

    # Subnet
    resource "aws_subnet" "public" {
      vpc_id                  = aws_vpc.main.id
      cidr_block              = "10.0.1.0/24"
      map_public_ip_on_launch = true
      availability_zone       = "us-east-1a"
    }

    # Security Group
    resource "aws_security_group" "web" {
      vpc_id = aws_vpc.main.id

      ingress {
        from_port   = 443
        to_port     = 443
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }

      egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }
    }

    # EC2
    resource "aws_instance" "web" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t3.micro"
      subnet_id     = aws_subnet.public.id

      tags = { Name = "web-server" }
    }

    # S3
    resource "aws_s3_bucket" "data" {
      bucket = "meu-bucket-unico"
    }

    resource "aws_s3_bucket_versioning" "data" {
      bucket = aws_s3_bucket.data.id
      versioning_configuration {
        status = "Enabled"
      }
    }
    ```

=== "GCP"

    ```hcl
    # VPC
    resource "google_compute_network" "main" {
      name                    = "main"
      auto_create_subnetworks = false
    }

    # Subnet
    resource "google_compute_subnetwork" "public" {
      name          = "public"
      network       = google_compute_network.main.id
      ip_cidr_range = "10.0.1.0/24"
      region        = "us-central1"
    }

    # Firewall
    resource "google_compute_firewall" "web" {
      name    = "allow-https"
      network = google_compute_network.main.name

      allow {
        protocol = "tcp"
        ports    = ["443"]
      }

      source_ranges = ["0.0.0.0/0"]
    }

    # VM
    resource "google_compute_instance" "web" {
      name         = "web-server"
      machine_type = "e2-micro"
      zone         = "us-central1-a"

      boot_disk {
        initialize_params {
          image = "debian-cloud/debian-11"
        }
      }

      network_interface {
        subnetwork = google_compute_subnetwork.public.id
        access_config {}  # IP público
      }
    }

    # Cloud Storage
    resource "google_storage_bucket" "data" {
      name     = "meu-bucket-unico"
      location = "US"

      versioning {
        enabled = true
      }
    }
    ```

=== "Azure"

    ```hcl
    # Resource Group
    resource "azurerm_resource_group" "main" {
      name     = "main-rg"
      location = "East US"
    }

    # VNet
    resource "azurerm_virtual_network" "main" {
      name                = "main-vnet"
      resource_group_name = azurerm_resource_group.main.name
      location            = azurerm_resource_group.main.location
      address_space       = ["10.0.0.0/16"]
    }

    # Subnet
    resource "azurerm_subnet" "public" {
      name                 = "public"
      resource_group_name  = azurerm_resource_group.main.name
      virtual_network_name = azurerm_virtual_network.main.name
      address_prefixes     = ["10.0.1.0/24"]
    }

    # NSG
    resource "azurerm_network_security_group" "web" {
      name                = "web-nsg"
      resource_group_name = azurerm_resource_group.main.name
      location            = azurerm_resource_group.main.location

      security_rule {
        name                       = "HTTPS"
        priority                   = 100
        direction                  = "Inbound"
        access                     = "Allow"
        protocol                   = "Tcp"
        destination_port_range     = "443"
        source_address_prefix      = "*"
        destination_address_prefix = "*"
        source_port_range          = "*"
      }
    }

    # Storage
    resource "azurerm_storage_account" "data" {
      name                     = "meustorageaccount"
      resource_group_name      = azurerm_resource_group.main.name
      location                 = azurerm_resource_group.main.location
      account_tier             = "Standard"
      account_replication_type = "LRS"
    }
    ```

## Loops e condicionais

### count

```hcl
resource "aws_instance" "web" {
  count = var.instance_count

  ami           = "ami-xxx"
  instance_type = "t3.micro"

  tags = {
    Name = "web-${count.index}"
  }
}

# Acesso
output "instance_ids" {
  value = aws_instance.web[*].id
}
```

### for_each (map)

```hcl
variable "instances" {
  default = {
    web  = "t3.micro"
    api  = "t3.small"
    worker = "t3.medium"
  }
}

resource "aws_instance" "app" {
  for_each = var.instances

  ami           = "ami-xxx"
  instance_type = each.value

  tags = {
    Name = each.key
  }
}

# Acesso
output "instance_ids" {
  value = { for k, v in aws_instance.app : k => v.id }
}
```

### for_each (set)

```hcl
resource "aws_iam_user" "users" {
  for_each = toset(["alice", "bob", "carol"])
  name     = each.key
}
```

### Condicional

```hcl
# Criar ou não
resource "aws_instance" "bastion" {
  count = var.environment == "prod" ? 1 : 0
  # ...
}

# Valor condicional
resource "aws_instance" "web" {
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
}
```

### for expression

```hcl
locals {
  # Lista
  upper_names = [for name in var.names : upper(name)]

  # Filtrada
  prod_instances = [for i in var.instances : i if i.env == "prod"]

  # Map
  instance_map = { for i in var.instances : i.name => i.id }
}
```

## Data sources

```hcl
# AMI mais recente
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "web" {
  ami = data.aws_ami.amazon_linux.id
  # ...
}

# VPC existente
data "aws_vpc" "main" {
  tags = { Name = "main" }
}

# Zona do Route53
data "aws_route53_zone" "main" {
  name = "exemplo.com."
}
```

## Outputs

```hcl
output "instance_ip" {
  value       = aws_instance.web.public_ip
  description = "IP público da instância"
}

output "sensitive_value" {
  value     = aws_db_instance.main.password
  sensitive = true
}

# Output estruturado
output "vpc" {
  value = {
    id         = aws_vpc.main.id
    cidr_block = aws_vpc.main.cidr_block
    subnet_ids = aws_subnet.public[*].id
  }
}
```

## Modules

### Estrutura

```
modules/vpc/
├── main.tf
├── variables.tf
└── outputs.tf
```

### Usando módulo local

```hcl
module "vpc" {
  source = "./modules/vpc"

  name = "main"
  cidr = "10.0.0.0/16"
}

# Acessar output
resource "aws_instance" "web" {
  subnet_id = module.vpc.subnet_ids[0]
}
```

### Usando módulo do Registry

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "main"
  cidr = "10.0.0.0/16"
  azs  = ["us-east-1a", "us-east-1b"]

  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
}
```

### Módulo de Git

```hcl
module "vpc" {
  source = "git::https://github.com/empresa/terraform-modules.git//vpc?ref=v1.2.0"
}
```

## Remote state

=== "AWS S3"

    ```hcl
    terraform {
      backend "s3" {
        bucket         = "empresa-terraform-state"
        key            = "projeto/terraform.tfstate"
        region         = "us-east-1"
        dynamodb_table = "terraform-locks"
        encrypt        = true
      }
    }
    ```

    !!! warning "Criar bucket e tabela antes"
        ```hcl
        resource "aws_s3_bucket" "state" {
          bucket = "empresa-terraform-state"
        }

        resource "aws_s3_bucket_versioning" "state" {
          bucket = aws_s3_bucket.state.id
          versioning_configuration {
            status = "Enabled"
          }
        }

        resource "aws_dynamodb_table" "locks" {
          name         = "terraform-locks"
          billing_mode = "PAY_PER_REQUEST"
          hash_key     = "LockID"

          attribute {
            name = "LockID"
            type = "S"
          }
        }
        ```

=== "GCP GCS"

    ```hcl
    terraform {
      backend "gcs" {
        bucket = "empresa-terraform-state"
        prefix = "projeto"
      }
    }
    ```

=== "Azure Blob"

    ```hcl
    terraform {
      backend "azurerm" {
        resource_group_name  = "terraform-state-rg"
        storage_account_name = "empresatfstate"
        container_name       = "tfstate"
        key                  = "projeto.tfstate"
      }
    }
    ```

### Acessar remote state de outro projeto

```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "empresa-terraform-state"
    key    = "network/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "web" {
  subnet_id = data.terraform_remote_state.network.outputs.subnet_ids[0]
}
```

## Workspaces

```bash
terraform workspace new dev
terraform workspace new prod
terraform workspace list
terraform workspace select dev
terraform workspace show
```

```hcl
locals {
  env = terraform.workspace

  config = {
    dev  = { instance_type = "t3.micro", count = 1 }
    prod = { instance_type = "t3.large", count = 3 }
  }
}

resource "aws_instance" "web" {
  count         = local.config[local.env].count
  instance_type = local.config[local.env].instance_type
}
```

!!! tip "Quando usar workspaces"
    Bom para ambientes similares (dev/staging). Para ambientes muito diferentes, prefira diretórios separados.

## State management

```bash
# Listar recursos
terraform state list

# Ver detalhes
terraform state show aws_instance.web

# Mover recurso
terraform state mv aws_instance.web aws_instance.app

# Remover do state (não destrói)
terraform state rm aws_instance.web

# Importar recurso existente
terraform import aws_instance.web i-1234567890abcdef0

# Pull/push state
terraform state pull > backup.tfstate
terraform state push backup.tfstate
```

## Lifecycle

```hcl
resource "aws_instance" "web" {
  # ...

  lifecycle {
    create_before_destroy = true   # Cria novo antes de destruir
    prevent_destroy       = true   # Impede destruição
    ignore_changes        = [tags] # Ignora mudanças em tags

    replace_triggered_by = [       # Recria quando mudar
      aws_security_group.web.id
    ]
  }
}
```

## Provisioners

!!! warning "Use com moderação"
    Prefira cloud-init, user_data ou ferramentas como Ansible.

```hcl
resource "aws_instance" "web" {
  # ...

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install -y nginx"
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> inventory.txt"
  }
}
```

## Funções úteis

```hcl
# Strings
upper("hello")                    # HELLO
lower("HELLO")                    # hello
replace("hello", "l", "L")        # heLLo
split(",", "a,b,c")               # ["a", "b", "c"]
join("-", ["a", "b", "c"])        # a-b-c
format("Hello, %s!", "World")     # Hello, World!
trimspace("  hello  ")            # hello

# Números
min(1, 2, 3)                      # 1
max(1, 2, 3)                      # 3
ceil(1.5)                         # 2
floor(1.5)                        # 1

# Coleções
length(["a", "b", "c"])           # 3
element(["a", "b", "c"], 1)       # b
contains(["a", "b"], "a")         # true
concat(["a"], ["b"])              # ["a", "b"]
flatten([["a"], ["b", "c"]])      # ["a", "b", "c"]
distinct(["a", "a", "b"])         # ["a", "b"]
merge({a=1}, {b=2})               # {a=1, b=2}
lookup({a=1, b=2}, "a", 0)        # 1
keys({a=1, b=2})                  # ["a", "b"]
values({a=1, b=2})                # [1, 2]

# Type conversion
tostring(123)
tonumber("123")
tolist(toset(["a", "a", "b"]))    # ["a", "b"]
toset(["a", "a", "b"])            # ["a", "b"] (set)
tomap({a = "b"})

# Encoding
jsonencode({a = 1})               # {"a":1}
jsondecode("{\"a\":1}")           # {a = 1}
base64encode("hello")
base64decode("aGVsbG8=")
yamlencode({a = 1})

# Files
file("script.sh")                 # Conteúdo do arquivo
filebase64("image.png")           # Base64 do arquivo
templatefile("script.tpl", {name = "web"})

# Network
cidrsubnet("10.0.0.0/16", 8, 1)   # 10.0.1.0/24
cidrhost("10.0.1.0/24", 5)        # 10.0.1.5

# Crypto
md5("hello")
sha256("hello")
uuid()
```

## .gitignore

```gitignore
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
*.tfvars
!example.tfvars
crash.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json
```

## Ferramentas complementares

| Ferramenta | Descrição |
|------------|-----------|
| **tflint** | Linter |
| **terraform-docs** | Gera documentação |
| **terragrunt** | DRY para múltiplos ambientes |
| **atlantis** | GitOps para Terraform |
| **infracost** | Estimativa de custos |
| **checkov** | Security scanning |

```bash
# tflint
brew install tflint
tflint

# terraform-docs
brew install terraform-docs
terraform-docs markdown . > README.md

# infracost
brew install infracost
infracost breakdown --path .
```

## Links

- [Terraform Docs](https://developer.hashicorp.com/terraform/docs){:target="_blank"}
- [Terraform Registry](https://registry.terraform.io/){:target="_blank"}
- [Terraform Best Practices](https://www.terraform-best-practices.com/){:target="_blank"}
