---
type: handoff
status: hub-vps-ativo
updated: 2026-10-01
---

# Cutover emergencial do Hub para VPS — 2026-09-27

## Resultado

O Hub transacional deixou de depender do Worker Cloudflare `paymentcontrol` e da branch Neon bloqueada por quota. A autoridade usada pelos módulos DACH agora é `https://hub.dachbyte.tech`, atendida pelo container `hub-web` e pelo PostgreSQL privado da VPS.

O Worker antigo ainda responde a `/health`, mas sua conexão transacional com o Neon retorna erro de quota. Ele não deve voltar a ser configurado como `HUB_BASE_URL` sem reconciliação explícita.

## Origem dos dados

- Snapshot usado: rehearsal criado em `2026-09-25T19:06:09Z`.
- Integridade: SHA-256 conferido antes da restauração.
- Comparação do rehearsal: `ok=true`, `failure_count=0`.
- Estrutura comparada: 49 tabelas, 59 migrations e 671 entradas de schema, sem diferenças de tabelas, contagens, schema, dados críticos, Magalu ou migrations.
- Limite conhecido: alterações realizadas no Hub depois do snapshot podem não existir na VPS. A origem Neon não permitiu um dump final por exceder a quota.

## Estado aplicado

| Item | Estado |
| --- | --- |
| URL pública do Hub | `https://hub.dachbyte.tech` |
| Runtime | `hub-web`, saudável |
| Banco | `dachbyte_hub`, PostgreSQL privado da VPS |
| Role runtime | `dachbyte_hub_app` |
| Role migration | `dachbyte_hub_migrator` |
| Portal administrativo | `https://hub.dachbyte.tech/ops-portal/login` |
| Gateway e módulos | `HUB_BASE_URL=https://hub.dachbyte.tech` |
| Scheduler novo | ativo e estável desde 01/10/2026 |
| Webhook Asaas | ativo em `https://hub.dachbyte.tech/v1/public/webhooks/payment?provider=asaas` |

Todos os 22 containers da stack ficaram saudáveis após a troca. Smoke público confirmou `200` para `/healthz`, `/login`, `/selecao-plataforma` e portal do Hub.

## Variáveis e arquivos envolvidos

Não registrar valores dos secrets.

| Arquivo | Variáveis principais |
| --- | --- |
| `infra/env/hub.env` | `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`, modos de login/autorização |
| `infra/env/hub-runtime.env` | `HUB_DATABASE_URL`, `HUB_PUBLIC_BASE_URL`, `HUB_INTERNAL_TOKEN`, `ADMIN_*`, `ASAAS_*`, `BREVO_*`, `RENEWAL_INTENT_SECRET` |
| `infra/env/hub-migrate.env` | `HUB_DATABASE_URL` da role `dachbyte_hub_migrator`, `MIGRATION_BACKUP_CONFIRMED` |
| `infra/env/postgres.env` | `DACHBYTE_HUB_DB`, `DACHBYTE_HUB_APP_ROLE`, `DACHBYTE_HUB_MIGRATION_ROLE`, `DACHBYTE_HUB_APP_DB_PASSWORD`, `DACHBYTE_HUB_MIGRATION_DB_PASSWORD` |
| `infra/env/hub-cutover.env` | `SOURCE_DATABASE_URL`, destino público, controles de cutover e Asaas |

As roles do runtime e da migration passaram a usar credenciais distintas. Antes do corte, as duas URLs ainda reutilizavam a role administrativa do PostgreSQL; isso foi corrigido na VPS.

## Validação funcional

- O endpoint interno `/v1/internal/auth/verify` autorizou o master do Core somente para `volt_core`.
- O login centralizado em `https://dachbyte.tech/api/auth/login` retornou `200`, assinatura ativa e somente `volt_core` para o master do Core.
- Foi corrigida no Gateway a ausência de `volt_core` em `HUB_AUTH_MODULE_CANDIDATES`.
- Os demais masters restaurados mantêm os escopos `ml`, `shopee` e `tracking` no banco do Hub.

