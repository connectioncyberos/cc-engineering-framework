---
id: SPC-0002
title: Módulo Products (CS-008)
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

# SPC-0002 — Módulo Products (CS-008)

## 1. Objetivo

Especificar o comportamento esperado do módulo Products: criação, listagem, atualização e remoção de Produtos vinculados a um Workspace, reaproveitando o mesmo padrão de persistência, isolamento multi-tenant e camada de feature já validado no Workspace (CS-007/SPC-0001).

## 2. Escopo

- Camada de feature completa em `features/products/` (`types`, `validations`, `services`, `queries`, `mutations`, `actions`, `components`), seguindo a mesma estrutura de `features/workspace/`.
- Server Actions para create/update/delete de Product, usando o cliente Supabase server-side (`lib/supabase/server.ts`), já com sessão de usuário autenticado.
- Validação com Zod (`product.schema.ts`) para nome, descrição e status.
- Listagem dinâmica e UI (`ProductForm`, `ProductCard`/`ProductList`) seguindo o padrão visual já estabelecido para Workspace.
- Página `app/(app)/products/page.tsx`, com `PageHeader` e ação "Novo Produto", mesmo padrão de `app/(app)/workspace/page.tsx`.
- Seleção de Workspace ativo: todo Product pertence a um `workspace_id`; nesta versão, assume-se um único Workspace ativo por sessão (ver Não Escopo).

## 3. Não Escopo

- Vínculo com Brand (`brand_id`) — depende de SPC-0003 (Brands), ainda não escrita. Coluna fica de fora desta versão; será uma migração aditiva depois.
- Workspace switcher (troca entre múltiplos Workspaces na mesma sessão) — mesma exclusão já registrada em SPC-0001.
- Campos de catálogo avançado (preço, variações, estoque, imagens) — fora do escopo do MVP; `status` (`draft`/outros) já existe na tabela e é suficiente por ora.
- Qualquer geração assistida por IA (isso é escopo do Offer Engine, ainda não especificado).

## 4. Conteúdo Principal

### 4.1 Estado atual verificado (auditoria desta Specification)

Diferente do Workspace, este módulo **não tem achado bloqueante**. Verificado diretamente em `supabase/migrations/`:

- `001_create_workspace_core.sql` já cria `public.products` com `id`, `workspace_id` (FK para `workspaces`, `on delete cascade`), `name`, `description`, `status` (default `'draft'`), `created_at`, `updated_at`.
- `002_create_workspace_members_and_rls.sql` já habilita RLS em `products` e já define a política `products_member_all` (`for all`, `using`/`with check` exigindo linha em `workspace_members` para o `workspace_id` do produto) — aplicada e confirmada em produção junto com as demais políticas do CS-007.

Ou seja: schema e RLS são reaproveitados sem alteração. O trabalho desta Specification é 100% camada de aplicação (feature + UI), replicando o padrão já testado no Workspace.

### 4.2 Modelo de dados (já existente, sem migração nova)

```sql
public.products
  id           uuid primary key default gen_random_uuid()
  workspace_id uuid not null references public.workspaces(id) on delete cascade
  name         text not null
  description  text
  status       text not null default 'draft'
  created_at   timestamptz not null default now()
  updated_at   timestamptz not null default now()
```

### 4.3 Políticas de RLS (já existentes, sem alteração)

`products_member_all` — select/insert/update/delete permitido apenas se o usuário autenticado tiver uma linha em `workspace_members` para o `workspace_id` do produto.

### 4.4 Server Actions

Seguir exatamente o padrão de `create-workspace.action.ts` corrigido em CS-007: gerar `id` no cliente (`crypto.randomUUID()`), inserir sem `.select().single()` quando a visibilidade da linha depender de outra tabela — neste caso não depende (a política de `products` usa `workspace_members`, que já existe antes do insert do produto), então o `RETURNING` é seguro aqui. Ainda assim, manter o mesmo formato de retorno (`{ error, success }`) por consistência.

### 4.5 Workspace ativo

Nesta versão, a Server Action recebe o `workspace_id` explicitamente do formulário (campo oculto ou parâmetro de rota), sem seletor de Workspace na UI — mesma simplificação já assumida implicitamente pelo módulo Workspace atual (não há troca de contexto).

## 5. Dependências

- SPC-0001 — Persistência do Workspace (padrão de referência para toda a camada de feature)
- STD-0002, STD-0003 — versionamento e naming
- ADR-0001 — Next.js 14 / React 18 (Server Actions com `useFormState`/`useFormStatus`)

## 6. Critérios de Aprovação

- [ ] Usuário autenticado consegue criar um Product vinculado ao seu Workspace pelo formulário e vê-lo persistido no Supabase.
- [ ] Um usuário sem vínculo com o Workspace não consegue ver nem alterar seus Products (RLS já existente, validar comportamento na aplicação).
- [ ] Listagem de Products na UI reflete o estado real do banco.
- [ ] Update e delete funcionam e respeitam a mesma restrição de acesso.
- [ ] `npm run build` e `tsc --noEmit` sem erro.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft — schema e RLS já existentes e verificados, sem achado bloqueante | Claude |
