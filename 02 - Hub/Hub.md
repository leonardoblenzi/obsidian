---
type: plataforma
status: ativo
updated: 2026-10-01
---

# Hub

O Hub é a camada transversal da suíte para identidade, acesso, uso de recursos e cobrança. Ele conecta produtos sem exigir que a documentação duplique a implementação de cada aplicação.

O runtime vigente está na VPS em `https://hub.dachbyte.tech`, com PostgreSQL local `dachbyte_hub`. O Worker Cloudflare/Neon é legado desde o cutover de 27/09/2026. Estado detalhado: [[Estado operacional DACH — Hub, VPS e módulos — 2026-10-01]].

## Áreas

- [[Identidade e acesso]]
- [[Cobrança e uso]]
- [[Davantti e Dachbyte]]
- [[Arquitetura compartilhada]]

## Limite desta nota

Os comportamentos confirmados apontam para caminhos do repositório. Qualquer fluxo ainda não verificado fica marcado como `A mapear`.

## Histórico: preparação da migração para VPS (25/09/2026)

Em 25/09/2026, o Hub Cloudflare ainda era a autoridade operacional. A equipe já havia preparado e testado a migração, mas ainda não tinha restaurado o banco final nem publicado o Hub VPS.

Próximas etapas:

1. Configurar Brevo no novo domínio para habilitar e-mails transacionais.
2. Confirmar propagação pública de `hub.dachbyte.tech`.
3. Marcar janela de manutenção para congelar escrita no Hub Cloudflare.
4. Executar snapshot final, restore no PostgreSQL da VPS, migrations e comparação.
5. Subir e validar o Hub VPS antes de trocar o webhook Asaas.
6. Após validação pública, ativar scheduler e consumidores e monitorar o corte.

Runbook e estado detalhado: [[Migração do Hub Cloudflare para VPS — 2026-09-25]].

## Histórico: incidente de administradores (25/09/2026)

Em 25/09/2026, houve relato de falha de operação/login dos administradores. Esta nota não registra a investigação nem a resolução; não tratar o relato antigo como estado atual.

Próxima investigação:

1. Identificar a URL/ambiente afetado e o fluxo exato que falha.
2. Reproduzir com um administrador sem expor senha ou token.
3. Consultar logs e resposta HTTP correspondente.
4. Verificar sessão, hash de senha, permissão administrativa e release recente antes de aplicar correção.
