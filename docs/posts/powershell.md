---
date: 2026-01-20
categories:
  - DevOps
  - Terminal
---

# PowerShell: referência

Referência completa de PowerShell para uso no dia-a-dia. Cobre desde conceitos básicos até funções avançadas.

<!-- more -->

## Conceitos básicos

### Variáveis

```powershell
# Variáveis começam com $
$nome = "João"
$idade = 30
$ativo = $true

# Tipos explícitos
[string]$texto = "Hello"
[int]$numero = 42
[bool]$flag = $false
[datetime]$data = Get-Date

# Constantes (não podem ser alteradas)
Set-Variable -Name PI -Value 3.14159 -Option Constant

# Variáveis de ambiente
$env:PATH
$env:HOME
$env:USERNAME
```

### Tipos de dados

```powershell
# Strings
$str = "texto"
$str = 'texto literal (sem interpolação)'

# Números
$int = 42
$float = 3.14
$decimal = 19.99d

# Boolean
$verdadeiro = $true
$falso = $false

# Array
$arr = @(1, 2, 3)
$arr = 1, 2, 3  # sintaxe alternativa

# Hashtable
$hash = @{
    Nome = "João"
    Idade = 30
}

# Null
$nulo = $null
```

### Variáveis automáticas

```powershell
$_              # objeto atual no pipeline
$PSVersionTable # informações da versão do PowerShell
$HOME           # diretório home do usuário
$PWD            # diretório atual
$PROFILE        # caminho do arquivo de perfil
$?              # status do último comando (True/False)
$LASTEXITCODE   # código de saída do último programa nativo
$args           # argumentos passados para script/função
$input          # entrada do pipeline em funções
$Error          # array de erros recentes
$null           # valor nulo
$true           # booleano verdadeiro
$false          # booleano falso
```

!!! tip "Verificar versão do PowerShell"
    Use `$PSVersionTable.PSVersion` para ver a versão instalada. PowerShell 7+ é cross-platform (Windows, Linux, macOS).

## Comandos de descoberta

```powershell
# Descobrir comandos disponíveis
Get-Command                          # todos os comandos
Get-Command -Name *process*          # comandos com "process"
Get-Command -Module Microsoft.PowerShell.Management

# Ajuda detalhada
Get-Help Get-Process                 # ajuda básica
Get-Help Get-Process -Detailed       # com exemplos
Get-Help Get-Process -Full           # completa
Get-Help Get-Process -Online         # abre no browser
Get-Help about_*                     # tópicos conceituais

# Inspecionar objetos
Get-Process | Get-Member             # propriedades e métodos
Get-Process | Get-Member -MemberType Property
Get-Process | Get-Member -MemberType Method
```

### Aliases comuns

```powershell
# Navegação
cd      # Set-Location
pwd     # Get-Location
ls      # Get-ChildItem
dir     # Get-ChildItem

# Arquivos
cp      # Copy-Item
mv      # Move-Item
rm      # Remove-Item
cat     # Get-Content
mkdir   # New-Item -ItemType Directory

# Outros
echo    # Write-Output
cls     # Clear-Host
where   # Where-Object
select  # Select-Object
foreach # ForEach-Object
%       # ForEach-Object
?       # Where-Object
```

!!! info "Descobrir alias"
    Use `Get-Alias -Name ls` para ver o cmdlet por trás de um alias, ou `Get-Alias -Definition Get-ChildItem` para ver todos os aliases de um cmdlet.

## Sistema de arquivos

### Navegação

```powershell
# Localização atual
Get-Location           # pwd
Set-Location C:\Users  # cd

# Voltar ao diretório anterior
Push-Location C:\Temp  # salva atual e vai para C:\Temp
Pop-Location           # volta ao diretório salvo

# Home do usuário
Set-Location ~
Set-Location $HOME
```

### Listar arquivos

