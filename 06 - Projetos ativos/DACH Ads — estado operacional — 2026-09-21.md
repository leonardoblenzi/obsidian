---
type: projeto
produto: dach_ads
status: em-validacao-autenticada
atualizado: 2026-09-21
---

# DACH Ads — estado operacional — 2026-09-21

## Papel do produto

DACH Ads é um produto DACH separado da linha Seller. Ele consolida leitura de Google Ads e Meta Ads, mantém seus tokens no backend próprio e consulta o Hub para confirmar se a identidade pode usar `dach_ads`. Ele não deve aparecer na seleção `/selecao-plataforma` do Seller.

## Limites de acesso

| Situação | Resposta esperada |
| --- | --- |
| Pessoa sem identidade Ads | acesso ao HTML protegido redireciona para `/ads/login`. |
| Sessão sem `dach_ads` | API protegida devolve `403` com negação de acesso. |
| Sessão com `dach_ads` | produto pode carregar a empresa e iniciar conexões OAuth autorizadas. |
| Token de provedor | fica no backend Ads, cifrado; não é devolvido para o navegador ou armazenado no Hub. |

## Serviços e deploy

| Serviço | Responsabilidade |
| --- | --- |
| `ads-api` | aplicação HTTP, login e callbacks OAuth. |
| `ads-worker` | sincronização assíncrona de dados de provedores. |
| `ads-migrate` | operação manual, perfil `ops`; não deve iniciar em deploy comum. |

Para alteração do Ads, reconstrua `ads-api` e `ads-worker`. Consulte o [[Runbook de deploy DACH na VPS]].

## OAuth e configuração

O usuário final conecta a própria conta por OAuth. O DACH fornece os identificadores do cliente OAuth e recebe callbacks no backend; o usuário não fornece segredo de aplicação.

Variáveis de configuração ficam fora do Git, na VPS. Registre no cofre de segredos o proprietário, a data de rotação e o ambiente de cada credencial, mas não copie valores nesta nota.

### Google Ads

Pontos a validar em sessão de QA:

1. Usuário com `dach_ads` abre o produto.
2. Ação de conectar Google inicia OAuth e retorna ao callback público correto.
3. A aplicação lista as contas permitidas para o usuário selecionar.
4. O worker sincroniza uma janela de dados sem expor refresh token no browser.
5. A revogação de `dach_ads` bloqueia API e telas protegidas.

### Meta Ads

Antes de habilitar usuários, validar:

1. App Meta, redirect URI e escopos de leitura aprovados no provedor.
2. Callback processado pelo `ads-api`.
3. Business Portfolio e contas de anúncio são listados somente para a identidade autenticada.
4. Worker trata token expirado e exige reconexão sem degradar outros tenants.

## Testes de código confirmados

O conjunto raiz do DACH passou com `161/161` em 21/09/2026. Os testes cobrem separação do produto, RLS, entrada Ads, negação por Hub, redirecionamento de anônimo e ausência do cartão Ads na seleção Seller.

## Pendências

- [ ] Rodar OAuth Google completo com empresa e usuário de QA.
- [ ] Concluir configuração e teste OAuth Meta.
- [ ] Confirmar execução, observabilidade e retentativa do `ads-worker` em produção.
- [ ] Definir política comercial para qualquer estado de acesso pago antes de liberar cobrança associada ao Ads.
- [ ] Especificar a próxima etapa MCP/Plugin somente depois da validação de dados e isolamento de tenant.

## Relações

- [[Mapa de rotas e autenticação DACH]]
- [[Hub]]
- [[Atualização de produção, seleção Seller e DACH Ads — 2026-09-21]]
