---
id: STD-0004
title: Repository Organization Standard
category: Standard
version: 0.1.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: ChatGPT
project: ConnectionCyber Engineering Framework
repository: cc-engineering-framework
created_at: 2026-07-08
updated_at: 2026-07-08
---

# STD-0004 — Repository Organization Standard

## 1. Objetivo

Definir o padrão de organização de repositórios do ecossistema ConnectionCyber.

## 2. Repositórios iniciais

| Repositório | Finalidade |
|------------|------------|
| cc-engineering-framework | Conhecimento, governança, standards e artefatos. |
| cc-shared-framework | Código reutilizável, SDKs, UI, auth, clients e utilities. |
| cc-commerce-studio | Produto SaaS Commerce Studio. |

## 3. Estrutura base do cc-engineering-framework

```text
cc-engineering-framework/
  README.md
  LICENSE
  CHANGELOG.md
  ROADMAP.md
  CONTRIBUTING.md
  CODE_OF_CONDUCT.md
  artifacts/
    standards/
    foundations/
    specifications/
    policies/
    templates/
    adr/
    roadmaps/
    playbooks/
    guides/
    journals/
  assets/
  .github/
  .vscode/
```

## 4. Desenvolvimento local

Estrutura recomendada:

`C:\Projetos\connectioncyber`

## 5. Dependências

- STD-0003 — Naming Convention Standard

## 6. Histórico de alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-08 | Primeira versão Draft | Joaquim Mario Soares Coelho |
