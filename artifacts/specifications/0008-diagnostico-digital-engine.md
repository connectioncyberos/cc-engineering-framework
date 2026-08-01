---
id: SPC-0008
title: Diagnóstico Digital Engine (CS-014)
category: Specification
version: 0.1.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Commerce Studio
repository: cc-commerce-studio
created_at: 2026-08-01
updated_at: 2026-08-01
---

# SPC-0008 — Diagnóstico Digital Engine (CS-014)

## 1. Objetivo

Especificar o primeiro módulo novo do MPI (Marketing e Posicionamento na Internet) dentro do Commerce Studio: um diagnóstico estruturado da presença digital atual do Workspace (site, redes sociais, concorrência, público-alvo, objetivo de negócio, maturidade digital autodeclarada), com uma síntese gerada por IA que recomenda por onde começar entre os Engines já existentes (Offer, Landing Page, Video Script, Creative). É o "diagnóstico de entrada" que a metodologia MPI previa antes de qualquer módulo de conteúdo — e o primeiro módulo executado depois da decisão de trazer o MPI para dentro do Commerce Studio (DP-016, cc-engineering-framework/ROADMAP.md), por não depender de nenhuma conta ou API externa nova.

## 2. Escopo

- Tabela nova `public.diagnostics`, vinculada a `workspace_id` e a `project_id` — reaproveitando `public.projects` (existente desde a migração 001, sem uso até hoje) como o "ciclo/projeto MPI" ao qual o diagnóstico pertence.
- Alteração aditiva em `public.projects`: adicionar `check` de status (`draft`, `in_progress`, `completed`) — hoje a coluna existe sem nenhuma restrição de valor.
- Camada de feature completa em `features/diagnostic-engine/`, mesmo padrão vertical-slice dos módulos anteriores.
- Formulário estruturado (não texto livre) cobrindo: canais digitais atuais, público-alvo declarado, concorrentes diretos, objetivo principal do Workspace, maturidade digital autodeclarada — armazenado em `answers` (`jsonb`), validado por Zod na camada de aplicação.
- Síntese gerada por IA (`summary`) a partir das respostas, reaproveitando a mesma integração Gemini já ativa (`GEMINI_API_KEY`, `@google/genai`) — nenhuma conta, chave ou serviço externo novo.
- Novo Prompt `PR-0005` (`features/diagnostic-engine/prompts/diagnostic-summary.md`), seguindo o ciclo de vida do STD-0007 (Draft → Ready → Active). Numeração `PR-0005` porque `PR-0003`/`PR-0004` já estão reservados para o Creative Engine em SPC-0007, mesmo esse ainda não tendo sido construído.
- Página autenticada de gestão: `app/(app)/diagnostics/page.tsx`, com criação do "projeto MPI" (`projects`) quando o Workspace ainda não tiver um.

## 3. Não Escopo

- Upload de arquivo (screenshots de site, referências visuais de concorrentes) — exigiria um bucket do Supabase Storage, que só será criado quando o Creative Engine (CS-013) resolver essa mesma dependência (Achado 2, SPC-0007). Reaproveitar o bucket dele é candidato natural para uma versão futura deste módulo, não para a primeira.
- Coleta automática de dados (scraping de site/redes sociais, APIs de concorrência) — todas as respostas desta primeira versão são preenchidas manualmente pelo usuário.
- Histórico/versionamento de diagnóstico (comparar diagnóstico de hoje com o de 3 meses atrás) — a primeira versão guarda um diagnóstico ativo por projeto, não uma série temporal.
- Qualquer execução automática de recomendação (a IA sugere, mas não cria Offers/Landing Pages sozinha) — a ação de seguir a recomendação continua manual, no Engine correspondente.

## 4. Conteúdo Principal

### 4.1 Achados da auditoria desta Specification

Auditado `supabase/migrations/001_create_workspace_core.sql`, `002_create_workspace_members_and_rls.sql` e todo o diretório `features/` em busca de qualquer uso de `projects`/`assets`.

**Achado 1 — `projects` está pronto para reaproveitamento, sem migração destrutiva.** A tabela já existe desde a migração 001 (`id`, `workspace_id`, `name`, `description`, `status default 'draft'`, timestamps) e já tem RLS aplicada desde a migração 002 (`projects_member_all`, mesmo padrão de membership de todas as outras tabelas). Nenhum arquivo em `features/` ou `app/` referencia `public.projects` — confirma o achado já registrado em DP-015 (cc-engineering-framework): a tabela é órfã, não descontinuada. Pode ser adotada como o "projeto/ciclo MPI" de cada Workspace sem nenhum risco de regressão, porque não existe nenhuma feature usando-a hoje para quebrar.

**Achado 2 — `assets` fica fora de escopo por dependência real, não por preferência.** A tabela também está pronta (RLS incluída), mas seu propósito original (anexar arquivo a um `product_id`/`project_id`) só faz sentido com upload de arquivo funcionando — e isso depende do bucket do Supabase Storage que o Creative Engine (SPC-0007, Achado 2) ainda vai criar. Usar `assets` agora, sem Storage configurado, seria antecipar uma dependência que ainda não existe. Fica registrado como reaproveitamento futuro, não como não-escopo permanente.

**Achado 3 — `projects.status` não tem `check` de valores, diferente de todas as tabelas mais recentes.** `offers`, `landing_pages`, `video_scripts` e `creatives` (proposto) todos têm `check (status in (...))`. `projects` (migração 001, mais antiga) não tem. Adicionar o `check` agora é aditivo (o valor padrão `'draft'` já cumpre a nova restrição) e alinha `projects` ao padrão dos módulos mais recentes antes de ele ganhar sua primeira feature real.

