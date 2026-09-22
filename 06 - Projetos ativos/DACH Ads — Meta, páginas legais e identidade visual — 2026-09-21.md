---
type: registro-operacional
produto: dach_ads
status: api-publicada-worker-pendente
data: 2026-09-21
repositorio: dachbyte
branch: main
commit: 7af3e2f3
---

# DACH Ads — Meta, páginas legais e identidade visual — 2026-09-21

## Resultado

O backend do DACH Ads recebeu a configuração do aplicativo Meta em produção. As páginas públicas exigidas pelo provedor foram criadas, o callback foi fixado no domínio `dachbyte.tech` e a identidade visual do Ads foi incorporada ao favicon e ao shell autenticado.

Esta entrega configura a infraestrutura do OAuth; o teste ponta a ponta com uma conta Meta real continua pendente.

> [!warning] Pendência operacional encontrada na auditoria
> O `ads-api` possui as variáveis Meta em runtime, mas o container atual do `ads-worker` não possui `META_APP_ID`, `META_APP_SECRET` nem `META_REDIRECT_URI`. O worker precisa ser recriado antes do primeiro teste de sincronização Meta. Não registrar os valores dessas variáveis no vault.

## Aplicativo Meta

O aplicativo foi criado como **Dach Ads** para uso da API de Marketing. O produto solicita acesso por OAuth somente quando o usuário conecta sua conta dentro do DACH Ads.

Configuração funcional:

| Item | Configuração |
| --- | --- |
| Callback OAuth | `https://dachbyte.tech/ads/api/meta/oauth/callback` |
| Política de privacidade | `https://dachbyte.tech/ads/privacidade` |
| Termos de serviço | `https://dachbyte.tech/ads/termos` |
| Instruções de exclusão | `https://dachbyte.tech/ads/exclusao-de-dados` |
| Graph API | `v26.0` |
| Escopos | `ads_read,business_management` |
| Modo atual recomendado | desenvolvimento, usando administrador/testador do app. |

A opção de exclusão usada no painel Meta é **URL de instruções de exclusão de dados**. Não há callback automatizado de exclusão nesta etapa.

## Variáveis e fluxo

```text
META_APP_ID
META_APP_SECRET
META_REDIRECT_URI
META_GRAPH_API_VERSION
META_OAUTH_SCOPES
ADS_TOKEN_ENCRYPTION_KEY
```

Os nomes acima devem permanecer iguais no código e na VPS. Valores de `META_APP_SECRET` e `ADS_TOKEN_ENCRYPTION_KEY` nunca entram nesta nota.

### Arquivos de ambiente e consumidores

| Arquivo na VPS | Consumidor | Conteúdo esperado |
| --- | --- | --- |
| `infra/env/ads.env` | `ads-api` e `ads-worker` | banco da aplicação/worker, Redis, identidade, chave do cofre, janelas de sincronização e variáveis Meta. |
| `infra/env/ads-google.env` | `ads-api` e `ads-worker` | credenciais OAuth/API do Google e, atualmente, uma segunda declaração da chave do cofre. |
| `infra/env/hub.env` | `ads-api` e `ads-worker` | endpoint/token interno do Hub e segredo compartilhado da sessão DACH. |
| `infra/env/ads-migrate.env` | `ads-migrate` | conexão do papel migrador e nomes dos papéis de runtime. |

No Compose, `ads-api` e `ads-worker` carregam os três arquivos na ordem `ads.env`, `ads-google.env`, `hub.env`. Após qualquer alteração nesses arquivos, é necessário recriar o container consumidor; reiniciar sem recriação pode manter o ambiente antigo.

Estado verificado sem exibir valores:

| Processo | Meta App ID | Meta App Secret | Meta Redirect | Token key | Banco/Redis |
| --- | --- | --- | --- | --- | --- |
| `ads-api` | presente | presente | produção | presente | configurados |
| `ads-worker` | **ausente** | **ausente** | **ausente** | presente | configurados |

Correção operacional pendente:

```bash
cd /opt/dachbyte/repository
docker compose --env-file infra/env/compose.env \
  -f infra/compose.vps.yml -p dachbyte-staging \
  up -d --force-recreate --no-deps ads-worker
```

Depois, confirmar somente presença — nunca imprimir segredos — e verificar `healthy` no Compose.

Fluxo esperado:

