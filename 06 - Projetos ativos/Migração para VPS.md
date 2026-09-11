---
type: projeto
status: staging-pendente
---

# Migração para VPS

## Objetivo

Executar a plataforma Dachbyte em VPS própria, com serviços isolados, bancos e filas privados, backup externo e uma entrada HTTPS.

## Estado

Staging pendente. A configuração local existe; não há autorização implícita para alterar DNS, dados de produção, callbacks OAuth ou encerrar Render e Neon.

## Próximos passos

1. Preparar VPS, acesso SSH e Docker.
2. Configurar ambientes e bases sanitizadas de staging.
3. Subir a stack e validar login, rotas, filas ML e Chat.
4. Configurar e testar restauração de backup.
5. Planejar corte apenas após critérios de saúde, autenticação, OAuth, fila e rollback.

## Referências

- `README-VPS.md`.
- `docs/operations/dachbyte-vps-staging-runbook.md`.
- [[VPS e staging]].
