# 🧠 Estudos Fullstack DevOps
### Sistema de Estudos — Angular + Spring Boot (Microserviços e DevOps)

Este repositório documenta o **Sistema de Estudos Fullstack DevOps**, um ambiente integrado composto por um **frontend em Angular 20** e um **backend em Java Spring Boot 3.3.x com microserviços e mensageria Kafka**.  
O objetivo é consolidar estudos e práticas sobre **arquitetura de microsserviços**, **integração assíncrona**, **CI/CD com GitHub Actions** e **boas práticas de DevOps**.

---

## 🧩 Estrutura do Sistema

O sistema é dividido em dois principais módulos:

| Módulo | Tecnologia | Descrição |
|--------|-------------|-----------|
| **Frontend** | Angular 20 + TypeScript | Painel web que consome os microserviços backend via REST e Kafka. |
| **Backend** | Java 21 + Spring Boot + Kafka | API distribuída em microsserviços independentes, responsáveis pela gestão de contas e processamento assíncrono. |

---

## 🧭 Arquitetura Geral

```text
TO-DO
```


### Fluxo de Comunicação
1. O **frontend Angular** consome endpoints do `conta-service` (porta 8081) e `kafka-service` (porta 8082).
2. O usuário realiza ações no painel, como **abrir conta** ou **consultar contas**.
3. O **conta-service** recebe a requisição e publica mensagens no tópico **Kafka `contas-abrir`**.
4. O **kafka-service** consome as mensagens, valida os dados e persiste as informações no **PostgreSQL**.
5. A resposta é retornada ao cliente com status HTTP `202 ACCEPTED`.

---

## 🚀 Tecnologias Utilizadas

### 🖥️ Frontend
- Angular 20
- TypeScript 5.8
- Angular Material
- RxJS e HttpClient
- Integração REST com backend (Spring Boot + Kafka)

### ⚙️ Backend
- Java 21 ☕
- Spring Boot 3.3.4 🍃
- Spring Security 🔐
- JWT (JSON Web Token) 🔑
- Spring Data JPA & Hibernate 🗄️
- Spring Kafka 🔄
- PostgreSQL 🐘
- Lombok 📝
- Maven ⚙️
- Docker & Docker Compose 🐳

### ☁️ DevOps / CI-CD
- GitHub Actions 🤖 — build e deploy automatizado
- Docker Compose — orquestração dos microsserviços
- Git & GitHub 🌐 — controle de versionamento e integração contínua

---

## ⚙️ Como Executar Localmente

### 🧩 Frontend — Painel Angular
```bash
git clone https://github.com/ricvecchio/painel-funcoes-teste-angular.git
cd painel-funcoes-teste-angular
npm install
npm start
```
---

## 🧩 Funcionalidades Principais

### Frontend
- **Abrir Conta:** envia requisição para `http://localhost:8081/api/contas/abrir`
- **Listar Endpoints:** consome `http://localhost:8081/api/endpoints`
- **Consultar Contas:** consome `http://localhost:8082/api/contas`

### Backend
- **Gestão de Contas:** abertura, listagem e atualização de contas
- **Processamento Kafka:** consumo e persistência de mensagens no PostgreSQL
- **Validação de CPF e nome**
- **Monitoramento via endpoints** `/health`

---

## 📦 Repositórios: 
- **Frontend:** 👉 [painel-funcoes-teste-angular](https://github.com/ricvecchio/painel-funcoes-teste-angular)
- **Backend:** 👉 [api-funcoes-teste-spring](https://github.com/ricvecchio/api-funcoes-teste-spring)

---

## 👨‍💻 Autor

**Ricardo Del Vecchio**  
📍 GitHub: [@ricvecchio](https://github.com/ricvecchio)

---