## Código publicado

- `7f1fc713` — corrige `\\gexec` no provisionamento de permissões do Hub.
- `1a2a9840` — permite login centralizado do master `volt_core`.
- `ced525c8` — corrige a sessão da seleção Seller para masters internos da plataforma.
- `b654b52e` — oculta cards Seller sem autorização confirmada pelo Hub.
- Hub `b83c1ad` — mantém o processo `hub-scheduler` vivo no runtime Node.

## Fechamento do Asaas — 01/10/2026

- `ASAAS_API_KEY`, `ASAAS_API_BASE_URL` e `ASAAS_WEBHOOK_TOKEN` estão preenchidas em `infra/env/hub-runtime.env` e carregadas pelo container `hub-web`; os valores não devem ser registrados.
- A credencial foi validada contra a API de produção do Asaas com resposta HTTP `200`.
- O webhook deixou de apontar para `paymentcontrol.davantti-suite.workers.dev` e passou a apontar para `https://hub.dachbyte.tech/v1/public/webhooks/payment?provider=asaas`.
- O webhook está habilitado, não interrompido e usa entrega sequencial.
- Eventos assinados e suportados pelo código: `PAYMENT_CREATED`, `PAYMENT_RECEIVED`, `PAYMENT_CONFIRMED`, `PAYMENT_OVERDUE`, `PAYMENT_REFUNDED` e `PAYMENT_DELETED`.
- A rota pública foi validada com o token real e payload propositalmente inválido; o retorno `400 invalid_json` confirmou domínio, rota e autenticação sem gerar evento financeiro.
- O `hub-scheduler` reiniciava porque `src/runtime/nodeScheduler.ts` aplicava `timer.unref()`. A chamada foi removida no commit `b83c1ad`; o container foi recriado e permaneceu `running`, com `restartCount=0` na validação.
- Estado anterior do webhook salvo em `infra/.cutover/asaas-webhook-before.json`; estado posterior salvo em `infra/.cutover/asaas-webhook-after-20261001.json`.

## Backups

- Dumps de rehearsal permanecem em `/opt/dachbyte/rehearsal/dachbyte/infra/backups/hub-rehearsal/`.
- Foi criado e validado um dump pós-cutover em `/opt/dachbyte/repository/infra/backups/hub-cutover/dach-hub-vps-post-cutover-20260927T175149Z.dump`, acompanhado de SHA-256.
- Os arquivos reais de ambiente possuem cópias anteriores à rotação das roles. Não copiar seus valores para Git ou para este vault.

## Pendências obrigatórias

1. Desbloquear temporariamente o Neon ou obter um backup posterior a 25/09 e comparar registros criados após o snapshot.
2. Decidir como reconciliar qualquer diferença antes de descartar definitivamente a origem Neon.
3. Executar um checkout Asaas controlado e validar `checkout → pagamento → webhook → banco VPS → recurso → acesso`.
4. Confirmar no banco a primeira linha real em `payment_provider_webhook_events` e validar idempotência por reenvio.
5. Testar login real dos masters de Shopee e Rastreio e abertura de seus respectivos módulos; ML foi validado com acesso somente a `ml`.
6. Testar recuperação de senha e envio de e-mail pelo Brevo no runtime VPS.
7. Configurar backup externo criptografado e executar um teste de restauração do `dachbyte_hub`.
8. Criar alertas para falhas do webhook e reinicializações do `hub-scheduler`.

## Regra de rollback

Os consumidores já foram trocados e o Hub VPS pode receber novas autenticações/escritas. Não apontar automaticamente de volta para Worker/Neon. Qualquer retorno exige freeze e reconciliação dos dados escritos na VPS.

## Relações

- [[Identidade e acesso]]
- [[VPS e staging]]
- [[Inventário de variáveis — Hub e staging]]
- [[Migração para VPS]]
