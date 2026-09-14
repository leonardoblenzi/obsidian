---
type: handoff
status: pronto-para-retomar
updated: 2026-09-13
---

# Handoff — Hub, VPS e masters — 2026-09-13

Use esta nota como ponto de retomada em outro chat. Ela descreve o estado **já aplicado e verificado**, os limites atuais e os próximos testes. Não contém valores de secrets.

## Estado em uma frase

O staging Dachbyte está saudável na VPS, todos os módulos apontam para o Hub Cloudflare `paymentcontrol`, e esse Worker está conectado à branch Neon piloto `paymentcontrol-pilot-20260910`.

O Core foi exercitado no navegador com uma empresa de QA, incluindo cadastro, venda, baixa de estoque, pagamento e recibo. O repositório `dachbyte/main` já é a origem Git do staging na VPS e o primeiro deploy dessa nova etapa publicou as landings enxutas da linha Seller.

## Topologia ativa

| Camada | Estado e referência |
| --- | --- |
| Staging público | `https://staging.dachbyte.tech` |
| VPS | Hostinger `srv1971387`; acesso SSH já validado. |
| Repositório na VPS | `/opt/dachbyte/repository`, branch `main`, remoto `dachbyte`. |
| Repositório canônico da nova etapa | `git@github.com:leonardoblenzi/dachbyte.git`, branch `main`. |
| Checkout local da nova etapa | `C:\\Users\\Administrator\\Documents\\Projetos\\Dachbyte`. |
| Entrada da VPS | Caddy; serviços e bancos em Docker na rede privada. |
| Hub ativo | `https://paymentcontrol.davantti-suite.workers.dev` (`/ops-portal`). |
| Código do Hub | Repositório `hubPagamentos`, branch Git `payment`; ambiente `[env.paymentcontrol]`. |
| Banco do Hub | Projeto Neon `hidden-haze-81550394`, branch `paymentcontrol-pilot-20260910`, filha de `040926branch`. |
| Versão Cloudflare ativa | `c6b67479`, promovida a 100% em 13/09/2026; alteração de configuração `NEON_DATABASE_URL`. |

## Estado do código e das origens Git

| Item | Estado atual | Regra de uso |
| --- | --- | --- |
| Repositório novo | `git@github.com:leonardoblenzi/dachbyte.git`, branch `main`; staging implantado em `20d7807`. | É a origem canônica para trabalho e deploy desta nova etapa. |
| Checkout novo local | `C:\\Users\\Administrator\\Documents\\Projetos\\Dachbyte`. | Acompanha `origin/main` por SSH. |
| Repositório histórico | `davanttiSuite`, branch de trabalho `dach`. | Preservado como referência; não apagar nem reescrever. |
| Checkout do staging | `/opt/dachbyte/repository` na VPS, branch `main`, acompanhando `dachbyte/main`. | Usar para os próximos deploys de staging; preservar o remoto e branch históricos apenas para rollback. |
| Arquivos de ambiente | `/opt/dachbyte/repository/infra/env/`. | Não são versionados e não devem ir para Git ou Obsidian. |

O commit inicial do novo repositório contém o estado rastreado do checkout anterior, sem valores de `.env` e sem trazer o histórico Git antigo. O primeiro corte foi validado em staging com backup dos ambientes, build do gateway e smoke tests HTTP.

## O que foi concluído

### Novo repositório Dachbyte

- Foi criado o repositório independente `dachbyte` em `github.com/leonardoblenzi/dachbyte` para a nova etapa do projeto.
- A versão atual rastreada no branch `dach` foi publicada como histórico inicial da nova `main`, no commit `74f6dd8` (`chore: iniciar repositorio dachbyte`).
- O novo checkout local está em `C:\\Users\\Administrator\\Documents\\Projetos\\Dachbyte` e acompanha `origin/main` via SSH.
- A VPS foi cortada para `dachbyte/main` no commit `20d7807`; o remoto e uma branch local do estado anterior foram preservados para rollback.

### Landings Seller

