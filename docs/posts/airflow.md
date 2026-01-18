---
date: 2025-08-24
categories:
  - Engenharia de Dados
  - Python
---

# Apache Airflow: referência

Orquestrador de workflows para pipelines de dados. Permite definir, agendar e monitorar DAGs (Directed Acyclic Graphs) de tarefas.

<!-- more -->

## O que é

Apache Airflow é uma plataforma para criar, agendar e monitorar workflows programaticamente. Criado pelo Airbnb em 2014, hoje é um projeto Apache.

## Que problemas resolve

- **Agendamento de pipelines**: executar tarefas em horários específicos ou intervalos
- **Dependências entre tarefas**: garantir que task B só execute após task A
- **Retry automático**: reexecutar tarefas que falharam
- **Monitoramento**: visualizar status, logs e histórico de execuções
- **Alertas**: notificar falhas via email, Slack, etc.
- **Backfill**: executar pipelines para datas passadas

## Conceitos principais

### DAG (Directed Acyclic Graph)

Define o pipeline completo: quais tarefas executar e em que ordem.

```python
from airflow import DAG
from datetime import datetime

with DAG(
    dag_id="etl_vendas",
    start_date=datetime(2025, 1, 1),
    schedule="@daily",  # ou "0 2 * * *" (cron)
    catchup=False,
) as dag:
    # tasks aqui
```

Parâmetros importantes:

| Parâmetro | Descrição |
|-----------|-----------|
| `dag_id` | Identificador único |
| `start_date` | Quando começar a agendar |
| `schedule` | Frequência de execução |
| `catchup` | Se True, executa runs passados |
| `default_args` | Args padrão para todas as tasks |

### Task

Unidade de trabalho dentro de uma DAG.

```python
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator

extract = BashOperator(
    task_id="extract",
    bash_command="python extract.py",
)

transform = PythonOperator(
    task_id="transform",
    python_callable=transform_data,
)
```

### Operator

Tipo de task. Airflow vem com muitos built-in e providers adicionais.

```python
# Python
from airflow.operators.python import PythonOperator

# Bash
from airflow.operators.bash import BashOperator

# SQL
from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator

# AWS
from airflow.providers.amazon.aws.operators.s3 import S3CreateBucketOperator

# GCP
from airflow.providers.google.cloud.operators.bigquery import BigQueryInsertJobOperator

# Docker
from airflow.providers.docker.operators.docker import DockerOperator
```

### Sensor

Task que espera por uma condição antes de prosseguir.

{% raw %}
```python
from airflow.sensors.filesystem import FileSensor
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

# Esperar arquivo local
wait_file = FileSensor(
    task_id="wait_file",
    filepath="/data/input.csv",
    poke_interval=60,  # verifica a cada 60s
    timeout=3600,      # timeout após 1h
)

# Esperar arquivo no S3
wait_s3 = S3KeySensor(
    task_id="wait_s3",
    bucket_name="meu-bucket",
    bucket_key="data/{{ ds }}/file.parquet",
)
```
{% endraw %}

### Dependências

Definir ordem de execução:

```python
# Sequencial
extract >> transform >> load

# Múltiplas dependências
[extract_a, extract_b] >> transform >> [load_dwh, load_lake]

# Método set_downstream/set_upstream
extract.set_downstream(transform)
transform.set_upstream(extract)  # equivalente
```

## TaskFlow API

Sintaxe moderna usando decorators (Airflow 2.0+):

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(
    dag_id="etl_taskflow",
    start_date=datetime(2025, 1, 1),
    schedule="@daily",
    catchup=False,
)
def etl_pipeline():

    @task
    def extract():
        return {"data": [1, 2, 3, 4, 5]}

    @task
    def transform(raw_data):
        return [x * 2 for x in raw_data["data"]]

    @task
    def load(processed_data):
        print(f"Carregando {len(processed_data)} registros")

    # Dependências implícitas pelo fluxo de dados
    raw = extract()
    processed = transform(raw)
    load(processed)

