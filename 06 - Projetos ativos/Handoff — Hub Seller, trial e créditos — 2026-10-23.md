---
type: handoff
area: hub-seller
status: integrado-pendente-validacao-ponta-a-ponta
updated: 2026-10-28
repository: hubPagamentos
branch: main
commit_preparado: b1dc24d
commit_dach_integrado: 7d84790
---

# Handoff — Hub Seller, trial e créditos — 23/10/2026

## Leitura rápida

- Atualização de 28/10: Dach integrado e enviado para `origin/main` em `7d84790`, incluindo o commit funcional `5c3778c` e os commits anteriores de trial/pós-pago. Nenhum deploy ou migration foi executado nessa integração.
- O novo estado está enviado para `origin/main` do Hub no commit `b1dc24d` (`feat: prepare seller trial billing and credit pricing`).
- **Este commit não foi deployado, nenhuma migration foi aplicada e nenhum saldo, contrato ou recurso foi alterado nesta etapa.**
- O último baseline de produção documentado para créditos ML é a nota [[Créditos ML — estado e validação — 2026-10-16]]: Hub em `26dcf21d`, matriz `price_matrix_sellers_2026_ml_prepaid`, migration 063 aplicada e 26 operações ML com `enforce_ready=true`.
- Antes de qualquer deploy, confirmar no checkout da VPS o SHA realmente ativo. A nota de 16/10 é referência documentada, não substitui essa verificação.

## Implementado, testado e publicado — posição em 28/10

| Componente | Implementado e enviado ao GitHub | Evidência de testes | Publicação em produção |
| --- | --- | --- | --- |
| Hub | `main`, `b1dc24d`: catálogo, medição como autoridade, portal e faturas | TypeScript; checkout 766/766; testes focados 29/29; migrations 29/29, executados na preparação anterior | Deploy desse conjunto não realizado neste chat; SHA ativo na VPS precisa ser consultado |
| Dach | `main`, `7d84790`: integração do conjunto Seller, incluindo `5c3778c` | Após merge: Seller 50/50 e arquitetura 29/29 | Não deployado nesta etapa |
| Catálogo/migration 065 | SQL versionado no Hub | Testes locais de contrato da migration | Aplicação pendente conforme último registro; banco não consultado em 28/10 |
| Jornada pós-paga completa | Serviços e interfaces implementados | Testes locais de partes do fluxo | Ensaio integrado com Asaas, corte por vencimento e desbloqueio ainda pendente |

Push para `main` não significa deploy. Não tratar a nota histórica de produção de 16/10 como verificação atual da VPS. Nesta atualização, não houve consulta de containers, banco, flags ou revisões em execução.

### Integração do Dach

O merge `7d84790` incorporou `origin/main` sem conflitos e sem alteração do conteúdo em relação ao primeiro pai local. As duas correções da calculadora existiam nos dois históricos como patches equivalentes com hashes diferentes. A interpretação anterior de que houve uma reversão remota de aproximadamente seis mil linhas estava incorreta: aquela diferença representava código local ainda não enviado.

O conjunto integrado inclui OAuth e retorno à Seleção de plataforma, medição do último mês fechado em ML/Shopee, entrega da medição ao Hub, proposta com preço definido pelo servidor, edição de nome da conta, avisos de preço, desativação de assinatura e listagem de faturas. Backups, outputs, scripts auxiliares e rascunhos não rastreados ficaram preservados fora do commit.

Os 50 testes específicos cobrem OAuth/relink, medição, destino pós-OAuth, contratação sem campos de preço editáveis e identidade confiável no gateway. Os 29 testes de arquitetura passaram com loopback local permitido. Esses testes não substituem uma jornada real de pagamento nem uma validação visual em navegador.

## O que entrou em `b1dc24d`

### Precificação fixa de ML e Shopee

A nova migration **não aplicada** é `sql/065_seller_fixed_tiers_and_ml_topups.sql`.

Ela cria/publica a matriz `price_matrix_sellers_2026_fixed_tiers`, preservando pedidos, carteiras e snapshots financeiros existentes. Para ML e Shopee, as faixas mensais passam a ser:

| Pedidos fechados/mês | Valor mensal |
| --- | ---: |
| 0–30 | R$ 17,90 |
| 31–200 | R$ 32,90 |
| 201–700 | R$ 44,90 |
| 701–1.500 | R$ 74,90 |
| 1.501–3.000 | R$ 109,90 |

- Não existe mais adicional por bloco nessas faixas.
- Acima de 3.000 pedidos deve cair em contato comercial; não publicar uma faixa adicional sem decisão comercial.
- `above_1500` é mantida como compatibilidade/histórico e a UI deve sinalizar que o plano antigo precisa de revisão, em vez de inventar uma tarifa.

### Crédito ML recalibrado — ainda pendente de migration

Os pacotes previstos são exibidos em créditos visíveis; a precisão interna continua `credit_unit_scale=100`.

| Pacote | Preço | Saldo interno gravado |
| --- | ---: | ---: |
| 500 c | R$ 29,00 | 50.000 pontos |
| 2.000 c | R$ 99,00 | 200.000 pontos |
| 5.000 c | R$ 199,00 | 500.000 pontos |

Tarifas aprovadas:

- Escritas massivas (promoções, atacado, características, modelo em massa, prazo, dimensões, estoque, ativar/pausar/encerrar anúncio): **1 c por 400 anúncios**.
- Operações destrutivas e relistagem: **1 c por 250 anúncios**.
- Validações/consultas técnicas: **1 c por 10.000 anúncios**.
- Filtro de anúncios: **1 c por 30.000 anúncios**.
- Sincronização de catálogo financeiro: **1 c por 15.000 unidades**.
- Clone mantém a própria regra já existente; não foi recalibrado neste commit.

