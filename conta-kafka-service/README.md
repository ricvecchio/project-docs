# 🚀 Guia Oficial de Deploy Local com Docker

Este projeto implementa uma arquitetura de **microserviços Java Spring Boot**, orquestrados com **Docker Compose**, integrando mensageria **Apache Kafka** e banco de dados **PostgreSQL**:
![Diagrama](https://github.com/ricvecchio/project-docs/blob/main/images/spring-kafka-container.png)

| Serviço              | Função Principal                           |
| -------------------- | ------------------------------------------ |
| 🐘 **PostgreSQL**    | Banco de dados relacional                  |
| 🧠 **Zookeeper**     | Coordenação e registro de serviços Kafka   |
| 🔄 **Kafka Broker**  | Sistema de mensageria distribuída          |
| 💳 **Conta Service** | Microserviço produtor de eventos Kafka     |
| 📬 **Kafka Service** | Microserviço consumidor que persiste dados |

---

### 🔄 Arquitetura Kafka — Comunicação entre Microserviços

O **Apache Kafka** atua como **barramento de eventos assíncrono** entre o `conta-service` (producer) e o `kafka-service` (consumer), permitindo o desacoplamento completo entre os dois.

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

### Fluxo de Mensagens

1. O **usuário realiza uma requisição HTTP** `(POST /api/contas/abrir)`.
2. O **conta-service** envia um evento JSON para o tópico Kafka `conta.aberturas.topic`.
3. O **kafka-service** consome o evento via `@KafkaListener` e salva os dados no PostgreSQL.
4. Todo o fluxo é **assíncrono**, sem dependência direta entre os dois serviços.

---

### 🧩 Detalhamento dos Componentes
| Componente            | Tipo           | Função                               |
| --------------------- | -------------- | ------------------------------------ |
| `conta-service`       | Producer       | Publica eventos de abertura de conta |
| `KafkaProducerConfig` | Configuração   | Define propriedades do producer      |
| `Kafka Broker`        | Middleware     | Armazena e distribui mensagens       |
| `kafka-service`       | Consumer       | Consome e processa mensagens         |
| `KafkaConsumerConfig` | Configuração   | Define comportamento do consumidor   |
| `PostgreSQL`          | Banco de dados | Persistência de clientes e contas    |

---

## ✅ Benefícios da Arquitetura com Kafka
- **Desacoplamento:** o `conta-service` não depende da disponibilidade do `kafka-service`.
- **Escalabilidade:** múltiplos consumidores podem ler o mesmo tópico em paralelo.
- **Tolerância a falhas:** mensagens permanecem armazenadas até o consumo bem-sucedido.
- **Consistência eventual:** dados propagados de forma assíncrona entre serviços.

---

## ⚙️ Funcionamento Técnico do Kafka

### **1️⃣ Produção da Mensagem (Producer)**
📍 **Local:** `conta-service → ContaService.java`
```java
kafkaTemplate.send(contaAberturasTopic, request.getCpf(), payload);
```

- Enviar mensagens para o **tópico Kafka** definido em `application.properties`.
- `KafkaTemplate` publica o JSON com o CPF como **key**, garantindo particionamento consistente.

- Configuração (`KafkaProducerConfig.java`):
  - `acks=all` — confirmação total
  - `retries=3` — tentativas de reenvio
  - `bootstrap-servers`, `StringSerializer`, etc.

---

### **2️⃣ Transporte da Mensagem**

📍 **Local:** Kafka Broker

Responsável por armazenar, ordenar e entregar as mensagens de forma confiável.
- Persiste mensagens em disco.
- Mantém offsets de leitura.
- Retém mensagens até o consumo com sucesso. 

**Tópico:** `conta.aberturas.topic`

**Mensagem:** JSON com dados da abertura de conta

---

### **3️⃣ Consumo da Mensagem (Consumer)**

📍 **Local:** `kafka-service → ContaConsumer.java`
```java
@KafkaListener(topics = "${conta.aberturas.topic}", groupId = "kafka-service")
public void consume(String message) { ... }
```

- O `@KafkaListener` aciona o método automaticamente ao receber nova mensagem.
- O `groupId` evita duplicidade de leitura entre consumidores do mesmo grupo.
- A mensagem é desserializada e utilizada para salvar cliente e conta no banco.

---

### **4️⃣ Persistência no Banco de Dados**

📍 **Local:** `kafka-service → ContaConsumer.java`
```java
Cliente cliente = clienteRepository.findByCpf(cpfLimpo)
        .orElseGet(() -> clienteRepository.save(new Cliente(request.getNomeCliente(), cpfLimpo)));

Conta conta = new Conta();
conta.setTipo(request.getTipoConta());
        conta.setStatus(StatusConta.ATIVA);
conta.setCliente(cliente);
contaRepository.save(conta);
```

- O consumidor executa a persistência no PostgreSQL via `ClienteRepository` e `ContaRepository`.

---

### **5️⃣ Configuração do Consumer**

📍 **Local:** `KafkaConsumerConfig.java`

| Propriedade                  | Função                                 |
| ---------------------------- | -------------------------------------- |
| `bootstrap-servers`          | Endereço do cluster Kafka              |
| `group.id`                   | Identificação do grupo de consumidores |
| `enable.auto.commit=false`   | Controle manual do offset              |
| `auto.offset.reset=earliest` | Consumo desde o início                 |
| `poll.timeout=1500`          | Tempo de espera por mensagens          |

---

### 🧠 Funções do Kafka no Projeto
| Etapa            | Serviço         | Função Kafka                | Localização                                  |
| ---------------- | --------------- | --------------------------- | -------------------------------------------- |
| 1️⃣ Produção     | `conta-service` | Publica evento              | `ContaService.abrirConta()`                  |
| 2️⃣ Transporte   | `Kafka Broker`  | Armazena e roteia mensagens | Servidor Kafka                               |
| 3️⃣ Consumo      | `kafka-service` | Processa evento             | `ContaConsumer.consume()`                    |
| 4️⃣ Persistência | `kafka-service` | Grava no banco              | `ClienteRepository`, `ContaRepository`       |
| 5️⃣ Configuração | Ambos           | Define comportamento        | `KafkaProducerConfig`, `KafkaConsumerConfig` |

---

### 🔍 Conclusão

O Kafka no seu projeto atua como um **barramento de eventos** entre **microsserviços**, permitindo:
- **Desacoplamento total** entre `conta-service` e `kafka-service`.
- **Escalabilidade horizontal** (vários consumidores por grupo).
- **Tolerância a falhas** (mensagens persistidas até consumo).
- **Consistência eventual** entre serviços.

---

## 🟢 Deploy Local com Docker

### 1️⃣ Pré-requisitos
- Docker e Docker Compose instalados.
- Java 21 + Maven (para execução manual)
- Portas livres: `5432`, `2181`, `9092`, `8081`, `8082`

---

### 2️⃣ Limpeza Completa do Ambiente Docker
⚠️ Remove **todas as imagens, containers e redes:**
```bash
docker stop $(docker ps -aq) && \
docker rm -f $(docker ps -aq) && \
docker rmi -f $(docker images -aq) && \
docker volume rm -f $(docker volume ls -q) && \
docker network rm $(docker network ls -q | grep -v "bridge\|host\|none") && \
docker builder prune -af
```

---

### 3️⃣ Verificação de Portas Livres

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

Para liberar:
```bash
sudo kill -9 <PID>
```
---

### 4️⃣ Execução do Ambiente
Navegue até a pasta de infraestrutura:
```bash
cd ~/"Projetos/Projeto para Estudos (Frontend + Backend)/api-funcoes-teste-spring/infra"
```
Suba o ambiente completo:
```bash
docker compose --env-file .env.local up -d --build
```
O comando:
1. Criar a **rede** `microservices-net`
2. Subir o **PostgreSQL**, **Zookeeper** e **Kafka Broker**
3. Construir as imagens do `conta-service` e `kafka-service`
4. Iniciar todos os containers em segundo plano

---

### 5️⃣ Verificando o Status dos Containers
```bash
docker ps
```

Saída esperada:
```nginx
CONTAINER ID   NAME             STATUS                    PORTS
abc123         conta-service    Up (healthy)  0.0.0.0:8081->8080/tcp
def456         kafka-service    Up (healthy)  0.0.0.0:8082->8080/tcp
ghi789         kafka-broker     Up (healthy)  0.0.0.0:9092->9092/tcp
jkl101         zookeeper        Up (healthy)  0.0.0.0:2181->2181/tcp
mno112         postgres-db      Up (healthy)  0.0.0.0:5432->5432/tcp
```

---

### 6️⃣ Teste Rápido (Smoke Test)
```bash
curl -s http://localhost:8081/actuator/health
curl -s http://localhost:8082/actuator/health
```
✅ Ambos devem retornar `"status": "UP"`

---

### 7️⃣ Logs e Debug
```bash
docker logs -f conta-service
docker logs -f kafka-service
```

---

## 🌐 Endpoints para Teste

### 🌿 Conta Service (`http://localhost:8081`)
**POST** `/api/contas/abrir`
```json
{
  "nomeCliente": "Ricardo Teste",
  "cpf": "123.456.789-01",
  "tipoConta": "CORRENTE"
}
```
**Resposta:**
```json
{
  "mensagem": "✅ Solicitação de abertura de conta processada!",
  "status": "ACCEPTED",
  "timestamp": "2025-10-20T14:12:08.304246174Z"
}
```

---

**GET** `/api/endpoints`, `/actuator/health`, `/health`

---

### 🌿 Kafka Service (`http://localhost:8082`)

**GET** `/api/contas`
Lista todas as contas persistidas.

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

## 🧠 Dicas de Troubleshooting
- Caso um serviço fique em `unhealthy`, use:
```bash
docker inspect <nome_container> | grep -A 10 "Health"
```
- Logs:
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
Para **parar e remover tudo** (containers, redes, volumes, cache e imagens):
```bash
docker compose down -v --remove-orphans
docker system prune -af
```

---