- A landing geral foi padronizada com a tipografia da linha Business e a paleta atual da Seller, mantendo conteúdo curto e foco em conversão.
- Foram publicadas páginas próprias para Mercado Livre, Shopee e Rastreio, além da página geral Seller.
- Rotas verificadas com HTTP `200`: `/seller`, `/seller/mercado-livre`, `/seller/shopee`, `/seller/rastreio`, `/seller-assets/seller-landing.css` e `/login`.
- A estrutura e os links de entrada de cada módulo foram conferidos no navegador integrado.
- Implementação no commit `20d7807` (`feat: padronizar landings da linha seller`). O gateway foi reconstruído e ficou `healthy`.

### Worker e Neon

- Acessos ao Cloudflare e Neon foram confirmados no navegador integrado.
- A branch Neon correta foi aberta e confirmada como ativa: `paymentcontrol-pilot-20260910`.
- O secret Cloudflare `NEON_DATABASE_URL` do Worker `paymentcontrol` foi rotacionado para essa branch.
- A versão criada pela rotação (`c6b67479`) foi promovida a 100% do tráfego.
- O portal do Hub continuou acessível após a promoção e mostra quatro operadores internos.

### VPS e módulos

- Foram encontrados 13 ambientes de módulo ainda apontando para o Worker legado `hub-pagamento`.
- Todos foram alterados para usar `HUB_BASE_URL=https://paymentcontrol.davantti-suite.workers.dev`.
- Foram recriados, sem rebuild e sem alteração de banco, 14 serviços: gateway, Core, ML web/worker, Shopee, Rastreio, Madeira, Leader, Log, Price, Stock, Chat, Chat API e portal Business.
- Todos os serviços ficaram `healthy`; staging e health do Core retornaram `200`.
- Backup recuperável dos ambientes anterior à troca: `/opt/dachbyte/backups/hub-base-url-before-paymentcontrol-20260913T142153Z`.

### Masters e escopos

No Hub ativo, os escopos observados no `/ops-portal` são:

| Master | Escopo permitido |
| --- | --- |
| Mercado Livre | `ml` |
| Shopee | `shopee` |
| Rastreio | `tracking` |
| Volt Core | `volt_core` |

- O master local do Core foi provisionado e testado com sucesso (`200`, `admin_master`).
- O operador do Volt Core existe no Hub e tem somente `volt_core`.
- A Shopee foi reparada na base existente `dachbyte_shopee`; antes houve dump recuperável e depois as migrations foram reaplicadas. Não foi criada uma base nova.
- O fallback de senha fixa do bootstrap da Shopee foi removido. O seed exige a variável `SUPER_ADMIN_PASSWORD`.

### Teste operacional completo do Core

O Core foi aberto em `https://staging.dachbyte.tech/core/app` com o master local e testado na empresa **QA Hub Volt Core Rebuild 20260911**. A empresa **Ótica Nacional Sabaudia não foi alterada**.

| Etapa | Resultado verificado |
| --- | --- |
| Login master | Sucesso; painel master exibiu cinco empresas e os controles de administração. |
| Entrada no ambiente QA | Sucesso; setor `Core padrão`, sem dados pré-existentes relevantes. |
| Produto | Criado `QA Core Operacao 2026-09-13`, SKU `QA-CORE-20260913-01`, estoque inicial 2 e mínimo 1. |
| Cliente | Criado `QA Cliente Operacao 2026-09-13` para o fluxo de QA. |
| Venda PDV | Venda #1 concluída por Pix no valor de R$ 9,99. |
| Pedido e pagamento | Pedido #1 exibido como concluído, cliente identificado e pagamento Pix concluído. |
| Estoque | Baixa automática confirmada: 2 para 1 unidade; alerta de mínimo exibido corretamente. |
| Dashboard | Atualizado para 1 venda e R$ 9,99 recebido no dia. |
| Recibo | Recibo da venda disponível para emissão. |

Os registros acima são dados de QA intencionais e devem permanecer enquanto forem úteis para regressão e auditoria. Não reutilizar essa empresa para dados comerciais reais.

## Situação por módulo

