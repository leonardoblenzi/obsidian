---
type: arquitetura
status: transicao
---

# Marca e compatibilidade

`platform/branding/` concentra o vocabulário público e `platform/compatibility/` mantém contratos de rotas, cookies e callbacks legados.

## Regras

- A marca pública segue Dachbyte.
- Contratos técnicos existentes permanecem até haver telemetria, validação e rollback documentado.
- Um novo caminho ou serviço deve preservar um adaptador ou alias para o contrato anterior.
- Desde 14/09/2026, `/business/price` é a landing pública DACHBYTE Price. `/volt-price` é mantida somente como rota técnica legada; o alias `/dach/business/price` continua apontando para ela.

## Relações

- [[Marca pública Dachbyte]]
- [[Compatibilidade durante a migração]]
- [[Refatoração Davantti para Dachbyte]]
- [[Landings Seller e Business — 2026-09-14]]
