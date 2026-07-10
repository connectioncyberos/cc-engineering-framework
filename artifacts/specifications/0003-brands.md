---
id: SPC-0003
title: Módulo Brands (CS-009)
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

# SPC-0003 — Módulo Brands (CS-009)

## 1. Objetivo

Especificar o módulo Brands: criação, listagem, atualização e remoção de Marcas vinculadas a um Workspace, e o vínculo (aditivo, não destrutivo) entre `products` e `brands`. Este é o último módulo de infraestrutura antes do Offer Engine, que consome Products + Brands para gerar ofertas.

## 2. Escopo

- Tabela nova `public.brands`, com campos mínimos para identidade de marca: `name`, `slug`, `description`, `tone_of_voice`, `logo_url`.
- Política de RLS para `brands`, no mesmo padrão de `products_member_all` (isolamento por `workspace_members`).
- Migração aditiva em `products`: nova coluna `brand_id` (nullable, `on delete set null`) — não quebra nenhum produto já existente.
- Camada de feature completa em `features/brands/`, seguindo exatamente o padrão de `features/products/` (CS-008).
- UI: `BrandForm`, `BrandCard`/`BrandList`, página `app/(app)/brands/page.tsx`.
- Campo de seleção de Brand (opcional) no `ProductForm` existente, para permitir associar um produto a uma marca já cadastrada.

## 3. Não Escopo

- Múltiplas marcas por produto (relação N:N) — nesta versão, um produto pertence no máximo a uma marca (`brand_id` único).
- Paleta de cores/guia visual completo de marca — apenas `tone_of_voice` (texto livre) nesta versão, suficiente para o Offer Engine consumir depois.
- Qualquer geração assistida por IA a partir do `tone_of_voice` — isso é escopo do Offer Engine (SPC seguinte).
- Upload de arquivo de logo — `logo_url` aceita apenas uma URL nesta versão; upload de asset fica para o módulo Assets.

## 4. Conteúdo Principal

### 4.1 Estado atual verificado (auditoria desta Specification)

Confirmado nas migrations existentes: não existe tabela `brands` em nenhum lugar do schema. `products` (criada em `001_create_workspace_core.sql`) não tem coluna `brand_id`. Isso é consistente com o que já havia sido registrado em DP-007 (`cc-engineering-framework/ROADMAP.md`): Brands exige schema novo, diferente de Products, que já existia pronto.

### 4.2 Modelo de dados (nova migração)

```sql
create table if not exists public.brands (
  id           uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references public.workspaces(id) on delete cascade,
  name         text not null,
  slug         text not null,
  description  text,
  tone_of_voice text,
  logo_url     text,
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now(),
  unique (workspace_id, slug)
);

create index if not exists idx_brands_workspace_id on public.brands(workspace_id);

-- Migração aditiva: produtos existentes continuam válidos (brand_id nulo).
alter table public.products
  add column if not exists brand_id uuid references public.brands(id) on delete set null;

create index if not exists idx_products_brand_id on public.products(brand_id);
```

### 4.3 Políticas de RLS (novas, mesmo padrão de `products_member_all`)

```sql
alter table public.brands enable row level security;

create policy "brands_member_all"
  on public.brands for all
  using (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = brands.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  )
  with check (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = brands.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  );
```

Nenhuma política de `products` precisa mudar: `brand_id` é apenas uma FK nova, e o acesso a produtos continua controlado por `products_member_all` (via `workspace_id`), não por marca.

### 4.4 Camada de feature e Server Actions

Replicar exatamente `features/products/` (CS-008): `types/brand.types.ts`, `validations/brand.schema.ts` (Zod: `name`, `slug`, `description`, `tone_of_voice`, `logo_url` opcionais/obrigatórios conforme uso), `services/brand.service.ts`, `queries/`, `mutations/`, `actions/`, `components/`. Como a política `brands_member_all` depende só de `workspace_members` (já existente antes de qualquer marca ser criada), o insert pode usar `.select().single()` com segurança — mesmo raciocínio já validado em CS-008 para `products`.

### 4.5 Integração com Products

Adicionar um campo opcional (select) de Brand no `ProductForm`, populado via `listBrandsQuery(supabase, workspaceId)`. Ao salvar, `brand_id` é passado no `CreateProductInput`/`UpdateProductInput`. Produtos sem marca (`brand_id = null`) continuam funcionando normalmente — nenhuma regressão em CS-008.

## 5. Dependências

- SPC-0002 — Módulo Products (CS-008), define o padrão de feature replicado aqui
- STD-0002, STD-0003 — versionamento e naming
- STD-0007 — Prompt & Script Governance Standard (o campo `tone_of_voice` é o dado que o Offer Engine vai consumir para gerar prompts de marca — conexão direta com a dívida futura de 0.2.0 registrada em STD-0007)

## 6. Critérios de Aprovação

- [ ] Usuário autenticado consegue criar uma Brand vinculada ao seu Workspace e vê-la persistida no Supabase.
- [ ] Um usuário sem vínculo com o Workspace não consegue ver nem alterar suas Brands (RLS).
- [ ] Produtos existentes (criados antes desta migração) continuam acessíveis e editáveis sem erro após o `ALTER TABLE`.
- [ ] É possível associar um Product existente a uma Brand pelo `ProductForm`.
- [ ] Update e delete de Brand funcionam e respeitam a mesma restrição de acesso.
- [ ] `npm run build` e `tsc --noEmit` sem erro.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft — schema novo, migração aditiva em `products`, sem regressão | Claude |
