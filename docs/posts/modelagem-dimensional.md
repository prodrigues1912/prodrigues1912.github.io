---
date: 2025-10-26
categories:
  - Engenharia de Dados
  - SQL
---

# Modelagem dimensional com SQL Server

Conceitos básicos de modelagem dimensional usando a stack Microsoft: SQL Server, SSIS e SSAS.

<!-- more -->

## O que é modelagem dimensional?

Técnica de organização de dados otimizada para análises. Diferente do modelo relacional (normalizado), prioriza:

- Facilidade de entendimento
- Performance de queries
- Flexibilidade para análises

## Conceitos fundamentais

### Fatos (Facts)

Tabelas que armazenam **eventos** e **métricas**. Centro do modelo.

- Registram eventos no tempo
- Contêm métricas numéricas (quantidade, valor)
- Têm muitas linhas
- Chaves estrangeiras para dimensões

### Dimensões (Dimensions)

Tabelas que descrevem o **contexto** dos fatos. O "quem, o quê, onde, quando".

- Atributos descritivos
- Relativamente poucas linhas
- Permitem filtrar e agrupar fatos

### Surrogate Keys

Chaves artificiais (INT, sequenciais) em vez de chaves naturais:

```sql
-- Chave natural
customer_id VARCHAR(10)  -- "C001"

-- Surrogate key (preferir)
customer_key INT  -- 101
```

Vantagens: performance em JOINs, suporte a histórico, independência do sistema fonte.

## Star Schema

Padrão mais comum. Fato no centro, dimensões ao redor.

```
                    dim_date
                       │
                       │
    dim_customer ──── fact_orders ──── dim_product
                       │
                       │
                    dim_store
```

## Criando o modelo no SQL Server

### Dimensão de data

```sql
CREATE TABLE dbo.dim_date (
    date_key INT PRIMARY KEY,  -- YYYYMMDD
    full_date DATE NOT NULL,
    day_of_week TINYINT,
    day_name VARCHAR(10),
    day_of_month TINYINT,
    day_of_year SMALLINT,
    week_of_year TINYINT,
    month_num TINYINT,
    month_name VARCHAR(10),
    quarter TINYINT,
    year SMALLINT,
    is_weekend BIT,
    is_holiday BIT
);
GO

-- Popular dimensão de data (2020-2030)
DECLARE @start DATE = '2020-01-01';
DECLARE @end DATE = '2030-12-31';

WITH dates AS (
    SELECT @start AS dt
    UNION ALL
    SELECT DATEADD(DAY, 1, dt) FROM dates WHERE dt < @end
)
INSERT INTO dbo.dim_date
SELECT
    CONVERT(INT, FORMAT(dt, 'yyyyMMdd')) AS date_key,
    dt AS full_date,
    DATEPART(WEEKDAY, dt) AS day_of_week,
    DATENAME(WEEKDAY, dt) AS day_name,
    DAY(dt) AS day_of_month,
    DATEPART(DAYOFYEAR, dt) AS day_of_year,
    DATEPART(WEEK, dt) AS week_of_year,
    MONTH(dt) AS month_num,
    DATENAME(MONTH, dt) AS month_name,
    DATEPART(QUARTER, dt) AS quarter,
    YEAR(dt) AS year,
    CASE WHEN DATEPART(WEEKDAY, dt) IN (1, 7) THEN 1 ELSE 0 END AS is_weekend,
    0 AS is_holiday
FROM dates
OPTION (MAXRECURSION 0);
```

### Dimensão de cliente

```sql
CREATE TABLE dbo.dim_customer (
    customer_key INT IDENTITY(1,1) PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL,
    name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(2),
    segment VARCHAR(20),
    created_at DATETIME2,
    -- SCD Type 2
    valid_from DATE NOT NULL,
    valid_to DATE NOT NULL,
    is_current BIT NOT NULL DEFAULT 1
);
GO

CREATE INDEX IX_dim_customer_id ON dbo.dim_customer(customer_id);
CREATE INDEX IX_dim_customer_current ON dbo.dim_customer(is_current) WHERE is_current = 1;
```

