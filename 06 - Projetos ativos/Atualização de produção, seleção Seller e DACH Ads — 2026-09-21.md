---
type: registro-operacional
status: publicado
data: 2026-09-21
repositorio: dachbyte
branch: main
commit: e95db29
---

# Atualização de produção, seleção Seller e DACH Ads — 2026-09-21

## Resumo executivo

O ambiente público DACH está ativo em `https://dachbyte.tech`. A seleção compartilhada do Seller foi reduzida aos três produtos que realmente usam a jornada de sessão comum: Mercado Livre, Shopee e Tracking. DACH Ads, Log, Madeira e Leader não aparecem mais nessa seleção; seus acessos permanecem independentes e protegidos nos respectivos fluxos e rotas.

O commit publicado foi `e95db29` (`Refine Seller module selection`) na branch `main` de `git@github.com:leonardoblenzi/dachbyte.git`.

## Decisão de produto e acesso

| Contexto | Regra atual |
| --- | --- |
| Seleção Seller | Mostra apenas `ml`, `shopee` e `tracking`. |
| Permissão dos três cartões | Continua sendo decidida pelo Hub. Sem o módulo retornado na sessão, o cartão não é exibido/acionável. |
| DACH Ads | Não integra a seleção Seller; mantém login e aplicação dedicados em `/ads`, com autorização Hub para `dach_ads`. |
| Log, Madeira e Leader | Não são mais portas de entrada da seleção Seller. Rotas compatíveis permanecem protegidas pelo Gateway para fluxos próprios e links existentes. |

Arquivos funcionais principais no repositório DACH:

- `apps/seller-ml/views/selecao-plataforma.html` — cartões e filtro visual de permissões.
- `apps/gateway/server.js` — entrega a seleção e mantém os handlers protegidos `/go/*`.
- `tests/seller-platform-catalog.test.js` — impede a reintrodução de Ads, Log, Madeira ou Leader na seleção.

## Correções de contratos realizadas

A suíte raiz tinha três falhas que não eram da seleção, mas precisavam ser corrigidas para que o baseline ficasse confiável:

1. `packages/design-system/src/tokens.css` não continha os tokens de acento do Ads já presentes no arquivo público. A fonte canônica foi sincronizada.
2. O contrato de compatibilidade ainda procurava mounts Seller no Node Gateway. Na arquitetura atual, os aliases `/ml`, `/shopee`, `/madeiramadeira`, `/avantracking`, `/davanttilog` e `/skuleader` são atendidos pelo Caddy, com proxy para os serviços correspondentes. O teste e o registro de compatibilidade foram atualizados para esse limite correto.
3. O contrato da landing Business esperava caminhos Volt legados. A landing usa os caminhos canônicos `/business/stock`, `/business/chat` e `/business/core`; o teste foi alinhado sem remover aliases legados do Caddy.

Validação local final: `node --test tests/*.test.js` → **161 testes aprovados, 0 falhas**.

## Publicação na VPS

| Item | Estado verificado |
| --- | --- |
| Checkout | `/opt/dachbyte/repository` no commit `e95db290`. |
| Serviços reconstruídos | `gateway` e `seller-ml-web`, somente os serviços necessários para a seleção. |
| Saúde | Ambos ficaram `healthy`. |
| Página pública | `https://dachbyte.tech/selecao-plataforma` retornou somente `ml`, `shopee` e `tracking`. |
| Health público | `https://dachbyte.tech/healthz` respondeu `{"ok":true,"brand":"DachByte"}`. |

O comando inicial de `up --build` não recriou containers já ativos. Para publicar efetivamente, foi necessário:

```bash
cd /opt/dachbyte/repository/infra
docker compose --env-file ./env/compose.env -f compose.vps.yml \
  up -d --build --force-recreate --no-deps gateway seller-ml-web
```

## Atenção operacional na VPS

O repositório da VPS possui dois remotos:

- `dachbyte` → repositório canônico atual, `git@github.com:leonardoblenzi/dachbyte.git`;
- `origin` → repositório histórico `davanttiSuite`.

Para deploys do DACH, usar explicitamente:

```bash
cd /opt/dachbyte/repository
git pull --ff-only dachbyte main
```

Não usar `git pull origin main`: ele diverge da história atual e falha por fast-forward. A VPS também contém modificações locais preexistentes em scripts e backups de ambiente; elas foram preservadas e não foram incluídas no commit nem no deploy.

## Andamento e próximos passos

1. Testar manualmente, com uma sessão por perfil, a visibilidade e a abertura de ML, Shopee e Tracking após o login via Hub.
2. Testar a jornada independente do DACH Ads: login, OAuth Google e a futura configuração Meta, usando uma empresa e usuário de QA.
3. Corrigir a operação de deploy: definir `dachbyte` como remoto padrão ou documentar um script que use esse remoto explicitamente; antes, inventariar e preservar as alterações locais da VPS.
4. Criar CI para rodar `node --test tests/*.test.js` antes de cada deploy e um deploy controlado que force recriação somente dos serviços alterados.
5. Manter o rollback por commit/branch e validar o procedimento em janela controlada. Nenhum segredo, variável de ambiente ou backup deve ser registrado no Git ou no vault.

## Relações

- [[VPS e staging]]
- [[Seller e operações]]
- [[Identidade e acesso]]
- [[Landings Seller e Business — 2026-09-14]]
- [[Migração para VPS]]
