---
type: projeto
status: producao-ativa
---

# Migração para VPS

## Objetivo

Executar a plataforma Dachbyte em VPS própria, com serviços isolados, bancos e filas privados, backup externo e uma entrada HTTPS.

## Estado

A plataforma está em execução na VPS, com serviços DACHBYTE e bancos PostgreSQL internos privados. Desde 27/09/2026, o Hub transacional também está na VPS em `https://hub.dachbyte.tech`, usando o banco local `dachbyte_hub`; o Worker Cloudflare e a branch Neon deixaram de ser a autoridade operacional. Em 01/10/2026, o webhook Asaas foi migrado para o Hub VPS e o `hub-scheduler` foi ativado após correção do ciclo de vida do processo.

## Próximos passos

1. Executar checkout Asaas controlado e validar o fluxo completo até a liberação do módulo.
2. Consolidar testes de login e abertura de cada módulo por escopo do Hub.
3. Configurar backup externo criptografado e testar restauração.
4. Automatizar deploy, smoke tests, alertas de webhook e rollback.

## Referências

- `README-VPS.md`.
- `docs/operations/dachbyte-vps-staging-runbook.md`.
- [[VPS e staging]].
- [[Inventário de variáveis — Hub e staging]].
