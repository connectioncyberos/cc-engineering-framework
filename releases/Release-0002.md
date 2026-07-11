# Release 0002 — MVP do ConnectionCyber Commerce Studio (Workspace → Products → Brands → Offer Engine com IA real)

## Período

2026-07-10

## Pacotes cobertos

- DP-003 (conclusão) — Product Constitution escrito, Engineering Principles virou Standard
- DP-005 — SPC-0001 confirmada e implementada (CS-007)
- DP-006 — STD-0007 criado (0.1.0)
- DP-007 — SPC-0002 (CS-008)
- DP-008 — SPC-0003 (CS-009)
- DP-009 — SPC-0004 (CS-010), STD-0007 expandido para 0.2.0
- DP-010 — Escopo final do MVP, integração real de IA, STD-0007 0.3.0, STD-0006, FND-0004, decisão sobre testes/CI
- CS-007 a CS-010 (`cc-commerce-studio/ROADMAP.md`) — Workspace, Products, Brands, Offer Engine

## Artefatos entregues

**Governança (`cc-engineering-framework`):**

- SPC-0001 — Persistência do Workspace
- SPC-0002 — Módulo Products
- SPC-0003 — Módulo Brands
- SPC-0004 — Offer Engine
- STD-0006 — Engineering Principles (12 princípios, cada um citando o caso real que o originou)
- STD-0007 — Prompt & Script Governance Standard (0.1.0 → 0.3.0: local/formato → contrato de entrada/saída, ciclo de vida, checklist → critérios quantitativos reais de custo/latência/segurança)
- FND-0004 — Product Constitution (ConnectionCyber Commerce Studio)
- STD-0003 — ajustado com tipos `PR` (Prompt) e `SCR` (Script)

**Produto (`cc-commerce-studio`):**

- Autenticação real (Supabase Auth + `@supabase/ssr`)
- CS-007 — Workspace: CRUD completo, RLS multi-tenant real
- CS-008 — Products: CRUD completo, reaproveitando schema/RLS já existentes
- CS-009 — Brands: schema novo, migração aditiva em `products` (`brand_id`), tom de voz
- CS-010 — Offer Engine: CRUD completo, `PR-0001` (primeiro Prompt do ecossistema), integração real com Gemini 3.1 Flash-Lite

## Achados e correções ao longo do pacote

- Bug de RLS no insert de `workspaces`: `RETURNING` pedido antes de o vínculo em `workspace_members` existir. Corrigido gerando `id` no cliente e removendo `.select().single()` do insert de `workspaces` (não se aplica a `products`/`brands`/`offers`, cuja política não depende de uma linha criada na mesma transação).
- Erro de encoding num comentário SQL (acento em "módulo") corrompeu o parser do editor SQL do Supabase — corrigido removendo acentos de comentários de migration.
- Modelo `gemini-2.5-flash-lite`, escolhido na SPC-0004, foi descontinuado para novos usuários entre a escrita do `PR-0001` e o primeiro teste real — corrigido para `gemini-3.1-flash-lite`. Motivou o campo `model_verified_at` no STD-0007 0.3.0.
- `generateOfferCopy()`/`generateOfferCopyAction()` ajustados para tratar resposta vazia/bloqueada por filtro de segurança e falhas de rede/cota, conforme exigido pelo STD-0007 §4.8.

## Decisão de escopo (DP-010)

MVP fechado = Workspace + Products + Brands + Offer Engine com IA real. Os 8 módulos restantes do `cc-commerce-studio/ROADMAP.md` (Landing Page, Creative, Video Script, Marketplace, Email/WhatsApp, Publishing, Analytics, Quality Engine + Prompt Lab) ficam fora do escopo desta rodada, documentados como próximas fases. Testes automatizados e CI ficam como backlog consciente, não pendência esquecida.

## Impacto

Fecha o primeiro ciclo completo do produto: da fundação (Workspace) ao primeiro Engine de IA real em produção, com toda a governança (Specification, Standard de Prompt, Princípios, Constituição do Produto) criada a partir de casos reais, não especulação — nenhum artefato de governança foi escrito antes de existir um caso concreto que o justificasse.

## Próximo pacote

Em aberto — os 8 módulos de produto fora do escopo do MVP (DP-010), a serem retomados quando houver decisão do usuário para continuar expandindo o produto.
