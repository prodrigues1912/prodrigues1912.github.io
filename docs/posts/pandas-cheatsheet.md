---
date: 2025-07-20
categories:
  - Python
  - Engenharia de Dados
---

# Pandas: referência rápida

Comandos e padrões para manipulação de dados com Pandas.

<!-- more -->

## Instalação

```bash
uv add pandas
# ou com todas as dependências opcionais
uv add "pandas[all]"
```

## Importação

```python
import pandas as pd
import numpy as np
```

## Criando DataFrames

```python
# De dicionário
df = pd.DataFrame({
    'nome': ['Ana', 'Bruno', 'Carla'],
    'idade': [25, 30, 28],
    'cidade': ['SP', 'RJ', 'BH']
})

# De lista de dicionários
df = pd.DataFrame([
    {'nome': 'Ana', 'idade': 25},
    {'nome': 'Bruno', 'idade': 30}
])

# De NumPy array
df = pd.DataFrame(
    np.random.randn(5, 3),
    columns=['A', 'B', 'C']
)
```

## Lendo dados

```python
# CSV
df = pd.read_csv('dados.csv')
df = pd.read_csv('dados.csv', sep=';', encoding='utf-8')
df = pd.read_csv('dados.csv', usecols=['col1', 'col2'])
df = pd.read_csv('dados.csv', dtype={'id': str})
df = pd.read_csv('dados.csv', parse_dates=['data'])
df = pd.read_csv('dados.csv', nrows=1000)  # primeiras N linhas

# Excel
df = pd.read_excel('dados.xlsx', sheet_name='Plan1')

# JSON
df = pd.read_json('dados.json')
df = pd.read_json('dados.json', lines=True)  # JSON Lines

# Parquet
df = pd.read_parquet('dados.parquet')

# SQL
from sqlalchemy import create_engine
engine = create_engine('postgresql://user:pass@host/db')
df = pd.read_sql('SELECT * FROM tabela', engine)
df = pd.read_sql_table('tabela', engine)
```

## Salvando dados

```python
# CSV
df.to_csv('saida.csv', index=False)
df.to_csv('saida.csv', index=False, sep=';')

# Excel
df.to_excel('saida.xlsx', index=False, sheet_name='Dados')

# JSON
df.to_json('saida.json', orient='records')
df.to_json('saida.json', orient='records', lines=True)

# Parquet
df.to_parquet('saida.parquet', index=False)

# SQL
df.to_sql('tabela', engine, if_exists='replace', index=False)
df.to_sql('tabela', engine, if_exists='append', index=False)
```

## Explorando dados

```python
df.head()          # primeiras 5 linhas
df.head(10)        # primeiras 10 linhas
df.tail()          # últimas 5 linhas
df.sample(5)       # 5 linhas aleatórias

df.shape           # (linhas, colunas)
df.columns         # nomes das colunas
df.dtypes          # tipos de cada coluna
df.info()          # resumo (tipos, não-nulos, memória)
df.describe()      # estatísticas numéricas

df['coluna'].value_counts()           # contagem de valores
df['coluna'].value_counts(normalize=True)  # percentual
df['coluna'].unique()                 # valores únicos
df['coluna'].nunique()                # quantidade de únicos

df.isnull().sum()       # nulos por coluna
df.duplicated().sum()   # linhas duplicadas
```

## Selecionando dados

```python
# Colunas
df['coluna']              # Series
df[['col1', 'col2']]      # DataFrame

# Linhas por índice
df.loc[0]                 # linha pelo label
df.loc[0:5]               # linhas 0 a 5 (inclusive)
df.loc[0:5, 'coluna']     # linhas e coluna

df.iloc[0]                # linha pela posição
df.iloc[0:5]              # linhas 0 a 4
df.iloc[0:5, 0:2]         # linhas e colunas por posição

# Filtros
df[df['idade'] > 25]
df[df['cidade'] == 'SP']
df[df['cidade'].isin(['SP', 'RJ'])]
df[df['nome'].str.contains('Ana')]
df[df['coluna'].notna()]
df[(df['idade'] > 25) & (df['cidade'] == 'SP')]
df[(df['idade'] > 25) | (df['cidade'] == 'RJ')]

# Query (mais legível)
df.query('idade > 25')
df.query('cidade == "SP"')
df.query('idade > 25 and cidade == "SP"')
```

## Modificando dados