```powershell
# Listar conteúdo
Get-ChildItem                        # ls
Get-ChildItem -Force                 # incluir ocultos
Get-ChildItem -Recurse               # recursivo
Get-ChildItem -File                  # só arquivos
Get-ChildItem -Directory             # só diretórios

# Filtrar por padrão
Get-ChildItem *.txt
Get-ChildItem -Filter *.log
Get-ChildItem -Include *.txt, *.md -Recurse
Get-ChildItem -Exclude *.tmp -Recurse
```

### Manipular arquivos

```powershell
# Criar
New-Item -Path arquivo.txt -ItemType File
New-Item -Path pasta -ItemType Directory
New-Item arquivo.txt -Value "conteúdo inicial"

# Copiar
Copy-Item origem.txt destino.txt
Copy-Item pasta -Destination nova-pasta -Recurse

# Mover/Renomear
Move-Item arquivo.txt nova-pasta/
Rename-Item arquivo.txt novo-nome.txt

# Remover
Remove-Item arquivo.txt
Remove-Item pasta -Recurse              # diretório com conteúdo
Remove-Item pasta -Recurse -Force       # forçar remoção
```

### Conteúdo de arquivos

```powershell
# Ler
Get-Content arquivo.txt
Get-Content arquivo.txt -Head 10        # primeiras 10 linhas
Get-Content arquivo.txt -Tail 10        # últimas 10 linhas
Get-Content arquivo.txt -Tail 10 -Wait  # como tail -f

# Escrever (sobrescreve)
Set-Content arquivo.txt -Value "novo conteúdo"
"texto" | Set-Content arquivo.txt

# Adicionar (append)
Add-Content arquivo.txt -Value "mais uma linha"
"texto" | Add-Content arquivo.txt

# Limpar conteúdo
Clear-Content arquivo.txt
```

### Paths

```powershell
# Construir paths
Join-Path -Path "C:\Users" -ChildPath "João"
Join-Path "C:\Users" "João" "Documents"  # múltiplos segmentos

# Separar paths
Split-Path "C:\Users\João\arquivo.txt"              # C:\Users\João
Split-Path "C:\Users\João\arquivo.txt" -Leaf        # arquivo.txt
Split-Path "C:\Users\João\arquivo.txt" -Extension   # .txt (PS 6+)

# Verificar existência
Test-Path arquivo.txt
Test-Path pasta -PathType Container     # é diretório?
Test-Path arquivo.txt -PathType Leaf    # é arquivo?

# Resolver path
Resolve-Path *.txt                      # paths completos que batem
Resolve-Path ~                          # expande ~
```

## Strings

### Interpolação

```powershell
$nome = "João"
$idade = 30

# Aspas duplas: interpola variáveis
"Olá, $nome!"                           # Olá, João!
"Idade: $($idade + 1)"                  # Idade: 31 (expressão)
"Home: $($env:HOME)"                    # variável de ambiente

# Aspas simples: literal
'Olá, $nome!'                           # Olá, $nome! (literal)
```

### Here-strings

```powershell
# Here-string com interpolação
$json = @"
{
    "nome": "$nome",
    "idade": $idade
}
"@

# Here-string literal
$template = @'
Texto com $variáveis que não serão interpoladas.
Útil para templates ou regex.
'@
```

### Métodos de string

```powershell
$str = "  Hello, World!  "

# Transformação
$str.ToUpper()                          # "  HELLO, WORLD!  "
$str.ToLower()                          # "  hello, world!  "
$str.Trim()                             # "Hello, World!"
$str.TrimStart()                        # "Hello, World!  "
$str.TrimEnd()                          # "  Hello, World!"

# Busca
$str.Contains("World")                  # True
$str.StartsWith("  Hello")              # True
$str.EndsWith("!  ")                    # True
$str.IndexOf("World")                   # 9

# Manipulação
$str.Replace("World", "PowerShell")     # "  Hello, PowerShell!  "
$str.Substring(2, 5)                    # "Hello"
$str.Split(",")                         # @("  Hello", " World!  ")
"a,b,c".Split(",")                      # @("a", "b", "c")

# Juntar array em string
$arr = @("a", "b", "c")
$arr -join ", "                         # "a, b, c"
[string]::Join("-", $arr)               # "a-b-c"
```

