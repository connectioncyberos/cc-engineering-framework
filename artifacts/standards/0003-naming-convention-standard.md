---
id: STD-0003
title: Naming Convention Standard
category: Standard
version: 0.1.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: ChatGPT
project: ConnectionCyber Engineering Framework
repository: cc-engineering-framework
created_at: 2026-07-08
updated_at: 2026-07-10
---

# STD-0003 — Naming Convention Standard

## 1. Objetivo

Definir o padrão oficial de nomenclatura para arquivos, diretórios, artefatos, IDs, repositórios, commits, branches e componentes do ecossistema ConnectionCyber.

## 2. IDs de artefatos

Formato: `TIPO-NNNN`

Exemplos: `STD-0001`, `FND-0001`, `SPC-0001`, `ADR-0001`.

## 3. Tipos oficiais

| Código | Tipo |
|-------|------|
| STD | Standard |
| FND | Foundation |
| SPC | Specification |
| ADR | Architecture Decision Record |
| POL | Policy |
| TMP | Template |
| GDE | Guide |
| PBK | Playbook |
| RDM | Roadmap |
| JNL | Journal |
| BLP | Blueprint |
| MDL | Model |
| PR | Prompt |
| SCR | Script |

## 4. Arquivos Markdown

Formato: `NNNN-nome-do-artefato.md`

Exemplo: `0001-documentation-artifact-standard.md`

## 5. Repositórios

Repositórios do ecossistema devem preferir prefixo `cc-`, como `cc-engineering-framework`, `cc-shared-framework` e `cc-commerce-studio`.

## 6. Commits

Utilizar Conventional Commits, como `docs: add versioning standard`.

## 7. Dependências

- STD-0001 — Documentation & Artifact Standard
- STD-0002 — Versioning Standard

## 8. Histórico de alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-08 | Primeira versão Draft | Joaquim Mario Soares Coelho |
| 0.1.0 | 2026-07-10 | Adicionados tipos PR (Prompt) e SCR (Script) à tabela de tipos oficiais (ver STD-0007) | Joaquim Mario Soares Coelho |