### Dimensão de produto

```sql
CREATE TABLE dbo.dim_product (
    product_key INT IDENTITY(1,1) PRIMARY KEY,
    product_id VARCHAR(50) NOT NULL,
    name VARCHAR(100),
    category VARCHAR(50),
    subcategory VARCHAR(50),
    brand VARCHAR(50),
    unit_price DECIMAL(10,2),
    valid_from DATE NOT NULL,
    valid_to DATE NOT NULL,
    is_current BIT NOT NULL DEFAULT 1
);
GO
```

### Tabela de fatos

```sql
CREATE TABLE dbo.fact_sales (
    sale_key BIGINT IDENTITY(1,1) PRIMARY KEY,
    order_id VARCHAR(50) NOT NULL,  -- degenerate dimension
    customer_key INT NOT NULL REFERENCES dbo.dim_customer(customer_key),
    product_key INT NOT NULL REFERENCES dbo.dim_product(product_key),
    date_key INT NOT NULL REFERENCES dbo.dim_date(date_key),
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    discount_amount DECIMAL(10,2) DEFAULT 0,
    total_amount DECIMAL(10,2) NOT NULL
);
GO

-- Índices para performance
CREATE INDEX IX_fact_sales_date ON dbo.fact_sales(date_key);
CREATE INDEX IX_fact_sales_customer ON dbo.fact_sales(customer_key);
CREATE INDEX IX_fact_sales_product ON dbo.fact_sales(product_key);
```

## ETL com SSIS

SSIS (SQL Server Integration Services) para extrair, transformar e carregar dados.

### Estrutura do pacote

```
Package.dtsx
├── Connection Managers
│   ├── Source_OLTP (conexão com sistema transacional)
│   └── DW (conexão com data warehouse)
│
├── Control Flow
│   ├── Sequence: Load Dimensions
│   │   ├── Data Flow: Load dim_customer
│   │   └── Data Flow: Load dim_product
│   │
│   └── Sequence: Load Facts
│       └── Data Flow: Load fact_sales
```

### Data Flow: Carga de dimensão (SCD Type 2)

```
[OLE DB Source] → [Lookup] → [Conditional Split] → [OLE DB Destination]
    (OLTP)         (DW)         (novo/alterado)       (INSERT)
                                      │
                                      └──────────→ [OLE DB Command]
                                                      (UPDATE is_current=0)
```

!!! tip "Componente SCD no SSIS"
    Use o componente **Slowly Changing Dimension** do SSIS para automatizar a lógica de SCD Type 1, 2 ou 3.

### Script: Carga incremental de fatos

```sql
-- Procedure para carga incremental
CREATE PROCEDURE dbo.usp_load_fact_sales
    @date_from DATE,
    @date_to DATE
AS
BEGIN
    SET NOCOUNT ON;

    INSERT INTO dbo.fact_sales (
        order_id, customer_key, product_key, date_key,
        quantity, unit_price, discount_amount, total_amount
    )
    SELECT
        s.order_id,
        COALESCE(c.customer_key, -1) AS customer_key,  -- -1 = não encontrado
        COALESCE(p.product_key, -1) AS product_key,
        CONVERT(INT, FORMAT(s.order_date, 'yyyyMMdd')) AS date_key,
        s.quantity,
        s.unit_price,
        s.discount_amount,
        s.quantity * s.unit_price - s.discount_amount AS total_amount
    FROM staging.sales s
    LEFT JOIN dbo.dim_customer c
        ON s.customer_id = c.customer_id
        AND c.is_current = 1
    LEFT JOIN dbo.dim_product p
        ON s.product_id = p.product_id
        AND p.is_current = 1
    WHERE s.order_date BETWEEN @date_from AND @date_to
      AND NOT EXISTS (
          SELECT 1 FROM dbo.fact_sales f
          WHERE f.order_id = s.order_id
      );
END;
```

### SQL Server Agent Job

Agendar execução diária:

