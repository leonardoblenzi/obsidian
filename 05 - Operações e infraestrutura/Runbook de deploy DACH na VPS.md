---
type: runbook
status: operacional
atualizado: 2026-09-21
---

# Runbook de deploy DACH na VPS

## Quando usar

Use este procedimento para publicar código que já passou nos testes locais no ambiente `dachbyte.tech`. Ele cobre atualização Git, rebuild seletivo, saúde dos containers e smoke público. Não executa migration de banco, não altera DNS e não atualiza segredos.

## Pré-requisitos

- O commit já existe em `dachbyte/main`.
- A suíte relevante passou localmente. Para mudança transversal, use `node --test tests/*.test.js`.
- Você sabe quais serviços foram afetados. Para a seleção Seller publicada em 21/09/2026: `gateway` e `seller-ml-web`.
- Você verificou que a mudança não requer migration, novo secret, callback OAuth ou alteração de Caddy.

## Estado especial da VPS

O checkout operacional é `/opt/dachbyte/repository`. Ele mantém dois remotos:

```text
dachbyte → git@github.com:leonardoblenzi/dachbyte.git   # usar para deploy
origin   → git@github.com:leonardoblenzi/davanttiSuite.git # histórico; não usar
```

Use sempre `git pull --ff-only dachbyte main`. O `origin` diverge da linha atual e um pull por ele falha. Não troque remotos, não execute `reset --hard` e não use `stash` para “limpar” o servidor sem uma tarefa de manutenção específica.

## Procedimento padrão

### 1. Inspecionar antes de atualizar

```bash
cd /opt/dachbyte/repository
git status --short
git branch -vv
git log -1 --oneline
```

Interrompa se houver modificação no mesmo arquivo que o deploy pretende atualizar. A VPS possui arquivos locais de ambiente e backups que não pertencem ao Git e devem permanecer intactos.

### 2. Atualizar o checkout

```bash
git pull --ff-only dachbyte main
git rev-parse --short HEAD
```

Confirme que o hash corresponde ao commit publicado. O pull precisa terminar por fast-forward; qualquer pedido de merge ou rebase exige investigação antes de seguir.

### 3. Validar configuração quando a mudança afetar infraestrutura

Execute apenas se o commit alterar `infra/compose.vps.yml`, `infra/Caddyfile`, Dockerfiles ou scripts de operação:

```bash
cd /opt/dachbyte/repository/infra
./business-staging-ops.sh config
```

Mesmo com o nome histórico do script, a validação usa o Compose da VPS atual. Não rode `up` global por padrão: ele reconstrói mais serviços do que o necessário.

### 4. Publicar somente os serviços afetados

Exemplo real para Gateway e seleção Seller:

```bash
cd /opt/dachbyte/repository/infra
docker compose --env-file ./env/compose.env -f compose.vps.yml \
  up -d --build --force-recreate --no-deps gateway seller-ml-web
```

`--force-recreate` é obrigatório quando o Compose mantém um container ativo mesmo após uma imagem ter sido reconstruída. `--no-deps` impede que PostgreSQL, Redis e módulos não relacionados sejam recriados.

Troque a lista final de serviços conforme o impacto real. Não use `docker compose down` para um deploy ordinário.

### 5. Aguardar e checar saúde

```bash
docker compose --env-file ./env/compose.env -f compose.vps.yml \
  ps gateway seller-ml-web
```

Espere `healthy`. Se um container ficar `unhealthy`, colete logs antes de recriar novamente:

```bash
docker compose --env-file ./env/compose.env -f compose.vps.yml logs --tail=200 gateway
docker compose --env-file ./env/compose.env -f compose.vps.yml logs --tail=200 seller-ml-web
```

### 6. Smoke público

```bash
curl -fsS https://dachbyte.tech/healthz
curl -fsS https://dachbyte.tech/selecao-plataforma | grep -Eo 'data-module="[^"]+"' | sort -u
```

Para a seleção Seller atual, o segundo comando deve listar somente `ml`, `shopee` e `tracking`.

## Checklist por tipo de mudança

| Mudança | Serviços prováveis | Verificação adicional |
| --- | --- | --- |
| Página ou sessão Seller | `gateway`, `seller-ml-web` | Login de QA, `/selecao-plataforma`, `/go/ml`, `/go/shopee`, `/go/tracking`. |
| DACH Ads | `ads-api`, `ads-worker` | `/ads/login`, sessão Hub, OAuth e fila do worker. |
| Core | `business-core` | `/business/core`, `/business/core/api/health`, login e fluxo de QA. |
| Caddy ou rotas | `caddy` e serviços envolvidos | `caddy validate`, aliases canônicos e legado, headers de depreciação. |
| Banco ou migration | serviço afetado + operação explícita | Backup, migration, health e rollback documentados antes do deploy. |

## Rollback

1. Pare e registre o erro, o commit e os logs relevantes sem expor segredos.
2. Identifique o último commit saudável e atualize o checkout por um procedimento Git controlado. Não use reset destrutivo no servidor.
3. Reconstrua somente os serviços afetados com `--force-recreate`.
4. Repita os healthchecks e o smoke público.
5. Registre a causa, o commit revertido e o estado final no vault.

O checkout possui referências históricas de rollback. Antes de usá-las, confirme se representam o banco, as variáveis e os containers do incidente atual.

## Não incluir em deploy comum

- `infra/env/*` e qualquer secret;
- DNS, origem OAuth, callback ou webhook externo;
- migration de banco sem backup e plano de rollback;
- `docker compose down`, limpeza de volume ou remoção de imagem;
- alteração do remoto histórico da VPS sem inventário das modificações locais.

## Relações

- [[VPS e staging]]
- [[Mapa de rotas e autenticação DACH]]
- [[Atualização de produção, seleção Seller e DACH Ads — 2026-09-21]]
