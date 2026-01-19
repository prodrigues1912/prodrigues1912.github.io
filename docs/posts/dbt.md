---
date: 2025-08-20
categories:
  - Engenharia de Dados
  - SQL
---

# dbt: referência

Ferramenta de transformação de dados usando SQL. Faz o **T** do ELT, assumindo que os dados já estão no warehouse.

<!-- more -->

## O que é

dbt (data build tool) transforma dados no warehouse usando SQL SELECT. Gerencia dependências entre modelos, testa qualidade dos dados e gera documentação automaticamente.

## Que problemas resolve

- **Modularidade**: SQL organizado em modelos reutilizáveis
- **Dependências**: ordem de execução automática baseada em `ref()`
- **Testes**: validar qualidade dos dados (unique, not_null, etc.)
- **Documentação**: gerada a partir dos schemas YAML
- **Versionamento**: tudo no Git, CI/CD padrão
- **Materialização**: views, tables, incremental sem código extra

## Instalação

```bash
# Com uv
uv add dbt-core

# Com pip
pip install dbt-core
```

Adapters disponíveis: `dbt-bigquery`, `dbt-snowflake`, `dbt-redshift`, `dbt-postgres`, `dbt-databricks`, `dbt-duckdb`.

## Estrutura de projeto

```bash
dbt init meu_projeto
```

```
meu_projeto/
├── dbt_project.yml      # Configuração do projeto
├── profiles.yml         # Conexões (normalmente em ~/.dbt/)
├── models/
│   ├── staging/         # Limpeza inicial
│   │   ├── _sources.yml
│   │   ├── _schema.yml
│   │   └── stg_orders.sql
│   └── marts/           # Modelos finais
│       ├── _schema.yml
│       └── dim_customers.sql
├── tests/               # Testes customizados
├── macros/              # Funções SQL reutilizáveis
├── seeds/               # CSVs para carregar
└── snapshots/           # SCD Type 2
```

## dbt_project.yml

```yaml
name: 'meu_projeto'
version: '1.0.0'
profile: 'meu_projeto'

model-paths: ["models"]
test-paths: ["tests"]
macro-paths: ["macros"]
seed-paths: ["seeds"]
snapshot-paths: ["snapshots"]

models:
  meu_projeto:
    staging:
      +materialized: view
      +schema: staging
    marts:
      +materialized: table
      +schema: analytics
```

## profiles.yml

```yaml
# ~/.dbt/profiles.yml
meu_projeto:
  target: dev
  outputs:
    dev:
      type: bigquery
      method: oauth
      project: meu-projeto-gcp
      dataset: dbt_dev
      threads: 4
    prod:
      type: bigquery
      method: service-account
      project: meu-projeto-gcp
      dataset: analytics
      keyfile: /path/to/keyfile.json
      threads: 8
```

## Sources

Definir fontes de dados raw:

```yaml
# models/staging/_sources.yml
version: 2

sources:
  - name: raw
    database: meu_projeto
    schema: raw_data
    freshness:
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}
    tables:
      - name: orders
        loaded_at_field: _loaded_at
        description: Pedidos brutos do sistema
        columns:
          - name: id
            description: ID do pedido
      - name: customers
```

{% raw %}
Uso: `{{ source('raw', 'orders') }}`
{% endraw %}

## Modelos

### Staging

Limpeza e padronização:

{% raw %}
```sql
-- models/staging/stg_orders.sql
select
    id as order_id,
    user_id as customer_id,
    cast(created_at as timestamp) as ordered_at,
    lower(status) as status,
    total_cents / 100.0 as total_amount
from {{ source('raw', 'orders') }}
where id is not null
```
{% endraw %}

### Marts

Modelos analíticos finais:

{% raw %}
```sql
-- models/marts/dim_customers.sql
with customers as (
    select * from {{ ref('stg_customers') }}
),

orders as (
    select * from {{ ref('stg_orders') }}
),

customer_orders as (
    select
        customer_id,
        count(*) as total_orders,
        sum(total_amount) as lifetime_value,
        min(ordered_at) as first_order_at,
        max(ordered_at) as last_order_at
    from orders
    group by 1
)

select
    c.customer_id,
    c.name,
    c.email,
    c.created_at,
    coalesce(co.total_orders, 0) as total_orders,
    coalesce(co.lifetime_value, 0) as lifetime_value,
    co.first_order_at,
    co.last_order_at
from customers c
left join customer_orders co using (customer_id)
```
{% endraw %}

