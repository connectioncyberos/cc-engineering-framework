---
id: POL-0001
title: Execution Handoff Policy
category: Policy
version: 1.0.0
status: Approved
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Engineering Framework
repository: cc-engineering-framework
created_at: 2026-07-09
updated_at: 2026-07-09
---

# POL-0001 — Execution Handoff Policy

## 1. Objetivo

Definir como qualquer trabalho que dependa de execução fora do alcance direto do assistente (comandos de terminal, Git, GitHub, Supabase) é entregue ao usuário, para que nenhuma etapa fique implícita ou seja perdida entre a atualização de arquivo e a ação manual necessária.

## 2. Escopo

Aplica-se a toda sessão de trabalho em que o assistente edita arquivos em um repositório sob controle de versão, mas não tem acesso direto a terminal, Git, GitHub ou banco de dados.

## 3. Regra

Toda vez que uma atualização de arquivo(s) exigir uma ação de Git/GitHub (add, commit, push) ou outro comando manual para ser efetivada, o assistente deve apresentar os comandos exatos — pasta, arquivo, comando, resultado esperado — imediatamente na mesma resposta em que o processo foi executado. Nunca esperar o usuário perguntar "o que eu rodo agora".

Isso estende a RO-001 (nenhuma instrução sem contexto completo: objetivo, repositório, pasta, arquivo, comando, validação esperada), fixando que a entrega dos comandos acontece no mesmo turno da atualização, não depois.

## 4. Não Escopo

Não define a estratégia de branching, o padrão de commits (ver STD-0002/STD-0003) nem quando uma mudança deve virar ADR.

## 5. Dependências

- STD-0001 — Documentation & Artifact Standard
- ADR-0001 — decisão registrada seguindo esta mesma disciplina de handoff

## 6. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 1.0.0 | 2026-07-09 | Primeira versão, formalizando RO-001 e a extensão pedida pelo usuário | Joaquim Mario Soares Coelho |
