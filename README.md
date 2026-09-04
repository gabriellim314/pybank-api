# 🏦 PyBank API

API REST de um sistema bancário simplificado, construída em Python com foco em boas práticas de arquitetura, consistência de dados financeiros e testes automatizados.

![Python](https://img.shields.io/badge/Python-3.12-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-REST-009688)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-336791)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

---

## 📖 Sobre o projeto

O **PyBank API** simula as operações centrais de uma instituição financeira: criação de contas, depósitos, saques, transferências entre contas e extrato de transações — tudo com autenticação JWT, migrations versionadas e transações atômicas no banco de dados.

O objetivo do projeto é demonstrar domínio de:
- Arquitetura em camadas (Router → Service → Repository)
- Modelagem correta de valores monetários (`Decimal`, nunca `float`)
- Atomicidade e consistência em operações financeiras
- Testes unitários e de integração
- Containerização e pipeline de CI

---

## ✨ Funcionalidades (planejadas)

- [ ] Cadastro e autenticação de usuários (JWT + hash de senha)
- [ ] Criação de contas bancárias
- [ ] Depósito
- [ ] Saque com validação de saldo
- [ ] Transferência entre contas com **atomicidade garantida**
- [ ] Extrato de transações
- [ ] Documentação automática via Swagger

---

## 🏗️ Arquitetura

```
Client
   │
   ▼
FastAPI
   │
   ├── Routers        → camada HTTP
   ├── Services        → regras de negócio
   ├── Repositories     → acesso a dados
   └── SQLAlchemy
          │
          ▼
      PostgreSQL
```

Separação de responsabilidades:

```
Router → Service → Repository → Database
```

---

## 🧱 Stack tecnológica

| Camada             | Tecnologia              |
|--------------------|--------------------------|
| Linguagem          | Python 3.12              |
| Framework web      | FastAPI                  |
| ORM                | SQLAlchemy 2.0           |
| Banco de dados     | PostgreSQL               |
| Migrations         | Alembic                  |
| Validação / DTOs   | Pydantic                 |
| Autenticação       | JWT + Argon2/Bcrypt      |
| Testes             | Pytest                   |
| Containerização    | Docker + Docker Compose  |
| CI/CD              | GitHub Actions           |
| Deploy (futuro)    | AWS                      |

---

## 🗄️ Modelagem de dados

```
User
 │
 └── 1:N ── Account
                │
                └── 1:N ── Transaction
```

**users** — id, name, email, password_hash, created_at
**accounts** — id, account_number, user_id, balance (`NUMERIC(15,2)`), status, created_at
**transactions** — id, account_id, type, amount, created_at

> 💡 Todos os valores monetários usam `Decimal` em Python e `NUMERIC(15,2)` no PostgreSQL — nunca `float`, para evitar erros de arredondamento em operações financeiras.

---

## 📡 Endpoints

### Autenticação
| Método | Rota             | Descrição              |
|--------|------------------|-------------------------|
| POST   | `/auth/register` | Cria um novo usuário    |
| POST   | `/auth/login`    | Retorna um access token |

### Contas
| Método | Rota                | Descrição            |
|--------|---------------------|------------------------|
| POST   | `/accounts`         | Cria uma nova conta   |
| GET    | `/accounts/{id}`    | Consulta uma conta    |

### Transações
| Método | Rota                              | Descrição                |
|--------|-------------------------------------|----------------------------|
| POST   | `/accounts/{id}/deposit`          | Realiza um depósito       |
| POST   | `/accounts/{id}/withdraw`         | Realiza um saque          |
| GET    | `/accounts/{id}/transactions`     | Lista o extrato da conta  |

### Transferências
| Método | Rota         | Descrição                          |
|--------|--------------|-------------------------------------|
| POST   | `/transfers` | Transfere valores entre duas contas |

Documentação interativa completa disponível em `/docs` (Swagger UI) após subir a aplicação.

---

## ⚙️ Como executar

### Pré-requisitos
- Docker e Docker Compose instalados

### Passo a passo
```bash
# clone o repositório
git clone https://github.com/seu-usuario/pybank-api.git
cd pybank-api

# copie as variáveis de ambiente
cp .env.example .env

# suba os containers
docker compose up --build
```

A API estará disponível em `http://localhost:8000` e a documentação em `http://localhost:8000/docs`.

### Rodando localmente sem Docker
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

pip install -r requirements.txt

# aplique as migrations
alembic upgrade head

# rode a aplicação
uvicorn app.main:app --reload
```

---

## 🔑 Variáveis de ambiente

| Variável        | Descrição                          | Exemplo                                   |
|------------------|--------------------------------------|---------------------------------------------|
| `DATABASE_URL`   | String de conexão com o PostgreSQL | `postgresql://user:pass@localhost:5432/db` |
| `JWT_SECRET_KEY` | Chave secreta para geração de token | `sua-chave-secreta`                        |
| `JWT_ALGORITHM`  | Algoritmo do JWT                    | `HS256`                                    |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Tempo de expiração do token | `30`                                       |

---

## 🔄 Migrations (Alembic)

```bash
# gerar uma nova migration a partir dos models
alembic revision --autogenerate -m "create accounts"

# aplicar as migrations pendentes
alembic upgrade head
```

---

## 🧪 Testes

```bash
pytest
```

Cobertura inclui:
- Testes unitários de depósito, saque e transferência
- Validação de saldo insuficiente e valores inválidos
- Teste de integração garantindo **rollback** em transferências que falham no meio da operação (ex.: conta destino não recebe se a origem falhar)

---

## 🔁 CI/CD

Pipeline no GitHub Actions executado a cada push:

```
push → install dependencies → lint (ruff) → tests (pytest) → build
```

---

## 📁 Estrutura do projeto

```
pybank-api/
├── app/
│   ├── main.py
│   ├── core/           # config, segurança, conexão com o banco
│   ├── models/          # entidades SQLAlchemy
│   ├── schemas/         # DTOs Pydantic
│   ├── routers/         # camada HTTP
│   ├── services/        # regras de negócio
│   └── repositories/    # acesso a dados
├── tests/
├── alembic/
├── Dockerfile
├── docker-compose.yml
├── alembic.ini
├── pyproject.toml
├── .env.example
└── README.md
```

---

## 🗺️ Roadmap

- [ ] **Fase 1 — MVP**: projeto FastAPI, PostgreSQL, models e endpoint de criação de conta
- [ ] **Fase 2 — Operações bancárias**: depósito, saque, transferência atômica, extrato
- [ ] **Fase 3 — Backend profissional**: JWT, hashing de senha, Alembic, testes, Docker, Swagger
- [ ] **Fase 4 — Produção**: GitHub Actions, imagem Docker, deploy AWS, HTTPS, monitoramento

---

## 📄 Licença

Distribuído sob a licença MIT. Veja `LICENSE` para mais detalhes.