1. Usuário autenticado com `dach_ads` abre `/ads/app/meta`.
2. O backend cria `state` de uso único no Redis e redireciona para a Meta.
3. A Meta retorna para `META_REDIRECT_URI`.
4. O `ads-api` valida sessão, identidade e `state`, troca o código e cifra o token.
5. O backend descobre portfólios e contas permitidas.
6. O usuário seleciona a conta a sincronizar; o worker executa leitura assíncrona.
7. Ao desconectar, o serviço tenta revogar as permissões e marca a conexão como revogada.

## Arquitetura e responsabilidades

```text
Navegador
  └─ suite_auth_token (cookie assinado pelo ecossistema DACH)
       └─ ads-api
            ├─ valida JWT e consulta Hub /v1/access/check
            ├─ cria state OAuth no Redis
            ├─ troca code por token com a Meta
            ├─ cifra token no banco dachbyte_ads
            └─ agenda sincronização
                 └─ ads-worker
                      ├─ reivindica job com lock
                      ├─ consulta Graph API read-only
                      ├─ normaliza campanhas, conjuntos, anúncios e métricas
                      └─ grava dados sob contexto RLS do tenant
```

| Componente | Arquivo principal | Responsabilidade |
| --- | --- | --- |
| Entrada HTTP | `apps/ads/app.js` | rotas públicas, páginas legais, assets e montagem das APIs protegidas. |
| Configuração | `apps/ads/config/env.js` | nomes, defaults e parsing das variáveis. |
| Identidade Hub | `apps/ads/infrastructure/identity/HubIdentityProvider.js` | valida `suite_auth_token` e consulta a concessão `dach_ads`. |
| Middleware | `apps/ads/http/middleware/requireIdentity.js` | diferencia anônimo, acesso negado e Hub indisponível. |
| Rotas Meta | `apps/ads/http/routes/metaAdsRoutes.js` | status, OAuth, descoberta, seleção, sync e desconexão. |
| Serviço Meta | `apps/ads/application/meta/metaAdsService.js` | regras do fluxo, vínculo ao workspace e persistência das credenciais. |
| Cliente OAuth | `apps/ads/infrastructure/meta/metaOAuthClient.js` | autorização, troca do código, token longo, debug e revogação. |
| Graph API | `apps/ads/infrastructure/meta/metaGraphClient.js` | descoberta e leitura paginada de ativos/insights. |
| Cofre | `apps/ads/infrastructure/security/tokenVault.js` | cifra autenticada dos tokens. |
| Worker | `apps/ads/worker.js` | heartbeat, polling e alternância justa entre Google e Meta. |
| Persistência Meta | `apps/ads/infrastructure/meta/metaAdsRepository.js` e `metaAdsSyncRepository.js` | conexões, contas, jobs e snapshots sob tenant. |

## Contrato HTTP Meta

Todas as rotas abaixo passam pelo `requireIdentity`. O callback também depende da sessão DACH ainda presente no navegador após o retorno da Meta.

| Método e rota | Papel | Resultado esperado |
| --- | --- | --- |
| `GET /ads/api/meta/status` | estado da configuração, conexão, portfólios e contas. | JSON sem credenciais do provedor. |
| `GET /ads/api/meta/oauth/start` | cria `state` e inicia o OAuth. | `302` para a Meta. |
| `GET /ads/api/meta/oauth/callback` | valida `code/state`, troca token e descobre ativos. | `302` para `/ads/app/meta?meta=connected`. |
| `POST /ads/api/meta/connections/:connectionId/discover` | repete descoberta de ativos. | lista atualizada para o tenant autenticado. |
| `DELETE /ads/api/meta/connections/:connectionId` | revoga permissões e marca conexão como revogada. | `200` sem expor token. |
| `PUT /ads/api/meta/accounts/:accountId/selection` | ativa/desativa sincronização da conta. | conta atualizada. |
| `POST /ads/api/meta/accounts/:accountId/sync` | agenda sincronização manual. | `202 queued`. |

Semântica de acesso:

| Situação | HTML protegido | API protegida |
| --- | --- | --- |
| Sem cookie válido | redireciona para `/ads/login`. | `401 authentication_required`. |
| Sem `dach_ads` | redireciona para `/ads/access-denied`. | `403 dach_ads_access_denied`. |
| Hub indisponível/mal configurado | não libera acesso. | `503 auth_unavailable`. |
| Sessão e concessão válidas | entrega o shell. | segue para a operação solicitada. |

## Segurança do OAuth e dos tokens

