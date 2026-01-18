---
date: 2025-04-06
categories:
  - SQL
  - Banco de Dados
---

# PostgreSQL: referência rápida

Comandos e padrões úteis para PostgreSQL.

<!-- more -->

??? note "Script para criar tabelas usadas nos exemplos"

    ```sql
    -- Departamentos
    CREATE TABLE departamentos (
        id SERIAL PRIMARY KEY,
        nome VARCHAR(100) NOT NULL
    );

    INSERT INTO departamentos (nome) VALUES
    ('Engenharia'), ('Vendas'), ('RH'), ('Financeiro');

    -- Funcionários
    CREATE TABLE funcionarios (
        id SERIAL PRIMARY KEY,
        nome VARCHAR(100) NOT NULL,
        email VARCHAR(100),
        departamento_id INT REFERENCES departamentos(id),
        gerente_id INT REFERENCES funcionarios(id),
        salario NUMERIC(10,2),
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
        id SERIAL PRIMARY KEY,
        nome VARCHAR(100) NOT NULL,
        email VARCHAR(100),
        cidade VARCHAR(50),
        metadata JSONB DEFAULT '{}'
    );

    INSERT INTO clientes (nome, email, cidade, metadata) VALUES
    ('Tech Solutions', 'contato@techsol.com', 'São Paulo', '{"plano": "enterprise", "ativo": true}'),
    ('Data Corp', 'info@datacorp.com', 'Rio de Janeiro', '{"plano": "pro", "ativo": true}'),
    ('Cloud Services', 'hello@cloudserv.com', 'Belo Horizonte', '{"plano": "starter", "ativo": false}'),
    ('Digital Labs', 'contact@digilabs.com', 'São Paulo', '{"plano": "pro", "ativo": true}');

    -- Produtos
    CREATE TABLE produtos (
        id SERIAL PRIMARY KEY,
        nome VARCHAR(100) NOT NULL,
        preco NUMERIC(10,2),
        estoque INT DEFAULT 0,
        tags TEXT[]
    );

    INSERT INTO produtos (nome, preco, estoque, tags) VALUES
    ('Licença Software A', 500.00, 100, ARRAY['software', 'licença']),
    ('Licença Software B', 1200.00, 50, ARRAY['software', 'licença', 'premium']),
    ('Consultoria (hora)', 250.00, 999, ARRAY['serviço', 'consultoria']),
    ('Suporte Anual', 3000.00, 999, ARRAY['serviço', 'suporte']);

    -- Pedidos
    CREATE TABLE pedidos (
        id SERIAL PRIMARY KEY,
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
        id SERIAL PRIMARY KEY,
        pedido_id INT REFERENCES pedidos(id),
        produto_id INT REFERENCES produtos(id),
        quantidade INT,
        valor_unitario NUMERIC(10,2)
    );

    INSERT INTO itens_pedido (pedido_id, produto_id, quantidade, valor_unitario) VALUES
    (1, 1, 10, 500.00),
    (1, 4, 1, 3000.00),
    (2, 2, 5, 1200.00),
    (2, 3, 8, 250.00),
    (3, 1, 20, 500.00),
    (4, 2, 3, 1200.00),
    (5, 3, 4, 250.00);

    -- Transações
    CREATE TABLE transacoes (
        id SERIAL PRIMARY KEY,
        data DATE,
        valor NUMERIC(10,2)
    );

    INSERT INTO transacoes (data, valor) VALUES
    ('2025-01-01', 1000.00),
    ('2025-01-02', -200.00),
    ('2025-01-03', 500.00),
    ('2025-01-04', -100.00),
    ('2025-01-05', 800.00);

    -- Contas
    CREATE TABLE contas (
        id INT PRIMARY KEY,
        titular VARCHAR(100),
        saldo NUMERIC(10,2)
    );

    INSERT INTO contas (id, titular, saldo) VALUES
    (1, 'Conta A', 5000.00),
    (2, 'Conta B', 3000.00);

    -- Logs (para particionamento)
    CREATE TABLE logs (
        id SERIAL,
        created_at TIMESTAMP NOT NULL,
        nivel VARCHAR(10),
        mensagem TEXT
    ) PARTITION BY RANGE (created_at);

    CREATE TABLE logs_2025_01 PARTITION OF logs
        FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
    ```

## Tipos de dados

```sql
-- Strings
CHAR(10)          -- fixo
VARCHAR(100)      -- variável
TEXT              -- ilimitado

-- Números
INT / INTEGER     -- -2bi a 2bi
BIGINT            -- muito grande
NUMERIC(10,2)     -- precisão exata (dinheiro)
REAL / FLOAT      -- aproximado

-- Data/hora
DATE              -- só data
TIME              -- só hora
TIMESTAMP         -- data + hora
TIMESTAMPTZ       -- com timezone (preferir)
INTERVAL          -- duração

-- Outros
BOOLEAN           -- true/false
UUID              -- identificador único
JSONB             -- JSON binário (preferir sobre JSON)
TEXT[]            -- array de texto
INT[]             -- array de inteiros
```

