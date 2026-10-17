# Achados & Perdidos — ambiente com Docker Compose

## Objetivo
Empacotar a aplicação web (Flask + Gunicorn) e o banco PostgreSQL em contêineres,
subindo tudo com um único comando, com dados e fotos persistidos em volumes.

## Pré-requisitos
- Docker e Docker Compose (nada mais precisa ser instalado).

## Configurar o `.env`
```
cp .env.example .env       # Windows: copy .env.example .env
```
Edite o `.env` e troque a senha. O arquivo `.env` **não** é versionado.

| Variável | Para que serve |
|---|---|
| DB_NAME | nome do banco |
| DB_USER | usuário do banco |
| DB_PASSWORD | senha do banco |

## Subir, testar, parar e destruir
```
docker compose up -d --build     # sobe o ambiente
docker compose ps                # web running, db healthy
curl http://localhost:8080/health
docker compose logs web --tail 20
docker compose down              # para e remove contêineres (volumes ficam)
docker compose down -v           # remove também os volumes (APAGA os dados)
```
(No PowerShell use `curl.exe`.)

## Rotas da API
| Método | Rota | Descrição |
|---|---|---|
| GET | /health | Mostra `status: ok` e `banco: conectado` |
| GET | /itens | Lista os itens |
| GET | /itens?local=texto | Lista filtrando por local |
| POST | /itens | Cadastra item (`descricao`, `local`, `foto` opcional) |
| GET | /fotos/<nome> | Devolve a foto |
| GET | /estatisticas | Total de itens no banco e de fotos no volume |

## Estrutura
```
app/ (app.py, requirements.txt, Dockerfile, .dockerignore)
db/init.sql
docs/ (arquitetura.md, RELATORIO.md, REFLEXAO.md, evidencias/)
docker-compose.yml  .env.example  .gitignore  README.md
```

## Diagrama
Veja `docs/arquitetura.md`.
