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

A arquitetura de mensageria do projeto é baseada no **Apache Kafka**, responsável como um mecanismo de **mensageria assíncrona** entre dois microsserviços:
👉 o **conta-service** (produtor) e o **kafka-service** (consumidor).
```text
+-------------------+        +-------------------+        +-------------------+        +---------------------+
|                   |        |                   |        |                   |        |                     |
|   Conta Service   | -----> |   Kafka Broker    | -----> |   Kafka Service   | -----> |     PostgreSQL      |
|  (Produz eventos) |        | (Gerencia tópicos)|        | (Consome eventos) |        |   (Persistência)    |
|                   |        |                   |        |                   |        |                     |
+--------+----------+        +---------+---------+        +---------+---------+        +----------+----------+
         |                             |                            |                             |
         | 1️⃣ POST /api/contas/abrir   |                            |                             |
         |    Envia evento (Producer)  |                            |                             |         
         |---------------------------->|                            |                             |
         |                             |                            |                             |
         | 2️⃣ kafkaTemplate.send()     |                            |                             |
         |---------------------------->| 3️⃣ Armazena mensagem       |                             |
         |                             |    no tópico               |                             |
         |                             |--------------------------->| 4️⃣ @KafkaListener consome   |
         |                             |                            |    e processa mensagem      |
         |                             |                            |---------------------------> |
         |                             |                            | 5️⃣ Salva Cliente e Conta    |
         |                             |                            |    no PostgreSQL            |
```

---

### 🧩 Visão Geral do Fluxo Kafka

1. O **usuário faz uma requisição HTTP** `(POST /api/contas/abrir)` para abrir uma nova conta.
2. O **conta-service** transforma essa requisição em uma **mensagem JSON** e **publica no tópico Kafka** (`conta.aberturas.topic`).
3. O **kafka-service** escuta esse tópico através de um **@KafkaListener, consome a mensagem**, e **cria/atualiza o cliente e a conta no banco PostgreSQL.**
4. Todo esse processo ocorre **de forma assíncrona e desacoplada**, sem dependência direta entre os dois serviços.

---

### 🔄 Componentes do Fluxo

| **Componente**         | **Tipo**         | **Responsabilidade Principal**                                    |
|-------------------------|------------------|-------------------------------------------------------------------|
| `conta-service`         | Producer         | Envia mensagens Kafka com dados de abertura de conta |
| `KafkaProducerConfig`   | Configuração     | Define propriedades do produtor Kafka                             |
| `Kafka Broker`        | Middleware       | Armazena e distribui mensagens entre serviços                     |
| `kafka-service`         | Consumer         | Lê mensagens e executa persistência no banco                      |
| `KafkaConsumerConfig`   | Configuração     | Controla comportamento e políticas de leitura                     |
| `PostgreSQL`          | Banco de dados   | Persistência final de Cliente e Conta                             |

---

### 🧭 Estrutura do Projeto

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
---

## ✅ Benefícios da Arquitetura com Kafka

- **Desacoplamento:** o `conta-service` não depende da disponibilidade do `kafka-service`.
- **Escalabilidade:** múltiplos consumidores podem ler o mesmo tópico em paralelo.
- **Tolerância a falhas:** mensagens permanecem armazenadas até o consumo bem-sucedido.
- **Consistência eventual:** o estado do sistema se propaga via eventos Kafka.

---

## 🧩 Visão Geral do Fluxo Kafka

O usuário faz uma requisição HTTP (`POST /api/contas/abrir`) para abrir uma nova conta.

O **conta-service** transforma essa requisição em uma **mensagem JSON** e publica no tópico Kafka (`conta.aberturas.topic`).

O **kafka-service** escuta esse tópico através de um `@KafkaListener`, consome a mensagem e cria/atualiza o cliente e a conta no banco **PostgreSQL**.

Todo esse processo ocorre de forma **assíncrona e desacoplada**, sem dependência direta entre os dois serviços.

---

## ⚙️ Funções e Responsabilidades do Kafka — Passo a Passo

---

### **1. Produção da Mensagem (Producer)**

📍 **Local:** `conta-service → ContaService.java`

