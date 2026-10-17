# Relatório técnico

## (a) Dockerfile (app/Dockerfile)
| Instrução | O que faz / por quê |
|---|---|
| `FROM python:3.12-slim` | Imagem oficial do Python 3.12, variante enxuta (menor). |
| `WORKDIR /app` | Define a pasta de trabalho dentro da imagem. |
| `COPY requirements.txt .` | Copia só as dependências primeiro. |
| `RUN pip install --no-cache-dir -r requirements.txt` | Instala as dependências sem guardar cache do pip (imagem menor). |
| `COPY app.py .` | Copia o código por último. |
| `RUN mkdir ... useradd ... chown` | Cria `/app/uploads`, cria o usuário `appuser` sem privilégios e o torna dono da pasta. |
| `USER appuser` | A aplicação roda sem ser root (RNF04). |
| `EXPOSE 5000` | Documenta a porta da aplicação. |
| `CMD ["gunicorn", "-b", "0.0.0.0:5000", "app:app"]` | Inicia o Gunicorn (forma exec) escutando em todas as interfaces na porta 5000. |

**Ordem:** requirements antes do código para aproveitar o cache de camadas: se só o
`app.py` mudar, o `pip install` não é refeito. O `USER` vem depois das instruções que
precisam de root (instalar, criar usuário).

## (b) docker-compose.yml
- **db**: imagem `postgres:16-alpine`; `environment` lê `POSTGRES_*` do `.env`; volume
  `dados_banco` guarda os dados; `init.sql` montado como bind mount somente leitura em
  `/docker-entrypoint-initdb.d/` (cria a tabela na 1ª execução); `healthcheck` com
  `pg_isready`; sem `ports` (isolado); `restart: unless-stopped`.
- **web**: `build: ./app` (imagem própria); variáveis `DB_*` e `UPLOAD_DIR` vindas do
  `.env`; `ports: 8080:5000`; volume `fotos` em `/app/uploads`; `depends_on` com
  `service_healthy` (só inicia com o banco pronto, RNF05); `restart: unless-stopped`.
- **networks**: `rede_interna` (bridge definida pelo usuário), com DNS interno: `web` acha `db` pelo nome.
- **volumes**: `dados_banco` e `fotos` (nomeados, gerenciados pelo Docker).
- Nenhuma senha escrita no arquivo: tudo vem de `${...}` do `.env` (RNF03).

## (c) Caminho de POST /itens com foto
1. O `curl -F ... http://localhost:8080/itens` envia a requisição multipart para a porta 8080 do host.
2. O Docker encaminha a porta 8080 do host para a porta 5000 do contêiner `web`.
3. O Gunicorn recebe a requisição e passa para a função `cadastrar()` do Flask.
4. O Flask lê `descricao` e `local` e valida (se faltar, responde 400).
5. Gera um nome único (uuid + extensão) e salva a foto em `/app/uploads`.
6. `/app/uploads` é o volume `fotos`, então o arquivo vai para o disco gerenciado pelo Docker.
7. O app resolve o nome `db` pelo DNS da `rede_interna` e conecta na porta 5432.
8. Executa o `INSERT` na tabela `itens`; o PostgreSQL grava em `/var/lib/postgresql/data` (volume `dados_banco`).
9. O app responde `201` com o `id` e o nome da foto, que volta ao `curl`.

## (d) Ciclo de vida dos volumes

Resultados obtidos na Tarefa 5 (prints em `EVIDENCIAS.md`, na raiz do repositório):

| Situação | Resultado obtido | Motivo |
|---|---|---|
| `docker compose down` + `up -d` | Antes e depois: 5 itens e 2 fotos (evidências 10 e 12). | `down` remove contêineres e rede, mas os volumes ficam. |
| `docker volume ls` / `inspect` | Existem `achados-perdidos_dados_banco` e `achados-perdidos_fotos` (evidências 13 e 14). | Os dados ficam no disco do host Docker (campo `Mountpoint`), fora do contêiner. |
| `docker compose down -v` + `up -d` | 3 itens (os do `init.sql`) e 0 fotos (evidência 17); a URL da foto antiga dá 404. | `-v` apaga os volumes. Com o banco vazio, o PostgreSQL executa o `init.sql` de novo. |

**Onde os dados ficam fisicamente:** em
`/var/lib/docker/volumes/achados-perdidos_dados_banco/_data` e
`/var/lib/docker/volumes/achados-perdidos_fotos/_data` (campo `Mountpoint`, evidência 14).
Esse caminho fica dentro da máquina virtual do Docker, não numa pasta do Windows. O
`ls -l /app/uploads` (evidência 15) mostra as 2 fotos pertencentes ao usuário `appuser`,
confirmando que a aplicação roda sem privilégios de root.

**Tamanho da imagem:** `docker image ls` (evidência 19) mostra `achados-perdidos-web:latest`
com 219 MB de uso em disco (53,8 MB de conteúdo).
