# GCP Local Emulator Suite

Este projeto simula localmente alguns serviços do Google Cloud Platform utilizando `docker-compose`. Para desenvolvimento offline, testes e ambientes de desenvolvimento integrados.

---

## Tecnologias e ferramentas utilizadas

Para realizar os mocks dos serviços da GCP, utilizamos as seguintes ferramentas:

- **Pub/Sub Emulator**: É o emulador oficial fornecido pelo Google Cloud SDK que simula o serviço de mensageria Pub/Sub localmente. Permite criar tópicos, subscriptions e publicar/consumir mensagens sem conexão com a nuvem.

- **Fake GCS Server**: Projeto open source que simula a API do Google Cloud Storage localmente. Permite criar buckets, enviar, listar e baixar objetos via REST, ideal para testes e desenvolvimento sem uso da conta real.

Essas ferramentas juntas permitem desenvolver e testar aplicações que usam Pub/Sub e Storage sem custos e sem dependência da nuvem.

---

## Serviços emulados

| Serviço GCP        | Emulador utilizado   | Porta local |
| ------------------ | -------------------- | ----------- |
| Pub/Sub            | gcloud beta emulator | `8085`      |
| Cloud Storage      | Fake GCS Server      | `4443`      |
| BigQuery           | gcloud beta emulator | `9050`      |

---

## 🚀 Iniciando o ambiente

### 1. Clone o repositório

```bash
git clone https://github.com/seu-usuario/gcp-local-dev.git
cd gcp-local-dev
```

### 2. Suba os containers

```bash
docker-compose up --build -d
```

---

## Endpoints

| Serviço GCP        | URL                                  |
| ------------------ | ------------------------------------ |
| Pub/Sub            | http://localhost:8085                |
| Cloud Storage      | http://localhost:4443/storage/v1/b   |
| BigQuery           | http://localhost:9050                |

---

## Configuração do ambiente local

Antes de usar os emuladores, defina estas variáveis no terminal atual:

```bash
export PUBSUB_EMULATOR_HOST=localhost:8085
export BIGQUERY_EMULATOR_HOST=localhost:9050
export GOOGLE_CLOUD_PROJECT=local-gcp-project
```

### Configurando o projeto gcloud localmente

Para que a ferramenta `gcloud` funcione corretamente com o emulador, ela precisa saber qual ID de projeto usar, mesmo que seja um projeto fictício. Você pode configurar isso para sua CLI com o seguinte comando.

Isso **não requer uma conta Google** e **não faz nenhuma chamada para a nuvem**. A configuração é puramente local.

```bash
gcloud config set project local-gcp-project
```

Com isso e a variável de ambiente `PUBSUB_EMULATOR_HOST` definidos, o `gcloud` direcionará todos os comandos para o seu emulador local.

---

## Pub/Sub Emulator

Os comandos abaixo permitem interagir com o emulador Pub/Sub usando a CLI `gcloud`.

> **Nota:** É necessário ter o [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) instalado para usar os comandos `gcloud`.

### Criar tópico

```bash
gcloud pubsub topics create test-topic --project=local-gcp-project
```

### Criar subscription

```bash
gcloud pubsub subscriptions create test-sub --topic=test-topic --project=local-gcp-project
```

### Publicar mensagem

```bash
gcloud pubsub topics publish test-topic --message="Teste mensagem" --project=local-gcp-project
```

### Ler mensagens

```bash
gcloud pubsub subscriptions pull test-sub --auto-ack --project=local-gcp-project
```

Nota: certifique-se de que o SDK do GCP esteja instalado e autenticado localmente.

### Exemplos de uso por linguagem

<details>
<summary>Node.js</summary>

### Instalação da dependência

```bash
pnpm install @google-cloud/pubsub
```

```javascript
// Exemplo de código para publicar uma mensagem em Node.js
import { PubSub } from '@google-cloud/pubsub';

const pubsub = new PubSub({
  projectId: 'local-gcp-project',
  apiEndpoint: 'localhost:8085',
});

async function publishMessage(topicName, data) {
  try {
    const payload =
      typeof data === 'string' ? data : JSON.stringify(data);

    const dataBuffer = Buffer.from(payload);

    const messageId = await pubsub
      .topic(topicName)
      .publishMessage({ data: dataBuffer });

    console.log(`Mensagem publicada no tópico "${topicName}" com ID: ${messageId}`);
  } catch (error) {
    console.error('Erro ao publicar mensagem:', error);
  }
}

publishMessage('test-topic', {
  userId: 42,
  event: 'signup',
  timestamp: new Date().toISOString(),
});
```

</details>

<details>
<summary>Python</summary>

### Instalação da dependência

```bash
pip install google-cloud-pubsub
```