```python
# Adicionar coluna
df['nova'] = df['col1'] + df['col2']
df['constante'] = 100
df['calculada'] = df['valor'] * 1.1

# Renomear colunas
df.rename(columns={'old': 'new'}, inplace=True)
df.columns = ['col1', 'col2', 'col3']

# Alterar tipos
df['coluna'] = df['coluna'].astype(int)
df['coluna'] = df['coluna'].astype(str)
df['data'] = pd.to_datetime(df['data'])
df['categoria'] = df['categoria'].astype('category')

# Alterar valores
df.loc[df['idade'] > 30, 'categoria'] = 'senior'
df['coluna'] = df['coluna'].replace({'old': 'new'})
df['coluna'] = df['coluna'].str.upper()
df['coluna'] = df['coluna'].str.strip()

# Apply
df['nova'] = df['coluna'].apply(lambda x: x * 2)
df['nova'] = df.apply(lambda row: row['a'] + row['b'], axis=1)

# Map
df['status_texto'] = df['status'].map({0: 'Inativo', 1: 'Ativo'})

# Dropar colunas
df.drop(columns=['col1', 'col2'], inplace=True)

# Dropar linhas
df.drop(index=[0, 1, 2], inplace=True)
```

## Lidando com nulos

```python
# Verificar
df.isnull()
df.isnull().sum()
df.isnull().any()

# Remover
df.dropna()                    # remove linhas com qualquer nulo
df.dropna(subset=['col1'])     # remove se col1 for nulo
df.dropna(how='all')           # remove se todas forem nulas

# Preencher
df.fillna(0)
df.fillna({'col1': 0, 'col2': 'N/A'})
df['coluna'].fillna(df['coluna'].mean())
df['coluna'].fillna(method='ffill')  # forward fill
df['coluna'].fillna(method='bfill')  # backward fill
```

## Duplicados

```python
# Verificar
df.duplicated()
df.duplicated(subset=['col1', 'col2'])

# Remover
df.drop_duplicates()
df.drop_duplicates(subset=['col1'], keep='first')
df.drop_duplicates(subset=['col1'], keep='last')
```

## Ordenação

```python
df.sort_values('coluna')
df.sort_values('coluna', ascending=False)
df.sort_values(['col1', 'col2'], ascending=[True, False])
df.sort_index()

# Rank
df['rank'] = df['valor'].rank()
df['rank'] = df['valor'].rank(ascending=False)
df['rank'] = df.groupby('grupo')['valor'].rank()
```

## Agregações

```python
# Básicas
df['coluna'].sum()
df['coluna'].mean()
df['coluna'].median()
df['coluna'].min()
df['coluna'].max()
df['coluna'].std()
df['coluna'].count()

# Múltiplas
df.agg({'col1': 'sum', 'col2': 'mean'})
df.agg({'col1': ['sum', 'mean'], 'col2': 'count'})
```

## GroupBy

```python
# Básico
df.groupby('categoria')['valor'].sum()
df.groupby('categoria')['valor'].mean()
df.groupby('categoria').size()

# Múltiplas colunas
df.groupby(['cat1', 'cat2'])['valor'].sum()

# Múltiplas agregações
df.groupby('categoria').agg({
    'valor': 'sum',
    'quantidade': 'mean',
    'id': 'count'
})

# Named aggregations
df.groupby('categoria').agg(
    total=('valor', 'sum'),
    media=('valor', 'mean'),
    contagem=('id', 'count')
)

# Transform (mantém shape original)
df['valor_normalizado'] = df.groupby('grupo')['valor'].transform(
    lambda x: (x - x.mean()) / x.std()
)

# Filter
df.groupby('grupo').filter(lambda x: x['valor'].sum() > 1000)
```

## Pivot e Melt

```python
# Pivot (long to wide)
df.pivot(index='data', columns='produto', values='vendas')

# Pivot table (com agregação)
df.pivot_table(
    index='data',
    columns='produto',
    values='vendas',
    aggfunc='sum',
    fill_value=0
)

# Melt (wide to long)
df.melt(
    id_vars=['data'],
    value_vars=['prod_a', 'prod_b'],
    var_name='produto',
    value_name='vendas'
)
```

## Joins

```python
# Merge (SQL-like)
pd.merge(df1, df2, on='id')
pd.merge(df1, df2, on='id', how='left')
pd.merge(df1, df2, on='id', how='right')
pd.merge(df1, df2, on='id', how='outer')
pd.merge(df1, df2, left_on='id1', right_on='id2')
pd.merge(df1, df2, on=['col1', 'col2'])

# Join (por índice)
df1.join(df2, how='left')

# Concat
pd.concat([df1, df2])                    # empilhar linhas
pd.concat([df1, df2], ignore_index=True)
pd.concat([df1, df2], axis=1)            # lado a lado
```