### Operadores de string

```powershell
# -like (wildcard)
"PowerShell" -like "Power*"             # True
"PowerShell" -like "*Shell"             # True
"PowerShell" -like "*wer*"              # True

# -match (regex)
"PowerShell 7.4" -match "\d+\.\d+"      # True
$Matches[0]                             # "7.4"

# -replace (regex)
"Hello World" -replace "World", "PS"    # "Hello PS"
"abc123" -replace "\d+", "XXX"          # "abcXXX"

# -split
"a,b,c" -split ","                      # @("a", "b", "c")
"a1b2c3" -split "\d"                    # @("a", "b", "c", "")
```

!!! warning "Case sensitivity"
    Por padrão, operadores de string são case-insensitive. Use versões com 'c' para case-sensitive: `-clike`, `-cmatch`, `-creplace`.

## Arrays e Hashtables

### Arrays

```powershell
# Criar
$arr = @(1, 2, 3)
$arr = 1, 2, 3                          # sintaxe simplificada
$arr = @()                              # array vazio
$arr = 1..10                            # range: 1 a 10

# Acessar
$arr[0]                                 # primeiro elemento
$arr[-1]                                # último elemento
$arr[0..2]                              # slice: primeiros 3
$arr[-3..-1]                            # últimos 3

# Modificar
$arr += 4                               # adicionar (cria novo array!)
$arr[0] = 99                            # alterar elemento

# Propriedades
$arr.Count                              # quantidade de elementos
$arr.Length                             # mesmo que Count

# Verificar conteúdo
$arr -contains 2                        # True
2 -in $arr                              # True (PS 3+)
$arr -notcontains 99                    # True
```

!!! warning "Performance com arrays"
    `$arr += item` cria um novo array a cada adição. Para muitas inserções, use `[System.Collections.ArrayList]` ou `[System.Collections.Generic.List[object]]`.

### ArrayList (mutável)

```powershell
$list = [System.Collections.ArrayList]@()
$list.Add("item")                       # retorna índice
[void]$list.Add("item2")                # suprime retorno
$list.Remove("item")
$list.RemoveAt(0)
$list.Clear()
```

### Hashtables

```powershell
# Criar
$hash = @{
    Nome = "João"
    Idade = 30
    Ativo = $true
}
$hash = @{}                             # hashtable vazia

# Acessar
$hash["Nome"]                           # "João"
$hash.Nome                              # "João" (sintaxe de propriedade)
$hash.Keys                              # todas as chaves
$hash.Values                            # todos os valores

# Modificar
$hash["Email"] = "joao@email.com"       # adicionar/atualizar
$hash.Cidade = "São Paulo"              # sintaxe alternativa
$hash.Remove("Idade")                   # remover

# Verificar
$hash.ContainsKey("Nome")               # True
$hash.ContainsValue("João")             # True
$hash.Count                             # quantidade de pares

# Iterar
foreach ($key in $hash.Keys) {
    "$key = $($hash[$key])"
}

$hash.GetEnumerator() | ForEach-Object {
    "$($_.Key) = $($_.Value)"
}
```

### Ordered hashtable

```powershell
# Mantém ordem de inserção
$ordered = [ordered]@{
    Primeiro = 1
    Segundo = 2
    Terceiro = 3
}
```

## Pipeline e objetos

### Pipeline básico

```powershell
# Passar objetos entre comandos
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5

# $_ representa o objeto atual
Get-Process | Where-Object { $_.CPU -gt 100 }

# Sintaxe simplificada (PS 3+)
Get-Process | Where-Object CPU -gt 100
```

### Where-Object (filtrar)