etl_pipeline()
```

## XCom

Comunicação entre tasks. Usado automaticamente pelo TaskFlow API.

```python
# Push manual
def push_data(**context):
    context["ti"].xcom_push(key="result", value={"count": 100})

# Pull manual
def pull_data(**context):
    result = context["ti"].xcom_pull(task_ids="push_task", key="result")
    print(result)  # {"count": 100}
```

XCom é para dados pequenos (metadados, contagens). Para dados grandes, usar storage externo (S3, GCS).

## Templating (Jinja)

Variáveis dinâmicas em parâmetros:

{% raw %}
```python
BashOperator(
    task_id="process",
    bash_command="python process.py --date {{ ds }} --env {{ var.value.environment }}",
)
```
{% endraw %}

Variáveis disponíveis:

{% raw %}
| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `{{ ds }}` | Data de execução (YYYY-MM-DD) | 2025-01-15 |
| `{{ ds_nodash }}` | Data sem traços | 20250115 |
| `{{ execution_date }}` | Datetime completo | 2025-01-15T00:00:00 |
| `{{ prev_ds }}` | Data anterior | 2025-01-14 |
| `{{ next_ds }}` | Próxima data | 2025-01-16 |
| `{{ var.value.key }}` | Variável do Airflow | - |
| `{{ conn.conn_id }}` | Conexão | - |
{% endraw %}

## Connections

Credenciais armazenadas no Airflow (via UI ou CLI):

```bash
# Criar via CLI
airflow connections add 'postgres_dwh' \
    --conn-type 'postgres' \
    --conn-host 'localhost' \
    --conn-schema 'dwh' \
    --conn-login 'user' \
    --conn-password 'pass' \
    --conn-port 5432
```

Usar em operators:

```python
from airflow.providers.postgres.operators.postgres import PostgresOperator

PostgresOperator(
    task_id="create_table",
    postgres_conn_id="postgres_dwh",  # referencia a connection
    sql="CREATE TABLE IF NOT EXISTS ...",
)
```

## Variables

Configurações globais:

```python
from airflow.models import Variable

# Ler
env = Variable.get("environment")
config = Variable.get("config", deserialize_json=True)

# Em templates Jinja: var.value.environment ou var.json.config.key
```

## Hooks

Camada de abstração para conexões:

```python
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.providers.amazon.aws.hooks.s3 import S3Hook

def query_postgres():
    hook = PostgresHook(postgres_conn_id="postgres_dwh")
    df = hook.get_pandas_df("SELECT * FROM tabela")
    return df

def upload_to_s3():
    hook = S3Hook(aws_conn_id="aws_default")
    hook.load_file(
        filename="/tmp/data.parquet",
        key="data/file.parquet",
        bucket_name="meu-bucket",
    )
```

## Branching

Execução condicional:

```python
from airflow.operators.python import BranchPythonOperator

def choose_branch(**context):
    if context["ds_nodash"] == "20250101":
        return "task_new_year"
    return "task_normal"

branch = BranchPythonOperator(
    task_id="branch",
    python_callable=choose_branch,
)

branch >> [task_new_year, task_normal]
```

## Trigger Rules

Quando uma task deve executar:

```python
from airflow.utils.trigger_rule import TriggerRule

task = PythonOperator(
    task_id="always_run",
    python_callable=cleanup,
    trigger_rule=TriggerRule.ALL_DONE,  # executa mesmo se upstream falhou
)
```

| Regra | Descrição |
|-------|-----------|
| `all_success` | Todas upstream ok (padrão) |
| `all_failed` | Todas upstream falharam |
| `all_done` | Todas upstream terminaram |
| `one_success` | Pelo menos uma ok |
| `one_failed` | Pelo menos uma falhou |
| `none_failed` | Nenhuma falhou (pode ter skipped) |

## SubDAGs e TaskGroups

Organizar tasks relacionadas:

```python
from airflow.utils.task_group import TaskGroup

with DAG(...) as dag:

    with TaskGroup("extract") as extract_group:
        extract_orders = PythonOperator(...)
        extract_customers = PythonOperator(...)

    with TaskGroup("transform") as transform_group:
        transform_orders = PythonOperator(...)
        transform_customers = PythonOperator(...)

    extract_group >> transform_group
