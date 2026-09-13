---
type: produto
linha: seller
status: ativo
---

# Mercado Livre

Produto Seller para operações e integrações do Mercado Livre.

## Código e serviços

- Aplicação: `apps/seller-ml/`.
- Worker de filas: `apps/seller-ml/worker.js`.
- Acesso e créditos no Hub: `apps/seller-ml/services/hubAccessService.js` e `apps/seller-ml/services/hubCreditsService.js`.

## Variáveis e OAuth

- `ML_PUBLIC_ORIGIN` define a origem pública; no staging é `https://staging.dachbyte.tech`.
- `ML_REDIRECT_URI` define o callback OAuth de staging. `ML_APP_ID` e `ML_CLIENT_SECRET` pertencem ao aplicativo Mercado Livre.
- `ML_ALLOWED_ORIGINS` e `ML_EXTENSION_ALLOWED_ORIGINS` controlam origens permitidas.
- O master local usa `ML_BOOTSTRAP_MASTER_ENABLED`, `ML_BOOTSTRAP_MASTER_EMAIL`, `ML_BOOTSTRAP_MASTER_PASSWORD` ou `ML_BOOTSTRAP_MASTER_PASSWORD_HASH`, e `ML_BOOTSTRAP_MASTER_UPDATE_PASSWORD`.
- `ML_DATABASE_URL`, `ML_JWT_SECRET` e `ML_TOKEN_ENCRYPTION_KEY` são secrets operacionais; o acesso global continua sendo validado pelo Hub com `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`, `HUB_LOGIN_MODE` e `HUB_AUTH_MODE`.

Inventário completo e regra de alteração: [[Inventário de variáveis — Hub e staging]].

## Relações

- [[Hub]]
- [[Gateway]]
- [[Refatoração Davantti para Dachbyte]]
