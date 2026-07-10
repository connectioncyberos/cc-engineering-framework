---
id: SPC-0004
title: Offer Engine (CS-010) — Primeiro Engine de IA
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

# SPC-0004 — Offer Engine (CS-010) — Primeiro Engine de IA

## 1. Objetivo

Especificar o primeiro Engine de IA do produto: geração de ofertas de venda a partir de um Product e, opcionalmente, uma Brand (tom de voz). Esta Specification é também o primeiro caso real que aciona a expansão do `STD-0007` (Prompt & Script Governance Standard) de 0.1.0 para 0.2.0, conforme já registrado como dívida em `DP-006`.

## 2. Escopo

- Tabela nova `public.offers`, vinculada a `workspace_id` e `product_id` (obrigatório), com `brand_id` opcional (herda do produto se não informado).
- Camada de feature completa em `features/offer-engine/`, seguindo o padrão já validado em Products (CS-008) e Brands (CS-009).
- Primeiro artefato do tipo Prompt (`PR-0001`), armazenado em `features/offer-engine/prompts/`, seguindo o formato mínimo definido em STD-0007 0.1.0 (front-matter YAML: `id`, `title`, `target_model`, `objective`, `version`, `status`).
- CRUD completo de Offers (criação manual do texto, edição, remoção, listagem) — infraestrutura necessária independente de a geração por IA estar ligada a um provedor real ou não.
- Estrutura de chamada de IA (função de serviço isolada, ex. `generateOfferCopy()`), preparada para receber um provedor real, mas **sem chamada de API ativa** nesta versão (ver achado bloqueante 4.1).

## 3. Não Escopo

- Chamada real a uma API de IA (Gemini, OpenAI, Anthropic, etc.) — depende de decisão de provedor, ver 4.1. Nesta versão, a "geração" é um placeholder que o usuário preenche manualmente, com o prompt já estruturado e pronto para ligar a um modelo real depois.
- Múltiplas variações de oferta por produto (A/B testing de copy).
- Publicação automática da oferta em canais externos (Instagram/TikTok/marketplace) — escopo do Publishing Engine.
- Custo, latência e critérios de segurança do modelo — dependem de um provedor real escolhido; ver seção 4.3 sobre o que o STD-0007 0.2.0 consegue (e não consegue) cobrir nesta fase.

## 4. Conteúdo Principal

### 4.1 Achado bloqueante identificado nesta auditoria

Confirmado por auditoria direta: não existe tabela `offers` em nenhuma migration, `package.json` não tem nenhum SDK de IA (`@google/generative-ai`, `openai`, `@anthropic-ai/sdk` etc.), e `.env.example` só define `NEXT_PUBLIC_SUPABASE_URL`/`NEXT_PUBLIC_SUPABASE_ANON_KEY` — nenhuma chave de provedor de IA configurada.

Isso **não bloqueia o módulo inteiro** (diferente do caso Auth em SPC-0001): a estrutura de dados, o CRUD manual de ofertas e o prompt já podem ser construídos e ficam prontos para uso. O que fica bloqueado é especificamente a chamada real ao modelo de IA — decisão que não deve ser tomada unilateralmente (qual provedor, como a chave é armazenada em produção). Ver pergunta ao final da resposta a este parecer.

### 4.2 Modelo de dados (nova migração)

```sql
create table if not exists public.offers (
  id           uuid primary key default gen_random_uuid(),
  workspace_id uuid not null references public.workspaces(id) on delete cascade,
  product_id   uuid not null references public.products(id) on delete cascade,
  brand_id     uuid references public.brands(id) on delete set null,
  title        text not null,
  copy         text,
  status       text not null default 'draft' check (status in ('draft', 'generated', 'published')),
  prompt_id    text,
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now()
);

create index if not exists idx_offers_workspace_id on public.offers(workspace_id);
create index if not exists idx_offers_product_id on public.offers(product_id);
create index if not exists idx_offers_brand_id on public.offers(brand_id);
```

`prompt_id` guarda qual Prompt (STD-0007, ex. `PR-0001`) foi usado para gerar aquela oferta — rastreabilidade entre o dado gerado e o artefato de governança que o gerou.

### 4.3 Políticas de RLS (mesmo padrão de `products_member_all`/`brands_member_all`)

```sql
alter table public.offers enable row level security;

create policy "offers_member_all"
  on public.offers for all
  using (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = offers.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  )
  with check (
    exists (
      select 1 from public.workspace_members
      where workspace_members.workspace_id = offers.workspace_id
        and workspace_members.user_id = auth.uid()
    )
  );
```

### 4.4 Primeiro Prompt (`PR-0001`)

Arquivo `features/offer-engine/prompts/offer-copy.md`, seguindo o formato do STD-0007:

```yaml
id: PR-0001
title: Geração de copy de oferta a partir de Product + Brand
target_model: a definir (ver achado bloqueante 4.1)
objective: Gerar um texto de oferta de venda persuasivo a partir dos dados de um produto e do tom de voz de uma marca.
version: 0.1.0
status: Draft
```

Este é o primeiro caso real que expõe uma lacuna do STD-0007 0.1.0: o formato mínimo não define **contrato de entrada** (quais variáveis o prompt exige) nem **contrato de saída** (formato esperado da resposta). Isso alimenta diretamente a expansão para STD-0007 0.2.0 (ver DP-006).

### 4.5 Camada de feature e Server Actions

Replicar o padrão de `features/brands/` (CS-009): `types/offer.types.ts`, `validations/offer.schema.ts`, `services/offer.service.ts`, `queries/`, `mutations/`, `actions/`, `components/`. A política `offers_member_all` depende só de `workspace_members` (já existente antes de qualquer oferta), então o insert pode usar `.select().single()` com segurança — mesmo raciocínio já validado em CS-008/CS-009.

Função de geração isolada: `generateOfferCopy(input: { product: Product; brand?: Brand }): Promise<string>`. Nesta versão, implementação stub que retorna instrução para preenchimento manual — a troca por uma chamada real de IA fica isolada nesta única função, sem impacto no resto da camada.

### 4.6 UI

`OfferForm` (seleciona Product, opcionalmente Brand, permite escrever/colar a copy manualmente), `OfferCard`/`OfferList`, página `app/(app)/offers/page.tsx` — mesmo padrão visual de Products/Brands.

## 5. Dependências

- SPC-0002 — Módulo Products (CS-008)
- SPC-0003 — Módulo Brands (CS-009)
- STD-0007 — Prompt & Script Governance Standard (0.1.0 nesta Specification; gera a expansão para 0.2.0)
- STD-0002, STD-0003 — versionamento e naming

## 6. Critérios de Aprovação

- [ ] Usuário autenticado consegue criar uma Offer vinculada a um Product do seu Workspace, com Brand opcional.
- [ ] Um usuário sem vínculo com o Workspace não consegue ver nem alterar suas Offers (RLS).
- [ ] `PR-0001` existe em `features/offer-engine/prompts/offer-copy.md`, com front-matter válido conforme STD-0007.
- [ ] `generateOfferCopy()` existe como função isolada, mesmo sem chamada de IA real ainda.
- [ ] Listagem, update e delete de Offers funcionam e respeitam a mesma restrição de acesso.
- [ ] `npm run build` e `tsc --noEmit` sem erro.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft — schema novo, achado bloqueante de provedor de IA registrado, primeiro Prompt (PR-0001) criado | Claude |
