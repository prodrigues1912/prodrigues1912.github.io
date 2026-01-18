---
date: 2025-01-18
categories:
  - SQL
  - Banco de Dados
---

# T-SQL: referência rápida

Comandos e padrões úteis para SQL Server.

<!-- more -->

??? note "Script para criar tabelas usadas nos exemplos"

    ```sql
    -- Departamentos
    CREATE TABLE departamentos (
        id INT PRIMARY KEY IDENTITY(1,1),
        nome VARCHAR(100) NOT NULL
    );

    INSERT INTO departamentos (nome) VALUES
    ('Engenharia'), ('Vendas'), ('RH'), ('Financeiro');

    -- Funcionários
    CREATE TABLE funcionarios (
        id INT PRIMARY KEY IDENTITY(1,1),
        nome VARCHAR(100) NOT NULL,
        email VARCHAR(100),
        departamento_id INT REFERENCES departamentos(id),
        gerente_id INT REFERENCES funcionarios(id),
        salario DECIMAL(10,2),
        data_contratacao DATE
    );

    INSERT INTO funcionarios (nome, email, departamento_id, gerente_id, salario, data_contratacao) VALUES
    ('Ana Silva', 'ana@empresa.com', 1, NULL, 15000.00, '2020-01-15'),
    ('Bruno Costa', 'bruno@empresa.com', 1, 1, 8000.00, '2021-03-10'),
    ('Carla Santos', 'carla@empresa.com', 1, 1, 9500.00, '2020-06-20'),
    ('Daniel Oliveira', 'daniel@empresa.com', 2, NULL, 12000.00, '2019-11-01'),
    ('Eva Lima', 'eva@empresa.com', 2, 4, 7500.00, '2022-02-14'),
    ('Fernando Dias', 'fernando@empresa.com', 3, NULL, 10000.00, '2021-08-30'),
    ('Gisele Ramos', 'gisele@empresa.com', 4, NULL, 11000.00, '2020-04-25');

    -- Clientes
    CREATE TABLE clientes (
        id INT PRIMARY KEY IDENTITY(1,1),
        nome VARCHAR(100) NOT NULL,
        email VARCHAR(100),
        cidade VARCHAR(50)
    );

    INSERT INTO clientes (nome, email, cidade) VALUES
    ('Tech Solutions', 'contato@techsol.com', 'São Paulo'),
    ('Data Corp', 'info@datacorp.com', 'Rio de Janeiro'),
    ('Cloud Services', 'hello@cloudserv.com', 'Belo Horizonte'),
    ('Digital Labs', 'contact@digilabs.com', 'São Paulo');

    -- Produtos
    CREATE TABLE produtos (
        id INT PRIMARY KEY IDENTITY(1,1),
        nome VARCHAR(100) NOT NULL,
        preco DECIMAL(10,2),
        estoque INT DEFAULT 0
    );

    INSERT INTO produtos (nome, preco, estoque) VALUES
    ('Licença Software A', 500.00, 100),
    ('Licença Software B', 1200.00, 50),
    ('Consultoria (hora)', 250.00, 999),
    ('Suporte Anual', 3000.00, 999);

    -- Pedidos
    CREATE TABLE pedidos (
        id INT PRIMARY KEY IDENTITY(1,1),
        cliente_id INT REFERENCES clientes(id),
        data DATE NOT NULL,
        status VARCHAR(20) DEFAULT 'pendente'
    );

    INSERT INTO pedidos (cliente_id, data, status) VALUES
    (1, '2025-01-05', 'concluido'),
    (1, '2025-01-10', 'concluido'),
    (2, '2025-01-12', 'pendente'),
    (3, '2025-01-15', 'pendente'),
    (4, '2025-01-16', 'cancelado');

    -- Itens do pedido
    CREATE TABLE itens_pedido (
        id INT PRIMARY KEY IDENTITY(1,1),
        pedido_id INT REFERENCES pedidos(id),
        produto_id INT REFERENCES produtos(id),
        quantidade INT,
        valor_unitario DECIMAL(10,2)
    );

    INSERT INTO itens_pedido (pedido_id, produto_id, quantidade, valor_unitario) VALUES
    (1, 1, 10, 500.00),
    (1, 4, 1, 3000.00),
    (2, 2, 5, 1200.00),
    (2, 3, 8, 250.00),
    (3, 1, 20, 500.00),
    (4, 2, 3, 1200.00),
    (5, 3, 4, 250.00);

    -- Transações (para exemplos de running total)
    CREATE TABLE transacoes (
        id INT PRIMARY KEY IDENTITY(1,1),
        data DATE,
        valor DECIMAL(10,2)
    );

    INSERT INTO transacoes (data, valor) VALUES
    ('2025-01-01', 1000.00),
    ('2025-01-02', -200.00),
    ('2025-01-03', 500.00),
    ('2025-01-04', -100.00),
    ('2025-01-05', 800.00);

    -- Contas (para exemplo de transação)
    CREATE TABLE contas (
        id INT PRIMARY KEY,
        titular VARCHAR(100),
        saldo DECIMAL(10,2)
    );

    INSERT INTO contas (id, titular, saldo) VALUES
    (1, 'Conta A', 5000.00),
    (2, 'Conta B', 3000.00);

    -- Vendas (para PIVOT)
    CREATE TABLE vendas (
        id INT PRIMARY KEY IDENTITY(1,1),
        produto VARCHAR(50),
        mes VARCHAR(3),
        quantidade INT
    );

    INSERT INTO vendas (produto, mes, quantidade) VALUES
    ('Produto X', 'Jan', 100),
    ('Produto X', 'Fev', 150),
    ('Produto X', 'Mar', 120),
    ('Produto Y', 'Jan', 80),
    ('Produto Y', 'Fev', 90),
    ('Produto Y', 'Mar', 110);

    -- Staging clientes (para MERGE)
    CREATE TABLE staging_clientes (
        id INT PRIMARY KEY,
        nome VARCHAR(100),
        email VARCHAR(100)
    );

    INSERT INTO staging_clientes (id, nome, email) VALUES
    (1, 'Tech Solutions Ltda', 'novo@techsol.com'),  -- update
    (5, 'New Company', 'hello@newco.com');            -- insert
    ```