```sql
-- Criar job para carga diária
EXEC msdb.dbo.sp_add_job
    @job_name = N'DW_Daily_Load';

EXEC msdb.dbo.sp_add_jobstep
    @job_name = N'DW_Daily_Load',
    @step_name = N'Execute SSIS Package',
    @subsystem = N'SSIS',
    @command = N'/FILE "C:\SSIS\DW_Load.dtsx"';

EXEC msdb.dbo.sp_add_schedule
    @schedule_name = N'Daily_2AM',
    @freq_type = 4,  -- daily
    @active_start_time = 020000;  -- 02:00
```

## Cubo OLAP com SSAS

SSAS (SQL Server Analysis Services) para criar cubos multidimensionais.

### Estrutura do projeto SSAS

```
Projeto.dwproj
├── Data Sources
│   └── DW.ds
├── Data Source Views
│   └── DW.dsv
├── Cubes
│   └── Sales.cube
├── Dimensions
│   ├── Date.dim
│   ├── Customer.dim
│   └── Product.dim
└── Mining Structures (opcional)
```

### Definindo dimensões no SSAS

No Data Source View, arraste as tabelas de dimensão e fato. O SSAS detecta relacionamentos automaticamente.

**Dimensão de data (configuração):**

| Propriedade | Valor |
|-------------|-------|
| KeyColumn | date_key |
| NameColumn | full_date |
| Type | Time |
| Hierarchies | Year → Quarter → Month → Date |

**Dimensão de cliente:**

| Propriedade | Valor |
|-------------|-------|
| KeyColumn | customer_key |
| NameColumn | name |
| Hierarchies | State → City → Customer |

### Medidas do cubo

```
Measure Group: fact_sales
├── Sum of quantity
├── Sum of total_amount
├── Count of sale_key (distinct count)
└── Average unit_price
```

### MDX básico

```mdx
-- Total de vendas por ano
SELECT
    [Measures].[Total Amount] ON COLUMNS,
    [Date].[Year].Members ON ROWS
FROM [Sales];

-- Top 10 clientes
SELECT
    [Measures].[Total Amount] ON COLUMNS,
    TOPCOUNT(
        [Customer].[Name].Members,
        10,
        [Measures].[Total Amount]
    ) ON ROWS
FROM [Sales];

-- Vendas do ano atual vs ano anterior
WITH
    MEMBER [Measures].[PY Amount] AS
        ([Measures].[Total Amount], [Date].[Year].CurrentMember.PrevMember)
    MEMBER [Measures].[YoY Growth] AS
        ([Measures].[Total Amount] - [Measures].[PY Amount]) / [Measures].[PY Amount]
SELECT
    {[Measures].[Total Amount], [Measures].[PY Amount], [Measures].[YoY Growth]} ON COLUMNS,
    [Date].[Year].Members ON ROWS
FROM [Sales];
```

## Slowly Changing Dimensions (SCD)

SCD (Slowly Changing Dimension) é a técnica para lidar com mudanças em atributos de dimensões ao longo do tempo. Por exemplo, quando um cliente muda de cidade ou um produto muda de categoria.

| Tipo | O que faz | Histórico |
|------|-----------|-----------|
| **Type 1** | Sobrescreve o valor antigo | Não mantém |
| **Type 2** | Cria nova linha com versão | Mantém completo |
| **Type 3** | Adiciona coluna para valor anterior | Limitado (só 1 versão) |

**Type 2 é o mais usado** quando você precisa analisar dados históricos com o contexto correto (ex: "vendas por região do cliente *na época da venda*").

### Type 1: Sobrescrever

Atualiza o registro. Perde histórico.

```sql
UPDATE dbo.dim_customer
SET city = 'Rio de Janeiro'
WHERE customer_id = 'C001' AND is_current = 1;
```

### Type 2: Nova linha

Cria novo registro. Mantém histórico completo.

