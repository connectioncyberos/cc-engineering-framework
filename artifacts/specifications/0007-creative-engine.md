---
id: SPC-0007
title: Creative Engine (CS-013)
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

# SPC-0007 — Creative Engine (CS-013)

## 1. Objetivo

Especificar o módulo Creative Engine: geração de título otimizado para marketplace, geração de imagens de criativo (a partir de imagens de referência + descrição) e geração de vídeo a partir dessas imagens — replicando o fluxo manual que o usuário já executa hoje fora do produto (upload de referências + descrição → imagens → vídeo). Terceiro módulo da ordem decidida em DP-012, mas o primeiro que exige infraestrutura nova (armazenamento de arquivo, billing de IA, job assíncrono).

## 2. Escopo

Dividido em três fases, cada uma com dependência técnica real diferente:

- **Fase A — Título SEO.** Geração de título otimizado para marketplace (Mercado Livre, Shopee, Amazon, Magalu) a partir de Product + Offer. Reaproveita 100% o padrão de texto já usado (Gemini 3.1 Flash-Lite, tier gratuito) — nenhuma infraestrutura nova.
- **Fase B — Geração de imagem.** Upload de 1 a 14 imagens de referência + descrição textual → gera imagem de criativo via Nano Banana Pro (`gemini-3-pro-image-preview`), usando a **Interactions API** (`ai.interactions.create()`), diferente da `generateContent()` usada em PR-0001/PR-0002.
- **Fase C — Geração de vídeo.** A partir das imagens geradas na Fase B, gera um vídeo curto via Veo 3.1 (`veo-3.1-generate-preview`), usando `ai.models.generateVideos()` — operação assíncrona de longa duração, com polling.

## 3. Não Escopo

- Edição manual de imagem/vídeo dentro do produto (crop, filtros, timeline) — o produto só orquestra a geração, não é um editor.
- Publicação direta em marketplaces/redes sociais — escopo do futuro Publishing Engine.
- Múltiplas variações simultâneas (A/B de criativos) na primeira versão.
- Legendas/dublagem automática do vídeo além do que o Veo já gera nativamente.

## 4. Conteúdo Principal

### 4.1 Achados da auditoria desta Specification (bloqueantes)

Auditado `features/offer-engine/` e `features/video-script-engine/` (ambos só geram texto), `package.json` (`@google/genai@^2.11.0`), e ausência de qualquer bucket do Supabase Storage no código atual. Quatro achados mudam o desenho, dois deles bloqueantes:

**Achado 1 (bloqueante) — nem imagem nem vídeo têm tier gratuito.** Diferente do texto (Gemini 3.1 Flash-Lite, grátis), os modelos de imagem (Nano Banana / Nano Banana Pro) custam US$ 0,034–US$ 0,134 por imagem, e o Veo 3.1 custa US$ 0,03–US$ 0,60 por segundo de vídeo. É necessário **billing habilitado na conta Google AI Studio/Vertex** antes de qualquer teste real — decisão e ação do usuário, fora do alcance deste assistente (ver Critérios de Aprovação).

**Achado 2 (bloqueante) — não existe upload de arquivo no produto hoje.** Todos os módulos anteriores (Products, Brands, Offer Engine, Landing Pages, Video Script Engine) só usam campos de texto. Referências de imagem exigem armazenamento de arquivo — o Supabase Storage não tem nenhum bucket configurado ainda. É necessário criar um bucket (ex.: `creative-references`) com política de RLS própria antes da Fase B funcionar.

**Achado 3 — geração de imagem usa uma API diferente da já validada.** PR-0001 e PR-0002 usam `ai.models.generateContent()`. Nano Banana/Nano Banana Pro usam `ai.interactions.create({ model, input: [...] })`, onde `input` mistura blocos de texto e imagem (base64), e a imagem gerada volta em `interaction.output_image.data` (base64). É um padrão novo, não uma reutilização direta do `offer.service.ts`.

**Achado 4 — geração de vídeo é assíncrona, não cabe numa Server Action síncrona única.** `ai.models.generateVideos()` retorna uma operação de longa duração; o próprio exemplo oficial do Google faz polling a cada 10 segundos até `operation.done` — e isso é só o tempo de exemplo, um vídeo real pode levar minutos. Diferente de Offer Engine/Video Script Engine (uma chamada, uma resposta), o Creative Engine precisa: (a) uma Server Action que **inicia** a geração e salva o nome da operação (`operation.name`) e status `processing` na tabela; (b) uma Server Action separada de **verificar status**, chamada repetidamente pelo cliente (poll a cada alguns segundos) até `done`; (c) ao concluir, baixar o vídeo e salvar a URL. Isso é uma mudança de padrão arquitetural real em relação a todos os módulos anteriores.

