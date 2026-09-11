---
type: arquitetura
area: hub
---

# Identidade e acesso

## Evidências no repositório

- Sincronização de identidades: `lib/hubIdentitySync.js`.
- Rotas de autenticação da suíte: `routes/suiteAuthRoutes.js`.
- Controle de acesso por produto: módulos `hubAccessService` em `apps/seller-ml/`, `apps/seller-shopee/`, `apps/seller-madeira/`, `apps/seller-tracking/` e `apps/seller-leader/`.

## Relações

- [[Hub]]
- [[Gateway]]
- [[Compatibilidade durante a migração]]

## A mapear

- Fonte canônica de permissões por plano e tenant.
- Fluxo completo de provisionamento de uma empresa entre os produtos.