```sql
-- 1. Fechar registro atual
UPDATE dbo.dim_customer
SET valid_to = CAST(GETDATE() AS DATE),
    is_current = 0
WHERE customer_id = 'C001' AND is_current = 1;

-- 2. Inserir novo registro
INSERT INTO dbo.dim_customer (customer_id, name, city, valid_from, valid_to, is_current)
VALUES ('C001', 'João', 'Rio de Janeiro', CAST(GETDATE() AS DATE), '9999-12-31', 1);
```

### Merge para SCD Type 2

```sql
-- Procedure genérica para SCD Type 2
CREATE PROCEDURE dbo.usp_scd2_customer
AS
BEGIN
    -- Fechar registros alterados
    UPDATE dim
    SET valid_to = CAST(GETDATE() AS DATE),
        is_current = 0
    FROM dbo.dim_customer dim
    INNER JOIN staging.customer stg ON dim.customer_id = stg.customer_id
    WHERE dim.is_current = 1
      AND (dim.name <> stg.name OR dim.city <> stg.city OR dim.segment <> stg.segment);

    -- Inserir novos e alterados
    INSERT INTO dbo.dim_customer (customer_id, name, email, city, state, segment, valid_from, valid_to, is_current)
    SELECT
        stg.customer_id, stg.name, stg.email, stg.city, stg.state, stg.segment,
        CAST(GETDATE() AS DATE), '9999-12-31', 1
    FROM staging.customer stg
    LEFT JOIN dbo.dim_customer dim
        ON stg.customer_id = dim.customer_id AND dim.is_current = 1
    WHERE dim.customer_key IS NULL;  -- novo ou foi fechado acima
END;
```

## Query analítica

```sql
-- Vendas por categoria e mês
SELECT
    p.category,
    d.month_name,
    d.year,
    SUM(f.quantity) AS total_quantity,
    SUM(f.total_amount) AS total_revenue,
    COUNT(DISTINCT f.order_id) AS total_orders
FROM dbo.fact_sales f
INNER JOIN dbo.dim_product p ON f.product_key = p.product_key
INNER JOIN dbo.dim_date d ON f.date_key = d.date_key
WHERE d.year = 2025
GROUP BY p.category, d.month_name, d.year, d.month_num
ORDER BY d.month_num, total_revenue DESC;
```

## Boas práticas

### Índices

```sql
-- Columnstore para fatos (SQL Server 2016+)
CREATE CLUSTERED COLUMNSTORE INDEX CCI_fact_sales ON dbo.fact_sales;

-- Ou índice columnstore não-clusterizado
CREATE NONCLUSTERED COLUMNSTORE INDEX NCCI_fact_sales
ON dbo.fact_sales (customer_key, product_key, date_key, quantity, total_amount);
```

### Particionamento

```sql
-- Particionar fato por ano
CREATE PARTITION FUNCTION pf_year (INT)
AS RANGE RIGHT FOR VALUES (20210101, 20220101, 20230101, 20240101, 20250101);

CREATE PARTITION SCHEME ps_year
AS PARTITION pf_year ALL TO ([PRIMARY]);

CREATE TABLE dbo.fact_sales (
    -- colunas
) ON ps_year(date_key);
```

### Registro "desconhecido" nas dimensões

```sql
-- Inserir registro para chaves não encontradas
SET IDENTITY_INSERT dbo.dim_customer ON;
INSERT INTO dbo.dim_customer (customer_key, customer_id, name, valid_from, valid_to, is_current)
VALUES (-1, 'UNKNOWN', 'Unknown Customer', '1900-01-01', '9999-12-31', 1);
SET IDENTITY_INSERT dbo.dim_customer OFF;
```

## Ferramentas complementares

| Ferramenta | Uso |
|------------|-----|
| **SSIS** | ETL (extração, transformação, carga) |
| **SSAS** | Cubos OLAP e modelos tabulares |
| **SSRS** | Relatórios |
| **Power BI** | Dashboards e self-service BI |

## Links

- [Kimball Group](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/){:target="_blank"}
- [SSIS Docs](https://learn.microsoft.com/en-us/sql/integration-services/){:target="_blank"}
- [SSAS Docs](https://learn.microsoft.com/en-us/analysis-services/){:target="_blank"}