| Módulo/camada | Concluído e verificado | Pendente antes de declarar pronto para produção |
| --- | --- | --- |
| Hub `paymentcontrol` | Worker ativo, ligação Neon piloto confirmada, portal acessível e quatro operadores internos com escopos isolados. | Validar o fluxo oficial de senha global dos operadores e acompanhar erros/métricas após uso contínuo. |
| Gateway/seleção de plataforma | Sessão central e redirecionamento foram testados para ML, Shopee e Rastreio; cada master vê somente seu módulo. | Repetir teste completo com credenciais globais definitivas, incluindo Core, após a definição de senha no Hub. |
| Landings Seller | Geral, ML, Shopee e Rastreio publicadas no staging; rotas, assets, títulos, CTAs e destinos dos módulos verificados. | Fazer revisão de conteúdo e responsividade com o usuário; depois medir conversão antes de acrescentar novas seções. |
| Volt Core | Health `200`, master local, empresa QA e fluxo produto → cliente → PDV → estoque → pagamento → recibo testados no navegador. | Corrigir/definir a rota pública esperada para `/core/login` (hoje retorna `Cannot GET`); testar abertura/fechamento de caixa, recebível a prazo, permissões de usuário não-master e relatórios. |
| Mercado Livre | Login/escopo central e rotas do módulo confirmados; origem e callback OAuth de staging configurados. | Executar OAuth real com conta de teste e testar leitura/atualização operacional no Mercado Livre. |
| Shopee | Escopo central e entrada no módulo confirmados; schema reparado na base existente `dachbyte_shopee`; bootstrap sem senha fixa no código. | Executar OAuth/credenciais reais de loja de teste e validar sincronização/ações da Shopee. |
| Rastreio (Avantracking) | Login/escopo central e abertura do módulo confirmados. | Validar consulta e operações reais com integração/conta de teste. |
| Serviços auxiliares na VPS | Serviços recriados e reportados `healthy` após troca de `HUB_BASE_URL`. | Executar testes funcionais específicos para Madeira, Leader, Log, Price, Stock, Chat, Chat API e portal Business. |
| Produção | Nenhum serviço Dachbyte de produção foi publicado. | Planejar domínio, banco, secrets, backups, smoke tests e rollback antes de qualquer publicação. |

## Separação de bancos

- Os módulos de staging usam PostgreSQL interno na VPS, com bases `dachbyte_*`; não há sufixo `_staging`, pois o isolamento é definido pela stack/ambiente.
- O Hub permanece no Cloudflare + Neon por decisão explícita; não migrar essa base para a VPS sem novo plano de corte.
- A branch Neon piloto é isolada da produção e da branch origem; não alterar `production` ou `040926branch` ao trabalhar no Hub de staging.

## Variáveis e arquivos a consultar

Antes de modificar qualquer ambiente, consultar [[Inventário de variáveis — Hub e staging]]. Os nomes críticos são:

| Fluxo | Variáveis principais |
| --- | --- |
| Hub Cloudflare | `NEON_DATABASE_URL`, `HUB_PUBLIC_BASE_URL`, `HUB_INTERNAL_TOKEN`, `HUB_ACCESS_ENFORCEMENT_MODE`, `HUB_PLATFORM_SCOPES_ENABLED` |
| Integração VPS | `HUB_BASE_URL`, `HUB_INTERNAL_TOKEN`, `HUB_LOGIN_MODE`, `HUB_AUTH_MODE`, `HUB_ENFORCEMENT` |
| Core | `VOLT_CORE_BOOTSTRAP_MASTER_ENABLED`, `VOLT_CORE_BOOTSTRAP_MASTER_EMAIL`, `VOLT_CORE_BOOTSTRAP_MASTER_NAME`, `VOLT_CORE_BOOTSTRAP_MASTER_PASSWORD` |
| ML/OAuth | `ML_PUBLIC_ORIGIN`, `ML_REDIRECT_URI`, `ML_APP_ID`, `ML_CLIENT_SECRET`, `ML_BOOTSTRAP_MASTER_*` |
| Shopee | `SHOPEE_DATABASE_URL`, `SHOPEE_*`, `SUPER_ADMIN_PASSWORD` |

