---
id: ADR-0001
title: Manter Next.js 14 e React 18 como baseline do Commerce Studio
category: ADR
version: 1.0.0
status: Approved
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Commerce Studio
repository: cc-commerce-studio
created_at: 2026-07-09
updated_at: 2026-07-09
---

# ADR-0001 — Manter Next.js 14 e React 18 como baseline do Commerce Studio

## Status

Approved

## Contexto

A Sprint 001 (Bootstrap do Commerce Studio) congelou como decisão definitiva: Next.js (App Router) + React 19. Na prática, o `create-next-app` executado gerou o projeto com Next.js 14.2.23 e React 18.3.1 — versões que já vinham sendo usadas desde o primeiro commit (`chore: bootstrap Commerce Studio`) e sobre as quais o módulo Workspace inteiro (queries, mutations, actions, componentes) já foi construído e validado em `npm run dev`.

Next.js 14 não oferece suporte estável a React 19 — essa combinação só é válida a partir do Next.js 15. Ou seja, a decisão escrita na Sprint 001 e o código real divergiram desde o início, sem que isso tivesse sido percebido até uma auditoria posterior do projeto.

## Decisão

Manter Next.js 14.2.23 e React 18.3.1 como baseline oficial do Commerce Studio, revogando a menção a React 19 feita na Sprint 001. Next 14 + React 18 é uma combinação estável e já validada em uso real; migrar para Next 15/React 19 agora obrigaria a revalidar toda a estrutura de rotas, Server Actions e dependências já construídas para o módulo Workspace, sem ganho funcional imediato para o MVP.

## Consequências

Positivas: elimina a divergência entre o documento congelado e o código real; nenhum retrabalho no módulo Workspace já entregue; menor risco de regressão nesta fase do projeto.

Negativas: adia o acesso a funcionalidades específicas do React 19 (Actions nativas, `useOptimistic`, `useFormStatus` mais maduro) e do Next 15; a migração terá de acontecer eventualmente, e quanto mais módulos forem construídos sobre o Next 14, maior o custo dessa migração futura.

Risco monitorado: esta decisão deve ser revisitada se algum Engine futuro (por exemplo, o Publishing Engine) exigir de forma inevitável um recurso específico do Next 15/React 19.

## Alternativas Consideradas

1. Migrar imediatamente para Next.js 15 + React 19 — rejeitada nesta fase por introduzir risco de regressão sem benefício imediato, já que o módulo Workspace funciona integralmente sobre Next 14.
2. Manter a ambiguidade (documento diz uma coisa, código faz outra) — rejeitada, por contrariar o objetivo central de rastreabilidade do projeto.

## Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 1.0.0 | 2026-07-09 | Primeira versão, decisão tomada durante a execução da Fase A (housekeeping) | Claude |
