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
- Engineering Handbook — **Planned** (a escrever por ondas — só capítulos com conteúdo real agora: Introdução, Filosofia, Specifications, ADR, Versionamento. História/Evolução/Roadmap ficam como stub até haver o que registrar)
- ~~Engineering Bible~~ — descontinuado, ver DP-003.1
- STD-0006 — Engineering Principles — **Planned** (substitui o que seria parte da Bible)

### DP-003.1 — Resolução: Handbook vs. Bible vs. Principles (2026-07-09)

Analisando o conteúdo já definido para Foundation ("artefato que define identidade, propósito, princípios e base estratégica" — STD-0001), o Engineering Bible tal como proposto seria redundante com o que Mission, Vision e Product Constitution já cobrem. Resolução:

- **Engineering Handbook** — mantido. É o índice/navegação único do ecossistema (mapa de Specifications, dependências, roadmap, histórico). Não contém conteúdo completo, só aponta para os artefatos canônicos.
- **Engineering Bible** — descontinuado como artefato próprio. Seu conteúdo pretendido (filosofia narrada) já é coberto por Mission, Vision e Product Constitution.
- **Engineering Principles** — deixa de ser parte do Handbook e passa a ser um Standard (`STD-0006 — Engineering Principles`), já que é uma lista prescritiva de regras, não narrativa.

**Status:** Resolvido

## DP-004 — Delivery Package History

**Objetivo:** preservar o histórico de cada entrega, em vez de sobrescrever um único `EXECUTE.md`/`VALIDATION.md`/`RELEASE_NOTES.md` na raiz.

### Entregáveis

- Pasta `releases/` — **Done**
- `releases/Release-0001.md` (retroativo, cobrindo DP-001, DP-002, FP-001 e o Housekeeping da Fase A) — **Done**
- Convenção: cada nova entrega gera seu próprio `Release-000X.md`; `EXECUTE.md`/`VALIDATION.md`/`RELEASE_NOTES.md` na raiz sempre descrevem o pacote **em andamento**, e são arquivados em `releases/` quando o pacote fecha

**Status:** Done

## DP-005 — Primeira Specification real

**Objetivo:** validar o processo de Specification na prática, com um caso real e imediato.

### Entregáveis

- SPC-0001 — Especificação da Persistência do Workspace (CS-007), cobrindo Server Action, validação Zod, RLS/multi-tenant e critério de aceite — **Done**. Implementação em `cc-commerce-studio` (CS-007) concluída e testada end-to-end.

**Status:** Done

## DP-006 — Governança de Prompts e Scripts

**Objetivo:** dar um local e um formato oficial para prompts de IA e scripts de automação usados no ecossistema, antes de o primeiro Engine de IA (CS-008+) ser construído.

### Entregáveis

- STD-0007 — Prompt & Script Governance Standard (0.1.0, Draft) — **Done**. Cobre local de armazenamento (`features/<engine>/prompts/` no produto, `scripts/` para automação), formato mínimo (front-matter YAML) e metadados obrigatórios. Critérios de avaliação/aprovação ficam explicitamente fora de escopo nesta versão.
- STD-0003 — ajuste na tabela de tipos oficiais, adicionando `PR` (Prompt) e `SCR` (Script) — **Done**.

### Dívida futura registrada

STD-0007 sobe para 0.2.0 (MINOR, expansão compatível, conforme STD-0002) quando o primeiro Engine real (Offer Engine, candidato natural em CS-008+) tiver sua própria Specification — nesse momento, incorporar os critérios reais de avaliação (qualidade, custo, segurança) antes de um prompt entrar em produção.

**Status:** Done (versão 0.1.0) — 0.2.0 depende de CS-008+ ter uma Specification real.

## DP-007 — Specification do módulo Products

**Objetivo:** especificar o próximo módulo do produto (CS-008), com prioridade definida por dependência real de schema, não por preferência.

### Entregáveis

- SPC-0002 — Módulo Products (CS-008) — **Done**. Auditoria confirmou que `products` (tabela + RLS) já existe e já está aplicada desde a migration 002 (CS-007) — sem achado bloqueante. Trabalho restante é só camada de feature + UI.

### Prioridade de módulos (CS-008+) — decisão registrada

Ordem definida por dependência de schema, verificada nas migrations reais (não suposição): Products (schema/RLS prontos) → Brands (exige schema novo + migração aditiva de `brand_id` em `products`) → Offer Engine (consome os dois; primeiro caso real de STD-0007).

**Status:** Done

## DP-008 — Specification do módulo Brands

**Objetivo:** especificar o módulo Brands (CS-009), próximo na ordem confirmada em DP-007.

### Entregáveis

- SPC-0003 — Módulo Brands (CS-009) — **Done**. Auditoria confirmou que não existe tabela `brands` nem coluna `brand_id` em `products` — schema novo necessário, migração de `products` é aditiva (`brand_id` nullable, `on delete set null`), sem regressão em CS-008.

**Status:** Done

## Dependência cruzada

O roadmap de produto (Commerce Studio) é mantido separadamente em `cc-commerce-studio/ROADMAP.md`. Este arquivo cobre apenas o Engineering Framework.

## Histórico de alterações

| Data | Alteração |
|------|-----------|
| 2026-07-07 | Versão inicial (DP-001 a DP-003 planejados) |
| 2026-07-09 | DP-001/DP-002 marcados como concluídos; DP-003 reconciliado com a execução real; DP-004 e DP-005 adicionados |
| 2026-07-10 | DP-006 adicionado: STD-0007 (Prompt & Script Governance Standard) criado, STD-0003 ajustado com tipos PR/SCR |
| 2026-07-10 | DP-005 reconciliado para Done: SPC-0001 confirmada e implementada em CS-007 |
| 2026-07-10 | DP-008 adicionado: SPC-0003 (Brands) escrita, schema novo + migração aditiva em products |
| 2026-07-10 | DP-007 adicionado: SPC-0002 (Products) escrita; prioridade de módulos CS-008+ decidida por dependência de schema |
