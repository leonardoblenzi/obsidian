---
type: projeto
status: em-andamento
---

# Refatoração Davantti para Dachbyte

## Objetivo

Consolidar os produtos públicos sob a marca Dachbyte sem romper logins, integrações, URLs, dados ou rollback.

## Estado

Em andamento. O catálogo público aprovado inclui Dach Seller, Dach Core, Dach Stock, Dach Chat, Dach Price e Dach Hub.

## Sequência

1. Marca visível e catálogo público.
2. Rotas canônicas mantendo aliases legados.
3. Configuração aditiva com fallback legado.
4. Domínios, OAuth e webhooks somente após validação em staging.

## Regras

Cookies, variáveis, bancos, callbacks e identificadores técnicos legados permanecem compatíveis durante a transição.

## Referências

- `docs/superpowers/specs/2026-09-11-dachbyte-brand-migration-design.md`.
- `docs/architecture/migration-dachbyte.md`.
- [[Marca pública Dachbyte]].
- [[Compatibilidade durante a migração]].