```powershell
# Filtrar com script block
Get-Process | Where-Object { $_.CPU -gt 100 }
Get-Process | Where-Object { $_.Name -like "*chrome*" }
Get-Process | Where-Object { $_.WorkingSet -gt 100MB }

# Sintaxe simplificada
Get-Process | Where-Object CPU -gt 100
Get-Process | Where-Object Name -like "*chrome*"

# Múltiplas condições
Get-Process | Where-Object { $_.CPU -gt 100 -and $_.Name -ne "Idle" }
```

### Select-Object (projetar)

```powershell
# Selecionar propriedades
Get-Process | Select-Object Name, CPU, WorkingSet
Get-Process | Select-Object -Property Name, CPU

# Limitar resultados
Get-Process | Select-Object -First 5
Get-Process | Select-Object -Last 3
Get-Process | Select-Object -Skip 10 -First 5

# Propriedades calculadas
Get-Process | Select-Object Name, @{
    Name = "MemoriaMB"
    Expression = { [math]::Round($_.WorkingSet / 1MB, 2) }
}

# Expandir propriedade (extrair valor)
Get-Process | Select-Object -ExpandProperty Name    # retorna strings, não objetos

# Propriedades únicas
Get-Process | Select-Object Name -Unique
```

### ForEach-Object (transformar)

```powershell
# Executar ação em cada item
Get-Process | ForEach-Object { $_.Name.ToUpper() }
1..5 | ForEach-Object { $_ * 2 }        # 2, 4, 6, 8, 10

# Sintaxe simplificada
Get-Process | ForEach-Object Name       # extrai propriedade

# Múltiplos script blocks
1..3 | ForEach-Object -Begin { "Início" } -Process { "Item: $_" } -End { "Fim" }
```

### Sort-Object (ordenar)

```powershell
Get-Process | Sort-Object CPU
Get-Process | Sort-Object CPU -Descending
Get-Process | Sort-Object CPU, Name                 # múltiplas propriedades
Get-Process | Sort-Object { $_.WorkingSet } -Descending
```

### Group-Object (agrupar)

```powershell
Get-Process | Group-Object ProcessName
Get-Service | Group-Object Status

# Acessar grupos
$grupos = Get-Process | Group-Object ProcessName
$grupos[0].Name                         # nome do grupo
$grupos[0].Count                        # quantidade no grupo
$grupos[0].Group                        # itens do grupo
```

### Measure-Object (estatísticas)

```powershell
# Contar
Get-Process | Measure-Object            # Count
Get-ChildItem | Measure-Object

# Estatísticas numéricas
Get-Process | Measure-Object CPU -Sum -Average -Maximum -Minimum
(Get-Content arquivo.txt | Measure-Object -Line -Word -Character)
```

### Outros cmdlets úteis

```powershell
# Comparar
Compare-Object $array1 $array2

# Tee (bifurcar output)
Get-Process | Tee-Object -Variable processos | Where-Object CPU -gt 100

# Out-Null (descartar output)
Remove-Item arquivo.txt | Out-Null
$null = Remove-Item arquivo.txt         # alternativa
```

!!! tip "Pipeline vs ForEach"
    Para operações simples, pipeline é mais idiomático. Para lógica complexa ou quando precisa de índice, use `foreach` statement.

## Condicionais

### if/elseif/else

```powershell
$idade = 25

if ($idade -lt 18) {
    "Menor de idade"
}
elseif ($idade -lt 65) {
    "Adulto"
}
else {
    "Idoso"
}

# Uma linha
if ($valor) { "Verdadeiro" } else { "Falso" }
```

### Operadores de comparação