## CTEs

```sql
WITH vendas_mes AS (
    SELECT
        cliente_id,
        SUM(i.quantidade * i.valor_unitario) AS total
    FROM pedidos p
    JOIN itens_pedido i ON i.pedido_id = p.id
    WHERE p.data >= CURRENT_DATE - INTERVAL '1 month'
    GROUP BY cliente_id
)
SELECT * FROM vendas_mes WHERE total > 1000;
```

### CTE recursiva

```sql
WITH RECURSIVE hierarquia AS (
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
    departamento_id,
    salario,
    ROW_NUMBER() OVER (ORDER BY salario DESC) AS ranking,
    RANK() OVER (ORDER BY salario DESC) AS rank_com_gaps,
    DENSE_RANK() OVER (ORDER BY salario DESC) AS rank_sem_gaps,
    SUM(salario) OVER (PARTITION BY departamento_id) AS total_dept,
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

## UPSERT (INSERT ON CONFLICT)

```sql
INSERT INTO clientes (id, nome, email)
VALUES (1, 'Tech Solutions Ltda', 'novo@techsol.com')
ON CONFLICT (id) DO UPDATE SET
    nome = EXCLUDED.nome,
    email = EXCLUDED.email;

-- Ignorar conflito
INSERT INTO clientes (id, nome, email)
VALUES (1, 'Tech Solutions', 'contato@techsol.com')
ON CONFLICT DO NOTHING;
```

## RETURNING

```sql
-- Retorna dados após INSERT/UPDATE/DELETE
INSERT INTO clientes (nome, email)
VALUES ('Nova Empresa', 'contato@nova.com')
RETURNING id, nome;

UPDATE produtos
SET preco = preco * 1.1
WHERE estoque < 10
RETURNING id, nome, preco;

DELETE FROM pedidos
WHERE status = 'cancelado'
RETURNING *;
```

## JSONB

```sql
-- Criar
SELECT '{"nome": "João", "idade": 30}'::JSONB;

-- Acessar campo
SELECT metadata->>'plano' FROM clientes;          -- como texto
SELECT metadata->'plano' FROM clientes;           -- como jsonb

-- Acessar nested
SELECT metadata->'endereco'->>'cidade' FROM clientes;

-- Filtrar
SELECT * FROM clientes
WHERE metadata->>'plano' = 'enterprise';

SELECT * FROM clientes
WHERE metadata @> '{"ativo": true}';  -- contém

-- Atualizar campo
UPDATE clientes
SET metadata = jsonb_set(metadata, '{plano}', '"pro"')
WHERE id = 1;

-- Adicionar campo
UPDATE clientes
SET metadata = metadata || '{"desconto": 10}'
WHERE id = 1;

-- Remover campo
UPDATE clientes
SET metadata = metadata - 'desconto'
WHERE id = 1;

-- Agregar em JSON
SELECT jsonb_agg(nome) FROM clientes;
SELECT jsonb_object_agg(id, nome) FROM clientes;
```

## Arrays

```sql
-- Criar
SELECT ARRAY['a', 'b', 'c'];
SELECT '{a,b,c}'::TEXT[];

-- Acessar (1-indexed)
SELECT tags[1] FROM produtos;

-- Contém
SELECT * FROM produtos WHERE 'premium' = ANY(tags);
SELECT * FROM produtos WHERE tags @> ARRAY['software'];

-- Adicionar
UPDATE produtos
SET tags = array_append(tags, 'novo')
WHERE id = 1;

-- Remover
UPDATE produtos
SET tags = array_remove(tags, 'novo')
WHERE id = 1;

-- Expandir (unnest)
SELECT id, unnest(tags) AS tag FROM produtos;

-- Agregar
SELECT array_agg(nome) FROM clientes;
```

## STRING_AGG

```sql
SELECT
    departamento_id,
    STRING_AGG(nome, ', ' ORDER BY nome) AS funcionarios
FROM funcionarios
GROUP BY departamento_id;
```

## Funções de data

```sql
NOW()                                    -- timestamp atual
CURRENT_DATE                             -- só data
CURRENT_TIMESTAMP                        -- com timezone

-- Aritmética
CURRENT_DATE + INTERVAL '7 days'
CURRENT_DATE - INTERVAL '1 month'

-- Diferença
AGE(data_fim, data_inicio)               -- retorna interval
DATE_PART('day', AGE(data_fim, data_inicio))  -- em dias

-- Extrair
EXTRACT(YEAR FROM data)
EXTRACT(MONTH FROM data)
DATE_TRUNC('month', data)                -- truncar para início do mês

