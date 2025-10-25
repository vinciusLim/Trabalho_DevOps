# 🐳 Projeto Flask + MySQL com Docker

## Descrição do Projeto
Este projeto foi desenvolvido para consolidar os conhecimentos sobre Dockerfile, Docker Compose, redes, volumes e variáveis de ambiente. O objetivo é criar um ambiente multi-container que execute uma API Flask integrada a um banco de dados MySQL, demonstrando um CRUD básico para usuários. A aplicação foi construída de forma leve, utilizando imagens Alpine e build multi-stage para otimização, além de boas práticas de segurança e isolamento de rede.

## Arquitetura da Solução
A aplicação é composta por dois containers: flask_app (API Flask) e mysql_db (MySQL) com volume persistente. Os containers se comunicam através de uma rede Docker customizada, garantindo isolamento e segurança.

## Estrutura de Pastas
```

Trabalho_DevOps/
│
├── app.py                 # Código principal da API Flask
├── requirements.txt       # Dependências Python
├── Dockerfile             # Build multi-stage da aplicação
├── docker-compose.yml     # Orquestração dos containers
├── init.sql               # Script de inicialização do banco
├── .env                   # Variáveis de ambiente
└── README.md              # Documentação do projeto

```

## Requisitos Técnicos Atendidos
- Dockerfile multi-stage usando Python 3.11 Alpine.
- Docker Compose multi-container com app Flask e MySQL.
- Volume persistente para o banco de dados.
- Rede customizada `app_network`.
- Variáveis de ambiente em `.env`.
- Usuário não-root criado no `init.sql`.
- CRUD funcional (GET e POST).
- Documentação detalhada neste README.

## Configuração do Ambiente
Crie um arquivo `.env` na raiz do projeto:
```

MYSQL_DATABASE=usersdb
MYSQL_USER=app_user
MYSQL_PASSWORD=12345
MYSQL_ROOT_PASSWORD=rootpass

````

O `init.sql` inicializa o banco:
```sql
CREATE DATABASE IF NOT EXISTS usersdb;
CREATE USER IF NOT EXISTS 'app_user'@'%' IDENTIFIED BY '12345';
GRANT ALL PRIVILEGES ON usersdb.* TO 'app_user'@'%';
FLUSH PRIVILEGES;

USE usersdb;

CREATE TABLE IF NOT EXISTS users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);
````

## Dockerfile

```dockerfile
# Etapa 1: build
FROM python:3.11-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt

# Etapa 2: runtime
FROM python:3.11-alpine
WORKDIR /app
COPY --from=builder /install /usr/local
COPY . .
CMD ["python", "app.py"]
```

## docker-compose.yml

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app_network

  app:
    build: .
    container_name: flask_app
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=db
      - DB_USER=${MYSQL_USER}
      - DB_PASSWORD=${MYSQL_PASSWORD}
      - DB_NAME=${MYSQL_DATABASE}
    depends_on:
      - db
    networks:
      - app_network

volumes:
  db_data:

networks:
  app_network:
    driver: bridge
```

## Como Executar o Projeto

1. Certifique-se de ter Docker e Docker Compose instalados.
2. Na raiz do projeto, execute:

```
docker compose up --build
```

3. Aguarde até ver: `Running on http://0.0.0.0:5000/`
4. Teste a API via Postman ou curl.

## Testando a API

**GET /users**:

```
http://localhost:5000/users
```

Resposta:

```json
[]
```

**POST /users**:

```
http://localhost:5000/users
```

Body (JSON):

```json
{
  "name": "Vinícius Lima",
  "email": "vinicius@example.com"
}
```

Resposta:

```json
{
  "message": "User added successfully"
}
```

**GET novamente**:

```json
[
  {
    "id": 1,
    "name": "Vinícius Lima",
    "email": "vinicius@example.com"
  }
]
```

## Acessando o Banco de Dados Manualmente

Dentro do container:

```
docker exec -it mysql_db bash
mysql -u app_user -p
```

Senha: definida no `.env` (ex: `12345`)

Comandos úteis:

```sql
SHOW DATABASES;
USE usersdb;
SHOW TABLES;
SELECT * FROM users;
```

## Boas Práticas e Segurança

* Usuário root não é usado pela aplicação.
* `app_user` tem acesso restrito ao banco `usersdb`.
* Variáveis sensíveis fora do código (em `.env`).
* Rede customizada para isolamento (`app_network`).
* Build multi-stage garante imagens menores e seguras.

## Conclusão

Este projeto cumpre todos os requisitos: multi-container, volume persistente, rede customizada, variáveis de ambiente, usuário seguro, CRUD funcional e documentação detalhada.

**Autor:** Vinícius Lima
**Disciplina:** DevOps
**Ano:** 2025