```python
from google.cloud import pubsub_v1
import json
import os

# Configura o ambiente para usar o emulador local
os.environ["PUBSUB_EMULATOR_HOST"] = "localhost:8085"
os.environ["GOOGLE_CLOUD_PROJECT"] = "local-gcp-project"

# Inicializa o cliente
publisher = pubsub_v1.PublisherClient()
project_id = os.environ["GOOGLE_CLOUD_PROJECT"]

def publish_message(topic_name: str, data):
    """Publica uma mensagem no Pub/Sub local"""
    topic_path = publisher.topic_path(project_id, topic_name)

    # Se o dado for dict, converte para JSON
    if isinstance(data, dict):
        data = json.dumps(data)

    # Codifica como bytes
    data_bytes = data.encode("utf-8")

    try:
        future = publisher.publish(topic_path, data=data_bytes)
        message_id = future.result()
        print(f"Mensagem publicada com ID: {message_id}")
    except Exception as e:
        print(f"Erro ao publicar mensagem: {e}")

publish_message("test-topic", {
    "user_id": 123,
    "action": "purchase",
    "amount": 59.90,
    "timestamp": "2025-06-21T12:34:56Z"
})
```

</details>

---

## Fake GCS Server

### Listar buckets disponíveis

```bash
curl http://localhost:4443/storage/v1/b
```

### Criar novo bucket (via API)

```bash
curl -X POST http://localhost:4443/storage/v1/b -H "Content-Type: application/json" -d '{"name": "novo-bucket"}'
```

### Fazer upload de arquivo

```bash
curl -X POST "http://localhost:4443/upload/storage/v1/b/local-bucket/o?uploadType=media&name=meuarquivo.txt" -H "Content-Type: text/plain" --data-binary @./README.md
```

### Fazer download de arquivo

```bash
curl http://localhost:4443/storage/v1/b/local-bucket/o/meuarquivo.txt?alt=media
```

---

## BigQuery Emulator

Os comandos abaixo permitem interagir com o emulador BigQuery usando a CLI `gcloud`.

> **Nota:** É necessário ter o [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) instalado para usar os comandos `gcloud`.

### Criar dataset

```bash
gcloud beta bigquery datasets create test_dataset --project=local-gcp-project --endpoint=http://localhost:9050
```

### Criar tabela

```bash
gcloud beta bigquery tables create test_dataset.test_table --project=local-gcp-project --endpoint=http://localhost:9050
```

### Inserir dados

```bash
gcloud beta bigquery tables insert test_dataset.test_table --project=local-gcp-project --endpoint=http://localhost:9050 --row '{"id":1,"name":"Alice"}'
```

### Consultar dados

```bash
gcloud beta bigquery query 'SELECT * FROM test_dataset.test_table' --project=local-gcp-project --endpoint=http://localhost:9050
```

### Exemplos de uso por linguagem

<details>
<summary>Node.js</summary>

### Instalação da dependência

```bash
pnpm install @google-cloud/bigquery
```

```javascript
import { BigQuery } from '@google-cloud/bigquery';

const bigquery = new BigQuery({
  projectId: 'local-gcp-project',
  apiEndpoint: 'http://localhost:9050',
});

async function queryBigQuery(sqlQuery) {
  try {
    const [job] = await bigquery.createQueryJob({
      query: sqlQuery,
      location: 'US',
    });
    const [rows] = await job.getQueryResults();
    console.log('Resultados:', rows);
  } catch (error) {
    console.error('Erro ao consultar BigQuery:', error);
  }
}

queryBigQuery('SELECT * FROM `test_dataset.test_table`');
```

</details>

<details>
<summary>Python</summary>

### Instalação da dependência

```bash
pip install google-cloud-bigquery
```

```python
from google.cloud import bigquery
import os

os.environ["BIGQUERY_EMULATOR_HOST"] = "localhost:9050"
os.environ["GOOGLE_CLOUD_PROJECT"] = "local-gcp-project"

client = bigquery.Client(
    project=os.environ["GOOGLE_CLOUD_PROJECT"],
    location="US"
)

def query_bigquery(sql_query: str):
    """Consulta dados no BigQuery emulado"""
    try:
        query_job = client.query(sql_query)
        results = query_job.result()
        for row in results:
            print(dict(row))
    except Exception as e:
        print(f"Erro ao consultar BigQuery: {e}")

query_bigquery("SELECT * FROM `test_dataset.test_table`")
```

</details>

---

## 📦 Requisitos

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) (opcional, para interagir com o emulador Pub/Sub via `gcloud`)
- `curl` (para testar endpoints REST, como Fake GCS e API Gateway)
- Terminal Bash, Zsh, ou outro shell compatível