Arquivos de ambiente na VPS ficam em `/opt/dachbyte/repository/infra/env/` e não são versionados. Valores de secrets não devem ir para Git, terminal compartilhado ou Obsidian.

## Pontos ainda abertos

1. **Senha global de masters no Hub:** criar operador no `/ops-portal` define escopo, mas não cria senha global. O Core possui bootstrap local; isso não atualiza automaticamente a credencial global do Hub. Usar o fluxo oficial de recuperação de senha do Hub ou uma alteração direta e controlada na branch Neon piloto.
2. **Core e login centralizado:** o master local do Core funciona em `/core/app`, porém `/core/login` retorna `Cannot GET`. Definir a rota canônica e validar o login pelo Hub com uma senha global antes de expor o fluxo a usuários.
3. **OAuth Mercado Livre:** validar o callback configurado em `ML_REDIRECT_URI`, confirmar `ML_PUBLIC_ORIGIN` e concluir uma autorização real de conta de teste no staging.
4. **Integrações reais:** testar uma loja Shopee de teste e uma conta Avantracking de teste; registrar resultados e erros por módulo.
5. **Backups:** concluir backup criptografado, retenção e teste de restauração para bancos VPS e plano de recuperação para o Hub/Neon.
6. **Pipeline do novo repositório:** o corte para `dachbyte/main` foi concluído manualmente; falta automatizar CI/deploy e testar formalmente o rollback.
7. **Produção:** não há serviço Dachbyte de produção ativo. Não alterar DNS, tráfego, callbacks produtivos ou encerrar Render/Neon sem autorização específica.

## Próximos passos sugeridos, na ordem

1. Definir/resetar a senha global de cada master no Hub e testar, em janela anônima, o acesso centralizado de ML, Shopee, Rastreio e Core.
2. Ajustar a rota pública de login do Core e repetir o teste com um usuário operacional de QA, não apenas com o master.
3. Executar OAuth real do Mercado Livre e da Shopee com contas de teste; registrar callback, permissões e primeira operação lida/escrita.
4. Executar o smoke test dos serviços auxiliares e registrar evidências no respectivo tópico do vault.
5. Criar CI/deploy repetível para `dachbyte/main`, incluindo testes de arquitetura, smoke tests e procedimento de rollback.
6. Só então abrir um plano separado para produção, com variáveis, DNS, segurança, monitoramento e restauração validados.

## Como verificar antes de avançar

- No Cloudflare: Worker `paymentcontrol` → Deployments; confirmar versão ativa e 100% de tráfego.
- No Neon: projeto `hubpagamento` → branch `paymentcontrol-pilot-20260910`; não selecionar `production` nem `040926branch` para mudanças de staging.
- Na VPS: confirmar branch `main`, commit esperado e serviços com `docker compose --env-file infra/env/compose.env -f infra/compose.vps.yml ps` dentro de `/opt/dachbyte/repository`.
- No Hub: abrir `/ops-portal#operators` e conferir os quatro escopos de master.
- No staging: verificar `https://staging.dachbyte.tech` e `https://staging.dachbyte.tech/core/healthz`.
- Nas landings: verificar `/seller`, `/seller/mercado-livre`, `/seller/shopee` e `/seller/rastreio`.

## Rollback conhecido

- **Worker:** a versão anterior `1aa57f70` continua no histórico do Cloudflare e pode ser promovida se a nova conexão apresentar falhas.
- **VPS/repositório:** o corte para `dachbyte/main` gerou o backup `/opt/dachbyte/backups/new-main-20260914T014403Z` e a branch local `rollback/dach-before-new-main-20260914T014403Z`; restaurar os ambientes do backup e reconstruir somente os serviços afetados.
- **Shopee:** o dump anterior à reparação do schema está em `/opt/dachbyte/backups/dachbyte_shopee_before_schema_repair_20260913T133153Z.dump`.

## Relações

- [[Migração para VPS]]
- [[VPS e staging]]
- [[Identidade e acesso]]
- [[Inventário de variáveis — Hub e staging]]
- [[Mercado Livre]]
- [[Shopee]]
