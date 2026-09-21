---
type: arquitetura
status: verificado-parcialmente
atualizado: 2026-09-21
---

# Mapa de rotas e autenticação DACH

## Escopo

Este mapa descreve a entrada pública atual, quem atende cada família de rota e onde o Hub decide acesso. Ele não contém segredos, URLs internas de banco, tokens ou conteúdo dos arquivos `infra/env/*`.

## Entrada pública

O domínio público ativo é `https://dachbyte.tech`. O Caddy recebe o tráfego HTTPS e encaminha cada prefixo para um container na rede Docker. O Gateway não monta todos os produtos dentro do próprio processo. Ele atende login, sessão, páginas Seller e redirecionamentos protegidos; cada produto continua sendo um serviço.

```text
Navegador
   │
   ▼
dachbyte.tech
   │
   ▼
Caddy
 ├─ /selecao-plataforma, /login, /go/* ─────────────► gateway
 ├─ /ml/* ──────────────────────────────────────────► seller-ml-web
 ├─ /shopee/* ──────────────────────────────────────► seller-shopee
 ├─ /avantracking/* ────────────────────────────────► seller-tracking
 ├─ /madeiramadeira/*, /davanttilog/*, /skuleader/* ► módulos Seller próprios
 ├─ /business/* ────────────────────────────────────► portal e módulos Business
 └─ /ads/* ─────────────────────────────────────────► ads-api
```

## Rotas canônicas e compatibilidade

| Família | Caminho canônico público | Compatibilidade vigente | Responsável pela compatibilidade |
| --- | --- | --- | --- |
| Business Core | `/business/core` e `/business/core/api` | `/core`, `/api/core` | Caddy redireciona UI; API mantém proxy direto. |
| Business Chat | `/business/chat` e `/business/chat/api` | `/chat`, `/chat-api`, `/business/chat-api`, `/voltchat`, `/volt_chat` | Caddy. |
| Business Stock | `/business/stock` e `/business/stock/api` | `/voltstock`, `/stock`, `/voltstock/api` | Caddy. |
| Business Price | `/business/price` e `/business/price/api` | `/volt-price`, `/volt-price/api` | Caddy. |
| DACH Ads | `/ads` | Não participa da seleção Seller. | Serviço Ads e Gateway para a entrada protegida. |
| Seller ML | `/ml` | rota operacional atual | Caddy para `seller-ml-web`. |
| Seller Shopee | `/shopee` | rota operacional atual | Caddy para `seller-shopee`. |
| Seller Tracking | `/avantracking` | rota operacional atual | Caddy para `seller-tracking`. |
| Seller Log, Madeira e Leader | rotas próprias | `/davanttilog`, `/madeiramadeira`, `/skuleader` | Caddy; não aparecem na seleção compartilhada. |

As rotas legadas não devem ser removidas por edição casual. Clientes antigos, callbacks OAuth, builds desktop e links salvos podem depender delas. A remoção exige telemetria de uso, plano de comunicação e rollback.

## Fluxo de identidade e acesso

```text
Usuário → /login → Gateway → Hub paymentcontrol
                                  │
                                  ├─ identidade global
                                  ├─ tenant/empresa
                                  ├─ usuário ativo ou inativo
                                  └─ módulos liberados
                                           │
                                           ▼
                          sessão da suíte / entitlements
                                           │
                    ┌──────────────────────┴──────────────────────┐
                    ▼                                             ▼
           /selecao-plataforma                              /go/<módulo>
           filtra cartões visíveis                    revalida acesso no servidor
```

O filtro no navegador serve para a experiência. A autorização efetiva precisa continuar no Gateway ou no produto. Uma pessoa que digite uma rota direta não ganha acesso por ter um link.

### Seleção Seller

`/selecao-plataforma` é entregue por `apps/gateway/server.js`, usando `apps/seller-ml/views/selecao-plataforma.html`. A página consulta a sessão em `/api/auth/me`, coleta os entitlements e mostra apenas:

| Slug | Rota de entrada | Aliases aceitos no entitlement |
| --- | --- | --- |
| `ml` | `/go/ml` | `ml`, `meli`, `mercadolivre` e variações históricas. |
| `shopee` | `/go/shopee` | `shopee`. |
| `tracking` | `/go/tracking` | `tracking`, `rastreio`, `avantracking` e variações logísticas. |

O desenho não lista `dach_ads`, `davanttilog`, `madeiramadeira` nem `skuleader`. Seus handlers continuam registrados para compatibilidade e validam o módulo solicitado antes do redirecionamento.

### DACH Ads

DACH Ads usa a permissão Hub `dach_ads`, mas não compartilha a seleção Seller. A aplicação possui login próprio e endpoints próprios sob `/ads`. O produto deve receber a sessão da suíte apenas como prova de identidade e consultar o Hub antes de liberar dados de uma empresa.

Estado confirmado em código e testes:

- HTML protegido sem identidade redireciona para `/ads/login`.
- API protegida devolve `403` quando o Hub nega `dach_ads`.
- Tokens de Google/Meta pertencem ao servidor Ads; não devem aparecer no navegador, no Hub ou no vault.

## Onde alterar cada comportamento

| Necessidade | Arquivo ou ambiente principal | Não fazer |
| --- | --- | --- |
| Adicionar/remover cartão da seleção Seller | `apps/seller-ml/views/selecao-plataforma.html` e teste de catálogo | Não alterar o catálogo administrativo do Hub para resolver apenas uma questão visual. |
| Proteger uma entrada compartilhada | `apps/gateway/server.js` e `createSuiteGoHandler` | Não confiar só em ocultar o cartão no HTML. |
| Mudar proxy, alias ou redirect público | `infra/Caddyfile` e contratos de VPS | Não editar Caddy na VPS sem validar a configuração e sem testar a rota pública. |
| Mudar permissão de produto | Hub `paymentcontrol` e sua base Neon | Não inventar permissão local em um módulo. |
| Mudar segredo ou callback OAuth | arquivo correspondente em `infra/env/` na VPS e console do provedor | Não registrar valores em Git, Obsidian ou capturas de terminal. |

## Verificações mínimas após mudança de rota ou acesso

```bash
# No repositório DACH
node --test tests/*.test.js

# Na VPS, depois do pull correto
cd /opt/dachbyte/repository
git pull --ff-only dachbyte main

# Verificar a rota pública sem expor sessão
curl -fsS https://dachbyte.tech/healthz
curl -fsS https://dachbyte.tech/selecao-plataforma
```

Para os passos de publicação, consulte [[Runbook de deploy DACH na VPS]]. Para o estado funcional mais recente, consulte [[Atualização de produção, seleção Seller e DACH Ads — 2026-09-21]].

## Relações

- [[Gateway]]
- [[Hub]]
- [[VPS e staging]]
- [[DACH Ads — estado operacional — 2026-09-21]]
- [[Compatibilidade durante a migração]]
