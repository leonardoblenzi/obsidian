---
type: handoff
status: operacional-com-pendencias
updated: 2026-10-01
---

# Estado operacional DACH — Hub, VPS e módulos — 2026-10-01

## Resumo executivo

A plataforma DACHBYTE está centralizada na VPS. O Hub, os bancos operacionais e os módulos usam a infraestrutura local; Cloudflare Worker/Neon deixaram de ser a autoridade transacional. O Hub responde em `https://hub.dachbyte.tech`, o domínio de produtos responde em `https://dachbyte.tech` e todos os serviços principais estavam ativos na última inspeção.

O Asaas foi finalmente cortado para a VPS em 01/10/2026. A API de produção foi validada, o webhook aponta para o Hub novo e o scheduler foi corrigido e ativado. Falta um pagamento controlado para provar o ciclo financeiro completo com dados reais.

## Repositórios e revisões implantadas

| Componente | Checkout VPS | Branch | Revisão verificada |
| --- | --- | --- | --- |
| Plataforma e módulos | `/opt/dachbyte/repository` | `main` | `d0ea2265` |
| Hub de pagamentos | `/opt/dachbyte/hub-pagamento` | `main` | `b83c1ad` |

O checkout do Hub usa `core.sshCommand` próprio com `/root/.ssh/hub_github_deploy_ed25519`, evitando conflito com a Deploy Key do repositório DACHBYTE.

## Hub e identidade

- Runtime público: `https://hub.dachbyte.tech`.
- Serviço: `hub-web`; health `/healthz` retorna banco `ok`.
- Banco: `dachbyte_hub`, privado na rede Docker.
- Migrations conhecidas até `058_allow_magalu_credit_resources.sql`.
- Portal administrativo: `/ops-portal`.
- Roles separadas: `dachbyte_hub_app` para runtime e `dachbyte_hub_migrator` para migrations.
- Consumidores usam `HUB_BASE_URL=https://hub.dachbyte.tech` e `HUB_INTERNAL_TOKEN` compartilhado.
- Masters internos permanecem como operadores internos no portal, mas a autorização efetiva vem dos escopos `module_master` por módulo.

## Seleção Seller e masters

- A seleção oferece os tipos ML, Shopee, Magalu e Tracking, mas só mostra cards autorizados em `entitlements.modules`.
- Cards nascem ocultos e desabilitados; não há navegação para módulo sem entitlement.
- O backend também bloqueia acesso direto por `/go/<módulo>`.
- O master ML foi testado: sessão contém somente `ml`, `/go/ml` libera e `/go/magalu` bloqueia.
- Correções: `ced525c8` (`Fix Seller selection for platform masters`) e `b654b52e` (`Hide unauthorized Seller modules`).
- Testes ainda pendentes: login completo e abertura de Shopee e Tracking com seus masters; repetir Magalu quando o escopo master for usado operacionalmente.

## Módulos e bancos

| Área | Serviço/banco | Estado resumido |
| --- | --- | --- |
| Mercado Livre | `seller-ml-web`, `seller-ml-worker`, `dachbyte_ml` | Serviços ativos; master ML e escopo central validados. |
| Shopee | `seller-shopee`, `dachbyte_shopee` | Serviço ativo; login master ponta a ponta ainda deve ser repetido após o cutover do Hub. |
| Tracking | `seller-tracking`, `dachbyte_tracking` | Serviço ativo; login master ponta a ponta ainda deve ser repetido. |
| Magalu | `seller-magalu-web`, `seller-magalu-worker`, banco no escopo local | Serviços ativos; integração de acesso depende do entitlement `magalu`. |
| Core | `business-core`, `dachbyte_core` | Master e login central `volt_core` validados. |
| Ads | `ads-api`, `ads-worker`, `dachbyte_ads` | Serviços ativos; OAuth/fluxos externos devem continuar sendo testados por provedor. |
| Chat | `business-chat`, `business-chat-api`, `dachbyte_chat` | Serviços ativos. |
| Stock/Price | `business-stock`, `business-price` | Serviços e bancos locais ativos. |
| Madeira/Leader/Log | serviços dedicados e bancos `dachbyte_madeira`, `dachbyte_leader`, `dachbyte_log` | Ativos; jornadas próprias, fora da seleção Seller. |

## Asaas

- Variáveis ativas em `infra/env/hub-runtime.env`: `PAYMENT_PROVIDER`, `ASAAS_API_KEY`, `ASAAS_API_BASE_URL`, `ASAAS_WEBHOOK_TOKEN`, `PAYMENT_SUCCESS_REDIRECT_URL`, `RENEWAL_INTENT_SECRET` e `SUITE_GATEWAY_BASE_URL`.
- Webhook ativo: `https://hub.dachbyte.tech/v1/public/webhooks/payment?provider=asaas`.
- Eventos: `PAYMENT_CREATED`, `PAYMENT_RECEIVED`, `PAYMENT_CONFIRMED`, `PAYMENT_OVERDUE`, `PAYMENT_REFUNDED`, `PAYMENT_DELETED`.
- Scheduler: `HUB_PIX_RENEWAL_CRON=0 * * * *` e `HUB_MAINTENANCE_CRON=17 9 * * *`, ambos em UTC.
- Bug corrigido no Hub `b83c1ad`: remoção de `timer.unref()` para impedir encerramento do processo Node.
- Estado pré/pós-cutover salvo em `infra/.cutover/asaas-webhook-before.json` e `infra/.cutover/asaas-webhook-after-20261001.json`.

## Variáveis: regra de manutenção

Os valores nunca devem ser copiados para Git ou Obsidian. Ao alterar algum fluxo, registrar sempre o nome da variável, arquivo, serviço consumidor e necessidade de recriação:

- Hub: `infra/env/hub-runtime.env`, `hub-migrate.env`, `hub-cutover.env`, `hub.env`.
- Gateway: `infra/env/gateway.env` e variáveis compartilhadas `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`, `HUB_LOGIN_MODE`, `HUB_AUTH_MODE`, `HUB_ENFORCEMENT`.
- ML: `infra/env/seller-ml.env`, incluindo `ML_*`, OAuth e bootstrap master.
- Shopee: `infra/env/seller-shopee.env`, incluindo `SHOPEE_*`, `SUPER_ADMIN_PASSWORD` e e-mails master.
- Tracking: `infra/env/seller-tracking.env`, incluindo banco, integrações e referências master.
- Core: `infra/env/business-core.env`, incluindo `VOLT_CORE_*`.
- Ads: `infra/env/ads.env`, `ads-google.env`, `ads-migrate.env`.

## Próximos passos prioritários

1. Criar checkout Asaas real e controlado para uma empresa de teste, preferencialmente FZ Tech.
2. Confirmar `payment_provider_webhook_events`, contrato, recurso, convite e entitlement após pagamento.
3. Reenviar o mesmo evento e provar idempotência.
4. Testar masters Shopee e Tracking desde `/login` até o módulo correto.
5. Testar recuperação de senha/Brevo no Hub VPS.
6. Configurar backup externo Restic/R2, retenção e restauração ensaiada.
7. Criar monitoramento para `/healthz`, webhook Asaas, filas, reinicializações e atraso do scheduler.
8. Revisar e remover Deploy Keys desnecessárias; manter a chave dedicada do Hub.

## Relações

- [[Cutover emergencial do Hub para VPS — 2026-09-27]]
- [[VPS e staging]]
- [[Inventário de variáveis — Hub e staging]]
- [[Identidade e acesso]]
- [[Cobrança e uso]]
- [[Seller e operações]]