## Tipos de dados

```sql
-- Strings
CHAR(10)          -- fixo, 10 caracteres
VARCHAR(100)      -- variável, até 100
VARCHAR(MAX)      -- até 2GB
NVARCHAR(100)     -- unicode

-- Números
INT               -- -2bi a 2bi
BIGINT            -- muito grande
DECIMAL(10,2)     -- precisão exata (dinheiro)
FLOAT             -- aproximado

-- Data/hora
DATE              -- só data
TIME              -- só hora
DATETIME2         -- data + hora (preferir sobre DATETIME)
DATETIMEOFFSET    -- com timezone

-- Outros
BIT               -- boolean (0/1)
UNIQUEIDENTIFIER  -- GUID
```

## CTEs

```sql
WITH vendas_mes AS (
    SELECT
        cliente_id,
        SUM(valor) AS total
    FROM pedidos
    WHERE data >= DATEADD(MONTH, -1, GETDATE())
    GROUP BY cliente_id
)
SELECT * FROM vendas_mes WHERE total > 1000;
```

### CTE recursiva

```sql
WITH hierarquia AS (
    -- Âncora
    SELECT id, nome, gerente_id, 0 AS nivel
    FROM funcionarios
    WHERE gerente_id IS NULL

    UNION ALL

    -- Recursão
    SELECT f.id, f.nome, f.gerente_id, h.nivel + 1
    FROM funcionarios f
    INNER JOIN hierarquia h ON f.gerente_id = h.id
)
SELECT * FROM hierarquia;
```

## Window functions

```sql
SELECT
    nome,
    departamento,
    salario,
    ROW_NUMBER() OVER (ORDER BY salario DESC) AS ranking,
    RANK() OVER (ORDER BY salario DESC) AS rank_com_gaps,
    DENSE_RANK() OVER (ORDER BY salario DESC) AS rank_sem_gaps,
    SUM(salario) OVER (PARTITION BY departamento) AS total_dept,
    AVG(salario) OVER () AS media_geral,
    LAG(salario) OVER (ORDER BY data_contratacao) AS salario_anterior,
    LEAD(salario) OVER (ORDER BY data_contratacao) AS salario_proximo
FROM funcionarios;
```

### Running total

```sql
SELECT
    data,
    valor,
    SUM(valor) OVER (ORDER BY data ROWS UNBOUNDED PRECEDING) AS acumulado
FROM transacoes;
```

## MERGE (upsert)

```sql
MERGE INTO clientes AS target
USING staging_clientes AS source
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET
        nome = source.nome,
        email = source.email,
        atualizado_em = GETDATE()
WHEN NOT MATCHED THEN
    INSERT (id, nome, email, criado_em)
    VALUES (source.id, source.nome, source.email, GETDATE())
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

## PIVOT / UNPIVOT

```sql
-- Linhas para colunas
SELECT *
FROM (
    SELECT produto, mes, quantidade
    FROM vendas
) AS src
PIVOT (
    SUM(quantidade)
    FOR mes IN ([Jan], [Fev], [Mar])
) AS pvt;

-- Colunas para linhas
SELECT produto, mes, quantidade
FROM vendas_pivot
UNPIVOT (
    quantidade FOR mes IN ([Jan], [Fev], [Mar])
) AS unpvt;
```

## STRING_AGG

```sql
-- Concatenar valores em uma string
SELECT
    departamento,
    STRING_AGG(nome, ', ') AS funcionarios
FROM funcionarios
GROUP BY departamento;

-- Com ordenação
SELECT
    pedido_id,
    STRING_AGG(produto, ', ') WITHIN GROUP (ORDER BY produto) AS produtos
FROM itens_pedido
GROUP BY pedido_id;
```

## JSON

```sql
-- Gerar JSON
SELECT id, nome, email
FROM clientes
FOR JSON PATH;

