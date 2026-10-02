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

## Seleção compartilhada Seller

- A seleção `/selecao-plataforma` contém Mercado Livre, Shopee, Magalu e Tracking, mas renderiza somente os cards presentes em `entitlements.modules` na sessão confirmada pelo Hub.
- Todos os cards começam ocultos e desabilitados; o frontend só revela um módulo após confirmação positiva do Hub. A rota `/go/<módulo>` mantém uma segunda barreira de autorização no backend.
- Masters internos usam seus escopos `module_master` e não são tratados como tenants comerciais da suíte. O master ML foi validado com acesso somente a `ml`; tentativa de abrir Magalu foi bloqueada.
- DACH Ads, Log, Madeira e Leader seguem com jornadas próprias e não devem voltar a ser adicionados à seleção Seller.
- As rotas compatíveis dos módulos removidos permanecem protegidas pelo Gateway; removê-las da seleção não remove nem concede acesso.
- Detalhes de publicação e validação: [[Atualização de produção, seleção Seller e DACH Ads — 2026-09-21]].
- Correções de sessão/visibilidade publicadas em `ced525c8` e `b654b52e`.
- Mapa de responsabilidades de rota e sessão: [[Mapa de rotas e autenticação DACH]].

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
