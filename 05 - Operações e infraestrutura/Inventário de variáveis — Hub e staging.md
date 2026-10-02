---
type: operacoes
status: ativo
updated: 2026-10-01
---

# Inventário de variáveis — Hub e staging

> **Estado vigente desde 27/09/2026:** o runtime ativo é `https://hub.dachbyte.tech`; as variáveis do Worker/Neon abaixo são referência histórica para reconciliação. Os arquivos ativos são `hub.env`, `hub-runtime.env`, `hub-migrate.env`, `hub-cutover.env` e `postgres.env` na VPS. Consulte [[Cutover emergencial do Hub para VPS — 2026-09-27]].

Esta nota registra **nomes, arquivo/ambiente e responsabilidade**. Nunca registrar valores de senhas, tokens, chaves de API ou URLs de conexão com credenciais.

## Hub Cloudflare: Worker `paymentcontrol`

| Variável | Tipo | Responsabilidade |
| --- | --- | --- |
| `NEON_DATABASE_URL` | secret | Conexão do Hub com a branch Neon `paymentcontrol-pilot-20260910`. Atualizada em 13/09/2026 e publicada na versão Cloudflare `c6b67479`. |
| `HUB_PUBLIC_BASE_URL` | variável | URL pública do Hub: `https://paymentcontrol.davantti-suite.workers.dev`. |
| `HUB_ACCESS_ENFORCEMENT_MODE` | variável | Modo de aplicação de política no Hub; atualmente `monitor`. |
| `HUB_PLATFORM_SCOPES_ENABLED` | variável | Habilita escopos de plataforma para operadores internos. |
| `HUB_INTERNAL_TOKEN` | secret | Autentica chamadas internas dos módulos para o Hub. Deve corresponder ao valor da VPS. |
| `ENVIRONMENT` | variável | Identifica o ambiente do Worker; atualmente `production`. |
| `PAYMENT_PROVIDER`, `ASAAS_API_BASE_URL`, `ASAAS_API_KEY`, `ASAAS_WEBHOOK_TOKEN` | variável/secrets | Configuração de cobrança Asaas. |
| `ADMIN_BASE_PATH`, `ADMIN_USERNAME`, `ADMIN_PASSWORD_HASH`, `ADMIN_PASSWORD_SALT`, `ADMIN_SESSION_SECRET` | variável/secrets | Portal administrativo em `/ops-portal`; não são as credenciais globais dos usuários de módulos. |
| `BREVO_API_KEY`, `BREVO_SENDER_EMAIL`, `BREVO_SENDER_NAME` | secrets | Entrega de e-mails, inclusive fluxo de identidade/recuperação. |
| `RENEWAL_INTENT_SECRET` | secret | Assinatura de intenções de renovação. |

## Integração comum na VPS

| Variável | Responsabilidade |
| --- | --- |
| `HUB_BASE_URL` | Deve apontar para o runtime ativo `https://hub.dachbyte.tech`. O Worker `paymentcontrol.davantti-suite.workers.dev` é legado e não deve voltar a ser usado sem reconciliação. |
| `HUB_INTERNAL_TOKEN` | Credencial de serviço compartilhada com o secret homônimo do Worker. |
| `HUB_LOGIN_MODE` | Define o modo de autenticação do módulo contra o Hub. |
| `HUB_AUTH_MODE` | Define o modo de validação/autorização do módulo contra o Hub. |
| `HUB_ENFORCEMENT` | Aplicado em `infra/env/hub.env`; controla a aplicação estrita da política pelo gateway. |

Arquivos principais: `infra/env/hub.env`, `infra/env/gateway.env`, `infra/env/business-core.env`, `infra/env/seller-ml.env`, `infra/env/seller-shopee.env` e `infra/env/seller-tracking.env`. Todos ficam fora do Git e não devem ter valores copiados para o vault.

## Hub VPS e Asaas

| Variável | Arquivo | Responsabilidade |
| --- | --- | --- |
| `ASAAS_API_KEY` | `infra/env/hub-runtime.env` | Chave da API de produção do Asaas; validada em 01/10/2026. |
| `ASAAS_API_BASE_URL` | `infra/env/hub-runtime.env` | Base da API; produção usa `https://api.asaas.com/v3`. |
| `ASAAS_WEBHOOK_TOKEN` | `infra/env/hub-runtime.env` | Token conferido pelo endpoint público do webhook. |
| `HUB_PIX_RENEWAL_CRON` | `infra/env/hub-runtime.env` | Agenda do job PIX; configuração ativa `0 * * * *`, interpretada em UTC. |
| `HUB_MAINTENANCE_CRON` | `infra/env/hub-runtime.env` | Agenda de manutenção; configuração ativa `17 9 * * *`, interpretada em UTC. |
| `ASAAS_WEBHOOK_ID` | `infra/env/hub-cutover.env` | Identificador do webhook alterado pelo procedimento de cutover. |
| `HUB_CUTOVER_WEBHOOK_URL` | `infra/env/hub-cutover.env` | Destino do webhook: `https://hub.dachbyte.tech/v1/public/webhooks/payment?provider=asaas`. |
| `CONFIRM_ASAAS_WEBHOOK_CHANGE` | `infra/env/hub-cutover.env` | Trava de alteração; deve permanecer `NO` fora da janela controlada de cutover. |

O checkout do Hub usa `PAYMENT_PROVIDER=asaas`, `PAYMENT_SUCCESS_REDIRECT_URL`, `SUITE_GATEWAY_BASE_URL` e `RENEWAL_INTENT_SECRET`. Valores sensíveis permanecem somente na VPS.

## Volt Core