-- Construir
MAKE_DATE(2025, 1, 15)
TO_DATE('15/01/2025', 'DD/MM/YYYY')
TO_TIMESTAMP('2025-01-15 10:30', 'YYYY-MM-DD HH24:MI')

-- Formatar
TO_CHAR(NOW(), 'DD/MM/YYYY HH24:MI')
```

## COALESCE / NULLIF / CASE

```sql
COALESCE(a, b, c)      -- primeiro não-nulo
NULLIF(a, b)           -- se a = b, retorna null

CASE
    WHEN nota >= 9 THEN 'A'
    WHEN nota >= 7 THEN 'B'
    WHEN nota >= 5 THEN 'C'
    ELSE 'F'
END
```

## LIMIT / OFFSET (paginação)

```sql
SELECT *
FROM produtos
ORDER BY nome
LIMIT 10 OFFSET 20;
```

## LATERAL JOIN

```sql
-- Subquery correlacionada como JOIN
SELECT c.nome, ultimos.*
FROM clientes c
CROSS JOIN LATERAL (
    SELECT p.data, p.status
    FROM pedidos p
    WHERE p.cliente_id = c.id
    ORDER BY p.data DESC
    LIMIT 3
) ultimos;
```

## FILTER

```sql
SELECT
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE status = 'concluido') AS concluidos,
    COUNT(*) FILTER (WHERE status = 'pendente') AS pendentes,
    SUM(valor) FILTER (WHERE valor > 0) AS entradas
FROM pedidos p
JOIN itens_pedido i ON i.pedido_id = p.id;
```

## DISTINCT ON

```sql
-- Primeiro registro de cada grupo
SELECT DISTINCT ON (cliente_id)
    cliente_id, data, status
FROM pedidos
ORDER BY cliente_id, data DESC;
```

## Transações

```sql
BEGIN;

UPDATE contas SET saldo = saldo - 100 WHERE id = 1;
UPDATE contas SET saldo = saldo + 100 WHERE id = 2;

-- Se tudo ok
COMMIT;

-- Se erro
ROLLBACK;
```

### Savepoints

```sql
BEGIN;
UPDATE contas SET saldo = saldo - 100 WHERE id = 1;

SAVEPOINT antes_transferencia;

UPDATE contas SET saldo = saldo + 100 WHERE id = 2;

-- Desfaz só até o savepoint
ROLLBACK TO antes_transferencia;

COMMIT;
```

## Índices

```sql
-- B-tree (padrão)
CREATE INDEX idx_pedidos_cliente ON pedidos(cliente_id);

-- Composto
CREATE INDEX idx_pedidos_data_cliente ON pedidos(data, cliente_id);

-- Parcial
CREATE INDEX idx_pedidos_pendentes ON pedidos(data)
WHERE status = 'pendente';

-- JSONB
CREATE INDEX idx_clientes_metadata ON clientes USING GIN(metadata);

-- Array
CREATE INDEX idx_produtos_tags ON produtos USING GIN(tags);

-- Texto (busca)
CREATE INDEX idx_clientes_nome ON clientes USING GIN(nome gin_trgm_ops);

-- Unique
CREATE UNIQUE INDEX idx_clientes_email ON clientes(email);

-- Concurrent (não bloqueia)
CREATE INDEX CONCURRENTLY idx_nome ON tabela(coluna);
```

## EXPLAIN

```sql
-- Plano estimado
EXPLAIN SELECT * FROM pedidos WHERE cliente_id = 1;

-- Plano real com execução
EXPLAIN ANALYZE SELECT * FROM pedidos WHERE cliente_id = 1;

-- Com buffers e timing
EXPLAIN (ANALYZE, BUFFERS, TIMING)
SELECT * FROM pedidos WHERE cliente_id = 1;
```

## Queries úteis

### Tabelas e tamanhos

```sql
SELECT
    relname AS tabela,
    n_live_tup AS linhas,
    pg_size_pretty(pg_total_relation_size(relid)) AS tamanho
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(relid) DESC;
```

### Índices não utilizados

```sql
SELECT
    indexrelname AS indice,
    relname AS tabela,
    idx_scan AS scans,
    pg_size_pretty(pg_relation_size(indexrelid)) AS tamanho
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Queries lentas (requer pg_stat_statements)

```sql
SELECT
    calls,
    mean_exec_time::INT AS avg_ms,
    SUBSTRING(query, 1, 100) AS query
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

### Locks ativos

```sql
SELECT
    pid,
    mode,
    relation::regclass,
    query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE relation IS NOT NULL;
```

### Conexões ativas

```sql
SELECT
    state,
    COUNT(*) AS conexoes,
    MAX(NOW() - state_change) AS mais_antiga
FROM pg_stat_activity
WHERE datname = current_database()
GROUP BY state;
```

## Links

- [PostgreSQL Docs](https://www.postgresql.org/docs/current/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [Explain Visualizer](https://explain.depesz.com/)