```powershell
# Igualdade
$a -eq $b                               # igual
$a -ne $b                               # diferente

# Numéricos
$a -gt $b                               # maior que
$a -ge $b                               # maior ou igual
$a -lt $b                               # menor que
$a -le $b                               # menor ou igual

# String (case-insensitive por padrão)
$str -like "padr*"                      # wildcard
$str -notlike "padr*"
$str -match "regex"                     # regex
$str -notmatch "regex"

# Versões case-sensitive
$str -ceq "Texto"                       # igual case-sensitive
$str -clike "Pad*"
$str -cmatch "Regex"

# Coleções
$arr -contains "item"                   # contém
"item" -in $arr                         # está em
$arr -notcontains "item"
"item" -notin $arr

# Tipo
$obj -is [string]                       # é do tipo
$obj -isnot [int]
```

### Operadores lógicos

```powershell
$a -and $b                              # E
$a -or $b                               # OU
-not $a                                 # NÃO
!$a                                     # NÃO (alternativa)
$a -xor $b                              # OU exclusivo
```

### switch

```powershell
$dia = "segunda"

switch ($dia) {
    "segunda" { "Início da semana" }
    "sexta" { "Quase fim de semana!" }
    "sábado" { "Fim de semana!" }
    "domingo" { "Fim de semana!" }
    default { "Dia comum" }
}

# Com wildcard
switch -Wildcard ($arquivo) {
    "*.txt" { "Arquivo de texto" }
    "*.ps1" { "Script PowerShell" }
    default { "Outro tipo" }
}

# Com regex
switch -Regex ($texto) {
    "^\d+$" { "Apenas números" }
    "^[a-z]+$" { "Apenas letras" }
    default { "Misto" }
}

# Múltiplos matches (sem break implícito)
switch ($valor) {
    { $_ -gt 0 } { "Positivo" }
    { $_ -lt 100 } { "Menor que 100" }
    10 { "É dez!" }
}
```

### Ternário (PS 7+)

```powershell
$resultado = $condicao ? "verdadeiro" : "falso"
$status = $ativo ? "Ativo" : "Inativo"
```

### Null-coalescing (PS 7+)

```powershell
$valor = $variavel ?? "padrão"          # usa "padrão" se $variavel é $null
$variavel ??= "padrão"                  # atribui se $null
```

!!! note "PowerShell 7+"
    Operador ternário (`?:`) e null-coalescing (`??`, `??=`) requerem PowerShell 7+.

## Loops

### foreach

```powershell
# foreach statement (mais rápido)
foreach ($item in $colecao) {
    $item
}

# Com índice
$arr = @("a", "b", "c")
for ($i = 0; $i -lt $arr.Count; $i++) {
    "$i : $($arr[$i])"
}

# ForEach-Object cmdlet (pipeline)
$colecao | ForEach-Object { $_ }
```

### for

```powershell
for ($i = 0; $i -lt 10; $i++) {
    "Iteração $i"
}

# Decremento
for ($i = 10; $i -gt 0; $i--) {
    $i
}
```

### while

```powershell
$i = 0
while ($i -lt 5) {
    $i
    $i++
}
```

### do-while / do-until

```powershell
# do-while: executa enquanto condição é verdadeira
$i = 0
do {
    $i
    $i++
} while ($i -lt 5)

# do-until: executa até condição ser verdadeira
$i = 0
do {
    $i
    $i++
} until ($i -ge 5)
```

### Controle de loop

```powershell
# break: sai do loop
foreach ($item in $colecao) {
    if ($item -eq "parar") { break }
    $item
}

# continue: pula para próxima iteração
foreach ($item in $colecao) {
    if ($item -eq "pular") { continue }
    $item
}

# return: sai da função/script (não apenas do loop)
```

### Ranges

```powershell
# Range simples
1..10                                   # 1 a 10
10..1                                   # 10 a 1 (decrescente)

# Iterar com range
foreach ($i in 1..5) {
    "Número: $i"
}

# Letras
[char]'a'..[char]'z' | ForEach-Object { [char]$_ }
```

## Funções

### Sintaxe básica

