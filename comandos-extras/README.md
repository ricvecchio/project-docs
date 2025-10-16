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

# Recompilar o frontend ou backend (Rebuildar containers após mudanças no código)
docker compose build

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
```


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

## 🔄 Comandos

### 🕵️‍♂️ Verificar quem esta ocupando uma porta. 
Exemplo porta: 5432
```bash
sudo lsof -i :5432
```
Esses comandos listam qual processo (PID e nome) está escutando na porta.
```graphql
COMMAND   PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
postgres  1234  ricardodelvecchio  7u  IPv4  0x...      0t0  TCP *:5432 (LISTEN)
```

### 🔪 Encerrar o processo que está travando a porta

Depois de identificar o PID, você pode encerrar com:
```bash
sudo kill -9 1234
```
### 🧹 Confirmar que a porta foi liberada

Verifique novamente:
```bash
sudo lsof -i :5432
```

Se não aparecer nada, está tudo limpo ✅