### 4.2 Modelo de dados (nova migração)

```sql
create table if not exists public.creatives (
  id                  uuid primary key default gen_random_uuid(),
  workspace_id        uuid not null references public.workspaces(id) on delete cascade,
  offer_id            uuid not null references public.offers(id) on delete cascade,
  seo_title           text,
  reference_image_urls text[] not null default '{}',
  generated_image_url text,
  video_operation_name text,
  video_status        text not null default 'idle' check (video_status in ('idle', 'processing', 'done', 'failed')),
  video_url           text,
  status              text not null default 'draft' check (status in ('draft', 'generated')),
  created_at          timestamptz not null default now(),
  updated_at          timestamptz not null default now()
);

create index if not exists idx_creatives_workspace_id on public.creatives(workspace_id);
create index if not exists idx_creatives_offer_id on public.creatives(offer_id);
```

### 4.3 Políticas de RLS

```sql
alter table public.creatives enable row level security;

create policy "creatives_member_all"
  on public.creatives for all
  using (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = creatives.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  )
  with check (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = creatives.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  );
```

Mesmo padrão de membership já validado em CS-008 a CS-012.

### 4.4 Supabase Storage (novo, Achado 2)

Bucket `creative-references` (privado), com política de storage restrita a membros do Workspace (mesmo padrão de `workspace_members`). Upload feito pelo cliente autenticado antes de chamar a geração de imagem; a URL assinada/pública do arquivo é o que entra em `reference_image_urls`.

### 4.5 Prompts novos

- `PR-0003` — título SEO de marketplace (Fase A, texto puro, mesmo padrão de PR-0001/PR-0002).
- `PR-0004` — geração de imagem de criativo a partir de referências (Fase B, Interactions API).

### 4.6 Camada de feature e Server Actions

`features/creative-engine/`: mesmo padrão vertical-slice, com adições específicas desta Specification: `services/creative.service.ts` (`generateSeoTitle()`, `generateCreativeImage()`, `startVideoGeneration()`, `checkVideoStatus()`), `actions/generate-video-status.action.ts` (chamado em poll pelo cliente).

### 4.7 UI de gestão

`CreativeForm`: upload de referências (até 14), campo de descrição, botão "Gerar título SEO", botão "Gerar imagem", e — só depois de uma imagem existir — botão "Gerar vídeo" que inicia o job e mostra estado "Processando..." com poll automático a cada alguns segundos até `video_status = 'done'` ou `'failed'`.

## 5. Dependências

- SPC-0004 — Offer Engine (CS-010) — todo criativo se origina de uma Offer
- STD-0007 — Prompt & Script Governance Standard (1.0.0, Approved) — PR-0003 e PR-0004 seguem o mesmo ciclo de vida
- STD-0006 — Engineering Principles (P4, decisão de arquitetura como opção explícita; P9, isolar dependência externa volátil)
- **Bloqueante externo:** billing habilitado na conta Google AI Studio/Vertex (ação do usuário, fora do alcance deste assistente)
- **Bloqueante técnico:** bucket do Supabase Storage criado e configurado (pode ser feito via código/migração de storage, a confirmar)

## 6. Critérios de Aprovação

- [ ] Usuário confirma billing habilitado na conta Google AI Studio/Vertex antes do início da Fase B/C.
- [ ] Fase A: título SEO gerado a partir de uma Offer, sem custo (tier gratuito de texto).
- [ ] Fase B: upload de ao menos 1 imagem de referência, geração de imagem de criativo real via Nano Banana Pro, imagem salva e exibida.
- [ ] Fase C: vídeo gerado a partir da imagem, com estado "Processando..." visível e transição correta para "Pronto" ou "Falhou".
- [ ] Falha de rede/cota/billing insuficiente vira erro tratável na UI, não exceção não tratada (mesmo padrão STD-0007 §4.8).
- [ ] Um usuário sem vínculo com o Workspace não consegue ver nem alterar criativos de outro Workspace (RLS de membro).
- [ ] `npm run build` e `tsc --noEmit` sem erro.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-11 | Primeira versão Draft — terceiro módulo pós-MVP, primeiro com custo real de IA (sem tier gratuito), armazenamento de arquivo e job assíncrono. Escopo completo (imagem + vídeo) decidido pelo usuário após apresentação de custos reais (DP-013) | Claude |