**Função do Kafka aqui:**
- Enviar mensagens para o tópico Kafka definido em `application.properties` (`conta.aberturas.topic`).
- Garantir entrega confiável e segura (usando `acks=all` e `retries=3`).

**Onde acontece:**
```java
kafkaTemplate.send(contaAberturasTopic, request.getCpf(), payload);
```
**Descrição técnica:**

- `KafkaTemplate` é o componente que **envia mensagens**.
- O método `.send()` publica a mensagem no **tópico**.
- A `key` (CPF) garante **particionamento consistente** — todas as mensagens do mesmo cliente vão para a mesma partição.
- O `payload` é um **JSON** com os dados da requisição (`AbrirContaRequest`).

**Configuração usada:**  
📍 **Arquivo:** `KafkaProducerConfig.java`

Define propriedades do producer:
- Servidor Kafka (`bootstrap-servers`)
- Serializadores (`StringSerializer`)
- Confirmação de envio (`ACKS_CONFIG = all`)
- Tentativas de reenvio (`RETRIES_CONFIG = 3`)

---

### **2️⃣ Transporte da Mensagem (Broker)**

📍 **Local:*** Entre os dois serviços — mediado pelo ***Kafka Broker**

**Funções:**
- Atuar como **middleware de mensageria**, armazenando e entregando as mensagens de forma confiável. 
- Garantir **durabilidade e ordenação** por partição. 
- Permitir que o consumidor processe as mensagens mesmo que o produtor ou o consumidor estejam temporariamente offline.

**Detalhes técnicos:**
- O Kafka **persiste a mensagem em disco** no cluster.
- Mantém o **offset de leitura** para cada grupo de consumidores.
- Caso o `kafka-service` esteja fora do ar, as mensagens **ficam retidas** até serem consumidas.

**Componente principal:**
- **Tópico:** `conta.aberturas.topic`
- **Mensagem:** JSON com dados da requisição de abertura de conta

---

### **3️⃣ Consumo da Mensagem (Consumer)**

📍 **Local:** `kafka-service → ContaConsumer.java`

**Fluxo:**
- **Entregar mensagens** publicadas no tópico (conta.aberturas.topic) para o serviço consumidor.
- Controlar o **offset** (posição de leitura).
- Permitir **reprocessamento** em caso de erro, pois o commit automático está desativado.

**Onde acontece:**
```java
@KafkaListener(topics = "${conta.aberturas.topic}", groupId = "kafka-service")
public void consume(String message) { ... }
```

**Descrição técnica:**
- O `@KafkaListener` faz o subscribe ao tópico e **aciona automaticamente** o método `consume()` ao receber novas mensagens.
- O `groupId` (`kafka-service`) garante que esse consumidor **faça parte de um grupo lógico,** evitando leitura duplicada por outros serviços iguais.
- A mensagem JSON é **desserializada** via `ObjectMapper` e usada para:
  - Criar ou atualizar o `Cliente` no banco.
  - Criar uma nova `Conta` associada ao cliente.

---

### **4️⃣ Persistência no Banco de Dados**

📍 **Local:** `kafka-service → ContaConsumer.java`

**Função do Kafka aqui:**
- Embora o Kafka não grave diretamente no banco, ele **dispara o evento** que inicia a persistência. 
- O consumidor é responsável por:
  - Interpretar a mensagem Kafka.
  - Executar operações no banco PostgreSQL via `ClienteRepository` e `ContaRepository`.

**Fluxo dentro do consumidor:**
```java
Cliente cliente = clienteRepository.findByCpf(cpfLimpo)
        .orElseGet(() -> clienteRepository.save(new Cliente(request.getNomeCliente(), cpfLimpo)));

Conta conta = new Conta();
conta.setTipo(request.getTipoConta());
        conta.setStatus(StatusConta.ATIVA);
conta.setCliente(cliente);
contaRepository.save(conta);
```

---

### **5️⃣ Persistência no Banco de Dados**

📍 **Local:** `KafkaConsumerConfig.java`

**Função:**
- Definir como o consumidor Kafka se conecta, processa e gerencia mensagens.

**Principais parâmetros:** 

