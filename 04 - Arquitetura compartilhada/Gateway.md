---
type: arquitetura
---

# Gateway

O gateway concentra login, sessão, portal e rotas canônicas. A implementação está em `apps/gateway/` e `platform/gateway/`.

## Relações

- [[Hub]]
- [[Mercado Livre]]
- [[Shopee]]
- [[Business]]
- [[Compatibilidade durante a migração]]

## Operação

Na arquitetura de VPS, o Caddy encaminha o tráfego para o gateway e os produtos na rede Docker privada. Consulte [[VPS e staging]].
