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