## Datas

```python
# Converter
df['data'] = pd.to_datetime(df['data'])
df['data'] = pd.to_datetime(df['data'], format='%d/%m/%Y')

# Extrair componentes
df['ano'] = df['data'].dt.year
df['mes'] = df['data'].dt.month
df['dia'] = df['data'].dt.day
df['dia_semana'] = df['data'].dt.dayofweek
df['nome_dia'] = df['data'].dt.day_name()
df['trimestre'] = df['data'].dt.quarter

# Aritmética
df['data'] + pd.Timedelta(days=7)
df['data'] - pd.Timedelta(hours=1)
(df['data_fim'] - df['data_inicio']).dt.days

# Filtrar
df[df['data'] > '2025-01-01']
df[df['data'].between('2025-01-01', '2025-12-31')]

# Resample (séries temporais)
df.set_index('data').resample('M')['valor'].sum()   # mensal
df.set_index('data').resample('W')['valor'].mean()  # semanal
df.set_index('data').resample('D')['valor'].sum()   # diário
```

## Strings

```python
# Acessar via .str
df['nome'].str.lower()
df['nome'].str.upper()
df['nome'].str.title()
df['nome'].str.strip()
df['nome'].str.replace('old', 'new')
df['nome'].str.contains('texto')
df['nome'].str.startswith('A')
df['nome'].str.endswith('a')
df['nome'].str.len()
df['nome'].str.split(',')
df['nome'].str.split(',').str[0]  # primeiro elemento

# Regex
df['nome'].str.extract(r'(\d+)')
df['nome'].str.findall(r'\d+')
df['nome'].str.replace(r'\s+', ' ', regex=True)
```

## Window functions

```python
# Rolling (janela móvel)
df['media_movel'] = df['valor'].rolling(window=7).mean()
df['soma_movel'] = df['valor'].rolling(window=7).sum()

# Expanding (acumulado)
df['acumulado'] = df['valor'].expanding().sum()
df['max_ate_aqui'] = df['valor'].expanding().max()

# Shift (lag/lead)
df['valor_anterior'] = df['valor'].shift(1)   # lag
df['valor_proximo'] = df['valor'].shift(-1)   # lead

# Diff
df['variacao'] = df['valor'].diff()

# Pct change
df['variacao_pct'] = df['valor'].pct_change()
```

## Categorias

```python
# Criar categoria
df['categoria'] = df['categoria'].astype('category')

# Ordenar categorias
df['tamanho'] = pd.Categorical(
    df['tamanho'],
    categories=['P', 'M', 'G', 'GG'],
    ordered=True
)

# Cut (bins)
df['faixa'] = pd.cut(df['idade'], bins=[0, 18, 30, 50, 100])
df['faixa'] = pd.cut(
    df['idade'],
    bins=[0, 18, 30, 50, 100],
    labels=['Menor', 'Jovem', 'Adulto', 'Senior']
)

# Qcut (quantis)
df['quartil'] = pd.qcut(df['valor'], q=4)
df['quartil'] = pd.qcut(df['valor'], q=4, labels=['Q1', 'Q2', 'Q3', 'Q4'])
```

## Performance

```python
# Usar tipos eficientes
df['inteiro'] = df['inteiro'].astype('int32')
df['categoria'] = df['categoria'].astype('category')

# Ler só colunas necessárias
df = pd.read_csv('dados.csv', usecols=['col1', 'col2'])

# Processar em chunks
chunks = pd.read_csv('grande.csv', chunksize=10000)
for chunk in chunks:
    processar(chunk)

# Usar query() para filtros complexos
df.query('col1 > 100 and col2 == "A"')  # mais rápido que &

# Vetorizar operações (evitar apply quando possível)
# Ruim:
df['nova'] = df['col'].apply(lambda x: x * 2)
# Bom:
df['nova'] = df['col'] * 2
```

## Method chaining

```python
# Encadear operações
resultado = (
    df
    .query('idade > 18')
    .assign(idade_meses=lambda x: x['idade'] * 12)
    .groupby('cidade')
    .agg(
        total=('valor', 'sum'),
        media_idade=('idade', 'mean')
    )
    .sort_values('total', ascending=False)
    .head(10)
)
```

## Links

- [Pandas Docs](https://pandas.pydata.org/docs/)
- [Pandas User Guide](https://pandas.pydata.org/docs/user_guide/index.html)
- [10 Minutes to Pandas](https://pandas.pydata.org/docs/user_guide/10min.html)