### 4.2 Modelo de dados

Alteração aditiva em `projects` (migração nova, não altera a 001):

```sql
alter table public.projects
  drop constraint if exists projects_status_check;

alter table public.projects
  add constraint projects_status_check
  check (status in ('draft', 'in_progress', 'completed'));
```

Tabela nova `diagnostics`:

```sql
create table if not exists public.diagnostics (
  id           uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references public.workspaces(id) on delete cascade,
  project_id   uuid not null references public.projects(id) on delete cascade,
  title        text not null,
  answers      jsonb not null default '{}'::jsonb,
  summary      text,
  status       text not null default 'draft' check (status in ('draft', 'generated')),
  prompt_id    text,
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now()
);

create index if not exists idx_diagnostics_workspace_id on public.diagnostics(workspace_id);
create index if not exists idx_diagnostics_project_id on public.diagnostics(project_id);
```

`answers` guarda o formulário estruturado (`canais_digitais`, `publico_alvo`, `concorrentes`, `objetivo_principal`, `maturidade_digital`) — validado por Zod em `features/diagnostic-engine/validations/`, não por `check` no banco (mesmo critério já usado em `offers.copy`/`video_scripts.script`: formato livre, validação na camada de aplicação).

### 4.3 Políticas de RLS

```sql
alter table public.diagnostics enable row level security;

create policy "diagnostics_member_all"
  on public.diagnostics for all
  using (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = diagnostics.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  )
  with check (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = diagnostics.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  );
```

`projects_member_all` (migração 002) já cobre o reaproveitamento — nenhuma policy nova é necessária para `projects`.

### 4.4 Camada de feature e Server Actions

`features/diagnostic-engine/`: `types/diagnostic.types.ts`, `validations/diagnostic.schema.ts` (schema Zod do formulário estruturado), `services/diagnostic.service.ts` (com `generateDiagnosticSummary()`, reaproveitando o mesmo padrão condicional a `GEMINI_API_KEY` de `offer.service.ts`), `queries/`, `mutations/` (incluindo `ensure-project.ts`, que cria o `project` do Workspace na primeira vez), `actions/` (`create-diagnostic.action.ts`, `generate-diagnostic-summary.action.ts`), `components/` (`DiagnosticForm`, `DiagnosticCard`, `DiagnosticSummary`).

### 4.5 Prompt PR-0005

Novo prompt, front-matter conforme STD-0007, iniciando em `Draft`/`0.1.0`. Entrada: as 5 respostas estruturadas (`canais_digitais`, `publico_alvo`, `concorrentes`, `objetivo_principal`, `maturidade_digital`) e a lista de Engines hoje disponíveis (Offer, Landing Page, Video Script, Creative). Saída esperada: texto corrido curto (síntese do diagnóstico) terminando com uma recomendação explícita de qual Engine usar primeiro e por quê.

### 4.6 UI de gestão

`DiagnosticForm` (formulário de 5 campos estruturados, botão "Gerar diagnóstico" chamando a Server Action, `DiagnosticSummary` exibindo o texto gerado e a recomendação), página `app/(app)/diagnostics/page.tsx`. Entrada "Diagnóstico" no `Sidebar`, primeira posição do menu (antes de "Produtos"), coerente com o papel de porta de entrada do módulo.

## 5. Dependências

- SPC-0001 — Workspace Persistence — `workspaces`/`workspace_members`/RLS, base de tudo
- STD-0007 — Prompt & Script Governance Standard (1.0.0, Approved) — PR-0005 segue o mesmo ciclo de vida
- STD-0006 — Engineering Principles (P2, auditar em vez de assumir — cumprido nesta própria Specification; P8, migrações aditivas — alteração em `projects.status` não quebra o valor padrão existente)
- DP-016 (cc-engineering-framework/ROADMAP.md) — decisão de trazer o MPI para dentro do Commerce Studio e de reaproveitar `projects`/`assets`
- **Dependência futura, não bloqueante:** bucket do Supabase Storage do Creative Engine (SPC-0007), candidato a reaproveitamento quando este módulo ganhar upload de arquivo

## 6. Critérios de Aprovação

- [ ] Usuário autenticado consegue criar (ou reaproveitar) um `project` do seu Workspace e preencher um diagnóstico vinculado a ele.
- [ ] "Gerar diagnóstico" chama o Gemini de verdade e preenche `summary` com uma recomendação explícita de Engine.
- [ ] Falha de rede/cota ou resposta vazia/bloqueada do Gemini vira erro tratável na UI, não exceção não tratada (mesmo padrão STD-0007 §4.8).
- [ ] Um usuário sem vínculo com o Workspace não consegue ver nem alterar diagnósticos nem projetos de outro Workspace (RLS de membro, reaproveitando `projects_member_all` já existente).
- [ ] Nenhuma regressão nos módulos existentes (Products, Brands, Offer/Landing/Video Engine) causada pela alteração aditiva em `projects.status`.
- [ ] `npm run build` e `tsc --noEmit` sem erro.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-08-01 | Primeira versão Draft — primeiro módulo novo do MPI dentro do Commerce Studio (DP-016), reaproveitando `projects` como ciclo/projeto e adiando `assets`/upload até o Storage do Creative Engine existir | Claude |
