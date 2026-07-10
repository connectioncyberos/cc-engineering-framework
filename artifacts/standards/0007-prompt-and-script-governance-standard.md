---
id: STD-0007
title: Prompt & Script Governance Standard
category: Standard
version: 0.1.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Engineering Framework
repository: cc-engineering-framework
created_at: 2026-07-10
updated_at: 2026-07-10
---

# STD-0007 — Prompt & Script Governance Standard

## 1. Objetivo

Definir onde prompts de IA e scripts de automação do ecossistema ConnectionCyber são armazenados, seu formato mínimo e os metadados obrigatórios. Esta versão (0.1.0) estabelece apenas a base estrutural — local, formato, identificação. Os critérios de avaliação e aprovação de um prompt antes de "entrar no motor" (produção) ficam para a versão 0.2.0, quando existir um Engine real para validar esses critérios na prática (ver seção 7).

## 2. Escopo

- Prompts de IA usados pelos Engines de produto (Offer Engine, Creative Engine, Video Script Engine, etc.), definidos em `cc-commerce-studio/ROADMAP.md` (CS-008+).
- Scripts de manutenção, automação e apoio ao desenvolvimento dos repositórios do ecossistema.

## 3. Não Escopo (nesta versão)

- Critérios de avaliação de qualidade, custo, segurança e aprovação de um prompt antes de produção.
- Ferramentas de teste automatizado de prompts (prompt evals).
- Processo de aprovação formal (quem assina, quando).

Esses itens são adiados intencionalmente para a versão 0.2.0, para evitar prescrever regras sem um caso real que as valide — mesmo critério já aplicado à Specification (SPC-0001 só foi escrita quando havia um caso concreto).

## 4. Conteúdo Principal

### 4.1 Tipos de artefato

Adicionados à tabela de tipos oficiais do STD-0003:

| Código | Tipo |
|-------|------|
| PR | Prompt |
| SCR | Script |

### 4.2 Local de armazenamento

- Prompt específico de uma feature/Engine: `features/<nome-do-engine>/prompts/`, dentro do repositório do produto (`cc-commerce-studio`) — consistente com a arquitetura de fatia vertical já em uso (`features/workspace/...`).
- Prompt compartilhado entre múltiplos Engines: candidato natural a `cc-shared-framework` (repositório reservado para código reutilizável, ver STD-0004). Decisão adiada até existir um segundo Engine real que confirme a necessidade — evita mover código para um repositório vazio de forma especulativa.
- Script de manutenção/automação: `scripts/` na raiz do repositório do produto (pasta já existente em `cc-commerce-studio`, hoje vazia).

### 4.3 Formato do arquivo de Prompt

Arquivo Markdown (`.md`) com front-matter YAML contendo, no mínimo:

```yaml
id: PR-NNNN
title: <nome do prompt>
target_model: <ex. Gemini, Nano Banana, Veo>
objective: <objetivo do prompt em uma frase>
version: 0.1.0
status: Draft
```

### 4.4 Versionamento

Git é a fonte de verdade — histórico, diff e rollback nativos, sem ferramenta adicional nesta fase. O versionamento semântico do conteúdo do próprio Standard segue o STD-0002.

## 5. Dependências

- STD-0001 — Documentation & Artifact Standard
- STD-0002 — Versioning Standard
- STD-0003 — Naming Convention Standard
- STD-0004 — Repository Organization Standard

## 6. Critérios de Aprovação (desta versão do Standard)

Permanece em Draft até ser usado por pelo menos um Engine real (primeiro caso previsto: Offer Engine, CS-008+).

## 7. Roadmap de evolução

- **0.2.0** — quando o primeiro Engine tiver sua própria Specification, expandir com os critérios reais de avaliação/aprovação (qualidade, custo, segurança) antes de um prompt entrar em produção.
- **1.0.0** — quando aprovado.

## 8. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft — local, formato e metadados mínimos | Joaquim Mario Soares Coelho |
