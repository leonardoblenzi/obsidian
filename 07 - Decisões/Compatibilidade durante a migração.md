---
type: decisao
status: vigente
---

# Compatibilidade durante a migração

## Contexto

Relocar entrypoints, rotas ou nomes sem uma ponte pode interromper deploys, OAuth, sessões e consumidores existentes.

## Decisão

Cada realocação deve manter adaptador ou alias compatível.

## Impacto

As mudanças de estrutura são independentes de alterações de schema, domínio, OAuth e permissões; remoções exigem telemetria e rollback documentado.

## Evidências

- `docs/architecture/migration-dachbyte.md`.
- `platform/compatibility/legacySurfaceRegistry.js`.
- [[Marca e compatibilidade]].
