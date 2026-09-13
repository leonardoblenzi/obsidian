---
type: arquitetura
---

# Gateway

O gateway concentra login, sessão, portal e rotas canônicas. A implementação está em `apps/gateway/` e `platform/gateway/`.

## Relações

- [[Hub]]
- [[Mercado Livre]]
- [[Shopee]]
- [[Business]]
- [[Compatibilidade durante a migração]]

## Operação

Na arquitetura de VPS, o Caddy encaminha o tráfego para o gateway e os produtos na rede Docker privada. Consulte [[VPS e staging]].

## Variáveis de identidade

- `HUB_BASE_URL` aponta para o Hub ativo `paymentcontrol`.
- `HUB_INTERNAL_TOKEN` autentica chamadas internas ao Hub.
- `HUB_LOGIN_MODE`, `HUB_AUTH_MODE` e `HUB_ENFORCEMENT` definem a aplicação da política de login e acesso.
- `ML_DATABASE_URL`, `ML_JWT_SECRET`, `SHOPEE_DATABASE_URL` e as credenciais `SHOPEE_*` são dependências de módulos atendidos pelo gateway; permanecem secrets na VPS.

Inventário e procedimento de alteração: [[Inventário de variáveis — Hub e staging]].