```powershell
function Get-Greeting {
    "Hello, World!"
}

# Com parâmetros
function Get-Greeting {
    param (
        [string]$Nome
    )
    "Hello, $Nome!"
}

# Sintaxe alternativa para parâmetros
function Get-Greeting($Nome) {
    "Hello, $Nome!"
}

# Chamar
Get-Greeting -Nome "João"
Get-Greeting "João"                     # posicional
```

### Parâmetros avançados

```powershell
function Get-UserInfo {
    [CmdletBinding()]
    param (
        # Obrigatório
        [Parameter(Mandatory)]
        [string]$Nome,

        # Com valor padrão
        [int]$Idade = 0,

        # Validação
        [ValidateRange(1, 150)]
        [int]$Anos,

        [ValidateSet("Ativo", "Inativo")]
        [string]$Status = "Ativo",

        [ValidateNotNullOrEmpty()]
        [string]$Email,

        [ValidateScript({ Test-Path $_ })]
        [string]$Caminho,

        # Switch (booleano)
        [switch]$Detalhado,

        # Aceita múltiplos valores
        [string[]]$Tags
    )

    # Corpo da função
    if ($Detalhado) {
        "Nome: $Nome, Idade: $Idade, Status: $Status"
    }
    else {
        $Nome
    }
}
```

### Pipeline input

```powershell
function Process-Item {
    [CmdletBinding()]
    param (
        [Parameter(ValueFromPipeline)]
        [string]$Item
    )

    begin {
        "Iniciando processamento"
    }

    process {
        "Processando: $Item"
    }

    end {
        "Finalizado"
    }
}

# Uso
"a", "b", "c" | Process-Item

# ValueFromPipelineByPropertyName
function Get-FileName {
    param (
        [Parameter(ValueFromPipelineByPropertyName)]
        [Alias("FullName")]
        [string]$Path
    )

    process {
        Split-Path $Path -Leaf
    }
}

Get-ChildItem | Get-FileName
```

### Retorno de valores

```powershell
function Get-Dobro {
    param ([int]$Numero)

    # Qualquer output vai para o retorno
    $Numero * 2
}

function Get-Info {
    param ([string]$Nome)

    # Retorno explícito
    return @{
        Nome = $Nome
        Data = Get-Date
    }
}

# Múltiplos valores
function Get-Numeros {
    1
    2
    3
    # ou: return 1, 2, 3
    # ou: 1, 2, 3
}

$resultado = Get-Numeros  # @(1, 2, 3)
```

!!! warning "Output não intencional"
    Qualquer expressão não atribuída a variável vai para o output. Use `[void]`, `$null =` ou `| Out-Null` para suprimir.

### Escopo

```powershell
$global:var = "global"                  # acessível em todo lugar
$script:var = "script"                  # acessível no script
$local:var = "local"                    # padrão, escopo atual
$private:var = "private"                # não visível em escopos filhos

function Test-Escopo {
    $var = "local na função"
    $script:var = "alterando script"
}
```

## Tratamento de erros

### try/catch/finally

```powershell
try {
    # Código que pode falhar
    $resultado = Get-Content "arquivo-inexistente.txt" -ErrorAction Stop
}
catch [System.IO.FileNotFoundException] {
    # Erro específico
    Write-Warning "Arquivo não encontrado: $_"
}
catch {
    # Qualquer outro erro
    Write-Error "Erro: $_"
    Write-Error $_.Exception.Message
}
finally {
    # Sempre executa
    "Limpeza"
}
```

### ErrorAction

```powershell
# Por comando
Get-Content arquivo.txt -ErrorAction SilentlyContinue
Get-Content arquivo.txt -ErrorAction Stop
Get-Content arquivo.txt -ErrorAction Continue  # padrão

# Valores de ErrorAction:
# - Continue: mostra erro, continua (padrão)
# - SilentlyContinue: ignora erro silenciosamente
# - Stop: converte em exceção terminadora
# - Inquire: pergunta ao usuário
# - Ignore: ignora completamente (nem vai para $Error)

# Variável de preferência (para todo o script)
$ErrorActionPreference = "Stop"
```

