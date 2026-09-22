---
type: operacoes
status: aplicado-sem-rollback
updated: 2026-09-22
---

# Migração de auditoria ML para partições — 2026-09-22

## Resultado

A tabela `ml.auth_audit` do Seller ML foi migrada de tabela regular para pai particionado mensalmente por `created_at`. O objetivo é tornar a retenção e a remoção futura de dados previsíveis, sem `VACUUM FULL` e sem varrer uma tabela de auditoria única cada vez maior.

- Ambiente: VPS `srv1971387`, checkout `/opt/dachbyte/repository`, branch `main`.
- Código publicado até o commit `35d6aae`.
- `seller-ml-web` e `seller-ml-worker` ficaram `healthy` após a operação.
- Migration aplicada: `072_auth_audit_partition_operations.sql`.
- Ledger ativo: `ml.auth_audit_partition_operations`.

## Execução realizada

1. Preflight aprovado para a tabela legacy regular, seu schema, FKs de saída, ausência de FKs de entrada, grants, capacidade e janela de manutenção.
2. Capacidade: tabela legacy com `2.211.479.552` bytes; estimativa de cópia com margem `2.874.923.418` bytes; espaço livre informado e validado: `56.444.706.816` bytes.
3. Cópia para `ml.auth_audit_partitioned_new`: `216.095` eventos, último `id` `902584`, batch de `10.000`.
4. Validação aprovada, sem divergências de contagem, IDs, datas, agregações nem vínculos de empresa/conta ML.
5. Janela curta: apenas `seller-ml-web` e `seller-ml-worker` foram parados; o swap executou catch-up final sob lock exclusivo. Não havia eventos adicionais a copiar.
6. O pai ativo passou a ter partições mensais de `auth_audit_2026_03` até `auth_audit_2028_03`, além de `auth_audit_default`. Os sete grants explícitos foram copiados.

## Retenção pós-swap

Na primeira manutenção automática após o restart, o serviço aplicou a retenção dinâmica já configurada no Master. O pai ativo ficou com `215.847` eventos, enquanto o legado ainda tinha `216.095`.

Foi feita comparação somente leitura: os `248` eventos ausentes do pai ativo estavam todos expirados pelas regras em `ml.auth_audit_retention_rules` (`248` expirados; `0` ainda retidos). Intervalo desses eventos: `2026-03-26` a `2026-06-24`.

O scheduler continua:

- criando e drenando partições de modo seguro;
- aplicando a retenção dinâmica do Master;
- registrando `maintenance` no ledger;
- mantendo o descarte de partições em *dry-run* — não há `DROP` automático de partição.

## Exceção de backup e decisão de descarte

O backup externo Restic/R2 não foi configurado. Por decisão explícita do responsável, a execução ocorreu sem backup e sem período de observação efetivo.

- A exceção exigiu `AUTH_AUDIT_PARTITION_ALLOW_NO_BACKUP=YES` e foi gravada no ledger como `backupMode: explicit_no_backup`.
- O mecanismo normal de rollback de 48 horas foi deliberadamente dispensado.
- A tabela `ml.auth_audit_legacy_15f92e3d3150` foi removida definitivamente sob lock exclusivo, após confirmar que `ml.auth_audit` era o pai particionado ativo.
- Não há mais rollback de dados para a estrutura anterior sem recuperação externa.

## Verificação atual

Verificação posterior ao descarte:

- `to_regclass('ml.auth_audit_legacy_15f92e3d3150')` = `null`;
- `ml.auth_audit` tem `relkind = 'p'`;
- eventos ativos no pai/partições: `215.847`;
- `seller-ml-web` e `seller-ml-worker`: `healthy`.

## Pendências e acompanhamento

1. Acompanhar logs e operações normais do Seller ML nos próximos dias; qualquer falha de escrita de auditoria deve ser tratada como incidente.
2. Configurar Restic + bucket R2 ou outro repositório externo antes de novas migrações/descarte estrutural.
3. Testar uma restauração de backup antes de considerar qualquer futuro `release-legacy` como procedimento rotineiro.
4. Manter a revisão das regras de `ml.auth_audit_retention_rules` no painel Master: elas determinam quais eventos expirados são removidos na manutenção.
5. Não executar manualmente `DROP TABLE` em partições; o scheduler mantém remoção de partições em simulação até uma decisão operacional específica.

## Referências

- [[VPS e staging]]
- [[Runbook de deploy DACH na VPS]]
- Código operacional: `docs/operations/ml-auth-audit-partition-cutover.md` no repositório Dachbyte.
