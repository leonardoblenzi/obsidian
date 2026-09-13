---
type: produto
linha: seller
status: ativo
---

# Shopee

Produto Seller para integração, catálogo, pedidos e recursos da Shopee.

## Código e serviços

- Aplicação: `apps/seller-shopee/`.
- Acesso e cobrança no Hub: `apps/seller-shopee/src/services/hubAccessService.js` e `apps/seller-shopee/src/services/hubResourceBillingService.js`.

## Variáveis de identidade e integração

- `SHOPEE_DATABASE_URL` é a conexão local do módulo; `SHOPEE_API_BASE`, `SHOPEE_PARTNER_ID`, `SHOPEE_PARTNER_KEY`, `SHOPEE_REDIRECT_URL` e `SHOPEE_PUSH_WEBHOOK_SECRET` atendem a integração principal.
- `SHOPEE_ADS_API_BASE`, `SHOPEE_ADS_PARTNER_ID` e `SHOPEE_ADS_PARTNER_KEY` atendem Shopee Ads.
- O seed do superadministrador exige `SUPER_ADMIN_PASSWORD`; o fallback de senha fixa foi removido. E-mails de master, quando usados, são configurados por `SHOPEE_MASTER_ADMIN_EMAILS`, `MASTER_ADMIN_EMAILS` ou `MASTER_ADMIN_EMAIL`.
- A seleção de plataforma e o escopo global usam `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`, `HUB_LOGIN_MODE` e `HUB_AUTH_MODE`.

Inventário completo e regra de alteração: [[Inventário de variáveis — Hub e staging]].

## Relações

- [[Hub]]
- [[Gateway]]
- [[Migração para VPS]]