| Variável | Responsabilidade |
| --- | --- |
| `VOLT_CORE_BOOTSTRAP_MASTER_ENABLED` | Habilita o provisionamento/atualização do master local ao iniciar o Core. |
| `VOLT_CORE_BOOTSTRAP_MASTER_EMAIL` | Identificador do master local. |
| `VOLT_CORE_BOOTSTRAP_MASTER_NAME` | Nome exibido do master local. |
| `VOLT_CORE_BOOTSTRAP_MASTER_PASSWORD` | Senha de bootstrap do master local; secret. |
| `VOLT_CORE_BOOTSTRAP_MASTER_PASSWORD_HASH` | Alternativa à senha em claro, quando o hash for fornecido. |
| `VOLT_CORE_APP_DATABASE_URL` | Conexão da base operacional local `dachbyte_core`; secret. |
| `VOLT_CORE_JWT_SECRET` | Assinatura de sessão local; secret. |
| `VOLT_CORE_ALLOWED_ORIGINS` | Origens CORS permitidas. |
| `VOLT_CORE_APP_BASE_PATH` | Base de rota pública do módulo. |

O master local do Core é distinto da credencial global do Hub. O Hub controla a senha global e o escopo `volt_core`; o Core usa o bootstrap para a conta administrativa local.

## Mercado Livre

| Variável | Responsabilidade |
| --- | --- |
| `ML_PUBLIC_ORIGIN` | Origem pública do ML; em staging, `https://staging.dachbyte.tech`. |
| `ML_REDIRECT_URI` | Callback OAuth Mercado Livre de staging. |
| `ML_ALLOWED_ORIGINS`, `ML_EXTENSION_ALLOWED_ORIGINS` | Origens CORS e extensão permitidas. |
| `ML_APP_ID`, `ML_CLIENT_SECRET` | Credenciais OAuth do aplicativo Mercado Livre. |
| `ML_DATABASE_URL`, `ML_JWT_SECRET`, `ML_TOKEN_ENCRYPTION_KEY` | Banco, sessão e cifragem local; secrets. |
| `ML_BOOTSTRAP_MASTER_ENABLED` | Habilita bootstrap do master local ML. |
| `ML_BOOTSTRAP_MASTER_EMAIL` | E-mail do master local ML. |
| `ML_BOOTSTRAP_MASTER_PASSWORD` ou `ML_BOOTSTRAP_MASTER_PASSWORD_HASH` | Credencial/alternativa em hash do bootstrap; secrets. |
| `ML_BOOTSTRAP_MASTER_NAME` ou `ML_BOOTSTRAP_MASTER_NOME` | Nome do master local. |
| `ML_BOOTSTRAP_MASTER_UPDATE_PASSWORD` | Define se o bootstrap pode atualizar a senha existente. |
| `ML_ALLOW_USER_LOGIN`, `ML_AUTH_RATE_LIMIT_MAX`, `ML_ADMIN_RATE_LIMIT_MAX`, `ML_OAUTH_RATE_LIMIT_MAX` | Política de login e limitação de tentativas. |

As credenciais por conta (`ML_*_ACCESS_TOKEN`, `ML_*_REFRESH_TOKEN`, `ML_*_CLIENT_SECRET` e variáveis correlatas) são secrets de integração, não credenciais de master.

## Shopee e Rastreio

| Variável | Responsabilidade |
| --- | --- |
| `SHOPEE_DATABASE_URL` | Conexão da base operacional `dachbyte_shopee`; secret. |
| `SHOPEE_API_BASE`, `SHOPEE_PARTNER_ID`, `SHOPEE_PARTNER_KEY` | Integração principal Shopee. |
| `SHOPEE_ADS_API_BASE`, `SHOPEE_ADS_PARTNER_ID`, `SHOPEE_ADS_PARTNER_KEY` | Integração Shopee Ads. |
| `SHOPEE_PUSH_WEBHOOK_SECRET` | Validação de webhook Shopee; secret. |
| `SHOPEE_REDIRECT_URL` | Callback/redirecionamento da Shopee. |
| `SUPER_ADMIN_PASSWORD` | Exigida pelo script de seed do administrador da Shopee; não há fallback fixo de senha. Também está presente em ambientes compartilhados que usam o fluxo legado. |
| `SHOPEE_MASTER_ADMIN_EMAILS`, `MASTER_ADMIN_EMAILS`, `MASTER_ADMIN_EMAIL` | Lista/identificador de master reconhecido pelo código Shopee quando configurados. |
| `MASTER_ADMIN_EMAIL` | Referência legada usada no código de Rastreio; não está definida no ambiente atual da VPS. O master de Rastreio em staging é controlado pelo escopo `tracking` do Hub. |

## Volt Price

| Variável | Responsabilidade |
| --- | --- |
| `VOLT_PRICE_BOOTSTRAP_MASTER_ENABLED` | Habilita bootstrap temporário do master. |
| `VOLT_PRICE_BOOTSTRAP_MASTER_EMAIL` | E-mail do master local. |
| `VOLT_PRICE_BOOTSTRAP_MASTER_PASSWORD` | Senha do bootstrap; secret. |
| `VOLT_PRICE_BOOTSTRAP_MASTER_TOTP_SECRET` | Segundo fator do bootstrap; secret. |

## Regra operacional de alteração

1. Identificar nesta nota a variável e o ambiente-alvo.
2. Criar backup do arquivo de ambiente ou uma nova versão do Worker antes de salvar.
3. Alterar somente o destino necessário; nunca copiar secrets para Git ou Obsidian.
4. Recriar apenas os serviços afetados na VPS ou promover a versão correspondente do Worker.
5. Validar saúde e o fluxo de login/OAuth relacionado.

## Relações

- [[Identidade e acesso]]
- [[VPS e staging]]
- [[Gateway]]
- [[Mercado Livre]]
- [[Shopee]]
- [[Seller e operações]]
