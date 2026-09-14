---
type: produto
linha: seller
status: ativo
---

# Seller e operações

## Produtos verificados

- Madeira: `apps/seller-madeira/`.
- Rastreio: `apps/seller-tracking/`.
- Log: `apps/seller-log/`.
- Leader: `apps/seller-leader/`.

## Variáveis de acesso compartilhadas

- Madeira, Rastreio, Log e Leader usam a integração central `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`, `HUB_LOGIN_MODE` e `HUB_AUTH_MODE`.
- No Rastreio, `MASTER_ADMIN_EMAIL` é uma referência legada do código; o ambiente atual não a define e o master de staging é liberado pelo escopo `tracking` do Hub.
- A lista completa de variáveis e o procedimento de mudança estão em [[Inventário de variáveis — Hub e staging]].

## Relações

- [[Mercado Livre]]
- [[Shopee]]
- [[Hub]]
- [[Migração para VPS]]
- [[Landings Seller e Business — 2026-09-14]]

## A mapear

- Owners operacionais e SLAs de cada produto.
