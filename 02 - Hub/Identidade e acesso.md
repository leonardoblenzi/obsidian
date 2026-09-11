---
type: arquitetura
area: hub
---

# Identidade e acesso

## Evidências no repositório

- Sincronização de identidades: `lib/hubIdentitySync.js`.
- Rotas de autenticação da suíte: `routes/suiteAuthRoutes.js`.
- Controle de acesso por produto: módulos `hubAccessService` em `apps/seller-ml/`, `apps/seller-shopee/`, `apps/seller-madeira/`, `apps/seller-tracking/` e `apps/seller-leader/`.
- Provisionamento de empresa Volt Core: `POST /v1/internal/identity/company` em `hub pagamento/src/modules/identity/internalRoutes.ts`.
- Sincronização e convite de usuário de módulo: `POST /v1/internal/identity/sync` com `volt_core` como origem.

## Checkpoint Volt Core

- O Hub `paymentcontrol` é a fonte de verdade para tenant, empresa, produto instalado, usuário, vínculo e convite.
- Empresa criada pelo master do Core: Core chama o Hub primeiro, recebe o `tenant_id` canônico, instala `volt_core` e só então cria o espelho operacional local.
- Usuário criado pelo Core: exige uma empresa vinculada ao Hub, é sincronizado no Hub com acesso pendente a `volt_core` e recebe convite para criar senha.
- O provisionamento de empresa é idempotente: a mesma chave retorna o mesmo tenant e uma tentativa após falha retoma o tenant já reservado.
- A sincronização de usuário Volt Core exige que o produto esteja ativo no tenant; não há criação de usuário operacional local sem Hub.
- Migration `053_internal_identity_provisioning.sql` foi aplicada em 11/09/2026 na branch Neon `payment` usada pelo sandbox do Hub.
- Hub publicado pelo commit `cbc65bc`; DACH publicado em `dach` pelos commits `101ec5f3` e `1b812ca0`.
- Testes locais concluídos: Hub `58/58`, TypeScript aprovado; Volt Core `236/236` e build aprovado.
- Staging comprovou a criação Hub-first: empresa criada no Core apareceu no Hub com o mesmo tenant e produto `volt_core` ativo; usuário criado no Core gerou vínculo e convite pendentes no Hub.
- No master do Core, empresa é seleção obrigatória ao criar usuário. Isso evita um vínculo acidental com a primeira empresa da lista.

## Worker e configuração ativa

- Worker em uso: `paymentcontrol`, publicado em `https://paymentcontrol.davantti-suite.workers.dev`.
- Ambiente do Worker: `paymentcontrol`, com `ENVIRONMENT=production`, `PAYMENT_PROVIDER=asaas` e API Asaas produtiva.
- Portal administrativo: `/ops-portal`; URL pública do Hub: `https://paymentcontrol.davantti-suite.workers.dev`.
- O Worker não herda projetos e integrações do Hub legado: os identificadores externos de ML, Shopee, Rastreio, Madeira e Skuleader estão vazios nesse ambiente.
- Agendamentos ativos no Worker: a cada 5 minutos e diariamente às `09:17`.
- `HUB_ACCESS_ENFORCEMENT_MODE=monitor` no Worker registra e permite observar a transição das políticas. Já os containers DACH usam `HUB_LOGIN_MODE=strict`, `HUB_AUTH_MODE=strict` e `HUB_ENFORCEMENT=strict`, portanto tratam o Hub como autoridade de login e acesso.

### Secrets presentes no Worker

Os valores não são documentados aqui nem versionados. A lista de nomes foi conferida em 11/09/2026:

- Administração: `ADMIN_PASSWORD_HASH`, `ADMIN_PASSWORD_SALT`, `ADMIN_SESSION_SECRET`.
- Hub interno: `HUB_INTERNAL_TOKEN`.
- Banco: `NEON_DATABASE_URL`.
- Asaas: `ASAAS_API_KEY`, `ASAAS_WEBHOOK_TOKEN`, `RENEWAL_INTENT_SECRET`.
- E-mail: `BREVO_API_KEY`, `BREVO_SENDER_EMAIL`, `BREVO_SENDER_NAME`.

### Contrato do DACH com o Hub

- `HUB_BASE_URL=https://paymentcontrol.davantti-suite.workers.dev`.
- `HUB_INTERNAL_TOKEN` deve ter o mesmo valor configurado como secret no Worker, sem ser exibido em logs, Git ou Obsidian.
- O arquivo da VPS é `infra/env/hub.env`, fora do Git e com permissão `600`.

## Relações

- [[Hub]]
- [[Gateway]]
- [[Compatibilidade durante a migração]]

## Próxima validação

- Repetir a mesma criação com uma chave idempotente para provar o replay.
- Aceitar convite, definir senha global e autenticar no Core.
- Validar bloqueios: empresa inativa, produto `volt_core` revogado e usuário inativo.
- Validar que reativação restaura somente os acessos que continuam válidos.
- Fonte canônica de permissões comerciais detalhadas por plano/faixa.
