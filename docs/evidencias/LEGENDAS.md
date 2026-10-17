# Legendas das evidências

Os prints estão documentados no `README.md`, com as imagens na pasta `img/`. As
saídas completas em texto estão nos arquivos `.txt` desta pasta.

| ID | Legenda |
|---|---|
| 02_ps | `docker compose ps` com web em execução e db healthy. |
| 03_health | GET /health retornando status ok e banco conectado. |
| 04_itens | GET /itens listando os 3 itens do init.sql. |
| 05_filtro_biblio | GET /itens?local=biblio filtrando por local. |
| 06_cadastro_foto | POST /itens com foto retornando id e nome da foto. |
| 07_estatisticas | GET /estatisticas no início: 3 itens e 0 fotos. |
| 08_foto_navegador | Fotos acessadas pela rota /fotos/<nome>. |
| 09_logs_web | Últimas 20 linhas de log do serviço web. |
| 10_stats_antes | Estatísticas antes do down: 5 itens, 2 fotos. |
| 11_down_up | `docker compose down` seguido de `up -d`. |
| 12_stats_depois | Estatísticas depois do down/up: iguais às de antes. |
| 13_volume_ls | Lista dos volumes dados_banco e fotos. |
| 14_volume_inspect | Mountpoint físico de cada volume. |
| 15_ls_uploads | Fotos dentro do volume, pertencentes a appuser. |
| 16_down_v | `docker compose down -v` removendo os volumes. |
| 17_stats_apos_down_v | Só os 3 itens do init.sql e 0 fotos. |
| 18_network_inspect | web e db conectados à rede_interna. |
| 19_image_ls | Tamanho da imagem do web (219 MB em disco). |
| 20_whoami | Processo do web rodando como appuser, não root. |
| 21_scale | Erro ao tentar subir 3 réplicas por conflito da porta 8080. |
