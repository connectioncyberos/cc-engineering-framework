# Roadmap — ConnectionCyber Engineering Framework

## DP-001 — Repository Bootstrap

**Objetivo:** inicializar o repositório e sua estrutura base.

### Entregáveis

- README.md
- LICENSE
- CHANGELOG.md
- ROADMAP.md
- CONTRIBUTING.md
- CODE_OF_CONDUCT.md
- .gitignore
- .editorconfig
- .gitattributes
- Estrutura inicial de diretórios
- Templates iniciais do GitHub

**Status:** Done

## DP-002 — Engineering Standards

**Objetivo:** criar os primeiros padrões oficiais do framework.

### Entregáveis

- STD-0001 — Documentation & Artifact Standard
- STD-0002 — Versioning Standard
- STD-0003 — Naming Convention Standard
- STD-0004 — Repository Organization Standard
- STD-0005 — Markdown Style Guide

**Status:** Done

## DP-003 — Foundation Artifacts

**Objetivo:** criar os artefatos fundacionais do programa.

### Reconciliação (2026-07-09)

O plano original previa FND-0001 Project Charter, FND-0002 Engineering Bible, FND-0003 Engineering Handbook, FND-0004 Product Vision, FND-0005 Product Constitution. A execução real seguiu ordem diferente: Charter e Vision foram entregues, e Mission foi criado sem estar no plano original. A numeração de arquivo segue STD-0003 (ordem de criação dentro da pasta `foundations/`), não a posição original do plano.

### Entregáveis

- `0001-project-charter.md` — **Done**
- `0002-mission.md` — **Done** (não previsto no plano original; adicionado por necessidade prática)
- `0003-vision.md` — **Done** (equivale ao antigo FND-0004 do plano original)
- Product Constitution — **Planned**
- Engineering Bible — **Planned** (bloqueado por DP-003.1)
- Engineering Handbook — **Planned** (bloqueado por DP-003.1; a escrever por ondas — só capítulos com conteúdo real agora: Introdução, Filosofia, Specifications, ADR, Versionamento. História/Evolução/Roadmap ficam como stub até haver o que registrar)

### DP-003.1 — Pré-requisito antes de Bible/Handbook

Definir a fronteira exclusiva entre Engineering Handbook (índice/navegação), Engineering Bible (filosofia narrada) e Engineering Principles (lista prescritiva) antes de escrever qualquer um dos três, para não duplicar conteúdo.

**Status:** In Progress

## DP-004 — Delivery Package History

**Objetivo:** preservar o histórico de cada entrega, em vez de sobrescrever um único `EXECUTE.md`/`VALIDATION.md`/`RELEASE_NOTES.md` na raiz.

### Entregáveis

- Pasta `releases/`
- `releases/Release-0001.md` (retroativo, cobrindo DP-001 a DP-003)
- Convenção: cada nova entrega gera seu próprio `Release-000X.md`

**Status:** Planned

## DP-005 — Primeira Specification real

**Objetivo:** validar o processo de Specification na prática, com um caso real.

### Entregáveis

- SPC-0001 (candidata: especificação do Offer Engine ou do módulo de Publishing multi-plataforma do Commerce Studio)

**Status:** Planned

## Dependência cruzada

O roadmap de produto (Commerce Studio) é mantido separadamente em `cc-commerce-studio/ROADMAP.md`. Este arquivo cobre apenas o Engineering Framework.

## Histórico de alterações

| Data | Alteração |
|------|-----------|
| 2026-07-07 | Versão inicial (DP-001 a DP-003 planejados) |
| 2026-07-09 | DP-001/DP-002 marcados como concluídos; DP-003 reconciliado com a execução real; DP-004 e DP-005 adicionados |
