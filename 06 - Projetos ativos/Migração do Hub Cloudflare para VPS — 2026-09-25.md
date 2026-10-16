---
type: projeto
status: pre-corte
updated: 2026-09-25
---

# Migração do Hub Cloudflare para VPS — 2026-09-25

## Decisão e escopo

O Hub DACH será migrado do Worker Cloudflare `paymentcontrol` com banco Neon para a VPS `srv1971387` (`177.7.52.6`). A finalidade é centralizar o runtime do Hub junto da suíte DACH, mantendo o PostgreSQL e Redis em rede privada Docker e Caddy como único ingresso HTTPS público.

O corte ainda **não ocorreu**. O Hub Cloudflare continua sendo a autoridade ativa para login, acesso, cobrança e webhook Asaas até a conclusão do procedimento controlado.

## Estado confirmado

| Item | Estado em 25/09/2026 |
| --- | --- |
| DACH no GitHub | `main` em `d513ef2`; checkout ativo da VPS sincronizado, sem reinício de containers. |
| Hub no GitHub | `main` e `hub-vps-migration` em `8cb5118`; `payment` permanece a linha Cloudflare e não foi alterada. |
| Checkout do Hub na VPS | `/opt/dachbyte/hub-pagamento`, clone de `main` em `8cb5118`. |
| Stack DACH ativa | Projeto Docker `dachbyte-staging`; nenhum container foi recriado nesta preparação. |
| Rehearsal | Concluído com sucesso em projeto Docker isolado `dachbyte-hub-rehearsal`, loopback `127.0.0.1:18787`, sem tocar o stack ativo. |
| Banco de destino | PostgreSQL interno da VPS; destino planejado `dachbyte_hub`. Ainda não restaurado nem migrado. |
| Hub definitivo | Não iniciado. Scheduler, consumidores e ferramentas de cutover permanecem desligados. |
| DNS | `hub.dachbyte.tech` foi criado na Hostinger como `A → 177.7.52.6`, TTL 300. Servidores autoritativos confirmaram o registro; resolvers públicos podem levar alguns minutos para atualizar cache. |
| Caddy | O `Caddyfile` versionado possui a rota `hub.dachbyte.tech → hub-web:8787`, mas não foi recarregado. |

## Trabalho concluído

### Código e isolamento

- Integração da migração VPS foi promovida para `main` nos dois repositórios após alinhamento de branches e validação.
- No Hub, a branch `payment` continua intocada para preservar o Worker Cloudflare durante a transição.
- O Compose passou a usar isolamento explícito no rehearsal; não compartilha projeto, redes ou volumes com o stack ativo.
- O serviço `hub-web` está definido no Compose. `hub-scheduler` exige o profile explícito `hub-scheduler`; migrations e ferramentas de corte exigem profiles operacionais próprios.

### Rehearsal aprovado

- Migração de ensaio foi executada contra a origem Neon em um banco isolado da VPS.
- O relatório do rehearsal retornou `ok: true`, sem falhas de contagem, schema, migrations ou dados críticos de Magalu.
- O runtime do Hub no loopback respondeu ao healthcheck e o login do portal administrativo foi validado.
- O rehearsal continua separado; não é o Hub definitivo e não recebe tráfego público.

### Preparação do ambiente definitivo

Arquivos criados em `/opt/dachbyte/repository/infra/env/`, todos com permissão `600` e fora do Git:

| Arquivo | Finalidade | Estado |
| --- | --- | --- |
| `hub-runtime.env` | Runtime do Hub definitivo | Banco local futuro, URL pública, enforcement estrito, token interno, administração, Asaas e segredo de renovação preenchidos; Brevo e integrações opcionais seguem pendentes. |
| `hub-migrate.env` | Migration do Hub para `dachbyte_hub` | Preparado; não executado. |
| `hub-cutover.env` | Orquestração do corte | Origem Neon, destino, IP, URL pública e ID do webhook registrados; confirmações perigosas permanecem `NO`. |

Notas de segurança:

- Nenhum valor de senha, token, API key ou URL de banco deve ser copiado para o vault, Git ou chat.
- A chave Asaas contém caractere `$`; em `hub-runtime.env` ela foi escapada para o Docker Compose entregar o valor literal ao container.
- O segredo de renovação foi criado novo no VPS. Intenções de renovação criadas com um segredo anterior não devem ser consideradas reutilizáveis após o corte.

### Asaas