-- Ler JSON
SELECT *
FROM OPENJSON(@json)
WITH (
    id INT '$.id',
    nome VARCHAR(100) '$.nome',
    email VARCHAR(100) '$.email'
);

-- Extrair valor
SELECT JSON_VALUE(@json, '$.nome');
```

## Funções de data

```sql
GETDATE()                              -- agora
GETUTCDATE()                           -- agora UTC
DATEADD(DAY, 7, GETDATE())             -- +7 dias
DATEDIFF(DAY, data_inicio, data_fim)   -- diferença em dias
EOMONTH(GETDATE())                     -- último dia do mês
DATEFROMPARTS(2025, 1, 15)             -- construir data
FORMAT(GETDATE(), 'yyyy-MM-dd')        -- formatar
```

## TRY_CAST / TRY_CONVERT

```sql
-- Retorna NULL em vez de erro
SELECT TRY_CAST('abc' AS INT);  -- NULL
SELECT TRY_CAST('123' AS INT);  -- 123

SELECT TRY_CONVERT(DATE, '2025-01-15');  -- funciona
SELECT TRY_CONVERT(DATE, 'invalid');     -- NULL
```

## COALESCE / ISNULL / NULLIF

```sql
COALESCE(a, b, c)      -- primeiro não-nulo
ISNULL(a, b)           -- se a é null, retorna b
NULLIF(a, b)           -- se a = b, retorna null
```

## IIF / CASE

```sql
-- IIF (ternário)
SELECT IIF(quantidade > 0, 'Em estoque', 'Esgotado') AS status;

-- CASE
SELECT
    CASE
        WHEN nota >= 9 THEN 'A'
        WHEN nota >= 7 THEN 'B'
        WHEN nota >= 5 THEN 'C'
        ELSE 'F'
    END AS conceito
FROM alunos;
```

## OFFSET/FETCH (paginação)

```sql
SELECT *
FROM produtos
ORDER BY nome
OFFSET 20 ROWS
FETCH NEXT 10 ROWS ONLY;
```

## Tabelas temporárias

```sql
-- Local (só na sessão)
CREATE TABLE #temp (
    id INT,
    nome VARCHAR(100)
);

-- Global (todas as sessões)
CREATE TABLE ##temp_global (
    id INT
);

-- Variável de tabela
DECLARE @tabela TABLE (
    id INT,
    nome VARCHAR(100)
);
```

## Transações

```sql
BEGIN TRY
    BEGIN TRANSACTION;

    UPDATE contas SET saldo = saldo - 100 WHERE id = 1;
    UPDATE contas SET saldo = saldo + 100 WHERE id = 2;

    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    THROW;
END CATCH;
```

## Índices

```sql
-- Índice simples
CREATE INDEX IX_pedidos_cliente ON pedidos(cliente_id);

-- Índice composto
CREATE INDEX IX_pedidos_data_cliente ON pedidos(data, cliente_id);

-- Índice com INCLUDE
CREATE INDEX IX_pedidos_cliente_inc
ON pedidos(cliente_id)
INCLUDE (valor, status);

-- Índice filtrado
CREATE INDEX IX_pedidos_pendentes
ON pedidos(data)
WHERE status = 'pendente';

-- Ver fragmentação
SELECT
    index_id,
    avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('pedidos'), NULL, NULL, 'LIMITED');
```

## Execution plan

```sql
-- Ver plano estimado
SET SHOWPLAN_TEXT ON;
GO
SELECT * FROM pedidos WHERE cliente_id = 1;
GO
SET SHOWPLAN_TEXT OFF;

-- Ver plano real com estatísticas
SET STATISTICS IO ON;
SET STATISTICS TIME ON;
SELECT * FROM pedidos WHERE cliente_id = 1;
```

## Queries úteis

### Tabelas e tamanhos

```sql
SELECT
    t.name AS tabela,
    SUM(p.rows) AS linhas,
    SUM(a.total_pages) * 8 / 1024 AS tamanho_mb
FROM sys.tables t
INNER JOIN sys.partitions p ON t.object_id = p.object_id
INNER JOIN sys.allocation_units a ON p.partition_id = a.container_id
WHERE p.index_id IN (0, 1)
GROUP BY t.name
ORDER BY tamanho_mb DESC;
```

### Queries lentas

```sql
SELECT TOP 10
    total_worker_time / execution_count AS avg_cpu,
    execution_count,
    SUBSTRING(st.text, 1, 200) AS query
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY avg_cpu DESC;
```

### Locks ativos

```sql
SELECT
    l.request_session_id,
    l.resource_type,
    l.request_mode,
    l.request_status,
    OBJECT_NAME(p.object_id) AS tabela
FROM sys.dm_tran_locks l
LEFT JOIN sys.partitions p ON l.resource_associated_entity_id = p.hobt_id
WHERE l.resource_database_id = DB_ID();
```

## Links

- [T-SQL Reference](https://learn.microsoft.com/en-us/sql/t-sql/language-reference)
- [Query Store](https://learn.microsoft.com/en-us/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store)
