---
type: projeto
status: em-andamento
area: identidade-e-acesso
---

# Provisionamento Volt Core via Hub

## Objetivo

Fazer do Hub `paymentcontrol` a fonte de verdade de empresa, usuário, acesso e convite do Volt Core. O Core continua sendo responsável pelas permissões operacionais e telas internas de cada empresa.

## Fluxo aprovado

1. Master cria empresa no Volt Core.
2. Core chama o Hub com nome, documento e chave de idempotência.
3. Hub reserva/cria o tenant, ativa `volt_core` e retorna o identificador canônico.
4. Core cria o espelho operacional com esse mesmo identificador global.
5. Master cria usuário dentro de uma empresa selecionada.
6. Core sincroniza usuário, vínculo, acesso pendente e convite no Hub antes de gravar o espelho local.
7. Usuário aceita o convite, define senha global e entra apenas se empresa, produto, usuário e acesso estiverem válidos.

## Regras de bloqueio

- Empresa inativa ou vencida bloqueia tudo.
- Produto `volt_core` não instalado ou revogado bloqueia o Core.
- Usuário inativo ou sem permissão explícita bloqueia o Core.
- O master técnico de bootstrap é exceção administrativa; o fluxo normal não cria usuários sem empresa Hub.

## Implementado

- Hub publicado no Worker `paymentcontrol` pelo commit `cbc65bc` (`Provision Volt Core identities through Hub`), reconciliado na branch Git `payment` pelo commit `2973136` em 14/09/2026.
- Endpoint idempotente de empresa no Hub e persistência da operação em `internal_identity_provisioning_operations`.
- Migration `053_internal_identity_provisioning.sql` aplicada no fluxo Neon em 11/09/2026; é aditiva e não remove dados. A auditoria de 14/09 confirmou a migration e a tabela `internal_identity_provisioning_operations` na branch piloto ativa `paymentcontrol-pilot-20260910`.
- DACH publicado na branch `dach` pelos commits `101ec5f3` (integração Hub-first) e `1b812ca0` (empresa explícita no convite).
- `business-core` reconstruído e recriado na VPS de staging; o container está saudável e confirma o cliente Hub-first e as variáveis estritas sem expor o token.
- Cliente interno do Core com autenticação Bearer, timeout e mensagens seguras.
- Criação de empresa Hub-first e usuário/convidado Hub-first.
- Modal master exige a seleção explícita da empresa; não assume a primeira empresa da lista.
- Validações locais: Hub `58/58` e TypeScript aprovado; Core `236/236`; build do Core aprovado durante a reconstrução da imagem.

### Reconciliação Git × Neon — 14/09/2026

- Foi identificada uma divergência de histórico: o banco piloto já possuía a migration 053, mas a branch Git `payment` não continha o commit de origem `cbc65bc` nem a migration.
- A correção foi um cherry-pick exclusivo de `cbc65bc` para `payment`, resultando em `2973136`. Não foi feito merge de `dev` e nenhuma migration foi reaplicada no banco.
- A validação posterior passou com `58/58` testes de identidade e TypeScript sem erros.

## Evidência integrada de staging

- Empresa sintética `QA Hub Volt Core Rebuild 20260911` criada pelo painel master do DACH Core.
- O Hub registrou o mesmo tenant canônico, com `access_state=active`, `tenant_status=active` e produto `volt_core=active`.
- Usuário sintético criado explicitamente nessa empresa: existe no Hub com vínculo ativo, produto ativo, `module_access_status=pending`, convite `pending` e `invite_kind=module_user`.
- O estado `pending` é esperado antes do usuário aceitar o convite e definir a senha global.
- A primeira tentativa visual foi feita antes do reload do navegador após a reconstrução e por isso ainda carregava o bundle antigo. Depois do reload, o modal exibiu corretamente `Selecione a empresa`.

## Próximos passos

1. Testar o replay da criação com a mesma chave de idempotência, garantindo que não cria um segundo tenant.
2. Abrir o convite de QA, aceitar e definir a senha global. A submissão final da troca de senha deve ser feita pelo usuário no navegador.
3. Autenticar o usuário convidado no DACH Core e confirmar a ativação do acesso `volt_core` após a senha.
4. Usando apenas a empresa de QA, validar visualmente: empresa inativa, produto `volt_core` pausado/revogado e usuário inativo. Cada bloqueio deve impedir o Core com motivo compreensível.
5. Reativar o cenário de QA ao fim e conferir que a volta respeita o estado próprio do produto e do usuário.
6. Conferir auditoria e correlação local: espelho Core deve manter `tenant_global_id` e `user_global_id` retornados pelo Hub.
7. Após o Volt Core estar aprovado, repetir o mesmo padrão para Seller ML, Shopee e Rastreio; não liberar os módulos Business ainda não integrados.

## Estado de ambientes

- Hub de integração ativo: `https://paymentcontrol.davantti-suite.workers.dev`.
- DACH staging integrado: `https://staging.dachbyte.tech/core/app`.
- A validação atual é exclusivamente em staging e com registros de QA. Não houve alteração de Render, DNS, callbacks OAuth ou banco de produção.

## Referências

- `C:\Users\USER\Documents\Projetos\hub pagamento\src\modules\identity\internalRoutes.ts`
- `C:\Users\USER\Documents\Projetos\hub pagamento\src\modules\identity\service.ts`
- `C:\Users\USER\Documents\Projetos\hub pagamento\sql\053_internal_identity_provisioning.sql`
- `C:\Users\USER\Documents\Projetos\davantti\apps\business\core\src\modules\auth\hubProvisioningClient.js`
- `C:\Users\USER\Documents\Projetos\davantti\docs\superpowers\plans\2026-09-11-volt-core-hub-company-user-sync.md`
