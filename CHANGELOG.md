# Changelog

Todas as mudanças relevantes deste repositório serão registradas neste arquivo.

## [Unreleased] - 2026-07-10

### Added

- ADR-0001 — Manter Next.js 14 e React 18 como baseline do Commerce Studio.
- ROADMAP.md reconciliado: DP-001 e DP-002 marcados como concluídos, DP-003 reconciliado com a execução real, DP-004 (Delivery Package History) e DP-005 (primeira Specification) adicionados.
- STD-0007 — Prompt & Script Governance Standard (0.1.0, Draft): local, formato e metadados mínimos para prompts e scripts do ecossistema.
- STD-0003 — tipos oficiais `PR` (Prompt) e `SCR` (Script) adicionados à tabela de tipos.
- ROADMAP.md: DP-006 (Governança de Prompts e Scripts) adicionado.

### Fixed

- ROADMAP.md: DP-005 reconciliado para Done (SPC-0001 confirmada e implementada em CS-007).

## [Unreleased] - 2026-07-10 (2)

### Added

- SPC-0003 — Módulo Brands (CS-009): schema novo (`brands`), migração aditiva em `products` (`brand_id` nullable), RLS no mesmo padrão de `products_member_all`.
- ROADMAP.md: DP-008 adicionado.

## [Unreleased] - 2026-07-10 (3)

### Added

- SPC-0004 — Offer Engine (CS-010): schema novo (`offers`), achado sobre ausência de provedor de IA registrado, primeiro Prompt (`PR-0001`).
- STD-0007 expandido para 0.2.0: contrato de entrada/saída, ciclo de vida do prompt, checklist de revisão.
- ROADMAP.md: DP-009 adicionado.

## [Unreleased] - 2026-07-10 (4)

### Added

- ROADMAP.md: DP-010 adicionado — escopo final do MVP decidido pelo usuário (Workspace+Products+Brands+Offer Engine com IA real, STD-0006, Product Constitution). Demais 8 módulos do CS-008+ ficam fora do escopo de "finalizar" por agora.

## [Unreleased] - 2026-07-10 (5)

### Added

- STD-0007 expandido para 0.3.0: critérios quantitativos de custo/latência de referência, tratamento obrigatório de resposta vazia/bloqueada e de falha de rede/cota, campo `model_verified_at`. Baseado na integração real de `PR-0001` com Gemini 3.1 Flash-Lite.

## [Unreleased] - 2026-07-10 (6)

### Added

- STD-0006 — Engineering Principles: 12 princípios prescritivos, cada um citando o caso real do projeto (DP-001 a DP-010) que o originou. Fecha a dívida registrada em DP-003.
- FND-0004 — Product Constitution (ConnectionCyber Commerce Studio): propósito, público-alvo, não escopo permanente, 5 princípios não negociáveis do produto. Fecha a dívida registrada em DP-003.
- `releases/Release-0002.md` — consolidação do fechamento do MVP (DP-003 conclusão, DP-005 a DP-010).

### Decided

- Testes automatizados e CI ficam fora do escopo do MVP — backlog consciente (DP-010, item 7).

## [0.1.0] - 2026-07-07

### Added

- Estrutura inicial do repositório `cc-engineering-framework`.
- README inicial.
- Licença proprietária.
- Roadmap inicial.
- Guia de contribuição.
- Código de conduta.
- Arquivos de padronização do repositório.
- Estrutura inicial de artefatos.
