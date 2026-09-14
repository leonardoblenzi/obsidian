---
type: operacoes
status: staging-ativo
---

# VPS e staging

## Fatos verificados

- O staging está ativo na VPS `srv1971387`, com entrada pública em `https://staging.dachbyte.tech`.
- O checkout operacional `/opt/dachbyte/repository` acompanha `git@github.com:leonardoblenzi/dachbyte.git`, branch `main`; o primeiro corte foi implantado no commit `20d7807`.
- Caddy é a entrada HTTPS; produtos executam em serviços e containers separados.
- PostgreSQL e Redis permanecem na rede Docker privada.
- As bases de staging são PostgreSQL interno na VPS e usam os nomes `dachbyte_*` sem sufixo `_staging`; o isolamento é feito pela stack e ambientes de staging, não pelo nome do banco.
- O Hub é a exceção deliberada: permanece como Worker Cloudflare e usa a branch Neon `paymentcontrol-pilot-20260910`.
- O plano inclui backup externo e restauração validada; antes da correção de schema da Shopee foi criado dump recuperável da base existente.

## Autenticação de staging

- O gateway usa o Hub `https://paymentcontrol.davantti-suite.workers.dev` como autoridade de acesso.
- Os masters de ML, Shopee, Rastreio e Volt Core têm acesso restrito ao respectivo módulo no Hub.
- O Volt Core usa as variáveis `VOLT_CORE_BOOTSTRAP_MASTER_ENABLED`, `VOLT_CORE_BOOTSTRAP_MASTER_EMAIL`, `VOLT_CORE_BOOTSTRAP_MASTER_NAME` e `VOLT_CORE_BOOTSTRAP_MASTER_PASSWORD`; valores ficam somente no ambiente da VPS.
- Em 13/09/2026, todos os ambientes de módulos que ainda usavam o Hub legado foram alinhados para `HUB_BASE_URL=https://paymentcontrol.davantti-suite.workers.dev`; 14 serviços foram recriados e ficaram saudáveis.
- Consulte [[Inventário de variáveis — Hub e staging]] antes de alterar qualquer ambiente, segredo ou fluxo de autenticação.

## Deploy das landings Seller

- O gateway foi reconstruído a partir de `dachbyte/main` e ficou `healthy`.
- Estão publicadas as rotas `/seller`, `/seller/mercado-livre`, `/seller/shopee` e `/seller/rastreio`; o CSS compartilhado é servido em `/seller-assets/seller-landing.css`.
- A revisão `1000c8f` adiciona navegação superior direta entre os três módulos, estado ativo nas páginas específicas, menu móvel acessível, faixa operacional e maior profundidade nos cards da landing geral.
- O deploy não adicionou variáveis novas. Ele preserva os arquivos não versionados em `/opt/dachbyte/repository/infra/env/` e os fluxos existentes, incluindo `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`, `ML_PUBLIC_ORIGIN`, `ML_REDIRECT_URI`, `ML_BOOTSTRAP_MASTER_*`, `SHOPEE_*`, `SUPER_ADMIN_PASSWORD` e `VOLT_CORE_BOOTSTRAP_MASTER_*`.
- Backup do corte: `/opt/dachbyte/backups/new-main-20260914T014403Z`; branch de rollback: `rollback/dach-before-new-main-20260914T014403Z`.

## Atualização visual Seller e Business — 14/09/2026

- Publicada a experiência compartilhada de landings Seller/Business e a landing pública DACHBYTE Price em `/business/price`.
- O commit final aplicado na VPS foi `ce0cfaaa451f7a3c25db4b94b2d84857b5c78dfb`; `business-portal` foi recriado e ficou `healthy`. Gateway, Core, Stock e Chat permaneceram `healthy`.
- Smoke tests externos confirmaram 200 para Business, Price, Seller Mercado Livre e os assets compartilhados.
- Não foram alterados banco, migrations, variáveis, segredos, DNS, callbacks OAuth ou arquivos de backup não versionados.
- Registro completo: [[Landings Seller e Business — 2026-09-14]].

## Próximos passos de staging

1. Validar a senha global do Hub para cada master via fluxo oficial de identidade/recuperação de senha.
2. Testar login e abertura de cada módulo liberado no gateway.
3. Configurar backup criptografado, retenção e testar restauração.
4. Fazer a revisão visual e responsiva das novas landings Seller e acompanhar conversão antes de aumentar o conteúdo.
5. Automatizar o deploy de `dachbyte/main` com smoke tests e rollback.
6. Somente em etapa aprovada: DNS e tráfego de produção, callbacks OAuth produtivos e monitoramento de tráfego.

## Bloqueado até aprovação específica

- DNS e tráfego de produção.
- Dados de produção.
- Callbacks OAuth e webhooks externos.
- Desligamento de Render e Neon.

Fonte: `README-VPS.md` e `docs/operations/dachbyte-vps-staging-runbook.md`.