{% raw %}
`{{ ref('model_name') }}` cria dependência automática.
{% endraw %}

## Materializations

{% raw %}
```sql
-- View (padrão)
{{ config(materialized='view') }}

-- Tabela
{{ config(materialized='table') }}

-- Ephemeral (CTE, não cria objeto)
{{ config(materialized='ephemeral') }}
```
{% endraw %}

### Incremental

Processa apenas dados novos:

{% raw %}
```sql
{{ config(
    materialized='incremental',
    unique_key='order_id',
    incremental_strategy='merge'  -- ou delete+insert
) }}

select
    order_id,
    customer_id,
    ordered_at,
    total_amount,
    updated_at
from {{ source('raw', 'orders') }}

{% if is_incremental() %}
where updated_at > (select max(updated_at) from {{ this }})
{% endif %}
```
{% endraw %}

Estratégias: `merge`, `delete+insert`, `insert_overwrite`, `append`.

## Testes

### Schema tests (genéricos)

```yaml
# models/staging/_schema.yml
version: 2

models:
  - name: stg_orders
    columns:
      - name: order_id
        tests:
          - unique
          - not_null
      - name: status
        tests:
          - accepted_values:
              values: ['pending', 'completed', 'cancelled']
      - name: customer_id
        tests:
          - not_null
          - relationships:
              to: ref('stg_customers')
              field: customer_id
```

### Data tests (customizados)

{% raw %}
```sql
-- tests/assert_positive_amounts.sql
select *
from {{ ref('stg_orders') }}
where total_amount < 0
```
{% endraw %}

Retornar linhas = falha.

### Singular tests por modelo

{% raw %}
```sql
-- tests/stg_orders_no_future_dates.sql
select *
from {{ ref('stg_orders') }}
where ordered_at > current_timestamp
```
{% endraw %}

## Macros

Funções SQL reutilizáveis:

{% raw %}
```sql
-- macros/cents_to_dollars.sql
{% macro cents_to_dollars(column_name) %}
    ({{ column_name }} / 100.0)
{% endmacro %}
```
{% endraw %}

Uso:

{% raw %}
```sql
select
    order_id,
    {{ cents_to_dollars('total_cents') }} as total_amount
from {{ source('raw', 'orders') }}
```
{% endraw %}

### Macro com lógica

{% raw %}
```sql
-- macros/generate_schema_name.sql
{% macro generate_schema_name(custom_schema_name, node) %}
    {% if custom_schema_name is none %}
        {{ target.schema }}
    {% else %}
        {{ target.schema }}_{{ custom_schema_name }}
    {% endif %}
{% endmacro %}
```
{% endraw %}

## Seeds

CSVs carregados como tabelas:

```
seeds/
└── country_codes.csv
```

```csv
code,name
BR,Brazil
US,United States
```

```bash
dbt seed
```

{% raw %}
Uso: `{{ ref('country_codes') }}`
{% endraw %}

## Snapshots

SCD Type 2 automático:

{% raw %}
```sql
-- snapshots/customers_snapshot.sql
{% snapshot customers_snapshot %}

{{
    config(
        target_schema='snapshots',
        unique_key='customer_id',
        strategy='timestamp',
        updated_at='updated_at'
    )
}}

select * from {{ source('raw', 'customers') }}

{% endsnapshot %}
```
{% endraw %}

```bash
dbt snapshot
```

## Documentação

```yaml
# models/marts/_schema.yml
version: 2

models:
  - name: dim_customers
    description: |
      Dimensão de clientes com métricas agregadas.
      Atualizado diariamente.
    columns:
      - name: customer_id
        description: ID único do cliente
      - name: lifetime_value
        description: Valor total gasto pelo cliente
```

Markdown em `models/marts/dim_customers.md`:

{% raw %}
```markdown
{% docs dim_customers %}

## Dimensão de Clientes

Contém todos os clientes com métricas de pedidos.

### Fontes
- `stg_customers`
- `stg_orders`

{% enddocs %}
```
{% endraw %}