### Variável $Error

```powershell
$Error                                  # lista de erros recentes
$Error[0]                               # erro mais recente
$Error[0].Exception.Message
$Error.Clear()                          # limpar histórico
```

### throw

```powershell
function Test-Valor {
    param ([int]$Valor)

    if ($Valor -lt 0) {
        throw "Valor não pode ser negativo"
    }

    $Valor
}

# Com tipo de exceção
throw [System.ArgumentException]::new("Argumento inválido")
```

### Write-Error vs throw

```powershell
# Write-Error: erro não-terminador
Write-Error "Algo deu errado"
"Esta linha executa"

# throw: erro terminador
throw "Erro fatal"
"Esta linha não executa"
```

!!! tip "Quando usar cada um"
    Use `throw` para erros que devem parar a execução. Use `Write-Error` para reportar problemas mas continuar processando outros itens.

## Módulos

### Gerenciar módulos

```powershell
# Listar módulos disponíveis
Get-Module -ListAvailable

# Listar módulos carregados
Get-Module

# Importar módulo
Import-Module NomeMódulo

# Remover módulo
Remove-Module NomeMódulo

# Encontrar no PSGallery
Find-Module -Name "*Azure*"

# Instalar do PSGallery
Install-Module -Name Az -Scope CurrentUser
Install-Module -Name Pester -Force

# Atualizar módulo
Update-Module -Name Az

# Desinstalar
Uninstall-Module -Name NomeMódulo
```

### Módulos úteis

```powershell
# Pester - framework de testes
Install-Module Pester -Scope CurrentUser

# PSReadLine - melhorias no console (já incluído no PS 5.1+)
# Configurações úteis:
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -EditMode Emacs

# Terminal-Icons - ícones no terminal
Install-Module Terminal-Icons -Scope CurrentUser
Import-Module Terminal-Icons

# Az - Azure PowerShell
Install-Module Az -Scope CurrentUser
```

### Criar módulo simples

```powershell
# Arquivo: MeuModulo.psm1
function Get-Saudacao {
    param ([string]$Nome)
    "Olá, $Nome!"
}

function Get-DataFormatada {
    (Get-Date).ToString("dd/MM/yyyy")
}

# Exportar funções
Export-ModuleMember -Function Get-Saudacao, Get-DataFormatada
```

## Perfil

### Caminhos do perfil

```powershell
$PROFILE                                # perfil do usuário atual, host atual
$PROFILE.CurrentUserCurrentHost         # mesmo que $PROFILE
$PROFILE.CurrentUserAllHosts            # usuário atual, todos os hosts
$PROFILE.AllUsersCurrentHost            # todos os usuários, host atual
$PROFILE.AllUsersAllHosts               # todos os usuários, todos os hosts

# Verificar se existe
Test-Path $PROFILE

# Criar se não existir
if (!(Test-Path $PROFILE)) {
    New-Item $PROFILE -ItemType File -Force
}

# Editar
code $PROFILE                           # VS Code
notepad $PROFILE                        # Notepad
```

### Exemplo de perfil

```powershell
# ~/.config/powershell/Microsoft.PowerShell_profile.ps1 (PS 7+ Linux/Mac)
# ~/Documents/PowerShell/Microsoft.PowerShell_profile.ps1 (PS 7+ Windows)
# ~/Documents/WindowsPowerShell/Microsoft.PowerShell_profile.ps1 (PS 5.1)

# Módulos
Import-Module Terminal-Icons
Import-Module PSReadLine

# PSReadLine
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

# Aliases/Funções
function ll { Get-ChildItem -Force $args }
function .. { Set-Location .. }
function ... { Set-Location ../.. }

# Alias para editor
Set-Alias -Name vim -Value nvim
Set-Alias -Name code -Value "code-insiders"

# Variáveis de ambiente
$env:EDITOR = "nvim"
```

