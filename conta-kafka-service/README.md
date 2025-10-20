# 🚀 Guia Oficial de Deploy Local com Docker

Este projeto utiliza um ambiente completo de microserviços Java Spring Boot orquestrados via Docker Compose, com os seguintes componentes:
![Diagrama](https://github.com/ricvecchio/project-docs/blob/main/images/spring-kafka-container.png)

🐘 PostgreSQL — Banco de dados relacional  
🧠 Zookeeper — Coordenação e registro de serviços para o Kafka  
🔄 Kafka Broker — Sistema de mensageria distribuída   
💳 Conta Service — Microserviço principal para operações de conta  
📬 Kafka Service — Microserviço para consumo e publicação de mensagens Kafka

---

### 🔄 Fluxo de Mensagens Kafka — Comunicação entre os Microserviços

A arquitetura de mensageria do projeto é baseada no Apache Kafka, responsável por garantir comunicação assíncrona e desacoplada entre os microserviços.
```text
+--------------------+        +--------------------+        +--------------------+
|                    |        |                    |        |                    |
|   Conta Service    | -----> |   Kafka Broker     | -----> |   Kafka Service    |
|  (Produz eventos)  |        | (Gerencia tópicos) |        | (Consome eventos)  |
|                    |        |                    |        |                    |
+--------------------+        +--------------------+        +--------------------+
          |                              |                              |
          | 1️⃣ Envia evento (Producer)   |                              |
          |------------------------------>|                              |
          |                              | 2️⃣ Armazena no tópico        |
          |                              |------------------------------>|
          |                              |                              | 3️⃣ Consome evento (Listener)
          |                              |                              |

```

### 🔄 Componentes do Fluxo

| Componente  | Função                  | Descrição                           |
|---------|---------------------------|-------------------------------------|
| Conta Service     | Producer            | Publica eventos no tópico Kafka `conta-events` sempre que ocorre uma ação (ex: criação ou atualização de conta). |
| Kafka Broker    | Mensageiro             | Garante entrega e persistência da mensagem no tópico.                |
| Kafka Service  | Consumer | Fica escutando o tópico `conta-events` e processa mensagens recebidas.                      |

---


### 🧭 3. Estrutura do Projeto

```text
api-funcoes-teste-spring/
├── .github/workflows/deploy(vps/test))
├── .mvn/wrapper/maven-wrapper.properties
├── infra/
│   ├── .env(dev/prod)
│   └── docker-compose.yml
├── pom.xml
└── README.md

conta-service/
├── .github/workflows/deploy-conta.yml
├── .mvn/wrapper/maven-wrapper.jar
├── src/main/
│   ├── java/com/funcoes/
│   │   ├── config/
│   │   ├── controller/
│   │   ├── model/
│   │   ├── repository/
│   │   ├── service/
│   │   └── ContaServiceApplication.java
│   └── resources/application.properties
├── target/conta-service-1.0.0.jar
├── Dockerfile
└── pom.xml

kafka-service/
├── .github/workflows/deploy-kafka.yml
├── .mvn/wrapper/maven-wrapper.jar
├── src/main/
│   ├── java/com/funcoes/
│   │   ├── config/
│   │   ├── consumer/
│   │   ├── controller/
│   │   ├── model/
│   │   ├── repository/
│   │   └── KafkaServiceApplication.java
│   └── resources/application.properties
├── target/kafka-service-1.0.0.jar
├── Dockerfile
└── pom.xml
```


## 🟢 Início: Passo a passo para subir localmente com Docker

---

### 🧼 1. Limpeza Completa do Ambiente Docker
Antes de subir o ambiente, **limpe todas as imagens, containers e redes antigas** para evitar conflitos:

```bash
docker stop $(docker ps -aq) && \
docker rm -f $(docker ps -aq) && \
docker rmi -f $(docker images -aq) && \
docker volume rm -f $(docker volume ls -q) && \
docker network rm $(docker network ls -q | grep -v "bridge\|host\|none") && \
docker builder prune -af
```
⚠️ **Atenção:** Esse comando remove **tudo** do Docker (containers, volumes, imagens e redes personalizadas).
Use apenas se deseja começar do zero.

---

### 🔎 2. Verificação de Portas Livres

Certifique-se de que as seguintes portas **não estão em uso** no seu sistema:

| Serviço | Porta | Descrição |
|---------|-------|-----------|
| PostgreSQL | 5432 | Banco de dados |
| Zookeeper | 2181 | Coordenação Kafka |
| Kafka Broker | 29092 | Comunicação Kafka |
| Conta Service | 8081 | API REST |
| Kafka Service | 8082 | API REST |

Verifique se há processos ativos nas portas (no macOS):

```bash
sudo lsof -i :5432
sudo lsof -i :2181
sudo lsof -i :9092
sudo lsof -i :8081
sudo lsof -i :8082
```

Para liberar uma porta ocupada:
```bash
sudo kill -9 <PID>
```
---

### 🧭 3. Navegação local do projeto

Acesse a pasta de infraestrutura local no terminal:

```bash
cd ~/Projetos/api-funcoes-teste-spring/infra
```
---

### 🧰 4. Subindo o Ambiente Completo

Com o **Docker Desktop** aberto e em execução, execute:

```bash
docker compose --env-file .env up -d --build
```

Esse comando vai:
1. Criar a **rede** `microservices-net`
2. Subir o **PostgreSQL**, **Zookeeper** e **Kafka Broker**
3. Construir as imagens do `conta-service` e `kafka-service`
4. Iniciar todos os containers em segundo plano
---

### 🩺 5. Verificando o Status dos Containers

Após alguns minutos (aguarde os health checks internos), execute:

```bash
docker ps
```

Saída esperada (exemplo):
```nginx
CONTAINER ID   NAME             STATUS                    PORTS
abc123         conta-service    Up (healthy)  0.0.0.0:8081->8080/tcp
def456         kafka-service    Up (healthy)  0.0.0.0:8082->8080/tcp
ghi789         kafka-broker     Up (healthy)  0.0.0.0:9092->9092/tcp
jkl101         zookeeper        Up (healthy)  0.0.0.0:2181->2181/tcp
mno112         postgres-db      Up (healthy)  0.0.0.0:5432->5432/tcp
```

---

### 🧾 6. Logs e Debug

Para visualizar logs de um serviço específico:

```bash
docker logs -f conta-service
```
ou 
```bash
docker logs -f kafka-service
```

---

### 🧪 7. Teste Completo (Smoke Test)
Após subir o ambiente, rode os health checks:
```bash
curl -s http://localhost:8081/actuator/health
curl -s http://localhost:8082/actuator/health
```
Ambos devem retornar `"status": "UP"`

---

### 🧩 8. Endpoints para Teste no Insomnia
📘 **Conta Service**

**Base URL:**  ``` http://localhost:8081 ``` 

🟢 **1. Criar Conta** - POST `/api/contas/abrir`
- **Body:** (JSON)
```json
{
  "nomeCliente": "Ricardo Teste",
  "cpf": "123.456.789-01",
  "tipoConta": "CORRENTE"
}
```
✅ **Resposta:**
```json
{
  "mensagem": "✅ Solicitação de abertura de conta processada!",
  "status": "ACCEPTED",
  "timestamp": "2025-10-20T14:12:08.304246174Z"
}
```

🟢 **2. Listar Endpoints** - GET `/api/endpoints`

✅ **Resposta:**
```json
[
  {
    "path": "/",
    "methods": "GET",
    "controller": "HealthController",
    "methodName": "home"
  },
  {
    "path": "/api/contas/abrir",
    "methods": "POST",
    "controller": "ContaController",
    "methodName": "abrirConta"
  },
  {
    "path": "/api/endpoints",
    "methods": "GET",
    "controller": "EndpointController",
    "methodName": "listarEndpoints"
  },
  {
    "path": "/health",
    "methods": "GET",
    "controller": "HealthController",
    "methodName": "health"
  }
]
```

🟢 **3. Actuator Health:** - GET `/actuator/health`

✅ **Resposta:**
```json
{
  "status": "UP",
  "groups": [
    "liveness",
    "readiness"
  ]
}
```

🟢 **4. Health Check:** - GET `/health`

✅ **Resposta:**
```json
{
  "service": "conta-service",
  "version": "1.0.0",
  "status": "UP",
  "timestamp": "2025-10-20T14:12:46.533994678"
}
```

📘 **Kafka Service**

**Base URL:**  ``` http://localhost:8082```

🟢 **1. Listar Contas** - GET `/api/contas`

✅ **Resposta:**
```json
[
  {
    "id": 1,
    "tipo": "CORRENTE",
    "status": "ATIVA",
    "cliente": {
      "id": 1,
      "nome": "Ricardo Teste Post 1",
      "cpf": "12345678901"
    }
  },
  {
    "id": 2,
    "tipo": "CORRENTE",
    "status": "ATIVA",
    "cliente": {
      "id": 1,
      "nome": "Ricardo Teste Post 1",
      "cpf": "12345678901"
    }
  }
]
```
🟢 **2. Actuator Health** - GET `/actuator/health`

✅ **Resposta:**
```json
{
  "status": "UP",
  "groups": [
    "liveness",
    "readiness"
  ]
}
```

🟢 **3. Health Check:** - GET `/health`

✅ **Resposta:**
```json
{
  "service": "kafka-service",
  "version": "1.0.0",
  "status": "UP",
  "timestamp": "2025-10-20T14:12:50.149249971"
}
```
---

### 🧠 9. Dicas de Troubleshooting
- Caso um serviço fique em `unhealthy`, use:
```bash
docker inspect <nome_container> | grep -A 10 "Health"
```
- Verifique logs:
```bash
docker logs <nome_container>
```
- Reinicie um container individual:
```bash
docker restart conta-service
```
- Para reconstruir as imagens sem cache:
```bash
docker compose build --no-cache
```

---

### 🧹 10. Encerrando o Ambiente
Para **parar e remover tudo** (containers, redes e volumes):
```bash
docker compose down -v --remove-orphans
```
Se quiser limpar completamente o cache e imagens:
```bash
docker system prune -af
```

---

## ✅ Resumo Rápido dos Comandos
```bash
# 1️⃣ Limpar ambiente antigo
docker stop $(docker ps -aq) && docker rm -f $(docker ps -aq)
docker rmi -f $(docker images -aq)
docker volume rm -f $(docker volume ls -q)
docker network rm $(docker network ls -q | grep -v "bridge\|host\|none")
docker builder prune -af

# 2️⃣ Entrar na pasta infra
cd ~/Projetos/api-funcoes-teste-spring/infra

# 3️⃣ Subir todos os serviços
docker compose up -d

# 4️⃣ Verificar status
docker ps

# 5️⃣ Testar serviços
curl http://localhost:8081/actuator/health
curl http://localhost:8082/actuator/health

# 6️⃣ Derrubar tudo
docker compose down -v --remove-orphans

```

---