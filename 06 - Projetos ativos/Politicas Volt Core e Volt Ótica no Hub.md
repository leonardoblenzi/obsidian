---
type: projeto
status: em-andamento
area: hub
repository: hub-pagamento
branch: payment
updated: 2026-09-15
---

# Políticas Volt Core e Volt Ótica no Hub

## Decisão de produto

- **Volt Core** é a instalação Business base da empresa.
- **Volt Ótica** é uma vertical Business adicional, dependente de um Core operacional.
- Nenhum dos dois representa conta externa, OAuth ou `billing_resource` artificial.
- A política comercial pertence ao produto instalado em `tenant_products`; Seller continua sendo administrado por conta/recurso.

## Planos comerciais do Volt Core

| Plano | Mensalidade | Usuários ativos no Core |
| --- | ---: | ---: |
| Start | R$ 39,90 | até 2 |
| Pro | R$ 69,90 | até 5 |
| Max | R$ 109,90 | até 12 |
| Enterprise | sob consulta | personalizado |

Volt Ótica custa **R$ 39,90/mês** e herda o limite de usuários do plano Core. Ela não cria vagas adicionais. Clientes, produtos, serviços e vendas do Business não recebem limite por esta política.

## Implementado no Hub

- Branch `payment`, commit `a876956` — `Add Volt Core and Optical installation policies` — enviado para `origin/payment` em 15/09/2026.
- Nova migration `055_business_installation_policies_and_volt_core_catalog.sql`:
  - adiciona política, vigência e referência de plano/faixa em `tenant_products`;
  - cadastra `volt_optical` como `business_vertical`, sem `module_slug`;
  - cria a dependência `volt_optical -> volt_core` em `billing_product_dependencies`.
- API administrativa para alterar política de uma instalação Business.
- Regras de segurança:
  - Ótica exige Core instalado e operacional;
  - Core não pode ser desativado se existir dependente operacional;
  - plano Core não pode ser reduzido abaixo do número de usuários ativos com acesso ao Core;
  - permissões de usuário só usam produtos operacionalmente ativos.
- Hub administrativo:
  - cards Business exibem plano, mensalidade, usuários e vigência;
  - modal de política separado de recursos Seller;
  - abas da ficha de empresa usam subcabeçalho sticky;
  - sidebar mantém largura reservada, sem navegação horizontal no desktop.

## Validação local

- `npm run test:checkout`: 391/391.
- `npm run test:admin`: 80/80.
- TypeScript, validação sintática do admin, invariantes do fluxo de pagamentos e `git diff --check`: aprovados.

## Migration 055 — aplicada no piloto

Em 15/09/2026, a URL da branch Neon piloto correta foi confirmada. O plano mostrou `053` e `054` aplicadas e somente a `055` pendente. A execução foi liberada explicitamente por versão, com confirmação de backup, e concluiu como `applied 055_business_installation_policies_and_volt_core_catalog`.

Verificação posterior: 053, 054 e 055 constam como aplicadas no histórico da branch piloto.

Próximos passos:

1. **Deploy concluído em 15/09/2026:** Worker `paymentcontrol`, versão Cloudflare `35b31327-aa86-4ccd-8e28-6cdde6c50368`, a partir da branch `payment` e do commit `a876956`.
2. Homologação técnica concluída: `GET /health` retornou `200`; o login administrativo serve o build `dach-seller-hub-20260915-3`; as cinco colunas de política existem em `tenant_products`; `volt_optical` está ativo como `business_vertical`, sem `module_slug`; e a dependência para `volt_core` está registrada como `requires_active`.
3. Homologação funcional autenticada pendente: validar no Hub a matriz Business publicada, atribuir plano ao Core, tentar ativar a Ótica sem Core e reduzir o Core abaixo dos usuários ativos. Essa etapa requer sessão administrativa válida e não foi simulada com credenciais não fornecidas.
4. Registrar o resultado funcional de staging e só então considerar a promoção de ambiente.

## Referências

- [[Provisionamento Volt Core via Hub]]
- [[Hub]]
- [[Catálogo de produtos]]
- `C:\Users\USER\Documents\Projetos\hub pagamento\sql\055_business_installation_policies_and_volt_core_catalog.sql`
