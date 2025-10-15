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
