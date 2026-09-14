---
type: projeto
status: publicado-em-staging
data: 2026-09-14
linha: [seller, business]
---

# Landings Seller e Business — 2026-09-14

## Objetivo

Elevar as landings de Seller e Business para uma jornada comercial coerente: a pessoa entende a família de produto, reconhece o módulo em que está e consegue experimentar uma decisão antes de seguir para contato ou acesso. A meta não foi simular o produto real nem usar dados de clientes; foi comunicar a utilidade de cada operação com clareza e personalidade.

## Direção visual e de UX

- Navegação e hierarquia alinhadas entre as duas famílias: faixa própria de navegação, hero curto, conteúdo longo alinhado à esquerda e CTA explícito.
- Business preserva o caráter analítico e integrado, com acentos ciano (`#30d6dc`) e verde (`#2dd9a8`). Seller tem linguagem operacional própria, com laranja (`#ff8a3f`) e azul logístico (`#6c93ff`).
- Fundo escuro `#090b10`, painéis `#131a27`, conteúdo em perspectiva e movimento condicionado à preferência de redução de movimento.
- Títulos mantêm Exo 2 quando já adotado; controles priorizam tipografia de sistema e números tabulares para leitura de valores.
- Nenhuma demonstração depende de API, conta ou dado real. Os cenários são fictícios e a fórmula exibida no Seller torna a estimativa verificável.

## Ajustes entregues

### Experiência compartilhada

- Criados `public/brand/dachbyte/landing-experience.css` e `landing-experience.js`.
- O componente monta shell de navegação por família e módulo, estados de foco, CTAs de contato e interações responsivas apenas nas landings.
- Business recebeu um cenário integrado por etapas, consequência da decisão e reinício. Seller recebeu demos distintas por contexto:
  - margem/preço/desconto/Ads para Mercado Livre;
  - fila selecionável, avanço e reset para Shopee;
  - mapa de exceções selecionáveis para Rastreio.

### Jornadas e conteúdo

- Seller: integração nas landings geral, Mercado Livre, Shopee e Rastreio; cada rota tem demonstração relacionada à sua rotina.
- Business: integração nas landings Business, Core, Stock e Chat; as páginas preservam o produto como destino e não confundem demonstração com aplicação autenticada.
- Criada a landing pública **DACHBYTE Price** em `https://staging.dachbyte.tech/business/price`, com retorno para Business e entrada explícita na aplicação.
- CTAs sem destino foram corrigidos; marcas antigas e alegações públicas que não eram demonstráveis foram removidas nos trechos abrangidos.

## Marca e compatibilidade

- A superfície pública usa **DACHBYTE Price**; "Volt" não aparece como marca comercial nessa jornada.
- `/volt-price` permanece como rota técnica legada para não interromper links existentes, sessão e callbacks OAuth.
- `/business/price` deixou de redirecionar para `/volt-price` e passou a servir a landing. `/dach/business/price` continua como adaptador técnico para a rota legada.
- A rota pública foi declarada no processo que efetivamente executa o portal na VPS: `apps/business/product-server.cjs`. A tentativa de atender a rota somente em `apps/business/app.js` não alcançava o container `business-portal`, que usa `product-server.cjs` com `DACHBYTE_PRODUCT=portal`.

## Publicação em staging

- Repositório operacional: `git@github.com:leonardoblenzi/dachbyte.git`, branch `main`.
- Commit publicado na VPS: `ce0cfaaa451f7a3c25db4b94b2d84857b5c78dfb`.
- Serviços reconstruídos no corte visual: `gateway`, `business-portal`, `business-core`, `business-stock` e `business-chat`; a correção final de Price recriou somente `business-portal`.
- O checkout da VPS permaneceu com os backups não versionados em `infra/env/` preservados. Não houve alteração de banco, migration, segredo, OAuth ou DNS.

## Verificação realizada

- Testes de experiência, rotas canônicas, fronteira de produtos e contratos de VPS/arquitetura: aprovados.
- Validação externa sem seguir redirects:
  - `https://staging.dachbyte.tech/business/price` → 200 e shell `Price`;
  - `https://staging.dachbyte.tech/business` → 200 e experiência Business;
  - `https://staging.dachbyte.tech/seller/mercado-livre` → 200 e assets Seller;
  - asset compartilhado `landing-experience.js` → 200.
- Na VPS, `gateway`, `business-portal`, `business-core`, `business-stock` e `business-chat` estavam `healthy` após o deploy.

## Commits relacionados

- `f12359f` — experiência unificada Seller/Business.
- `2d58ad6`, `a91b692` e `18a025d` — investigação e cobertura de rota Price no host antigo.
- `a4760cc` — reserva `/business/price` para a landing pública.
- `ce0cfaa` — rota no `product-server.cjs`, processo realmente executado pelo portal em staging.

## Próximos passos

1. Fazer revisão visual manual em desktop e 390 px: teclado, foco, overflow, contraste e preferência por movimento reduzido.
2. Medir conversão por CTA antes de aumentar conteúdo ou adicionar mais efeitos 3D.
3. Planejar a aposentadoria de aliases técnicos apenas após telemetria, testes OAuth e plano de rollback aprovado.

## Relações

- [[Business]]
- [[Seller e operações]]
- [[Marca e compatibilidade]]
- [[VPS e staging]]
- [[Compatibilidade durante a migração]]
