# 🚛 Sistema de Gestão de Clientes, Pedidos e Usuários (CRUD)

---
### Projeto Fullstack — Angular + Spring Boot (Transportadora)

Este repositório documenta o **Sistema de Gestão de Clientes, Pedidos e Usuários**, desenvolvido para uma **transportadora**.  
A aplicação é composta por um **frontend em Angular 14** e um **backend em Java Spring Boot 3.2.2** com banco de dados **PostgreSQL**, oferecendo uma solução completa de **gestão operacional, controle de pedidos e administração de usuários**.

---

## 🧭 Estrutura do Sistema

| Módulo | Tecnologia | Descrição |
|--------|-------------|-----------|
| **Frontend** | Angular 14 + TypeScript | Interface web para gestão de clientes, pedidos e usuários. |
| **Backend** | Java 21 + Spring Boot + PostgreSQL | API REST responsável pelas operações CRUD e lógica de negócios. |

---

## 🧩 Arquitetura Geral
```text
+----------------------+           HTTP REST          +----------------------+
|     Angular App      |  --------------------------> |   Spring Boot API    |
|  (Frontend - 4200)   |                              |   (Backend - 8080)   |
+----------------------+                              +----------------------+
          |                                                      |
          |                                                      |
          v                                                      v
   Interface do Usuário                             Banco de Dados PostgreSQL
```

### 🔁 Fluxo Resumido
1. O usuário acessa o painel web em Angular.
2. A aplicação consome endpoints da API Spring Boot para cadastrar, atualizar, listar e excluir clientes, pedidos e usuários.
3. Os dados são persistidos em um banco **PostgreSQL**, garantindo integridade e consistência.
4. As ações são validadas e registradas no backend, que retorna respostas RESTful.

---

## 🚀 Tecnologias Utilizadas

### 🖥️ Frontend
- Angular 14
- TypeScript
- Angular Material
- RxJS e HttpClient
- Integração REST com API Spring Boot

### ⚙️ Backend
- Java 21 ☕
- Spring Boot 3.2.2 🍃
- Spring Data JPA & Hibernate 🗄️
- Spring Security 🔐
- JWT (JSON Web Token) 🔑
- PostgreSQL 🐘
- Maven ⚙️
- Docker Compose 🐳

### ☁️ DevOps / Infra
- Render — hospedagem da aplicação frontend e backend (Free Tier)
- VPS Hostinger — execução do JAR do backend em produção
- PostgreSQL — banco gerenciado na nuvem
- GitHub Actions (em desenvolvimento) — automação de build e deploy

---

## 📊 Funcionalidades Principais
### 👥 Gestão de Clientes
- Cadastro, edição e exclusão de clientes
- Associação de volumes e preços
- Validação de valores de referência antes de emissão de pedidos

### 📦 Gestão de Pedidos
- Emissão e controle de pedidos
- Cálculo automático de preço final
- Campos opcionais como **ajudante, adicional e exibir preço**
- Validação automática dos checkboxes 
- Bloqueio de emissão sem preço de referência

### 👤 Gestão de Usuários
- Cadastro e autenticação JWT
- Controle de permissões e papéis
- Edição inline de permissões via dropdown no Angular Material

---

## 📈 Dashboard Analítico
O sistema conta com um **Dashboard em Angular** integrado à API, exibindo métricas de faturamento por cliente e mês.

### 📊 Gráficos
- **Barras empilhadas:** top 5 clientes com maiores gastos mensais
- **Pizza:** percentual de gastos por mês, com valor total em reais

---

## 📦 Repositórios
- **Frontend:** 👉 [transp-crud-angular](https://github.com/ricvecchio/transp-crud-angular)
- **Backend:** 👉 [transp-api-crud-spring](https://github.com/ricvecchio/transp-api-crud-spring)

---

## 👨‍💻 Autor

**Ricardo Del Vecchio**  
📍 GitHub: [@ricvecchio](https://github.com/ricvecchio)

---
