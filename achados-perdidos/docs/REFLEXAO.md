# Reflexão e equivalência em nuvem

**Q1.** O `web` usa o nome `db` porque o Docker tem um servidor DNS interno que, em redes
bridge definidas pelo usuário (`rede_interna`), resolve o nome do serviço para o IP do
contêiner. Se fosse criado com `docker run` na bridge padrão, não haveria resolução por nome:
seria preciso usar o IP (que muda) ou `--link` (obsoleto).

**Q2.** Volume nomeado é gerenciado pelo Docker (fica em área própria do Docker) e é ideal
para dados da aplicação. Bind mount liga um caminho específico do host ao contêiner. O
`init.sql` é bind mount porque é um arquivo do projeto, que eu edito e versiono. Os dados do
banco são volume nomeado porque o Docker cuida do local e das permissões, e eles não
dependem da estrutura de pastas do host.

**Q3.** Não é aplicada. Os scripts de `/docker-entrypoint-initdb.d/` só rodam quando o
diretório de dados está vazio (primeira execução). Como `dados_banco` já tem dados, o
`init.sql` é ignorado. Para aplicar sem perder dados: executar o `ALTER TABLE` manualmente
(`docker compose exec db psql -U <usuario> -d <banco> -c "ALTER TABLE itens ADD COLUMN ..."`).

**Q4.** `depends_on` simples só espera o contêiner do db *iniciar*, não ficar pronto. O
PostgreSQL leva alguns segundos para inicializar (e rodar o `init.sql`), então o `web` pode
tentar conectar antes e falhar. Com `condition: service_healthy` o Compose espera o
`pg_isready` do healthcheck dar sucesso.

**Q5.** Cada instrução gera uma camada em cache, e o cache é invalidado a partir da primeira
camada que mudou. Como `requirements.txt` e o `pip install` vêm antes do `COPY app.py`, quando
só o `app.py` muda o Docker reaproveita as camadas do `pip install` e refaz apenas o `COPY
app.py` (e as seguintes). O build fica bem mais rápido.

**Q6.** Publicar a 5432 deixaria o banco acessível a qualquer um que alcance o host,
permitindo tentativas de acesso direto aos dados. No meu `docker-compose.yml` o `db` não tem
`ports`: só o `web`, dentro da `rede_interna`, o acessa. Pela LGPD (segurança e minimização
de dados pessoais), isso reduz o risco de vazamento de fotos de documentos. Medidas
adicionais: (1) senha forte no `.env` e acesso restrito ao servidor e aos volumes; (2) validar
o tipo de arquivo enviado e definir um prazo para apagar fotos de documentos pessoais.

**Q7.** O comando não consegue subir as 3 réplicas: o `web` publica a porta fixa 8080 no host
e só um contêiner pode usá-la (erro de porta já alocada). Para rodar três réplicas seria
preciso remover a porta fixa do `web` e colocar um proxy/balanceador (ex.: nginx) na frente.
Compartilhar o volume `fotos`: no mesmo host funciona (os nomes são uuid, não colidem), mas
em vários servidores um volume local não é compartilhado; seria necessário armazenamento
compartilhado (NFS ou armazenamento de objetos).

**Evidência:** ao rodar `docker compose up -d --scale web=3`, o Docker retornou `Bind for 0.0.0.0:8080 failed: port is already allocated` (`docs/evidencias/21_scale.txt`).

## 5.1 Equivalência em nuvem
| Peça local | Microsoft Azure | AWS | Google Cloud |
|---|---|---|---|
| Contêiner web | Azure Container Apps | ECS Fargate (ou App Runner) | Cloud Run |
| Contêiner db + volume dados_banco | Azure Database for PostgreSQL | Amazon RDS for PostgreSQL | Cloud SQL for PostgreSQL |
| Volume fotos | Blob Storage (ou Azure Files) | Amazon S3 (ou EFS) | Cloud Storage |
| rede_interna | Virtual Network (VNet) | VPC (sub-redes e security groups) | VPC |
| Arquivo .env (segredos) | Key Vault | Secrets Manager | Secret Manager |
| Imagem construída localmente | Azure Container Registry | Amazon ECR | Artifact Registry |

**Peça que mais muda:** o volume `fotos`. Localmente é uma pasta em disco montada no
contêiner; na nuvem os contêineres são efêmeros e podem ter várias réplicas, então as fotos
devem ir para um armazenamento de objetos (Blob/S3/Cloud Storage). Isso exige mudar o código
do app (hoje grava em `/app/uploads`), diferente das outras peças, que só trocam de serviço.
