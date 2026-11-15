# Relatório — Esteira CI/CD para API na AWS

> RASCUNHO: preencha todos os trechos `<...>` com dados reais. Não invente evidências.

## 1. Integrantes e tarefas
| Integrante | Tarefas |
|---|---|
| <nome> | <ex.: código, testes, EC2, CI/CD, documentação> |

Repositório: <URL GitHub> · Aplicação: http://<IP_PUBLICO>/health (disponível até <data>)

## 2. Diagrama
```
Dev -> branch feature/... -> Pull Request -> CI (pytest + docker build)
                                   |
                              merge na main
                                   v
              CI: build + push ghcr.io/<owner>/<repo>:<SHA>
                                   v
              CD (workflow_run, só se CI ok): SSH -> EC2
                        docker pull <SHA> + docker run -> GET /health
```

## 3. Infraestrutura e portas
| Item | Valor |
|---|---|
| EC2 | Ubuntu 24.04 LTS, x86_64, tipo <t3.micro?>, região <...> |
| Security Group | TCP/80 de <origens>; TCP/22 somente de <IPs confiáveis / runner>. Nunca 0.0.0.0/0 no SSH |
| Docker | instalado por `scripts/setup-ec2-ubuntu.sh` |
| Registry | GHCR, tag = SHA do commit, pacote público |
| Persistência | volume Docker/diretório <caminho> montado em `DB_PATH` |
| Segredos | `EC2_HOST`, `EC2_SSH_KEY`, `EC2_KNOWN_HOSTS` (secrets); `EC2_USER` (variable). Nenhum no código/README/logs |

## 4. Evolução funcional
- `DELETE /tasks/<id>`: 204 se removida, 404 se não existe. Testes: `test_delete_task`, `test_delete_not_found`.
- Filtro `GET /tasks?done=true|false` (400 se inválido). Teste: `test_filter_by_done`.
- Resultado local: 8 testes passando (`python -m pytest -q`).

## 5. Evidências (anexar prints/links)
- [ ] PR: <link> (revisão: <quem>)
- [ ] Run de CI aprovado no PR: <link>
- [ ] Run de CI na `main` publicando imagem: <link> · Pacote GHCR: <link>
- [ ] Run de CD aprovado: <link> · commit <SHA>
- [ ] `/health` retornando `{"status":"ok"}`: <print>
- [ ] Persistência: criar tarefa, recriar contêiner (`docker rm -f cloud-tasks-api` + novo deploy), `GET /tasks` ainda lista: <print>

## 6. Custos e encerramento
| Recurso | Custo estimado | Excluído em |
|---|---|---|
| EC2 | <...> | <data/hora> |
| EBS | <...> | <data/hora> |
| IP elástico (se usado) | <...> | <data/hora> |

Evidência da exclusão: <print do console>

## 7. Dificuldades
<o que falhou e como resolveram, ex.: SSH do runner sem IP fixo>
