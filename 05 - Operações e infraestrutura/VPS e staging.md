---
type: operacoes
status: staging-pendente
---

# VPS e staging

## Fatos verificados

- Caddy é a entrada HTTPS planejada.
- Produtos executam em serviços e containers separados.
- PostgreSQL e Redis devem permanecer na rede Docker privada.
- O plano inclui backup externo e restauração validada.

## Próximos passos de staging

1. Disponibilizar VPS e acesso SSH; validar capacidade e Docker.
2. Preencher ambientes e restaurar schemas sanitizados em staging.
3. Construir imagens e subir a stack de staging.
4. Configurar backup criptografado, retenção e testar restauração.
5. Somente em etapa aprovada: cópia de banco, DNS, callbacks OAuth e monitoramento de tráfego.

## Bloqueado até aprovação específica

- DNS e tráfego de produção.
- Dados de produção.
- Callbacks OAuth e webhooks externos.
- Desligamento de Render e Neon.

Fonte: `README-VPS.md` e `docs/operations/dachbyte-vps-staging-runbook.md`.