Exemplo de referência comercial: uma aplicação em 10.000 anúncios cobra 25 c. Vinte aplicações desse porte por semana consomem 500 c.

As novas linhas afetam somente a matriz publicada depois da migration. Recargas já pagas usam seus snapshots; wallets e ledger não são atualizados pela migration.

### Seleção de plataforma e contratação Seller

O Hub agora é autoridade para ML e Shopee na contratação baseada em medição, sem o usuário declarar manualmente pedidos ou escolher a própria faixa:

1. A conta é vinculada por OAuth.
2. O Hub/integração mede pedidos do último mês fechado.
3. Com medição atual e válida, o Hub determina plano e faixa automaticamente.
4. Para dados ausentes, incompletos, antigos ou acima de 3.000 pedidos, o checkout falha fechado com o motivo apropriado; não há preço inventado.
5. Alteração manual de plano para ML/Shopee fica bloqueada. A faixa deve ser reapurada mensalmente a partir da medição.

Também entrou:

- Renomeação auditável da conta vinculada: `POST /v1/internal/portal/accounts/label`.
- Recursos inativos/encerrados no pós-pago podem voltar ao fluxo de ativação adequado, preservando OAuth; não é necessário pedir nova conexão apenas porque a assinatura foi desativada.
- O portal expõe estado e período da medição (`succeeded`, `stale`, `pending` ou erro) para a UI explicar “Validando faixa…”.
- A tela de seleção é o ponto de entrada de contratação, renovação, desativação e reativação; módulos pagos não devem liberar acesso só pela volta do OAuth.

### Trial e pós-pago

- O modelo de trial continua por empresa e módulo, com operações ilimitadas durante os 7 dias, conforme definido. Não há franquia de créditos no trial.
- Foi preparado um endpoint interno de faturas pós-pagas: `GET /v1/internal/postpaid/invoices`.
- Esse endpoint e os fluxos pós-pagos continuam protegidos por `HUB_SELLER_POSTPAID_ENABLED=true`. Enquanto a flag estiver ausente/falsa, nada é exposto/ativado pelo runtime.
- A cobrança pós-paga, runner e medição automática continuam fora de produção até haver validação operacional específica. Não alterar flags ou env sem uma etapa explícita.

## Estado de produção que permanece válido até deploy

- `consumption_model=prepaid_credits` cobra operações reais quando o recurso é controlado; `access_policy=unlimited` sozinho não dá consumo ilimitado.
- `consumption_model=unlimited` mantém “Tokens ilimitado” e bloqueia compra de recarga.
- Drossi é a conta piloto controlada por créditos; as demais contas ainda devem conservar sua política atual até ajuste explícito.
- O fluxo do Asaas já trabalha com créditos visíveis e escala interna: o webhook converte `visibleCredits × 100`. A compra em `unlimited + prepaid_credits` é permitida; consumo ilimitado é bloqueado.
- A prova ponta a ponta obrigatória continua sendo: quote → reserva → job → `billable_units` → settlement → ledger/wallet → XLSX/telemetria. Não confundir job concluído ou container healthy com prova de settlement.

## Verificações executadas antes do commit

- `npx tsc --noEmit --pretty false`: passou.
- Testes focados de pacotes, escala e operação: 29/29 passaram.
- `npm run test:migrations`: 29/29 passaram.
- `npm run test:checkout`: 766/766 passaram.
- `git diff --check`: sem erro de whitespace.

## Próxima sequência segura

Antes de publicar, concluir o ensaio integrado em ambiente de teste: duas contas na mesma empresa → medição → proposta → aceite → acesso → proporcional e fatura consolidada → vencimento sem pagamento → módulos bloqueados e portal acessível → webhook pago → desbloqueio. Cobrir webhook duplicado, retry e reativação sem novo trial. Conferir o catálogo de créditos separadamente da mensalidade pós-paga.

A sequência de produção abaixo depende dessa validação e de autorização de deploy; a instrução vigente em 28/10 foi integrar a main **sem deploy**. Incluir o Dach `7d84790` no planejamento de publicação junto ao Hub e levantar todas as migrations pendentes de ambos os repositórios, não apenas a 065.

1. Na VPS, verificar SHA do checkout do Hub, saúde de `hub-web` e migrations pendentes antes de agir.
2. Fazer deploy do commit `b1dc24d` **sem habilitar flags pós-pagas**.
3. Avaliar e então aplicar a migration 065 de forma explícita. Ela publica uma nova matriz e muda os preços futuros; não deve ser tratada como alteração cosmética.
4. Após 065, validar no Hub o catálogo publicado, os três pacotes, as tarifas por operação e a integridade de carteiras/snapshots históricos.
5. No Seller, conferir visual da Seleção de plataforma: conta recém-OAuth deve aparecer como “Validando faixa…”, seguida da faixa apurada e do caminho de pagamento. Não permitir seleção manual de faixa em ML/Shopee.
6. Fazer uma recarga Asaas controlada e validar webhook, idempotência, saldo visível e estorno.
7. Fazer um job pequeno controlado na Drossi e confirmar reserva, `billable_units`, settlement proporcional, reembolso e relatório XLSX.
8. Só depois considerar habilitar pós-pago/runner/medição automática, com flags documentadas e rollback definido.

## Arquivos locais preservados fora do commit

- `dist/` gerado no checkout do Hub.
- `docs/superpowers/plans/2026-09-24-migracao-hub-para-vps.md`.
- `docs/superpowers/plans/2026-09-28-business-catalog-activation.md`.

Esses arquivos não entraram em `b1dc24d`; revisar separadamente antes de qualquer commit futuro.

## Relações

- [[Créditos ML — estado e validação — 2026-10-16]]
- [[Cobrança e uso]]
- [[Hub]]
- [[Seller e operações]]