- `state` possui 32 bytes aleatórios, é salvo no Redis e consumido uma única vez.
- TTL padrão do `state` Meta: `600` segundos; mínimo aceito pelo parser: `120` segundos.
- O callback exige que `tenantId` e `userId` coincidam com os gravados no `state`.
- O token é validado em `debug_token` antes de ser persistido.
- O serviço tenta trocar o token inicial por token de maior duração; se a troca falhar, registra aviso e mantém o token original.
- Tokens são cifrados com `AES-256-GCM`, IV aleatório de 12 bytes e tag de autenticação.
- O cofre recusa descriptografar credencial sem o prefixo cifrado `dach::ads::v1::`.
- Chamadas Graph incluem `appsecret_proof` HMAC-SHA256 quando o segredo Meta está configurado.
- Tokens não são enviados ao frontend nem armazenados no Hub.
- `ADS_TOKEN_ENCRYPTION_KEY` precisa representar exatamente 32 bytes. Não rotacionar sem plano de recifragem dos tokens existentes.

## Banco local e isolamento

Banco: `dachbyte_ads`, no PostgreSQL da VPS. O Hub de pagamentos permanece no Neon; os dados operacionais do Ads não ficam no Neon.

Migrations confirmadas no banco ativo:

| Migration | Aplicada em UTC | Escopo |
| --- | --- | --- |
| `001_foundation.sql` | `2026-09-20T15:22:34.714Z` | workspaces, memberships, conexões, contas e execuções de sync. |
| `002_google_ads.sql` | `2026-09-20T15:22:34.762Z` | entidades e métricas Google Ads. |
| `003_meta_ads.sql` | `2026-09-20T15:22:34.818Z` | portfólios Meta, criativos, ações e métricas adicionais. |
| `004_multichannel_fz.sql` | `2026-09-20T15:22:34.837Z` | metas do negócio, mapeamentos de conversão e diagnósticos FZ. |

Todas as migrations `001` a `004` estão aplicadas. O serviço `ads-migrate` fica no profile `ops` e não sobe no deploy comum.

Grupos de tabelas existentes:

| Grupo | Tabelas |
| --- | --- |
| Tenant e acesso | `ads_workspaces`, `ads_workspace_memberships`. |
| Provedores | `ads_provider_connections`, `ads_provider_credentials`, `ads_ad_accounts`. |
| Estrutura de mídia | `ads_campaigns`, `ads_ad_groups`, `ads_ads`, `ads_meta_businesses`, `ads_meta_creatives`. |
| Métricas | `ads_metrics_daily`, `ads_meta_action_metrics_daily`, `ads_conversion_actions`, `ads_google_keywords`, `ads_google_search_terms_daily`. |
| Sincronização | `ads_sync_jobs`, `ads_sync_runs`, `ads_sync_cursors`. |
| Inteligência | `ads_business_targets`, `ads_conversion_mappings`, `ads_fz_rule_runs`, `ads_fz_findings`. |

As tabelas sensíveis usam `tenant_id` e RLS com `current_setting('app.tenant_id', true)`. Repositórios de runtime executam as consultas dentro de `withTenant`; o papel migrador é separado dos papéis da API e do worker.

## Sincronização Meta

Descoberta inicial:

- `me/businesses` para portfólios acessíveis;
- `me/adaccounts` para contas de anúncio autorizadas.

Snapshot read-only por conta selecionada:

- campanhas;
- conjuntos de anúncios (`adsets` → `ad_group` no modelo DACH);
- anúncios e criativos;
- insights diários em níveis de conta, campanha, conjunto e anúncio;
- impressões, cliques, alcance, frequência, gasto, cliques únicos, link/outbound clicks;
- ações e valores mantidos por `action_type`, sem inventar uma conversão total única.

Parâmetros operacionais padrão:

| Variável | Default | Função |
| --- | --- | --- |
| `ADS_META_INITIAL_LOOKBACK_DAYS` | `90` | janela inicial. |
| `ADS_META_RECENT_LOOKBACK_DAYS` | `14` | janela incremental recente. |
| `ADS_META_SYNC_INTERVAL_MINUTES` | `120` | intervalo após sucesso. |
| `ADS_SYNC_WORKER_POLL_MS` | `10000` | frequência de polling. |
| `ADS_SYNC_JOB_LOCK_MINUTES` | `30` | expiração do lock de um job. |
| `ADS_WORKER_HEARTBEAT_TTL_SECONDS` | `60` | validade do heartbeat no Redis. |