```

## Dynamic Tasks

Criar tasks dinamicamente:

```python
# Mapeamento dinâmico (Airflow 2.3+)
@task
def get_files():
    return ["file1.csv", "file2.csv", "file3.csv"]

@task
def process_file(filename):
    print(f"Processando {filename}")

@dag(...)
def dynamic_pipeline():
    files = get_files()
    process_file.expand(filename=files)  # cria 3 tasks
```

## Pools

Limitar execuções paralelas:

```python
# Criar pool via UI ou CLI
# Nome: "database_pool", Slots: 2

task = PythonOperator(
    task_id="query_db",
    python_callable=query,
    pool="database_pool",  # máximo 2 tasks simultâneas neste pool
)
```

## Retry e Timeout

```python
from datetime import timedelta

task = PythonOperator(
    task_id="flaky_task",
    python_callable=might_fail,
    retries=3,
    retry_delay=timedelta(minutes=5),
    retry_exponential_backoff=True,
    max_retry_delay=timedelta(hours=1),
    execution_timeout=timedelta(hours=2),
)
```

## Callbacks

Executar código em eventos:

```python
def on_failure(context):
    # Enviar alerta
    send_slack_alert(f"Task {context['task_instance'].task_id} falhou")

def on_success(context):
    print("Sucesso!")

task = PythonOperator(
    task_id="task_with_callbacks",
    python_callable=do_something,
    on_failure_callback=on_failure,
    on_success_callback=on_success,
)
```

## Estrutura de projeto

```
airflow/
├── dags/
│   ├── etl_vendas.py
│   ├── etl_clientes.py
│   └── utils/
│       └── helpers.py
├── plugins/
│   └── custom_operators.py
├── include/
│   └── sql/
│       └── queries.sql
├── tests/
│   └── test_dags.py
├── docker-compose.yaml
└── requirements.txt
```

## Testes

```python
import pytest
from airflow.models import DagBag

@pytest.fixture
def dag_bag():
    return DagBag(include_examples=False)

def test_dag_loaded(dag_bag):
    assert "etl_vendas" in dag_bag.dags
    assert dag_bag.import_errors == {}

def test_dag_structure(dag_bag):
    dag = dag_bag.dags["etl_vendas"]
    assert len(dag.tasks) == 3
    assert "extract" in [t.task_id for t in dag.tasks]
```

## CLI útil

```bash
# Listar DAGs
airflow dags list

# Testar task
airflow tasks test dag_id task_id 2025-01-01

# Executar DAG
airflow dags trigger dag_id

# Backfill
airflow dags backfill dag_id --start-date 2025-01-01 --end-date 2025-01-31

# Pausar/despausar
airflow dags pause dag_id
airflow dags unpause dag_id

# Ver logs
airflow tasks logs dag_id task_id 2025-01-01
```

## Docker Compose (desenvolvimento)

```yaml
version: '3'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow

  airflow-webserver:
    image: apache/airflow:2.8.0
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    volumes:
      - ./dags:/opt/airflow/dags
    ports:
      - "8080:8080"
    command: webserver

  airflow-scheduler:
    image: apache/airflow:2.8.0
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    volumes:
      - ./dags:/opt/airflow/dags
    command: scheduler
```

## Executors

| Executor | Uso |
|----------|-----|
| `SequentialExecutor` | Desenvolvimento (1 task por vez) |
| `LocalExecutor` | Produção pequena (multiprocess) |
| `CeleryExecutor` | Produção distribuída (workers) |
| `KubernetesExecutor` | Kubernetes (pod por task) |

## Managed services

- **AWS MWAA**: Amazon Managed Workflows for Apache Airflow
- **GCP Cloud Composer**: Airflow gerenciado no Google Cloud
- **Astronomer**: Plataforma Airflow gerenciada

## Links

- [Airflow Docs](https://airflow.apache.org/docs/)
- [Providers](https://airflow.apache.org/docs/apache-airflow-providers/)
- [Best Practices](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html)
