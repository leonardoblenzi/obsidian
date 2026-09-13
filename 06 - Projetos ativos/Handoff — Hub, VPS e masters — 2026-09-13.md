---
type: handoff
status: pronto-para-retomar
updated: 2026-09-13
---

# Handoff — Hub, VPS e masters — 2026-09-13

Use esta nota como ponto de retomada em outro chat. Ela descreve o estado **já aplicado e verificado**, os limites atuais e os próximos testes. Não contém valores de secrets.

## Estado em uma frase

O staging Dachbyte está saudável na VPS, todos os módulos apontam para o Hub Cloudflare `paymentcontrol`, e esse Worker está conectado à branch Neon piloto `paymentcontrol-pilot-20260910`.

## Topologia ativa

| Camada | Estado e referência |
| --- | --- |
| Staging público | `https://staging.dachbyte.tech` |
| VPS | Hostinger `srv1971387`; acesso SSH já validado. |
| Repositório na VPS | `/opt/dachbyte/repository`, branch `dach`. |
| Entrada da VPS | Caddy; serviços e bancos em Docker na rede privada. |
| Hub ativo | `https://paymentcontrol.davantti-suite.workers.dev` (`/ops-portal`). |
| Código do Hub | Repositório `hubPagamentos`, branch Git `payment`; ambiente `[env.paymentcontrol]`. |
| Banco do Hub | Projeto Neon `hidden-haze-81550394`, branch `paymentcontrol-pilot-20260910`, filha de `040926branch`. |
| Versão Cloudflare ativa | `c6b67479`, promovida a 100% em 13/09/2026; alteração de configuração `NEON_DATABASE_URL`. |

## O que foi concluído

### Worker e Neon

- Acessos ao Cloudflare e Neon foram confirmados no navegador integrado.
- A branch Neon correta foi aberta e confirmada como ativa: `paymentcontrol-pilot-20260910`.
- O secret Cloudflare `NEON_DATABASE_URL` do Worker `paymentcontrol` foi rotacionado para essa branch.
- A versão criada pela rotação (`c6b67479`) foi promovida a 100% do tráfego.
- O portal do Hub continuou acessível após a promoção e mostra quatro operadores internos.

### VPS e módulos

- Foram encontrados 13 ambientes de módulo ainda apontando para o Worker legado `hub-pagamento`.
- Todos foram alterados para usar `HUB_BASE_URL=https://paymentcontrol.davantti-suite.workers.dev`.
- Foram recriados, sem rebuild e sem alteração de banco, 14 serviços: gateway, Core, ML web/worker, Shopee, Rastreio, Madeira, Leader, Log, Price, Stock, Chat, Chat API e portal Business.
- Todos os serviços ficaram `healthy`; staging e health do Core retornaram `200`.
- Backup recuperável dos ambientes anterior à troca: `/opt/dachbyte/backups/hub-base-url-before-paymentcontrol-20260913T142153Z`.

### Masters e escopos

No Hub ativo, os escopos observados no `/ops-portal` são:

| Master | Escopo permitido |
| --- | --- |
| Mercado Livre | `ml` |
| Shopee | `shopee` |
| Rastreio | `tracking` |
| Volt Core | `volt_core` |

- O master local do Core foi provisionado e testado com sucesso (`200`, `admin_master`).
- O operador do Volt Core existe no Hub e tem somente `volt_core`.
- A Shopee foi reparada na base existente `dachbyte_shopee`; antes houve dump recuperável e depois as migrations foram reaplicadas. Não foi criada uma base nova.
- O fallback de senha fixa do bootstrap da Shopee foi removido. O seed exige a variável `SUPER_ADMIN_PASSWORD`.

## Separação de bancos

- Os módulos de staging usam PostgreSQL interno na VPS, com bases `dachbyte_*`; não há sufixo `_staging`, pois o isolamento é definido pela stack/ambiente.
- O Hub permanece no Cloudflare + Neon por decisão explícita; não migrar essa base para a VPS sem novo plano de corte.
- A branch Neon piloto é isolada da produção e da branch origem; não alterar `production` ou `040926branch` ao trabalhar no Hub de staging.

