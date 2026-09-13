---
type: operacoes
status: staging-ativo
---

# VPS e staging

## Fatos verificados

- O staging está ativo na VPS `srv1971387`, com entrada pública em `https://staging.dachbyte.tech`.
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

## Próximos passos de staging

1. Validar a senha global do Hub para cada master via fluxo oficial de identidade/recuperação de senha.
2. Testar login e abertura de cada módulo liberado no gateway.
3. Configurar backup criptografado, retenção e testar restauração.
4. Somente em etapa aprovada: DNS e tráfego de produção, callbacks OAuth produtivos e monitoramento de tráfego.

## Bloqueado até aprovação específica

- DNS e tráfego de produção.
- Dados de produção.
- Callbacks OAuth e webhooks externos.
- Desligamento de Render e Neon.

Fonte: `README-VPS.md` e `docs/operations/dachbyte-vps-staging-runbook.md`.
