---
id: SPC-0001
title: Persistência Real do Workspace (CS-007)
category: Specification
version: 0.1.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Commerce Studio
repository: cc-commerce-studio
created_at: 2026-07-10
updated_at: 2026-07-10
---

# SPC-0001 — Persistência Real do Workspace (CS-007)

## 1. Objetivo

Especificar o comportamento esperado da persistência real do módulo Workspace: criação, listagem, atualização e remoção de Workspaces gravados no Supabase, com isolamento por usuário autenticado (multi-tenant), antes de a Sprint CS-007 avançar para código.

## 2. Escopo

- Tabela `workspace_members` vinculando usuário autenticado a Workspace, com papel (`owner`/`member`).
- Políticas de RLS no Supabase garantindo que um usuário só veja e altere Workspaces aos quais pertence.
- Server Action real (`"use server"`) para criação de Workspace, usando cliente Supabase com contexto de sessão (não o cliente anônimo atual).
- Validação com Zod já existente (`workspace.schema.ts`) ligada ao formulário.
- `WorkspaceForm` funcional (hoje o botão é `type="button"` e o `<form>` não tem `action` nem `onSubmit` — não envia nada).
- Listagem dinâmica ligando `list-workspaces.ts` à UI (hoje a lista não é renderizada em nenhuma página).
- CRUD completo ponta a ponta (create/read/update/delete).

## 3. Não Escopo

- Convite de outros membros para um Workspace existente.
- Papéis granulares além de `owner`/`member`.
- Troca de contexto entre múltiplos Workspaces (workspace switcher).
- Interface de login/cadastro completa (fora do mínimo necessário para autenticação funcionar).

## 4. Conteúdo Principal

### 4.1 Achado bloqueante identificado nesta auditoria

Não existe autenticação implementada no projeto: `lib/auth/` está vazio (só `.gitkeep`), o pacote `@supabase/ssr` não está instalado, e `lib/supabase/client.ts` usa apenas a chave anônima, sem contexto de sessão. Isso significa que `workspace_members` vinculado a `auth.uid()` e políticas de RLS baseadas em usuário logado **não funcionam sem autenticação real** — toda a cadeia dependeria de um usuário que hoje não existe no sistema.

Duas rotas possíveis, que mudam o desenho da RLS:

**Opção A — Autenticação real agora.** Implementar Supabase Auth (e-mail/senha ou magic link) com `@supabase/ssr`, sessão via cookies, e RLS genuína baseada em `auth.uid()`. Mais trabalho agora, mas entrega o isolamento multi-tenant que o Database Blueprint promete desde o início.

**Opção B — Placeholder de usuário único para o MVP.** Adiar autenticação real; usar um `owner_id` fixo (ou desabilitar RLS temporariamente com uma política permissiva documentada como dívida técnica) até haver necessidade real de múltiplos usuários. Mais rápido agora, mas contraria o princípio "multi-tenant desde o início" já registrado no `DATABASE_BLUEPRINT.md`, e adia um problema que só cresce em custo depois.

**Esta escolha não foi feita unilateralmente — ver pergunta ao final da resposta.**

### 4.2 Modelo de dados (assumindo Opção A)

```sql
workspace_members
  id           uuid primary key
  workspace_id uuid references workspaces(id) on delete cascade
  user_id      uuid references auth.users(id) on delete cascade
  role         text not null default 'owner'  -- 'owner' | 'member'
  created_at   timestamptz not null default now()
  unique (workspace_id, user_id)
```

### 4.3 Políticas de RLS (assumindo Opção A)

- `workspaces`: select/update/delete permitido apenas se existir linha em `workspace_members` com `user_id = auth.uid()` para aquele `workspace_id`. Insert permitido para qualquer usuário autenticado (o registro em `workspace_members` como `owner` acontece na mesma transação/Server Action).
- `workspace_members`: select restrito às linhas do próprio usuário; insert/delete restrito ao próprio `owner`.

### 4.4 Server Action

Corrigir `create-workspace.action.ts` para incluir `"use server"`, obter o usuário autenticado via cliente Supabase server-side, inserir o Workspace, inserir a linha correspondente em `workspace_members` com `role: 'owner'`, e tratar erro de cada etapa.

### 4.5 Formulário

Corrigir `WorkspaceForm.tsx`: o botão precisa ser `type="submit"`, o `<form>` precisa de `action={createWorkspaceAction}` (ou equivalente com `useActionState`), e o resultado (sucesso/erro) precisa de feedback visual.

## 5. Dependências

- ADR-0001 — Next.js 14 / React 18 (define se o padrão de Server Action usa `useActionState` do React 18, não `useFormState` legado nem APIs exclusivas do React 19)
- `DATABASE_BLUEPRINT.md` — princípio de modelagem multi-tenant
- STD-0002, STD-0003 — versionamento e naming

## 6. Critérios de Aprovação

- [ ] Usuário autenticado consegue criar um Workspace pelo formulário e vê-lo persistido no Supabase.
- [ ] Um segundo usuário autenticado não consegue ver nem alterar o Workspace do primeiro (testado manualmente ou via política de RLS).
- [ ] Listagem de Workspaces na UI reflete o estado real do banco.
- [ ] Update e delete funcionam e respeitam a mesma restrição de acesso.
- [ ] `npm run build` e `tsc --noEmit` sem erro.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft, com achado bloqueante de autenticação registrado | Claude |
