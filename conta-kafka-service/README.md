# 🚀 Guia Oficial de Deploy Local com Docker

Este projeto utiliza um ambiente completo de microserviços Java Spring Boot orquestrados via Docker Compose, com os seguintes componentes: 

🐘 PostgreSQL — Banco de dados relacional 
🧠 Zookeeper — Coordenação e registro de serviços para o Kafka 
🔄 Kafka Broker — Sistema de mensageria distribuída 
💳 Conta Service — Microserviço principal para operações de conta 
📬 Kafka Service — Microserviço para consumo e publicação de mensagens Kafka

---

## 🧠 Visão Geral da Arquitetura de Microserviços

A arquitetura da aplicação é composta pelos seguintes serviços:

```text
+--------------------+         +---------------------+         +----------------------+
|   Angular Frontend | <-----> |   Spring Boot API   | <-----> |   PostgreSQL Banco   |
+--------------------+         +---------------------+         +----------------------+
         ↑
         │
         └──> Comunicação via HTTP (porta 4200 → 8080)

```
- **💻 Frontend (Angular):** Interface web do sistema, acessível via navegador.
- **⚙️ Backend (Spring Boot):** API responsável pelas regras de negócio e persistência.
- **🐘 Banco de Dados (PostgreSQL):** Armazena todas as entidades da aplicação.
- **🐳 Docker Compose:** Orquestração dos containers.

---

## 🟢 Início: Passo a passo para subir localmente com Docker

Todos os serviços são definidos no arquivo:

```text
api-funcoes-teste-spring/infra/docker-compose.yml
```

---

### 🧼 1. Limpeza Completa do Ambiente Docker
Antes de subir o ambiente, limpe todas as imagens, containers e redes antigas para evitar conflitos:

```bash
docker stop $(docker ps -aq) && \
docker rm -f $(docker ps -aq) && \
docker rmi -f $(docker images -aq) && \
docker volume rm -f $(docker volume ls -q) && \
docker network rm $(docker network ls -q | grep -v "bridge\|host\|none") && \
docker builder prune -af
```
⚠️ **Atenção:** Esse comando remove tudo do Docker (containers, volumes, imagens e redes personalizadas).
Use apenas se deseja começar do zero.

---

### 🔎 2. Verificação de Portas Livres

Certifique-se de que as seguintes portas não estão em uso no seu sistema:

| Serviço | Porta | Descrição |
|---------|-------|-----------|
| PostgreSQL | 5432 | Banco de dados |
| Zookeeper | 2181 | Coordenação Kafka |
| Kafka Broker | 9092 / 29092 | Comunicação Kafka |
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

### 🧭 3. Navegação e Estrutura do Projeto

A estrutura local é a seguinte:

```text
api-funcoes-teste-spring/
├── conta-service/
├── kafka-service/
├── infra/
│   └── docker-compose.yml
└── README.md
```

Acesse a pasta de infraestrutura no terminal:

```bash
cd ~/Projetos/api-funcoes-teste-spring/infra
```
---

### 🧰 4. Subindo o Ambiente Completo


