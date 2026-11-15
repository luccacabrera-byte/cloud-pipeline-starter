# Relatório — Esteira CI/CD para API na AWS

## 1. Integrantes e tarefas
| Integrante | Tarefas |
|---|---|
| Lucca Trazzi | Trabalho individual: código (DELETE e filtro), testes, repositório, CI, EC2, deploy, documentação |

Repositório: https://github.com/luccacabrera-byte/cloud-pipeline-starter
Aplicação: http://54.198.10.239/health (disponível até o encerramento dos recursos)

## 2. Diagrama
```
Dev -> branch feature/... -> Pull Request -> CI (pytest + docker build)
                                   |
                              merge na main
                                   v
              CI: build + push ghcr.io/luccacabrera-byte/cloud-pipeline-starter:<SHA>
                                   v
              CD (workflow_run, só se CI ok): SSH -> EC2 -> docker pull/run -> GET /health
```
Nesta entrega o último passo foi executado manualmente com o mesmo script do CD (`scripts/deploy.sh`), ver seção 7.

## 3. Infraestrutura e portas
| Item | Valor |
|---|---|
| EC2 | `cloud-tasks-api`, Ubuntu Server 26.04 LTS, t3.micro, us-east-1 (AWS Academy Learner Lab) |
| Security Group | TCP/80 aberto (HTTP); TCP/22 apenas do IP do aluno ("Meu IP"). Nenhuma regra SSH para 0.0.0.0/0 |
| Docker | Instalado por `scripts/setup-ec2-ubuntu.sh` (Docker 29.1.3) |
| Registry | GHCR, tag = SHA do commit, pacote público |
| Persistência | Volume Docker `cloud-tasks-data` montado em `/data`, `DB_PATH=/data/tasks.db` |
| Chave SSH | Par `cloud-tasks-key`, fora do repositório |

## 4. Evolução funcional
- `DELETE /tasks/<id>`: 204 se removida, 404 se não existe. Testes: `test_delete_task`, `test_delete_not_found`.
- Filtro `GET /tasks?done=true|false` (400 se inválido). Teste: `test_filter_by_done`.
- 8 testes passando (`python -m pytest -q`).

## 5. Evidências
- PR integrado: https://github.com/luccacabrera-byte/cloud-pipeline-starter/pull/1
- CI aprovada no PR (sem publicar imagem): https://github.com/luccacabrera-byte/cloud-pipeline-starter/actions/runs/43760163171
- CI na `main` publicando a imagem: https://github.com/luccacabrera-byte/cloud-pipeline-starter/actions/runs/43760319975
- Imagem implantada: `ghcr.io/luccacabrera-byte/cloud-pipeline-starter:0de58d0a9e103044941d0a5a5a38a5cf4df0974e`
- `/health` na EC2: `{"status":"ok"}`
- API verificada na EC2: `DELETE /tasks/2` -> 204; `DELETE /tasks/99` -> 404.
- Persistência: tarefa "Persistencia" criada, contêiner recriado com `scripts/deploy.sh`, `GET /tasks` continuou retornando a tarefa.
- Prints: <ANEXAR prints do console AWS, do PR e do /health>

## 6. Custos e encerramento
| Recurso | Custo estimado | Excluído em |
|---|---|---|
| EC2 t3.micro (Linux sob demanda: US$ 0,0104/h) | <horas x 0,0104> | <data/hora> |
| EBS 8 GiB | <...> | <data/hora> |

Evidência da exclusão: <ANEXAR print do console AWS após excluir a instância e o volume>

## 7. Dificuldades
- **CD automático:** o runner hospedado do GitHub não tem IP fixo e o SSH não pode ficar aberto para 0.0.0.0/0 (regra do enunciado). As execuções de CD falharam por ausência de secrets e de rota SSH segura. O deploy foi feito manualmente com `scripts/deploy.sh`, o mesmo script do CD. Pendente: orientação do professor (runner auto-hospedado ou SSM).
- **Escopo `workflow` do token do GitHub CLI:** o push dos workflows foi recusado até autorizar o escopo `workflow`.
- **Versão do Ubuntu:** o guia cita 24.04; o console ofereceu 26.04 LTS e o script de setup funcionou nela.