| Configuração                 | Função                                                         |
| ---------------------------- | -------------------------------------------------------------- |
| `bootstrap-servers`          | Endereço do cluster Kafka                                      |
| `group.id`                   | Identifica o grupo de consumidores                             |
| `enable.auto.commit=false`   | Desativa commit automático (maior controle de reprocessamento) |
| `auto.offset.reset=earliest` | Começa a consumir desde o início caso não haja offset salvo    |
| `poll.timeout=1500`          | Tempo máximo de espera para novas mensagens                    |

---

## 🧠 Funções do Kafka no Projeto

| **Etapa** | **Serviço**       | **Função Kafka**                          | **Localização**                                  |
|------------|-------------------|--------------------------------------------|--------------------------------------------------|
| 1️⃣ Produção de mensagem | `conta-service`   | Publicar evento de abertura de conta        | `ContaService.abrirConta()`                     |
| 2️⃣ Transporte assíncrono | `Kafka Broker`  | Armazenar e rotear mensagens                | Servidor Kafka                                  |
| 3️⃣ Consumo de mensagem   | `kafka-service`   | Escutar e processar eventos                 | `ContaConsumer.consume()`                       |
| 4️⃣ Persistência          | `kafka-service`   | Converter evento em ação no BD              | `ClienteRepository` e `ContaRepository`         |
| 5️⃣ Configuração técnica  | **Ambos**         | Controlar comportamento de producer e consumer | `KafkaProducerConfig`, `KafkaConsumerConfig` |

---

## 🔍 Conclusão

O Kafka no seu projeto atua como um **barramento de eventos** entre microsserviços, permitindo:
- **Desacoplamento total** entre `conta-service` e `kafka-service`.
- **Escalabilidade horizontal** (vários consumidores por grupo).
- **Tolerância a falhas** (mensagens persistidas até consumo).
- **Consistência eventual** entre serviços.

---

## 🟢 Início: Passo a passo para subir localmente com Docker

---

### 1️⃣ Pré-requisitos
- Docker e Docker Compose instalados.
- Java 21 + Maven (caso queira rodar manualmente).
- Porta 5432 (PostgreSQL) e 9092 (Kafka) livres.

---

### 2️⃣ Limpeza Completa do Ambiente Docker
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

### 3️⃣ Verificação de Portas Livres

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

### 4️⃣ Navegação local do projeto

Acesse a pasta de infraestrutura local no terminal:

```bash
cd ~/"Projetos/Projeto para Estudos (Frontend + Backend)/api-funcoes-teste-spring/infra"
```
---

### 5️⃣ Subindo o Ambiente Completo

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

### 6️⃣ Verificando o Status dos Containers

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

### 7️⃣ Teste Completo (Smoke Test)
Após subir o ambiente, rode os health checks:
```bash
curl -s http://localhost:8081/actuator/health
curl -s http://localhost:8082/actuator/health
```
Ambos devem retornar `"status": "UP"`

---

### 💾 🐞 Logs e Debug

Para visualizar logs de um serviço específico:

```bash
docker logs -f conta-service
```
ou 
```bash
docker logs -f kafka-service
```

---

## 🌐 Endpoints para Teste no Insomnia

### 🌿 Conta Service (`http://localhost:8081`)

---

1️⃣ **Criar Conta** - POST `/api/contas/abrir`
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

---

2️⃣ **Listar Endpoints** - GET `/api/endpoints`

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

---

3️⃣ **Actuator Health:** - GET `/actuator/health`

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
---

4️⃣ **Health Check:** - GET `/health`

✅ **Resposta:**
```json
{
  "service": "conta-service",
  "version": "1.0.0",
  "status": "UP",
  "timestamp": "2025-10-20T14:12:46.533994678"
}
```
---

### 🌿 Kafka Service (`http://localhost:8082`)

1️⃣ **Listar Contas** - GET `/api/contas`

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

---

2️⃣ **Actuator Health** - GET `/actuator/health`

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

---

3️⃣ **Health Check:** - GET `/health`

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

## 🧠 Dicas de Troubleshooting
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

## 🧹 Encerrando o Ambiente
Para **parar e remover tudo** (containers, redes e volumes):
```bash
docker compose down -v --remove-orphans
```
Se quiser limpar completamente o cache e imagens:
```bash
docker system prune -af
```

---
