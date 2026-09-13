---
type: projeto
status: staging-ativo
---

# Migração para VPS

## Objetivo

Executar a plataforma Dachbyte em VPS própria, com serviços isolados, bancos e filas privados, backup externo e uma entrada HTTPS.

## Estado

O staging está em execução na VPS, com serviços Dachbyte e bancos internos privados. O Hub continua intencionalmente no Cloudflare Worker `paymentcontrol`, usando a branch Neon `paymentcontrol-pilot-20260910`. Não há autorização implícita para alterar DNS, dados de produção, callbacks OAuth produtivos ou encerrar Render e Neon.

## Próximos passos

1. Consolidar os testes de login centralizado e abertura de módulos por escopo.
2. Validar rotas, filas ML e Chat no staging.
3. Configurar e testar restauração de backup.
4. Planejar corte apenas após critérios de saúde, autenticação, OAuth, fila e rollback.

## Referências

- `README-VPS.md`.
- `docs/operations/dachbyte-vps-staging-runbook.md`.
- [[VPS e staging]].
- [[Inventário de variáveis — Hub e staging]].
