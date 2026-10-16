---
type: projeto
area: seller-ml
status: validacao-producao
updated: 2026-10-16
---

# Créditos ML — estado e validação (16/10/2026)

## Estado confirmado

- O Hub de produção roda em `https://hub.dachbyte.tech`, com banco `dachbyte_hub` na VPS. A `main` do Hub e o checkout da VPS estão no commit `26dcf21d4cd8b0b8ae4b4f3e259280163b583cbf`.
- A migration `062_ml_prepaid_credit_shadow.sql` criou a matriz publicada `price_matrix_sellers_2026_ml_prepaid`. A migration `063_ml_prepaid_credit_enforce_ready.sql` registrou a liberação controlada: em produção, as 26 operações ML ativas da matriz estão com `enforce_ready=true` (consulta ao banco: `26|26`). Não houve mudança de preços nessa migration.
- Em 16/10, a migration `063` constou como aplicada, sem migrations pendentes. `hub-web`, `seller-ml-web` e `seller-ml-worker` estavam `healthy` após o deploy. O deploy dessa etapa recriou apenas `hub-web`; o Seller ML já estava publicado.
- O Seller ML opera com `HUB_RESOURCE_CREDITS_MODE=enforce` na VPS. O Hub decide se cada operação pode debitar pela matriz publicada e pelo `consumption_model` do recurso.
- A separação entre acesso e consumo está implantada. `access_policy=unlimited` não torna os créditos ilimitados quando `consumption_model=prepaid_credits`. Contas com `consumption_model=unlimited` continuam com consumo ilimitado.
- No último levantamento operacional, havia seis recursos ML `prepaid_credits + acesso unlimited` e seis `unlimited + acesso unlimited`. A ativação da matriz não alterou políticas ou saldos dessas contas. Confirmar a contagem novamente antes de usar esse número em relatórios atuais.

## Caminho funcional

1. Uma operação em lote pede ao Hub um quote para sua `operation_key` e quantidade prevista.
2. A interface mostra anúncios/unidades, custo estimado e saldo da conta. Saldo insuficiente impede iniciar o job em conta pré-paga.
3. A confirmação inicia reserva e job; o worker processa os itens e informa sucessos, falhas e unidades faturáveis.
4. O settlement deve cobrar apenas unidades faturáveis e devolver a parcela reservada para itens sem cobrança. O painel de jobs deve exibir progresso, concluídos, erros, créditos cobrados/reembolsados e XLSX quando o fluxo prevê relatório.
5. Contas com consumo ilimitado não precisam de saldo nem devem sofrer débito, mesmo que passem pelo mesmo fluxo de confirmação.

Este é o contrato esperado. A verificação ponta a ponta de uma operação real depois da migration `063` ainda está pendente; não tratar a saúde dos containers ou `26|26` como prova de settlement correto.

## Evolução até aqui

- Hub: Pontos 1 e 2 separaram `access_policy` de `consumption_model`, atualizaram o Admin e impediram que políticas em lote reescrevessem o consumo. O checkout Asaas passou a bloquear recarga apenas para `consumption_model=unlimited`.
- Seller ML: Ponto 3 passou a usar o `consumption_model`/wallet/quote do Hub como autoridade. Metadados legados (`legacy_active`, `billing_mode`, `usage_policy`, `courtesy_unlimited`) não concedem créditos ilimitados.
- A interface mostra saldo em créditos visíveis para contas pré-pagas e “Tokens ilimitado” para consumo ilimitado. `credit_unit_scale=100` é precisão interna: 100 créditos visíveis correspondem a 10.000 unidades de saldo interno.
- O Atacado foi o primeiro fluxo testado com cobrança efetiva, modal de confirmação, job em backend, progresso e relatório XLSX. O bloqueio por saldo insuficiente foi observado em uma conta com 0 c.
- A confirmação de créditos e o painel de jobs foram estendidos às operações em lote acordadas, incluindo remoção de promoções; clonar anúncio também entrou no escopo de cobrança. A regra de cobrança da consulta de anúncios ficou adiada: a intenção é cobrar somente enriquecimento, sem mudança por enquanto.
- Houve um caso em `promotions.apply`: quote de 422 unidades e estimativa de 1 c, seguido de HTTP 409 `credit_operation_not_enforce_ready` ao iniciar o job. A liberação das 26 flags removeu essa trava de configuração; repetir a operação para comprovar o fluxo completo.

## Contas piloto e alcance

- MV Estofados foi a primeira conta configurada como acesso ilimitado com consumo pré-pago e saldo visível de 100 c. Isso demonstrou a separação de acesso e consumo e a escala 100:1.
- Drossi também foi configurada para consumo controlado por créditos e usada nos testes de Atacado e promoções. O saldo mostrado em capturas anteriores mudou durante os testes; consultar a wallet atual no Hub antes de registrar um número.
- As demais contas com `consumption_model=unlimited` mantêm o direito de consumo ilimitado. A ativação de `enforce_ready` não converte contas nem debita saldos por si só.

## Preço e compra de créditos

- A matriz atual permite iniciar a validação real, mas o preço ainda precisa de revisão. O quote de 422 anúncios por 1 c foi questionado como possivelmente barato. Não alterar a tabela até comparar custo de execução, uso real e referência comercial (inclusive Mercado Turbo) e aprovar uma regra por operação.
- Pacotes informados no Hub: 1.000 c por R$ 29, 5.000 c por R$ 99 e 15.000 c por R$ 199. No webhook, o Hub converte créditos visíveis em unidades internas usando `credit_unit_scale=100`; a interface deve exibir somente créditos visíveis.
- Compra Asaas para `unlimited + prepaid_credits` é permitida; `consumption_model=unlimited` bloqueia compra. Confirmar em transação real o checkout, webhook, crédito da wallet, idempotência e eventual estorno antes de considerar o fluxo financeiro encerrado.

## Próximo teste controlado

1. Na Drossi, selecionar poucos anúncios elegíveis em uma operação em lote, registrar saldo e quote antes da confirmação.
2. Confirmar uma única vez; verificar criação de **um** job e uma reserva com `operation_id` estável.
3. Conferir no job quantidade prevista, processada, aplicada, falhas e `billable_units`; baixar o XLSX quando disponível.
4. Conferir settlement no Hub: débito proporcional apenas aos itens faturáveis, liberação/reembolso do restante, saldo final e telemetria. Repetir a consulta para garantir idempotência, sem executar a operação novamente.
5. Testar saldo insuficiente e uma conta de consumo ilimitado sem converter outras contas.
6. Só depois revisar a tabela de preços e o texto do modal. Não inferir que 1 c para centenas de itens é o preço final desejado.

## Limites desta atualização

- Não houve mudança de preço, saldo, `access_policy` ou `consumption_model` no commit `26dcf21`.
- Não houve deploy novo do Seller ML nessa etapa.
- Não existe, nesta nota, prova de débito/settlement real após a ativação geral. A próxima evidência deve vir de um job controlado e da wallet/ledger correspondente.

## Relações

- [[Cobrança e uso]]
- [[Mercado Livre]]
- [[Estado operacional DACH — Hub, VPS e módulos — 2026-10-01]]
