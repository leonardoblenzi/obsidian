---
status: em andamento
produto: Seller ML
repositorio: dachbyte
branch: main
commit: 1b8652a
atualizado: 2026-09-22
---

# Seller ML — atualização de estoque em massa

Relacionados: [[Mercado Livre]], [[Runbook de deploy DACH na VPS]]

## Estado atual

As Etapas 1, 2 e 3 foram aplicadas e enviadas para `main` no commit `1b8652a` (`Add ML stock update workflow`). O deploy na VPS ainda não foi executado.

## Etapa 1 — Painel

- Os comparativos do Painel mostram o período explícito: dia anterior, 7, 14 ou 30 dias anteriores.
- O bloco passou a usar “Ritmo do mês”, “Esperado até hoje” e a referência ao ritmo necessário para igualar o mês passado.
- Nenhuma fórmula financeira ou de ritmo foi modificada.

## Etapa 2 — Estoque: consulta e preview

- Estoque foi movido visualmente para Operações.
- A tela ganhou a aba “Atualizar estoque”, com carga de anúncios ativos ou MLBs/SKUs específicos, filtros, seleção, edição individual e preenchimento em massa.
- O preview reconsulta o Mercado Livre, separa itens simples de variações por `variation_id`, bloqueia itens inativos e identifica estoque alterado desde a carga.
- Esta etapa não realizava escrita no Mercado Livre.

## Etapa 3 — Estoque: execução em fila

- A confirmação cria jobs Bull/Redis processados pelo `seller-ml-worker`.
- Cada linha é revalidada antes da escrita; o resultado confirma a quantidade depois do `PUT` e mostra antes → solicitado → confirmado.
- Há cancelamento cooperativo, CSV final, auditoria e retry apenas para erros seguros sem escrita confirmada.
- Variações do mesmo MLB são agrupadas para evitar escritas concorrentes.

### Proteções ativas

- Estoque Full não é alterado.
- Contas com `warehouse_management` são bloqueadas.
- Linhas desatualizadas ficam como `stale` e não sofrem sobrescrita.
- Conflitos de quantidade para o mesmo User Product bloqueiam as linhas envolvidas.

## Validação local

- Sintaxe dos JavaScripts verificada.
- Testes de preview e serviço de estoque: 13/13 aprovados.
- Não houve migration nem nova variável obrigatória de ambiente.

## Próximas etapas

- [ ] Publicar `1b8652a` na VPS, reconstruindo e reiniciando `seller-ml-web` e `seller-ml-worker`.
- [ ] Executar QA com uma conta de teste: um anúncio simples e uma variação, usando quantidades fáceis de conferir.
- [ ] Validar rechecagem `stale`, bloqueios de Full e multi-origem, cancelamento, CSV e retry seguro.
- [ ] Etapa 4: implementar suporte nativo a estoque multi-origem por User Product, depósito (`seller_warehouse`) e controle de versão exigido pelo Mercado Livre.
- [ ] Só liberar o fluxo para lotes maiores após o QA em conta de teste e a confirmação do resultado no Mercado Livre.

## Deploy Seller ML na VPS

Usar o checkout `/opt/dachbyte/repository`, branch `main`, e o Compose em `/opt/dachbyte/repository/infra`. Antes de atualizar, conferir alterações locais e preservar arquivos não rastreados e `infra/env/`.

```bash
cd /opt/dachbyte/repository
git fetch origin
git pull --ff-only origin main
cd infra
docker compose --env-file ./env/compose.env -f compose.vps.yml build seller-ml-web seller-ml-worker
docker compose --env-file ./env/compose.env -f compose.vps.yml up -d --no-deps seller-ml-web seller-ml-worker
docker compose --env-file ./env/compose.env -f compose.vps.yml ps seller-ml-web seller-ml-worker
```