- `ASAAS_API_KEY` e `ASAAS_WEBHOOK_TOKEN` foram configurados no arquivo protegido do runtime.
- Webhook atual identificado: `880b391f-8557-4afe-bc87-b04067d2e150` (`Payment Control - Produção`).
- O webhook está ativo, sem fila interrompida, e **continua apontando para o Worker Cloudflare**:
  `https://paymentcontrol.davantti-suite.workers.dev/v1/public/webhooks/payment?provider=asaas`.
- O ID foi salvo em `hub-cutover.env` apenas para permitir a troca controlada posterior.
- Não trocar URL, token, estado ou eventos do webhook antes de o Hub VPS estar saudável publicamente.

## Pendências antes de iniciar o Hub definitivo

### Incidente a investigar antes do corte

- Foi reportado em 25/09/2026 que administradores do Hub não estão funcionando normalmente. A falha ainda não foi reproduzida nem classificada por ambiente. Como o Hub Cloudflare segue ativo e o Hub VPS não iniciou, não atribuir este incidente à migração sem evidência.
- Antes da janela de corte, reproduzir o fluxo afetado, registrar URL/resposta HTTP e logs, e validar credencial, sessão e permissão administrativa. Não promover o Hub VPS com um incidente de acesso administrativo não compreendido.

### Obrigatórias

1. Confirmar resolução pública de `hub.dachbyte.tech` para `177.7.52.6`.
2. Configurar Brevo no novo domínio: `BREVO_API_KEY` e `BREVO_SENDER_EMAIL` devem referenciar uma chave e remetente verificados. Sem isso, convites, recuperação de senha e outros e-mails transacionais não funcionam.
3. Executar preflight do corte e revisar que todas as confirmações continuam bloqueadas (`NO`).
4. Definir e comunicar uma janela de manutenção para congelar escrita comercial no Hub Cloudflare.

### Recomendadas antes do corte

1. Verificar o certificado HTTPS emitido por Caddy depois de o registro DNS estar propagado.
2. Validar novamente que o checkout ativo DACH continua na `main` prevista e que os serviços atuais estão saudáveis.
3. Confirmar mecanismo de backup: o backup Restic externo continua pendente. O snapshot final do Hub é obrigatório no procedimento; a ausência de Restic deve ser uma exceção consciente documentada.
4. Revisar se há intents de renovação pendentes no Hub Cloudflare, pois o segredo de renovação do VPS é novo.

## Procedimento de corte aprovado somente durante janela

O script `infra/hub-cutover/run-cutover.sh` deve ser usado como orquestrador; não executar comandos isolados de restore/migration fora do runbook.

1. **Preflight:** validar rehearsal aprovado, configuração Caddy, target local, variáveis e saúde da VPS.
2. **Freeze comercial:** suspender novas alterações/cobranças no Hub Cloudflare e confirmar scheduler antigo desligado.
3. **Snapshot final:** gerar dump final da origem Neon e preservar o artefato do corte.
4. **Restore e migration:** restaurar para `dachbyte_hub` no PostgreSQL privado e executar `hub-migrate`.
5. **Comparação:** conferir contagens, schema, migrations e dados comerciais/Magalu antes de expor o novo Hub.
6. **Subida fechada:** iniciar somente `hub-web`; validar healthcheck interno e login administrativo.
7. **Publicação:** recarregar Caddy, validar HTTPS e smoke público em `https://hub.dachbyte.tech`.
8. **Webhook:** somente após o endpoint público saudável, atualizar o webhook Asaas identificado para `https://hub.dachbyte.tech/webhooks/asaas`.
9. **Ativação final:** iniciar scheduler e consumidores explicitamente, retomar o fluxo comercial e monitorar logs/eventos.

## Rollback

Enquanto o corte não é concluído, o Worker Cloudflare, o banco Neon e a URL antiga do webhook permanecem intactos.

Se uma etapa falhar antes da troca de webhook, interromper o Hub VPS e manter o Cloudflare ativo. Se falhar depois da troca, reverter a URL do webhook para o Worker Cloudflare, pausar o Hub VPS e usar o snapshot final para análise; não repetir migrations ou reprocessar cobrança sem revisar a comparação e a idempotência.

## Relações

- [[Hub]]
- [[VPS e staging]]
- [[Inventário de variáveis — Hub e staging]]
- [[Runbook de deploy DACH na VPS]]
- [[Migração para VPS]]
