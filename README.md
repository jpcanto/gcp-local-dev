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

---

## Configuração do ambiente local

Antes de usar os emuladores, defina estas variáveis no terminal atual:

```bash
export PUBSUB_EMULATOR_HOST=localhost:8085
export GOOGLE_CLOUD_PROJECT=local-gcp-project
```

---

## Pub/Sub Emulator

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

## 📦 Requisitos

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Google Cloud SDK](https://cloud.google.com/sdk) (opcional, para interagir com o emulador Pub/Sub via `gcloud`)
- `curl` (para testar endpoints REST, como Fake GCS e API Gateway)
- Terminal Bash, Zsh, ou outro shell compatível