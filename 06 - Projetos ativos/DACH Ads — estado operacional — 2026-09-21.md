---
type: projeto
produto: dach_ads
status: meta-configurada-em-validacao
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

O aplicativo **Dach Ads** foi criado no Meta for Developers e a configuração de produção foi ligada ao backend Ads. O Hub continua responsável somente por identidade e concessão de `dach_ads`; tokens Meta permanecem no banco e backend próprios do Ads.

Variáveis usadas pelo fluxo:

| Variável | Uso | Estado em produção |
| --- | --- | --- |
| `META_APP_ID` | Identificador público do aplicativo Meta. | configurada; valor não documentado no vault. |
| `META_APP_SECRET` | Segredo usado pelo backend na troca do código OAuth. | configurada; valor nunca deve ser copiado para Git ou vault. |
| `META_REDIRECT_URI` | Callback exato usado na autorização e na troca do código. | `https://dachbyte.tech/ads/api/meta/oauth/callback`. |
| `META_GRAPH_API_VERSION` | Versão da Graph API usada pelo adaptador. | `v26.0`. |
| `META_OAUTH_SCOPES` | Permissões solicitadas ao usuário. | `ads_read,business_management`. |
| `ADS_TOKEN_ENCRYPTION_KEY` | Cifra os tokens dos provedores no backend Ads. | obrigatória; valor não documentado no vault. |

URLs públicas registradas para o aplicativo Meta:

- política de privacidade: `https://dachbyte.tech/ads/privacidade`;
- termos de serviço: `https://dachbyte.tech/ads/termos`;
- instruções de exclusão: `https://dachbyte.tech/ads/exclusao-de-dados`;
- callback OAuth: `https://dachbyte.tech/ads/api/meta/oauth/callback`.

As três páginas legais retornaram HTTP `200` após o deploy. A exclusão usa a opção **URL de instruções de exclusão de dados**. O usuário pode revogar a integração dentro do Ads e solicitar a exclusão pelo canal público; nunca deve enviar senha ou token.

Antes de habilitar clientes externos, validar:

1. OAuth completo com uma conta Meta administradora/testadora enquanto o app estiver em desenvolvimento.
2. Callback processado pelo `ads-api` e retorno correto para `/ads/app/meta`.
3. Business Portfolios e contas de anúncio listados somente para a identidade autenticada.
4. Seleção e sincronização read-only de uma conta real de QA.
5. Worker tratando token expirado e exigindo reconexão sem degradar outros tenants.
6. Antes da abertura a clientes: vínculo ao portfólio empresarial DACHBYTE, acesso avançado e App Review das permissões exigidas.

Não criar token manual nem conectar o portfólio da empresa cliente ao aplicativo. Cada cliente autoriza suas próprias contas pelo OAuth.

## Entrada, páginas públicas e identidade visual

O Ads possui jornada independente do Seller:

| Rota | Papel |
| --- | --- |
| `/ads` | landing pública. |
| `/ads/login` | login próprio do Ads, validado pelo Hub. |
| `/ads/access-denied` | empresa autenticada sem concessão `dach_ads`. |
| `/ads/app` | shell protegido do produto. |

Após definição de senha no Hub, usuários com apenas `dach_ads` são enviados para `/go/ads`, em vez da seleção Seller legada. O Ads não deve reaparecer em `/selecao-plataforma`.

O ícone Ads está em `apps/ads/public/ads-mark.png`. Ele possui transparência alfa real e é usado:

- como favicon em todas as páginas do Ads;
- na marca da navbar do shell autenticado;
- não aparece como ilustração na landing, login ou páginas legais.

O query versionado atual do asset é `v=20260922.1`, usado para invalidar o favicon antigo em cache.

## Estado publicado

| Item | Estado confirmado |
| --- | --- |
| Repositório | `git@github.com:leonardoblenzi/dachbyte.git`, branch `main`. |
| Commit atual documentado | `7af3e2f3` (`Bust cached DACH Ads icon`). |
| Serviço publicado | `ads-api`, saudável após a reconstrução. |
| Domínio usado pelo produto | `https://dachbyte.tech`. |
| Compose da VPS | ainda usa o project name técnico `dachbyte-staging`, apesar de atender o domínio público atual. |
| Testes Ads | `27/27` aprovados após páginas legais e ajustes de identidade visual. |

## Testes de código confirmados

O conjunto raiz do DACH passou com `161/161` em 21/09/2026. Os testes cobrem separação do produto, RLS, entrada Ads, negação por Hub, redirecionamento de anônimo e ausência do cartão Ads na seleção Seller.

## Pendências

- [ ] Rodar OAuth Google completo com empresa e usuário de QA.
- [x] Criar aplicativo Meta, configurar variáveis e cadastrar URLs públicas de produção.
- [ ] Rodar OAuth Meta completo com uma conta administradora/testadora e empresa de QA.
- [ ] Confirmar callback, descoberta, seleção e sincronização read-only de uma conta Meta Ads.
- [ ] Preparar App Review/acesso avançado e portfólio DACHBYTE antes de abrir o OAuth Meta para clientes externos.
- [ ] Confirmar execução, observabilidade e retentativa do `ads-worker` em produção.
- [ ] Definir política comercial para qualquer estado de acesso pago antes de liberar cobrança associada ao Ads.
- [ ] Especificar a próxima etapa MCP/Plugin somente depois da validação de dados e isolamento de tenant.

## Relações

- [[Mapa de rotas e autenticação DACH]]
- [[Hub]]
- [[Atualização de produção, seleção Seller e DACH Ads — 2026-09-21]]
- [[DACH Ads — Meta, páginas legais e identidade visual — 2026-09-21]]
