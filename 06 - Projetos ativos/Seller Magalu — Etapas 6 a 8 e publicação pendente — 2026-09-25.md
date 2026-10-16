---
type: projeto
status: publicado-validacao-funcional
atualizado: 2026-09-25
repositorio: dachbyte
branch: main
commit_publicado: e759b25
---

# Seller Magalu — Etapas 6 a 8 e publicação pendente — 2026-09-25

## Estado atual

- As Etapas 6, 7 e 8 foram integradas, testadas e publicadas na `main` do repositório `dachbyte`.
- Commit remoto: `e759b25` — `Expand Magalu operations and delivery workflows`.
- A suíte do módulo passou em **158/158** testes e `git diff --check` não apontou erro.
- Publicação realizada na VPS em 25/09/2026. Checkout atualizado para `e759b25`; migrations e containers foram validados.
- As flags de escrita nova continuam desligadas por segurança:
  - `MAGALU_INVOICE_WRITE_ENABLED=false`
  - `MAGALU_DELIVERY_WRITE_ENABLED=false`

## Entregas incorporadas

### Etapa 6 — Master e saúde de integrações

- Central Master para PostgreSQL, Redis, Hub, heartbeat do worker, OAuth por conta, scopes, recursos Hub, syncs e webhooks.
- Reconexão OAuth de Master vinculada à conta selecionada: o callback exige o mesmo tenant Magalu esperado.
- Migration `008_master_integration_health.sql` amplia `oauth_states` com o contexto seguro da reconexão.
- Reconciliação de Hub e webhooks é somente de leitura/metadados; não cria nem rotaciona secrets remotos.

### Etapa 7 — Pedidos e entregas somente leitura

- Tela `/magalu/pedidos`, filtros, KPIs, detalhe e reconciliação individual.
- Espelho local em `magalu.orders`, `magalu.order_items`, `magalu.deliveries` e `magalu.delivery_items`.
- Fila `magalu-orders-sync`; jobs de sincronização completa e reconciliação individual.
- Scopes necessários: `open:order-order-seller:read` e `open:order-delivery-seller:read`.
- Dados pessoais do comprador são minimizados; não se persiste CPF/CNPJ, telefone, e-mail ou endereço completo.

### Etapa 8 — Expedição protegida

- Fluxos protegidos para NF-e e finalização de entrega, exclusivamente por preview, confirmação explícita, fila, checagem remota e revalidação `WRITE magalu` no Hub.
- Endpoints remotos limitados a NF-e e finalização da entrega. Não foram incluídas ações de frete, etiquetas, cancelamento ou alteração genérica de status.
- XML de NF-e fica cifrado com `MAGALU_ENCRYPTION_KEY`; nunca vai ao frontend e é removido do preview/operação ao avançar o fluxo.
- Fila `magalu-delivery-write` e migration `010_delivery_write_operations.sql`.
- Em ambiguidade depois de `dispatching`, o sistema apenas reconcilia via GET: não repete automaticamente uma escrita remota.

## Migrations aplicadas na VPS

Aplicadas pela ordem natural do migrator, sem pular nenhuma:

1. `006_audit_v2_retention.sql`
2. `007_mass_sku_operations.sql`
3. `008_master_integration_health.sql`
4. `009_orders_read_model.sql`
5. `010_delivery_write_operations.sql`

## Publicação realizada e evidências

- VPS: `srv1971387`, acesso confirmado com `root@177.7.52.6` e a chave privada `C:\Users\USER\.ssh\codex-dachbyte-deploy-2026-09-22`.
- O IP anteriormente usado tinha um dígito incorreto (`177.57.52.6`); não houve troca de host key necessária após corrigir para `177.7.52.6`.
- Checkout atualizado por fast-forward para `e759b25`.
- Migrator confirmou `001`–`010` aplicadas, sem pendências.
- `seller-magalu-web` e `seller-magalu-worker` foram reconstruídos e recriados; ambos ficaram `healthy`.
- Smoke tests confirmados:
  - `https://dachbyte.tech/healthz` → HTTP 200.
  - `https://dachbyte.tech/magalu/health` → HTTP 200 (`seller-magalu`).
- Worker confirmou em log as filas novas `magalu-audit-maintenance`, `magalu-sku-update`, `magalu-orders-sync` e `magalu-delivery-write`.

## Próximos testes funcionais após deploy

- Sincronização de pedidos e entregas com conta de teste.
- Visibilidade de scopes ausentes e reconexão OAuth.
- Master > Integrações: probes e reconciliação não destrutiva.
- Conferir que NF-e/finalização permanecem indisponíveis enquanto as duas flags estiverem `false`.
- Antes de habilitar escrita: testar uma única NF-e e uma única finalização controlada, validar logs/auditoria e a reconciliação posterior.

## Cuidados de repositório

- Backups de aplicadores e diretórios `.codex`/`.superpowers` locais não fazem parte do commit do projeto.
- Não executar `git add .` no checkout local enquanto esses diretórios existirem.
