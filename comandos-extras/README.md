# ⚙️ Comandos e Anotações Úteis

Repositório de comandos, lembretes e anotações para **desenvolvimento**, **deploy** e **manutenção** de projetos.

---

## 🐳 Docker

### 🔍 Gerenciamento de Containers
```bash
# Listar containers ativos
docker ps

# Listar todos os containers (ativos e inativos)
docker ps -a

# Parar e remover todos os containers
docker compose down
docker container prune -f
```

### 🧱 Imagens e Volumes
```bash
# Listar imagens
docker images

# Remover imagens não utilizadas
docker image prune -a -f

# Limpar volumes específicos (ex: banco de dados)
docker volume rm project-root_postgres_data
```

### 🚀 Subir e Reconstruir Containers
```bash
# Subir containers com Docker Compose
docker compose up -d

# Reconstruir containers do zero
docker compose down
docker compose build --no-cache
docker compose up -d

# Rebuild rápido (apenas imagens alteradas)
docker compose up -d --build
```

### 🧭 Logs e Monitoramento
```bash
# Logs de todos os serviços
docker compose logs -f

# Logs de um serviço específico
docker logs conta-service

# Seguir logs em tempo real
docker logs -f conta-service

# Filtrar logs
docker logs conta-service | grep ERROR
docker logs conta-service | grep Exception
```

### 🗃️ Banco de Dados (PostgreSQL)
```bash
# Acessar o banco dentro do container
docker exec -it postgres-db psql -U admin -d minha_base

# Acessar o banco via serviço infra
docker exec -it infra-postgres-1 psql -U postgres -d conta_db

# Comandos SQL úteis
\dt                    # Listar tabelas
SELECT * FROM contas;  # Consultar dados
\q                     # Sair
```

### 🧩 Kafka (Mensageria)
```bash
# Entrar no container Kafka
docker exec -it infra-kafka-1 bash

# Listar tópicos
kafka-topics.sh --list --bootstrap-server localhost:9092

# Produzir mensagens
kafka-console-producer.sh --topic contas-topic --bootstrap-server localhost:9092

# Consumir mensagens
kafka-console-consumer.sh --topic contas-topic --from-beginning --bootstrap-server localhost:9092
```

---

## 🔄 Kafka Broker — Testes Locais

Diretório local (exemplo):
```bash
cd /Users/ricardodelvecchio/Projetos/api-funcoes-teste-spring/kafka-service
```

### 1️⃣ Criar Tópico
```bash
docker exec -it kafka-broker kafka-topics --create --topic teste --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

### 2️⃣ Produzir Mensagens
```bash
docker exec -it kafka-broker kafka-console-producer --topic teste --bootstrap-server localhost:9092
```

💬 Digite mensagens linha a linha:
```
Olá Kafka
Teste 123
```

### 3️⃣ Consumir Mensagens
```bash
docker exec -it kafka-broker kafka-console-consumer --topic teste --from-beginning --bootstrap-server localhost:9092
```

---

## 🧩 Git — Configuração e Versionamento

### 1️⃣ Inicializar Repositório Local
```bash
git init
git branch -M main
echo "node_modules" >> .gitignore
git add .
git commit -m "chore: rename project to painel-funcoes-teste-angular"
```

### 2️⃣ Criar Repositório Remoto
1. Vá em [GitHub › New Repository](https://github.com/new)  
2. Nome: `painel-funcoes-teste-angular`  
3. Escolha **público** ou **privado**  
4. **Não adicione README.md**  
5. Copie a URL SSH:
   ```
   git@github.com:seu-usuario/painel-funcoes-teste-angular.git
   ```

### 3️⃣ Conectar e Enviar ao Remoto
```bash
git remote add origin git@github.com:<seu-usuario>/painel-funcoes-teste-angular.git
git push -u origin main
```

Se já existir um remoto anterior:
```bash
git remote set-url origin git@github.com:<seu-usuario>/painel-funcoes-teste-angular.git
git push -u origin main
```

---

## 🧠 Git — Comandos Úteis

```bash
# Ver arquivos modificados
git status

# Listar arquivos staged (prontos para commit)
git diff --name-only --cached

# Listar modificações ainda não staged
git diff --name-only

# Adicionar e confirmar alterações
git add .
git commit -m "Adiciona imagem minha-imagem.png na pasta images"

# Enviar alterações para o remoto
git push origin main
```

---

## ☕ Java e Maven

### 🔧 Instalar Java (macOS / Homebrew)
```bash
brew install openjdk
brew install openjdk@21
export JAVA_HOME=/opt/homebrew/opt/openjdk
export PATH=$JAVA_HOME/bin:$PATH
java -version
javac -version
```

### 🧱 Build e Execução (Maven)
```bash
mvn clean install
mvn clean compile -DskipTests
mvn clean package -DskipTests
mvn spring-boot:run
```

---

## 💻 Comandos Gerais do Sistema

---

### 🔍 Verificar Portas em Uso
```bash
lsof -iTCP -sTCP:LISTEN -P -n
```

### 🔍 Verificar e Liberar Porta
```bash
# Exemplo: porta 5432
sudo lsof -i :5432
sudo kill -9 <PID>
sudo lsof -i :5432  # Confirmar liberação
```

### 🔑 Permitir Execução de Scripts
```bash
chmod +x bin/*.sh
```

Permite executar scripts diretamente:
```bash
./nome-do-script.sh
```

---

## 📝 To-Do
```bash
# Espaço reservado para novos comandos e anotações
```

---