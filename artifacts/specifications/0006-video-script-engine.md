---
id: SPC-0006
title: Video Script Engine (CS-012)
category: Specification
version: 0.1.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Commerce Studio
repository: cc-commerce-studio
created_at: 2026-07-11
updated_at: 2026-07-11
---

# SPC-0006 — Video Script Engine (CS-012)

## 1. Objetivo

Especificar o módulo Video Script: geração e gestão de roteiros de vídeo de venda (estilo VSL — Video Sales Letter) a partir de uma Offer existente, usando o mesmo provedor de IA já ativo (Gemini 3.1 Flash-Lite). Segundo módulo escolhido entre os que ficaram fora do escopo do MVP (DP-010), depois do Landing Page Engine (CS-011). Ordem de retomada decidida pelo usuário por critério de menor dependência externa nova (ver DP-012, cc-engineering-framework/ROADMAP.md): Video Script Engine → Creative Engine → Analytics Engine.

## 2. Escopo

- Tabela nova `public.video_scripts`, vinculada a `workspace_id` e `offer_id` (obrigatório) — mesmo padrão de proveniência do Landing Page Engine.
- Camada de feature completa em `features/video-script-engine/`, mesmo padrão vertical-slice dos módulos anteriores.
- Geração de roteiro via IA reaproveitando a mesma integração Gemini já ativa (`GEMINI_API_KEY`, `@google/genai`) — nenhuma conta, chave ou serviço externo novo.
- Novo Prompt `PR-0002` (`features/video-script-engine/prompts/video-script.md`), seguindo o ciclo de vida do STD-0007 (Draft → Ready → Active).
- Página autenticada de gestão: `app/(app)/video-scripts/page.tsx`.
- Roteiro é só texto estruturado (cenas/falas), editável antes de salvar — sem geração de vídeo real, sem upload de mídia.

## 3. Não Escopo

- Geração de vídeo real (áudio, imagem, edição) — fora de alcance de qualquer provedor de texto; ficaria para um módulo futuro dedicado, se decidido.
- Publicação/agendamento em redes sociais — escopo do futuro Publishing Engine.
- Analytics de desempenho do vídeo — escopo do futuro Analytics Engine.
- Múltiplas variações de roteiro por Offer (A/B) — mesmo não-escopo já aplicado em Landing Pages.

## 4. Conteúdo Principal

### 4.1 Achado da auditoria desta Specification

Auditado `features/offer-engine/` (services, actions, prompts) e `supabase/migrations/004_create_offers_and_rls.sql` como referência mais próxima, já que Video Script Engine gera conteúdo textual a partir de uma Offer, igual ao padrão já validado em CS-010/CS-011.

**Achado 1 — nenhuma rota pública necessária.** Diferente do Landing Page Engine, o roteiro de vídeo não tem consumo público nesta fase (não existe ainda Publishing Engine para postar o vídeo). Por isso `video_scripts` não precisa de política de RLS para acesso anônimo — só a política padrão de membership, igual a `products`/`brands`/`offers`.

**Achado 2 — reaproveitamento total da infraestrutura de IA.** `generateOfferCopy()` em `offer.service.ts` já resolve: cliente Gemini condicional a `GEMINI_API_KEY`, tratamento de erro de rede/cota e de resposta vazia/bloqueada (STD-0007 §4.8). O mesmo padrão será replicado em `video-script.service.ts::generateVideoScript()`, com um prompt diferente (`PR-0002`) — nenhuma mudança de infraestrutura, só um novo prompt e uma nova função isolada.

### 4.2 Modelo de dados (nova migração)

```sql
create table if not exists public.video_scripts (
  id           uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references public.workspaces(id) on delete cascade,
  offer_id     uuid not null references public.offers(id) on delete cascade,
  title        text not null,
  script       text,
  status       text not null default 'draft' check (status in ('draft', 'generated')),
  prompt_id    text,
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now()
);

create index if not exists idx_video_scripts_workspace_id on public.video_scripts(workspace_id);
create index if not exists idx_video_scripts_offer_id on public.video_scripts(offer_id);
```

### 4.3 Políticas de RLS

```sql
alter table public.video_scripts enable row level security;

create policy "video_scripts_member_all"
  on public.video_scripts for all
  using (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = video_scripts.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  )
  with check (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = video_scripts.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  );
```

Mesmo raciocínio já validado em CS-008/CS-009/CS-010/CS-011: a política depende só de `workspace_members`, que já existe antes de qualquer roteiro ser criado — insert com `.select().single()` é seguro.

### 4.4 Camada de feature e Server Actions

Replicar o padrão de `features/offer-engine/`: `types/video-script.types.ts`, `validations/video-script.schema.ts`, `services/video-script.service.ts` (com `generateVideoScript()`), `queries/`, `mutations/`, `actions/` (incluindo `generate-video-script.action.ts`), `components/` (`VideoScriptForm`, `VideoScriptCard`, `VideoScriptList`).

### 4.5 Prompt PR-0002

Novo prompt, front-matter conforme STD-0007, iniciando em `Draft`/`0.1.0`. Entrada: `offer.title`, `offer.copy`, `product.name`, `brand.tone_of_voice` (opcional). Saída esperada: roteiro estruturado em cenas curtas (gancho, problema, solução, prova, chamada para ação), texto simples, sem markdown.

### 4.6 UI de gestão

`VideoScriptForm` (título, seleciona a Offer de origem, botão "Gerar roteiro" chamando a Server Action, campo de roteiro editável, status), `VideoScriptCard`/`VideoScriptList`, página `app/(app)/video-scripts/page.tsx` — mesmo padrão visual dos demais módulos. Entrada "Roteiros de Vídeo" no `Sidebar`.

## 5. Dependências

- SPC-0004 — Offer Engine (CS-010) — todo roteiro se origina de uma Offer
- STD-0007 — Prompt & Script Governance Standard — novo prompt PR-0002 segue o mesmo ciclo de vida
- STD-0006 — Engineering Principles (P8, migrações aditivas; P9, isolar dependência externa volátil)

## 6. Critérios de Aprovação

- [ ] Usuário autenticado consegue criar um roteiro de vídeo vinculado a uma Offer do seu Workspace.
- [ ] "Gerar roteiro" chama o Gemini de verdade e preenche o campo de roteiro.
- [ ] Falha de rede/cota ou resposta vazia/bloqueada do Gemini vira erro tratável na UI, não exceção não tratada.
- [ ] Um usuário sem vínculo com o Workspace não consegue ver nem alterar roteiros de outro Workspace (RLS de membro).
- [ ] `npm run build` e `tsc --noEmit` sem erro.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-11 | Primeira versão Draft — segundo módulo pós-MVP, ordem decidida pelo usuário (DP-012) | Claude |
