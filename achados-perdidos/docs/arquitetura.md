# Diagrama de arquitetura

```mermaid
flowchart TB
    U["Navegador / curl (HOST)"] -->|"localhost:8080 (publicada)"| W

    subgraph REDE["rede_interna (bridge)"]
        W["web (Flask + Gunicorn)<br/>porta 5000 no contêiner<br/>usuário appuser"]
        D["db (postgres:16-alpine)<br/>porta 5432 - NÃO publicada"]
        W -->|"SQL host 'db':5432"| D
    end

    W --- VF[("volume fotos<br/>/app/uploads")]
    D --- VD[("volume dados_banco<br/>/var/lib/postgresql/data")]
    I["./db/init.sql (HOST)"] -.->|"bind mount :ro<br/>/docker-entrypoint-initdb.d/init.sql"| D
```

- **Fora da rede interna:** o host (navegador/curl) e o arquivo `init.sql`.
- **Dentro da rede interna:** `web` e `db`.
- **Única porta publicada:** 8080 (host) → 5000 (web). O `db` não publica porta.
