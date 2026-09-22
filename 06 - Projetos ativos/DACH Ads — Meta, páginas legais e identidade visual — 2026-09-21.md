---
type: registro-operacional
produto: dach_ads
status: publicado-em-validacao
data: 2026-09-21
repositorio: dachbyte
branch: main
commit: 7af3e2f3
---

# DACH Ads — Meta, páginas legais e identidade visual — 2026-09-21

## Resultado

O backend do DACH Ads recebeu a configuração do aplicativo Meta em produção. As páginas públicas exigidas pelo provedor foram criadas, o callback foi fixado no domínio `dachbyte.tech` e a identidade visual do Ads foi incorporada ao favicon e ao shell autenticado.

Esta entrega configura a infraestrutura do OAuth; o teste ponta a ponta com uma conta Meta real continua pendente.

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

Fluxo esperado:

1. Usuário autenticado com `dach_ads` abre `/ads/app/meta`.
2. O backend cria `state` de uso único no Redis e redireciona para a Meta.
3. A Meta retorna para `META_REDIRECT_URI`.
4. O `ads-api` valida sessão, identidade e `state`, troca o código e cifra o token.
5. O backend descobre portfólios e contas permitidas.
6. O usuário seleciona a conta a sincronizar; o worker executa leitura assíncrona.
7. Ao desconectar, o serviço tenta revogar as permissões e marca a conexão como revogada.

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
- favicon público retornando HTTP `200`.
- páginas legais retornando HTTP `200`.

Comando de publicação utilizado:

```bash
cd /opt/dachbyte/repository
git pull --ff-only dachbyte main
docker compose --env-file infra/env/compose.env \
  -f infra/compose.vps.yml -p dachbyte-staging \
  up -d --build --force-recreate --no-deps ads-api
```

## Próximo teste

Usar a empresa de QA definida para o Ads e uma conta Meta que seja administradora ou testadora do aplicativo enquanto ele estiver em desenvolvimento. Confirmar OAuth, callback, descoberta de contas, seleção, sincronização e desconexão.

Antes de uso por clientes externos, concluir acesso avançado/App Review e vincular o aplicativo ao portfólio empresarial da DACHBYTE. O portfólio de uma empresa cliente não deve ser proprietário do aplicativo.

## Relações

- [[DACH Ads — estado operacional — 2026-09-21]]
- [[Mapa de rotas e autenticação DACH]]
- [[Runbook de deploy DACH na VPS]]
- [[Próximas]]