O worker processa no máximo quatro jobs por ciclo e alterna o cursor entre `google_ads` e `meta_ads`. Jobs usam `FOR UPDATE SKIP LOCKED`. Em falha, a retentativa cresce em passos de cinco minutos até o máximo de uma hora; em sucesso, tentativas voltam a zero.

## Páginas legais

Foram adicionadas páginas públicas, independentes de login:

- `apps/ads/public/privacy.html`;
- `apps/ads/public/terms.html`;
- `apps/ads/public/data-deletion.html`;
- `apps/ads/public/legal.css`.

As rotas são entregues pelo `ads-api` e responderam HTTP `200` em produção.

## Identidade visual

O símbolo Ads foi ajustado para transparência alfa real. O arquivo consumido pelo produto é:

```text
apps/ads/public/ads-mark.png
```

Decisão de uso:

- favicon em landing, login, acesso negado, app e páginas legais;
- símbolo na marca da navbar do shell autenticado;
- nenhuma ilustração de símbolo no corpo da landing ou das páginas públicas.

Commits relacionados:

- `57242fcd` — marca inicial do DACH Ads;
- `2126520c` — páginas legais públicas;
- `c08b6be3` — posicionamento do ícone;
- `31492df4` — PNG com transparência real;
- `7af3e2f3` — invalidação do favicon antigo em cache.

## Validação e deploy

- `npm run ads:test` → `27/27` aprovados.
- `ads-api` reconstruído e confirmado como `healthy`.
- `ads-worker`, PostgreSQL e Redis confirmados como `healthy`.
- favicon público retornando HTTP `200`.
- páginas legais retornando HTTP `200`.
- migrations `001` a `004` confirmadas na tabela `schema_migrations` do banco ativo.

Comando de publicação utilizado:

```bash
cd /opt/dachbyte/repository
git pull --ff-only dachbyte main
docker compose --env-file infra/env/compose.env \
  -f infra/compose.vps.yml -p dachbyte-staging \
  up -d --build --force-recreate --no-deps ads-api
```

O nome técnico do projeto Compose continua sendo `dachbyte-staging`, mas o domínio utilizado nesta fase é `https://dachbyte.tech`. Não deduzir o ambiente apenas pelo nome do container.

### Comandos de diagnóstico seguros

Status dos serviços:

```bash
cd /opt/dachbyte/repository
docker compose --env-file infra/env/compose.env \
  -f infra/compose.vps.yml -p dachbyte-staging \
  ps ads-api ads-worker postgres redis
```

Logs recentes sem seguir indefinidamente:

```bash
docker compose --env-file infra/env/compose.env \
  -f infra/compose.vps.yml -p dachbyte-staging \
  logs --tail 200 ads-api ads-worker
```

Health público:

```bash
curl -fsS https://dachbyte.tech/ads/healthz
```

Não usar `printenv`, `docker inspect` completo ou comandos que despejem arquivos `infra/env/*.env` em logs compartilhados. Para checar configuração, imprimir somente booleanos de presença e valores não secretos, como a URI de callback.

### Deploy por tipo de alteração

| Alteração | Serviços a recriar |
| --- | --- |
| HTML/CSS/JS, rotas ou API Ads | `ads-api`. |
| Worker, sincronização ou qualquer `META_*`/`GOOGLE_*` usada em jobs | `ads-worker`; recriar `ads-api` também se ele consumir a mesma variável. |
| Migration SQL nova | executar `ads-migrate` pelo profile `ops`, validar e então recriar os runtimes afetados. |
| Caddy/roteamento | validar configuração e recriar/recarregar somente o proxy após revisão. |

Para configuração OAuth, a forma segura é:

```bash
docker compose --env-file infra/env/compose.env \
  -f infra/compose.vps.yml -p dachbyte-staging \
  up -d --force-recreate --no-deps ads-api ads-worker
```

### Rollback

1. Identificar o último commit DACH conhecido como saudável.
2. Reverter por novo commit na `main` ou publicar um checkout isolado desse commit; não usar `git reset --hard` no checkout compartilhado da VPS.
3. Reconstruir apenas `ads-api` e/ou `ads-worker` conforme o impacto.
4. Confirmar health, rotas públicas e logs.
5. Migration exige análise própria: as migrations são forward-only e não possuem downgrade automático documentado.

## Checklist de QA Meta

### Pré-condições

