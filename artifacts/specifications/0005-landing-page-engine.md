---
id: SPC-0005
title: Landing Page Engine (CS-011)
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

# SPC-0005 — Landing Page Engine (CS-011)

## 1. Objetivo

Especificar o módulo Landing Page: criação, listagem, atualização e remoção de páginas de venda vinculadas a uma Offer, com uma versão pública (sem login) acessível por URL/slug quando publicada. Primeiro módulo escolhido entre os 8 que ficaram fora do escopo do MVP (DP-010) — segue o mesmo ciclo já validado: auditoria → Specification → código → teste.

## 2. Escopo

- Tabela nova `public.landing_pages`, vinculada a `workspace_id` e `offer_id` (obrigatório).
- Camada de feature completa em `features/landing-pages/`, mesmo padrão de Products/Brands/Offer Engine.
- Página autenticada de gestão: `app/(app)/landing-pages/page.tsx`.
- Página pública (sem login): `app/lp/[slug]/page.tsx`, renderizando a landing page só quando `status = 'published'`.
- RLS com duas regras distintas: membros do Workspace têm acesso total (select/insert/update/delete); visitantes públicos (sem sessão) têm select restrito a páginas com `status = 'published'`.
- Conteúdo inicial da página herda a `copy` da Offer vinculada (pré-preenchido, editável antes de publicar).

## 3. Não Escopo

- Domínio customizado (a página vive em `/lp/<slug>` no domínio do próprio Commerce Studio nesta versão).
- Analytics de conversão/visitas — escopo do futuro Analytics Engine.
- Editor visual de blocos/seções — mesmo padrão simples de texto já usado nos outros módulos (título + conteúdo em texto), sem builder de páginas.
- Múltiplas landing pages testando variações (A/B) da mesma oferta.

## 4. Conteúdo Principal

### 4.1 Achado da auditoria desta Specification

Dois achados, nenhum bloqueante, mas que mudam o desenho:

**Achado 1 — roteamento público.** Verificado em `middleware.ts` e `app/(app)/layout.tsx`: a autenticação não é aplicada globalmente no middleware (que só atualiza a sessão), é aplicada no próprio layout de `app/(app)/` via `redirect("/login")`. Isso significa que uma rota criada fora do grupo `(app)` (ex.: `app/lp/[slug]/page.tsx`) já nasce pública, sem exigir login — nenhuma mudança de arquitetura necessária, só não colocar a rota dentro de `(app)/`.

**Achado 2 — RLS não cobre acesso público.** Todas as políticas de RLS existentes (`workspaces_select_member`, `products_member_all`, `brands_member_all`, `offers_member_all`) exigem `auth.uid()` presente em `workspace_members` — um visitante sem sessão sempre recebe zero linhas. Isso nunca foi um problema até agora porque nenhuma tela pública existia. A política de `landing_pages` precisa de uma regra adicional de select público, restrita a `status = 'published'` (ver 4.3) — as demais operações (insert/update/delete, e select de rascunhos) continuam exigindo membership, sem enfraquecer o isolamento multi-tenant já existente.

### 4.2 Modelo de dados (nova migração)

```sql
create table if not exists public.landing_pages (
  id           uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references public.workspaces(id) on delete cascade,
  offer_id     uuid not null references public.offers(id) on delete cascade,
  title        text not null,
  slug         text not null,
  content      text,
  status       text not null default 'draft' check (status in ('draft', 'published')),
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now(),
  unique (slug)
);

create index if not exists idx_landing_pages_workspace_id on public.landing_pages(workspace_id);
create index if not exists idx_landing_pages_offer_id on public.landing_pages(offer_id);
```

`slug` é único globalmente (não só por workspace) porque a URL pública `/lp/<slug>` não carrega o workspace no caminho.

### 4.3 Políticas de RLS (duas regras, não uma)

```sql
alter table public.landing_pages enable row level security;

-- Membros do Workspace: acesso total (rascunho ou publicada).
create policy "landing_pages_member_all"
  on public.landing_pages for all
  using (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = landing_pages.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  )
  with check (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = landing_pages.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  );

-- Público (sem sessão): só linhas publicadas.
create policy "landing_pages_select_public_published"
  on public.landing_pages for select
  using (status = 'published');
```

Duas políticas de `select` coexistem (Postgres aplica `OR` entre políticas permissivas da mesma ação) — um membro sempre vê tudo do seu Workspace; um visitante público só vê o que está publicado, de qualquer Workspace.

### 4.4 Camada de feature e Server Actions

Replicar o padrão de `features/offer-engine/` (CS-010): `types/landing-page.types.ts`, `validations/landing-page.schema.ts`, `services/landing-page.service.ts`, `queries/`, `mutations/`, `actions/`, `components/`. Mesmo raciocínio de RETURNING seguro já validado (política de membro não depende de linha criada na mesma transação).

### 4.5 Página pública

`app/lp/[slug]/page.tsx` — Server Component fora do grupo `(app)`, sem `AppShell`, sem checagem de login. Busca a landing page pelo `slug`; se não encontrada ou não publicada, renderiza 404 (`notFound()` do Next.js). Renderiza título e conteúdo em texto simples, sem sidebar/topbar.

### 4.6 UI de gestão

`LandingPageForm` (título, slug, seleciona a Offer de origem, conteúdo pré-preenchido com a `copy` da Offer, status), `LandingPageCard`/`LandingPageList` com link para a versão pública quando publicada, página `app/(app)/landing-pages/page.tsx` — mesmo padrão visual dos demais módulos.

## 5. Dependências

- SPC-0004 — Offer Engine (CS-010) — toda landing page se origina de uma Offer
- STD-0006 — Engineering Principles (P8, migrações aditivas; P4, decisão de arquitetura como esta precisa ser explícita)
- STD-0002, STD-0003 — versionamento e naming

## 6. Critérios de Aprovação

- [ ] Usuário autenticado consegue criar uma Landing Page vinculada a uma Offer do seu Workspace.
- [ ] Um usuário sem vínculo com o Workspace não consegue ver nem alterar Landing Pages de outro Workspace (RLS de membro).
- [ ] Uma Landing Page com `status = 'draft'` retorna 404 na URL pública.
- [ ] Uma Landing Page com `status = 'published'` é visível em `/lp/<slug>` **sem estar logado** (testar em aba anônima).
- [ ] Update e delete respeitam a mesma restrição de membership.
- [ ] `npm run build` e `tsc --noEmit` sem erro.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft — primeiro módulo com rota pública; RLS de select público restrita a `status = 'published'` | Claude |
