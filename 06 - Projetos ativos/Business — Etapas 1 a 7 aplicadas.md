---
type: projeto
status: aplicado-no-codigo
data: 2026-09-16
---

# Business — Etapas 1 a 7 aplicadas

## Resultado

As sete etapas recebidas em `C:\Users\USER\Downloads\etapas business` foram aplicadas no monorepo `C:\Users\USER\Documents\Projetos\dachbyte`, na branch `main`. Foram copiados/atualizados 141 arquivos. Nenhum segredo real foi incluído.

## Principais alterações

- Hardening e provisionamento seguro do Chat, Core, Price e Stock.
- Rotas canônicas `/business/core`, `/business/chat`, `/business/stock` e `/business/price`.
- Compatibilidade temporária das rotas antigas (`/core`, `/chat`, `/voltstock`, `/volt-price` e APIs legadas).
- Redirect 308 somente para interfaces antigas; APIs continuam proxyadas para não quebrar OAuth, POST ou WebSocket.
- Headers e logging privacy-safe para acompanhar uso de rotas legadas.
- Compose e scripts para Postgres, Redis, migração, staging, validação, rollback e cutover.
- Canais Desktop separados para staging e produção, com appId e prefixo R2 distintos.

## Arquivos operacionais

- `infra/business-staging-ops.sh`
- `infra/business-production-ops.sh`
- `infra/business-db-ops.sh`
- `infra/business-legacy-ops.sh`
- `infra/compose.migration.yml`
- `infra/compose.staging-validation.yml`
- `infra/compose.production-validation.yml`
- `infra/DB_MIGRATION_VPS.md`
- `infra/STAGING_VALIDATION.md`
- `infra/PRODUCTION_CUTOVER.md`
- `infra/LEGACY_DEPRECATION.md`

## Variáveis a configurar na VPS

Copiar os exemplos em `infra/env/` para arquivos sem `.example` e preencher somente na VPS:

- `compose.env`: `DACHBYTE_SITE`, `DACHBYTE_BIND_IP`.
- `postgres.env`: credenciais e banco PostgreSQL.
- `hub.env`: `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`.
- `business-core.env`: `VOLT_CORE_*`, incluindo bootstrap apenas durante provisionamento controlado.
- `business-chat.env` e `business-chat-api.env`: URLs, secrets, OAuth, Brevo e R2.
- `business-stock.env` e `business-price.env`: credenciais, integrações e flags de bootstrap.
- `staging-validation.env`: usuários de teste, tokens e URLs de smoke test.
- `production-validation.env` e `production-cutover.env`: hostname final e confirmações explícitas.

Não commitar arquivos `.env` reais.

## Comandos de staging

```bash
cd /opt/dachbyte/repository/infra
cp env/staging-validation.env.example env/staging-validation.env
./business-staging-ops.sh up
./business-staging-ops.sh all
```

Executar também `./business-staging-ops.sh public-smoke` quando apenas as rotas públicas precisarem ser verificadas.

## Próximas etapas

1. Instalar dependências ausentes do workspace e repetir a suite Business completa (`vite` está ausente no ambiente local).
2. Configurar os arquivos reais de `infra/env/` na VPS, sem versioná-los.
3. Rodar o gate completo de staging: saúde dos containers, bancos/roles, autenticação, OAuth, filas, WebSocket, uploads, isolamento de tenant e Hub.
4. Validar backup criptografado e restauração real.
5. Testar Desktop staging separado (login, WebSocket, upload/download, reconexão e atualização).
6. Monitorar e reduzir gradualmente o uso de rotas legadas.
7. Somente após aprovação explícita, executar `business-production-ops.sh preflight`, `prepare` e `cutover` com `CONFIRM_PRODUCTION_CUTOVER=YES`.

## Validações realizadas localmente

- Sintaxe JavaScript e Python: aprovada.
- Testes Price: 189/189 aprovados.
- Suite Business: 238 testes passaram; 1 bloqueado por dependência ausente (`vite`).
- Docker/Caddy/VPS reais ainda precisam ser executados no ambiente de staging.

## Git

Commit/push desta aplicação: a registrar após a revisão final da branch `main`.