## CLI

```bash
# Executar modelos
dbt run                           # todos
dbt run --select stg_orders       # específico
dbt run --select staging.*        # pasta
dbt run --select +dim_customers   # modelo + upstream
dbt run --select dim_customers+   # modelo + downstream
dbt run --select tag:daily        # por tag

# Testes
dbt test                          # todos
dbt test --select stg_orders      # por modelo

# Build (run + test)
dbt build
dbt build --select staging.*

# Documentação
dbt docs generate
dbt docs serve

# Outros
dbt compile              # compilar SQL sem executar
dbt debug                # verificar conexão
dbt deps                 # instalar pacotes
dbt seed                 # carregar seeds
dbt snapshot             # executar snapshots
dbt source freshness     # verificar freshness
dbt clean                # limpar artefatos
```

## Seletores

```bash
# Por nome
dbt run --select model_name

# Por caminho
dbt run --select models/staging/

# Upstream (+) e downstream (+)
dbt run --select +model_name      # modelo e dependências
dbt run --select model_name+      # modelo e dependentes
dbt run --select +model_name+     # ambos

# Por tag
dbt run --select tag:daily

# Por source
dbt run --select source:raw.orders

# Exclusão
dbt run --exclude stg_legacy

# Combinação
dbt run --select staging.* --exclude stg_legacy
```

## Packages

```yaml
# packages.yml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1
  - package: dbt-labs/codegen
    version: 0.12.1
  - package: calogica/dbt_expectations
    version: 0.10.3
```

```bash
dbt deps
```

### dbt_utils

{% raw %}
```sql
-- Surrogate key
{{ dbt_utils.generate_surrogate_key(['customer_id', 'order_id']) }}

-- Pivot
{{ dbt_utils.pivot(
    'status',
    dbt_utils.get_column_values(ref('stg_orders'), 'status')
) }}

-- Date spine
{{ dbt_utils.date_spine(
    datepart="day",
    start_date="cast('2020-01-01' as date)",
    end_date="current_date"
) }}
```
{% endraw %}

## Hooks

Executar SQL antes/depois:

```yaml
# dbt_project.yml
models:
  meu_projeto:
    marts:
      +post-hook:
        - "GRANT SELECT ON {{ this }} TO ROLE analyst"
```

{% raw %}
```sql
-- No modelo
{{ config(
    post_hook="ANALYZE TABLE {{ this }}"
) }}
```
{% endraw %}

## Variáveis

```yaml
# dbt_project.yml
vars:
  start_date: '2020-01-01'
  enable_feature_x: true
```

{% raw %}
```sql
select *
from {{ ref('stg_orders') }}
where ordered_at >= '{{ var("start_date") }}'
```
{% endraw %}

```bash
dbt run --vars '{"start_date": "2023-01-01"}'
```

## Exposures

Documentar uso downstream:

```yaml
# models/_exposures.yml
version: 2

exposures:
  - name: weekly_sales_dashboard
    type: dashboard
    maturity: high
    url: https://bi.company.com/dashboard/123
    description: Dashboard de vendas semanais
    depends_on:
      - ref('fct_orders')
      - ref('dim_customers')
    owner:
      name: Data Team
      email: data@company.com
```

## Métricas

```yaml
# models/_metrics.yml
version: 2

metrics:
  - name: total_revenue
    label: Total Revenue
    model: ref('fct_orders')
    description: Receita total
    calculation_method: sum
    expression: total_amount
    timestamp: ordered_at
    time_grains: [day, week, month]
    dimensions:
      - customer_segment
      - product_category
```

## Ambiente de execução

{% raw %}
```sql
-- Acessar target
{{ target.name }}      -- dev, prod
{{ target.schema }}    -- schema atual
{{ target.database }}  -- database atual

-- Condicional por ambiente
{% if target.name == 'prod' %}
    -- lógica de produção
{% endif %}
```
{% endraw %}

## Links

- [dbt Docs](https://docs.getdbt.com/){:target="_blank"}
- [dbt Learn](https://courses.getdbt.com/){:target="_blank"}
- [dbt Hub](https://hub.getdbt.com/){:target="_blank"}
- [dbt Utils](https://github.com/dbt-labs/dbt-utils){:target="_blank"}
