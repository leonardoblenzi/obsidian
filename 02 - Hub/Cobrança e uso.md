---
type: arquitetura
area: hub
updated: 2026-10-01
---

# Cobrança e uso

## Evidências no repositório

- Relato de consumo: módulos `hubUsageReporter` em produtos Seller.
- Cobrança de recursos: `hubResourceBillingService` em Shopee e Tracking.
- Créditos de Mercado Livre: `apps/seller-ml/services/hubCreditsService.js`.

## Estado operacional

- O Hub transacional e seu banco estão na VPS: `https://hub.dachbyte.tech` e `dachbyte_hub`.
- Provider ativo: Asaas de produção, configurado por `PAYMENT_PROVIDER`, `ASAAS_API_KEY`, `ASAAS_API_BASE_URL` e `ASAAS_WEBHOOK_TOKEN` em `infra/env/hub-runtime.env`.
- O webhook público é `https://hub.dachbyte.tech/v1/public/webhooks/payment?provider=asaas`.
- O acesso pago somente deve ser liberado após evento financeiro confirmado pelo webhook; o retorno do navegador não é autoridade de pagamento.
- Eventos tratados: criação, recebimento/confirmação, vencimento, estorno e exclusão.
- `hub-scheduler` executa renovação PIX conforme `HUB_PIX_RENEWAL_CRON` e manutenção conforme `HUB_MAINTENANCE_CRON`, em UTC.
- O histórico idempotente é armazenado em `payment_provider_webhook_events`.
- Em 01/10/2026, rota, autenticação, API e scheduler foram validados; o primeiro checkout/pagamento real após o cutover ainda está pendente.

## Relações

- [[Hub]]
- [[Davantti e Dachbyte]]

## A validar

- Checkout real controlado, ativação do recurso e convite/acesso do usuário.
- Reenvio idempotente e comportamento de vencimento/estorno.
- Alertas operacionais e reconciliação periódica com o Asaas.