- [x] `META_APP_ID` presente no `ads-api`.
- [x] `META_APP_SECRET` presente no `ads-api`.
- [x] `META_REDIRECT_URI=https://dachbyte.tech/ads/api/meta/oauth/callback` no `ads-api`.
- [x] páginas legais públicas retornando `200`.
- [x] migrations `001` a `004` aplicadas.
- [ ] recriar `ads-worker` e confirmar presença das variáveis `META_*`.
- [ ] usuário de QA com sessão válida e concessão Hub `dach_ads`.
- [ ] conta Facebook/Meta usada no teste cadastrada como administradora ou testadora do app em modo desenvolvimento.
- [ ] conta de anúncios de QA acessível por essa identidade Meta.

### Jornada funcional

- [ ] Abrir `https://dachbyte.tech/ads/login` e autenticar.
- [ ] Confirmar entrada direta em `/ads/app`, sem seleção Seller.
- [ ] Abrir a área Meta e conferir `configuration.configured=true`.
- [ ] Clicar em conectar e confirmar redirecionamento para o app **Dach Ads** na Meta.
- [ ] Recusar uma vez e conferir retorno controlado `meta=cancelled`.
- [ ] Autorizar e confirmar callback sem erro de redirect URI ou `state`.
- [ ] Confirmar descoberta de portfólio e conta de anúncios esperados.
- [ ] Confirmar que contas não autorizadas não aparecem.
- [ ] Selecionar a conta de QA e solicitar sync manual.
- [ ] Confirmar resposta `202`, job processado pelo worker e execução `succeeded`.
- [ ] Conferir métricas de uma janela curta sem comparar valores secretos ou dados de outro tenant.
- [ ] Desconectar e confirmar status revogado/reconexão necessária.

### Isolamento e falhas

- [ ] Usuário sem `dach_ads` recebe acesso negado.
- [ ] API sem sessão devolve `401`.
- [ ] Hub indisponível faz o Ads falhar fechado com `503`, sem liberar acesso por cache permissivo.
- [ ] `state` reutilizado ou expirado é recusado.
- [ ] Token expirado gera erro recuperável e orientação de reconexão.
- [ ] Falha de uma conta/tenant não paralisa jobs dos demais tenants.
- [ ] Nenhuma resposta HTTP ou log apresenta access token, app secret ou chave do cofre.

## Critérios para liberar clientes externos

Não considerar Meta Ads pronto para clientes apenas porque o OAuth funciona com o administrador do app. Antes da liberação comercial:

1. concluir o QA acima;
2. vincular o aplicativo ao portfólio empresarial da DACHBYTE;
3. solicitar acesso avançado/App Review para as permissões exigidas;
4. revisar textos legais e dados societários com o responsável jurídico/comercial;
5. confirmar canal real de atendimento do endereço exibido nas páginas legais;
6. definir retenção e procedimento operacional de exclusão;
7. validar observabilidade, alertas, expiração/reconexão e recuperação do worker;
8. registrar evidências do teste por tenant sem copiar dados pessoais ou segredos para o vault.

## Pontos de atenção conhecidos

- O worker atual está saudável, mas ainda sem as variáveis Meta carregadas.
- O callback Meta é protegido e depende do cookie DACH sobreviver ao retorno do provedor; isso precisa ser confirmado no navegador real.
- O app Meta permanece em desenvolvimento; usuários externos que não tenham papel no app não conseguirão completar o OAuth.
- A página de exclusão contém instruções e e-mail, não um callback assinado automatizado da Meta.
- O contato público usado nas páginas legais deve corresponder a uma caixa realmente monitorada.
- `ads-google.env.example` ainda usa staging como exemplo de callback Google; não copiar esse valor para produção sem ajustar o domínio.
- O repositório da VPS mantém `dachbyte` como remoto canônico e `origin` histórico; deploy deve usar `git pull --ff-only dachbyte main`.

## Próximo teste

Usar a empresa de QA definida para o Ads e uma conta Meta que seja administradora ou testadora do aplicativo enquanto ele estiver em desenvolvimento. Confirmar OAuth, callback, descoberta de contas, seleção, sincronização e desconexão.

Antes de uso por clientes externos, concluir acesso avançado/App Review e vincular o aplicativo ao portfólio empresarial da DACHBYTE. O portfólio de uma empresa cliente não deve ser proprietário do aplicativo.

## Relações

- [[DACH Ads — estado operacional — 2026-09-21]]
- [[Mapa de rotas e autenticação DACH]]
- [[Runbook de deploy DACH na VPS]]
- [[Próximas]]
