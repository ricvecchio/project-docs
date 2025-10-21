# ⚙️ Comandos Extras

Este repositório armazena comandos e anotações úteis para desenvolvimento, manutenção e deploy de projetos.

---

## 🐳 Docker

```bash
# Listar containers ativos
docker ps

# Remover todos os containers parados
docker container prune -f

# Remover imagens não utilizadas
docker image prune -a -f

# Subir containers com Docker Compose
docker compose up -d

# Para parar todos os containers
docker-compose down

# Listar todos os containers (ativos e inativos)
docker ps -a

# Limpar volumes (apagar dados do banco)
docker volume rm project-root_postgres_data

# Acessar o banco de dados
docker exec -it postgres-db psql -U admin -d minha_base

# Ver logs de todos os serviços
docker-compose logs -f

# Logs apenas do serviço conta-service:
docker logs conta-service

# Seguir logs em tempo real
docker logs -f conta-service

# Filtrar logs por erros
docker logs conta-service | grep ERROR

# Filtrar logs por exceções 
docker logs conta-service | grep Exception

# Para garantir que tudo seja reconstruído do zero
docker compose down
docker compose build --no-cache
docker compose up -d

# Recompilar o frontend ou backend (Rebuildar containers após mudanças no código)
docker compose up -d --build

# Listar os containers em execução no Docker
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```
---

## 🔄 Kafka Broker — Sistema de mensageria distribuída   

Teste o **endpoint Kafka** para enviar e consumir mensagens do broker em execução no contêiner `kafka-broker`.
Pode ser executado de qualquer diretório, desde que o contêiner esteja rodando.

Diretório local do Kafka Service:
```bash
cd /Users/ricardodelvecchio/Projetos/api-funcoes-teste-spring/kafka-service
```

### 1️⃣ Criar/usar um tópico (ex: teste)
```bash
docker exec -it kafka-broker kafka-topics --create \
--topic teste \
--bootstrap-server localhost:9092 \
--partitions 1 \
--replication-factor 1
```
Se o tópico já existir, ele vai avisar.

### 2️⃣ Enviar mensagens para o tópico
```bash
docker exec -it kafka-broker kafka-console-producer \
--topic teste \
--bootstrap-server localhost:9092
```
Depois de rodar esse comando, digite qualquer mensagem, por exemplo:

```css
Olá Kafka
Teste 123
```

Cada linha será enviada como uma mensagem.

### 3️⃣ Consumir mensagens do tópico
```bash
docker exec -it kafka-broker kafka-console-consumer \
--topic teste \
--from-beginning \
--bootstrap-server localhost:9092
```
Isso vai mostrar todas as mensagens enviadas para o tópico teste.

---
## 🧩 Configuração do repositório Git

---

### 1️⃣ Criar ou usar um tópico (ex: “teste”)
```bash
git init
git branch -M main
echo "node_modules" >> .gitignore
git add .
git commit -m "chore: rename project to painel-funcoes-teste-angular"
```
⚠️ Se o projeto já possuía Git, pule para o passo 3.

### 2️⃣ Enviar mensagens para o tópico, cada linha digitada será enviada como uma mensagem separada.
1. Acesse GitHub > [New Repository](https://github.com/new)
2. Nome: `painel-funcoes-teste-angular`
3. Escolha **público** ou **privado**
4. **Não adicione README** (já temos commit local)
5. Copie a URL do repositório, por exemplo:
`git@github.com:ricvecchio/painel-funcoes-teste-angular.git`

### 3️⃣ Conectar e enviar para o repositório remoto
```bash
git remote add origin git@github.com:<seu-usuario>/painel-funcoes-teste-angular.git
git push -u origin main
```
🔁 Se já existir um remoto anterior:
```bash
git remote set-url origin git@github.com:<seu-usuario>/painel-funcoes-teste-angular.git
git push -u origin main
```

---
## ⚙️ Comandos

---
### 💻 Verificar e encerrar quem esta ocupando uma porta. 
- Exemplo porta: 5432
```bash
sudo lsof -i :5432
```
- Esses comandos listam qual processo (PID e nome) está escutando na porta.
```graphql
COMMAND   PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
postgres  1234  ricardodelvecchio  7u  IPv4  0x...      0t0  TCP *:5432 (LISTEN)
```
- Encerrar o processo que está travando a porta através do PID:
```bash
sudo kill -9 1234
```
- Confirmar que a porta foi liberada
```bash
sudo lsof -i :5432
```
- Se não aparecer nada, está tudo limpo

---

### 💻 Dar permissão de execução aos scripts

Este comando altera as permissões dos arquivos `.sh` dentro da pasta `bin/`, tornando-os **executáveis**.
```bash
chmod +x bin/*.sh
```
Após sua execução, será possível rodar os scripts diretamente no terminal utilizando `./nome-do-script.sh`.

---

### 💻 Ver arquivos modificados ou não rastreados
```bash
git status
```

---

### 💻 Lista apenas os arquivos staged (prontos para commit)
```bash
git diff --name-only --cached
```

---

### 💻 Listar arquivos modificados ainda não adicionados ao stage (não adicionados para commit)
```bash
git diff --name-only
```

---

### 💻 Adiciona todos os arquivos modificados/novos e exibe quais serão incluídos no próximo commit
```bash
git add .
git status
```

---
### 💻 Commit adicionando um comentário pelo Git
```bash
git commit -m "Adiciona imagem minha-imagem.png na pasta images"
```

---
### 💻 Envie as alterações para o seu repositório remoto no GitHub
```bash
git push origin main
```

---

### 💻 TO-DO
```bash

```

---

### 💻 TO-DO
```bash

```

---

### 💻 TO-DO
```bash

```

---