## Variáveis e arquivos a consultar

Antes de modificar qualquer ambiente, consultar [[Inventário de variáveis — Hub e staging]]. Os nomes críticos são:

| Fluxo | Variáveis principais |
| --- | --- |
| Hub Cloudflare | `NEON_DATABASE_URL`, `HUB_PUBLIC_BASE_URL`, `HUB_INTERNAL_TOKEN`, `HUB_ACCESS_ENFORCEMENT_MODE`, `HUB_PLATFORM_SCOPES_ENABLED` |
| Integração VPS | `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`, `HUB_LOGIN_MODE`, `HUB_AUTH_MODE`, `HUB_ENFORCEMENT` |
| Core | `VOLT_CORE_BOOTSTRAP_MASTER_ENABLED`, `VOLT_CORE_BOOTSTRAP_MASTER_EMAIL`, `VOLT_CORE_BOOTSTRAP_MASTER_NAME`, `VOLT_CORE_BOOTSTRAP_MASTER_PASSWORD` |
| ML/OAuth | `ML_PUBLIC_ORIGIN`, `ML_REDIRECT_URI`, `ML_APP_ID`, `ML_CLIENT_SECRET`, `ML_BOOTSTRAP_MASTER_*` |
| Shopee | `SHOPEE_DATABASE_URL`, `SHOPEE_*`, `SUPER_ADMIN_PASSWORD` |

Arquivos de ambiente na VPS ficam em `/opt/dachbyte/repository/infra/env/` e não são versionados. Valores de secrets não devem ir para Git, terminal compartilhado ou Obsidian.

## Pontos ainda abertos

1. **Senha global de masters no Hub:** criar operador no `/ops-portal` define escopo, mas não cria senha global. O Core possui bootstrap local; isso não atualiza automaticamente a credencial global do Hub. Usar o fluxo oficial de recuperação de senha do Hub ou uma alteração direta e controlada na branch Neon piloto.
2. **Teste de login centralizado:** autenticar os masters de ML, Shopee, Rastreio e Core, abrir a seleção de plataforma e verificar que cada um vê e entra apenas no módulo do próprio escopo.
3. **OAuth Mercado Livre:** validar o callback configurado em `ML_REDIRECT_URI` no staging e confirmar `ML_PUBLIC_ORIGIN`.
4. **Backups:** concluir configuração de backup criptografado, retenção e um teste de restauração.
5. **Produção:** não há serviço Dachbyte de produção ativo. Não alterar DNS, tráfego, callbacks produtivos ou encerrar Render/Neon sem autorização específica.

## Como verificar antes de avançar

- No Cloudflare: Worker `paymentcontrol` → Deployments; confirmar versão ativa e 100% de tráfego.
- No Neon: projeto `hubpagamento` → branch `paymentcontrol-pilot-20260910`; não selecionar `production` nem `040926branch` para mudanças de staging.
- Na VPS: verificar serviços com `docker compose --env-file infra/env/compose.env -f infra/compose.vps.yml ps` dentro de `/opt/dachbyte/repository`.
- No Hub: abrir `/ops-portal#operators` e conferir os quatro escopos de master.
- No staging: verificar `https://staging.dachbyte.tech` e `https://staging.dachbyte.tech/core/healthz`.

## Rollback conhecido

- **Worker:** a versão anterior `1aa57f70` continua no histórico do Cloudflare e pode ser promovida se a nova conexão apresentar falhas.
- **VPS:** restaurar os arquivos de ambiente a partir do backup datado acima e recriar somente os serviços afetados.
- **Shopee:** o dump anterior à reparação do schema está em `/opt/dachbyte/backups/dachbyte_shopee_before_schema_repair_20260913T133153Z.dump`.

## Relações

- [[Migração para VPS]]
- [[VPS e staging]]
- [[Identidade e acesso]]
- [[Inventário de variáveis — Hub e staging]]
- [[Mercado Livre]]
- [[Shopee]]