## Comandos úteis

### Saída e formatação

```powershell
# Formatar como tabela
Get-Process | Format-Table Name, CPU, WorkingSet -AutoSize

# Formatar como lista
Get-Process | Format-List *

# Saída para arquivo
Get-Process | Out-File processos.txt
Get-Process | Out-File processos.txt -Append

# Exportar CSV
Get-Process | Export-Csv processos.csv -NoTypeInformation
Import-Csv processos.csv

# JSON
Get-Process | ConvertTo-Json | Out-File processos.json
Get-Content processos.json | ConvertFrom-Json

# XML
Get-Process | Export-Clixml processos.xml
Import-Clixml processos.xml

# Clipboard
Get-Process | clip                      # Windows
Get-Process | Set-Clipboard             # PS 5.1+
Get-Clipboard
```

### Web requests

```powershell
# Download simples
Invoke-WebRequest -Uri "https://example.com" -OutFile pagina.html

# GET com headers
$response = Invoke-WebRequest -Uri "https://api.example.com/data" -Headers @{
    "Authorization" = "Bearer $token"
}
$response.Content
$response.StatusCode

# REST API (retorna objetos)
$dados = Invoke-RestMethod -Uri "https://api.example.com/users"
$dados.name

# POST JSON
$body = @{
    nome = "João"
    email = "joao@email.com"
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://api.example.com/users" `
    -Method Post `
    -Body $body `
    -ContentType "application/json"
```

### Processos e serviços

```powershell
# Processos
Get-Process
Get-Process -Name chrome
Get-Process | Where-Object CPU -gt 100
Stop-Process -Name notepad
Stop-Process -Id 1234
Start-Process notepad
Start-Process "https://google.com"      # abre no browser

# Serviços (requer elevação no Windows)
Get-Service
Get-Service -Name wuauserv
Start-Service -Name wuauserv
Stop-Service -Name wuauserv
Restart-Service -Name wuauserv
```

### Jobs (background)

```powershell
# Iniciar job
$job = Start-Job -ScriptBlock { Get-Process }

# Verificar status
Get-Job

# Aguardar conclusão
Wait-Job -Id $job.Id

# Obter resultado
Receive-Job -Id $job.Id

# Remover job
Remove-Job -Id $job.Id

# Job com parâmetros
Start-Job -ScriptBlock { param($nome) "Olá, $nome" } -ArgumentList "João"
```

### Remoting (se habilitado)

```powershell
# Sessão interativa
Enter-PSSession -ComputerName servidor

# Executar comando remoto
Invoke-Command -ComputerName servidor -ScriptBlock { Get-Process }

# Em múltiplas máquinas
Invoke-Command -ComputerName srv1, srv2, srv3 -ScriptBlock { hostname }
```

### Data e hora

```powershell
Get-Date
Get-Date -Format "dd/MM/yyyy HH:mm:ss"
Get-Date -Format "yyyy-MM-dd"
(Get-Date).AddDays(7)
(Get-Date).AddHours(-2)

# Diferença entre datas
$inicio = Get-Date "2024-01-01"
$fim = Get-Date
($fim - $inicio).Days
```

### Matemática

```powershell
[math]::Round(3.14159, 2)               # 3.14
[math]::Floor(3.9)                      # 3
[math]::Ceiling(3.1)                    # 4
[math]::Abs(-5)                         # 5
[math]::Max(1, 5)                       # 5
[math]::Min(1, 5)                       # 1
[math]::Pow(2, 8)                       # 256
[math]::Sqrt(16)                        # 4
```

## Links

- [Documentação PowerShell](https://learn.microsoft.com/en-us/powershell/){:target="_blank"}
- [PowerShell Gallery](https://www.powershellgallery.com/){:target="_blank"}
- [Oh-My-Posh](https://ohmyposh.dev/){:target="_blank"}
