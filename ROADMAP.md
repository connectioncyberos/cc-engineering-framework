# Roadmap — ConnectionCyber Engineering Framework

## DP-001 — Repository Bootstrap

**Objetivo:** inicializar o repositório e sua estrutura base.

### Entregáveis

- README.md
- LICENSE
- CHANGELOG.md
- ROADMAP.md
- CONTRIBUTING.md
- CODE_OF_CONDUCT.md
- .gitignore
- .editorconfig
- .gitattributes
- Estrutura inicial de diretórios
- Templates iniciais do GitHub

**Status:** Done

## DP-002 — Engineering Standards

**Objetivo:** criar os primeiros padrões oficiais do framework.

### Entregáveis

- STD-0001 — Documentation & Artifact Standard
- STD-0002 — Versioning Standard
- STD-0003 — Naming Convention Standard
- STD-0004 — Repository Organization Standard
- STD-0005 — Markdown Style Guide

**Status:** Done

## DP-003 — Foundation Artifacts

**Objetivo:** criar os artefatos fundacionais do programa.

### Reconciliação (2026-07-09)

O plano original previa FND-0001 Project Charter, FND-0002 Engineering Bible, FND-0003 Engineering Handbook, FND-0004 Product Vision, FND-0005 Product Constitution. A execução real seguiu ordem diferente: Charter e Vision foram entregues, e Mission foi criado sem estar no plano original. A numeração de arquivo segue STD-0003 (ordem de criação dentro da pasta `foundations/`), não a posição original do plano.

### Entregáveis

- `0001-project-charter.md` — **Done**
- `0002-mission.md` — **Done** (não previsto no plano original; adicionado por necessidade prática)
- `0003-vision.md` — **Done** (equivale ao antigo FND-0004 do plano original)
- `0004-product-constitution.md` — **Done** (ver DP-010, item 6). Diferente de Charter/Mission/Vision (sobre o Framework), este é sobre o produto Commerce Studio: propósito, público-alvo, não escopo permanente e princípios não negociáveis.
- Engineering Handbook — **Planned** (a escrever por ondas — só capítulos com conteúdo real agora: Introdução, Filosofia, Specifications, ADR, Versionamento. História/Evolução/Roadmap ficam como stub até haver o que registrar)
- ~~Engineering Bible~~ — descontinuado, ver DP-003.1
- STD-0006 — Engineering Principles — **Done** (ver DP-010, item 5). 12 princípios extraídos da execução real do projeto (DP-001 a DP-010).

### DP-003.1 — Resolução: Handbook vs. Bible vs. Principles (2026-07-09)

Analisando o conteúdo já definido para Foundation ("artefato que define identidade, propósito, princípios e base estratégica" — STD-0001), o Engineering Bible tal como proposto seria redundante com o que Mission, Vision e Product Constitution já cobrem. Resolução:

- **Engineering Handbook** — mantido. É o índice/navegação único do ecossistema (mapa de Specifications, dependências, roadmap, histórico). Não contém conteúdo completo, só aponta para os artefatos canônicos.
- **Engineering Bible** — descontinuado como artefato próprio. Seu conteúdo pretendido (filosofia narrada) já é coberto por Mission, Vision e Product Constitution.
- **Engineering Principles** — deixa de ser parte do Handbook e passa a ser um Standard (`STD-0006 — Engineering Principles`), já que é uma lista prescritiva de regras, não narrativa.

**Status:** Resolvido

## DP-004 — Delivery Package History

**Objetivo:** preservar o histórico de cada entrega, em vez de sobrescrever um único `EXECUTE.md`/`VALIDATION.md`/`RELEASE_NOTES.md` na raiz.

### Entregáveis

- Pasta `releases/` — **Done**
- `releases/Release-0001.md` (retroativo, cobrindo DP-001, DP-002, FP-001 e o Housekeeping da Fase A) — **Done**
- Convenção: cada nova entrega gera seu próprio `Release-000X.md`; `EXECUTE.md`/`VALIDATION.md`/`RELEASE_NOTES.md` na raiz sempre descrevem o pacote **em andamento**, e são arquivados em `releases/` quando o pacote fecha

**Status:** Done

## DP-005 — Primeira Specification real

**Objetivo:** validar o processo de Specification na prática, com um caso real e imediato.

### Entregáveis

- SPC-0001 — Especificação da Persistência do Workspace (CS-007), cobrindo Server Action, validação Zod, RLS/multi-tenant e critério de aceite — **Done**. Implementação em `cc-commerce-studio` (CS-007) concluída e testada end-to-end.

**Status:** Done

## DP-006 — Governança de Prompts e Scripts

**Objetivo:** dar um local e um formato oficial para prompts de IA e scripts de automação usados no ecossistema, antes de o primeiro Engine de IA (CS-008+) ser construído.

### Entregáveis

- STD-0007 — Prompt & Script Governance Standard (0.1.0, Draft) — **Done**. Cobre local de armazenamento (`features/<engine>/prompts/` no produto, `scripts/` para automação), formato mínimo (front-matter YAML) e metadados obrigatórios. Critérios de avaliação/aprovação ficam explicitamente fora de escopo nesta versão.
- STD-0003 — ajuste na tabela de tipos oficiais, adicionando `PR` (Prompt) e `SCR` (Script) — **Done**.

### Dívida futura registrada

STD-0007 sobe para 0.2.0 (MINOR, expansão compatível, conforme STD-0002) quando o primeiro Engine real (Offer Engine, candidato natural em CS-008+) tiver sua própria Specification — nesse momento, incorporar os critérios reais de avaliação (qualidade, custo, segurança) antes de um prompt entrar em produção.

**Status:** Done (versão 0.1.0) — 0.2.0 depende de CS-008+ ter uma Specification real.

## DP-007 — Specification do módulo Products

**Objetivo:** especificar o próximo módulo do produto (CS-008), com prioridade definida por dependência real de schema, não por preferência.

### Entregáveis

- SPC-0002 — Módulo Products (CS-008) — **Done**. Auditoria confirmou que `products` (tabela + RLS) já existe e já está aplicada desde a migration 002 (CS-007) — sem achado bloqueante. Trabalho restante é só camada de feature + UI.

### Prioridade de módulos (CS-008+) — decisão registrada

Ordem definida por dependência de schema, verificada nas migrations reais (não suposição): Products (schema/RLS prontos) → Brands (exige schema novo + migração aditiva de `brand_id` em `products`) → Offer Engine (consome os dois; primeiro caso real de STD-0007).

**Status:** Done

## DP-008 — Specification do módulo Brands

**Objetivo:** especificar o módulo Brands (CS-009), próximo na ordem confirmada em DP-007.

### Entregáveis

- SPC-0003 — Módulo Brands (CS-009) — **Done**. Auditoria confirmou que não existe tabela `brands` nem coluna `brand_id` em `products` — schema novo necessário, migração de `products` é aditiva (`brand_id` nullable, `on delete set null`), sem regressão em CS-008.

**Status:** Done

## DP-009 — Specification do Offer Engine e expansão do STD-0007

**Objetivo:** especificar o primeiro Engine de IA (CS-010) e usar esse caso real para cumprir a dívida já registrada em DP-006 (expansão do STD-0007 para 0.2.0).

### Entregáveis

- SPC-0004 — Offer Engine (CS-010) — **Done**. Auditoria confirmou schema novo necessário (`offers`) e nenhum provedor de IA configurado (achado não-bloqueante para o módulo, bloqueante apenas para a chamada real de IA).
- STD-0007 — expandido para 0.2.0 — **Done**. Adicionado contrato de entrada/saída, ciclo de vida do prompt (Draft/Ready/Active/Deprecated) e checklist de revisão. Critérios quantitativos (custo/latência/segurança) adiados para 0.3.0, quando houver provedor de IA real escolhido.

**Status:** Done

## DP-010 — Escopo final do MVP (decisão do usuário, 2026-07-10)

**Objetivo:** definir o que significa "projeto finalizado" nesta fase, antes de continuar abrindo módulos novos.

### Decisão

MVP fechado = Workspace + Products + Brands + Offer Engine (já entregues, CS-007 a CS-010) **com um provedor de IA real ligado** ao Offer Engine, mais as duas dívidas de governança já registradas como Planned (`STD-0006`, Product Constitution).

Os 8 módulos restantes do CS-008+ (Landing Page, Creative, Video Script, Marketplace, Email/WhatsApp, Publishing, Analytics, Quality Engine + Prompt Lab) ficam **fora do escopo de "finalizar"** nesta rodada — permanecem documentados no roadmap como próximas fases, sem Specification ainda.

### Plano de ação (ordem de execução)

1. Configurar provedor de IA real (Gemini API — free tier) e ligar ao `generateOfferCopy()` do Offer Engine. — **Concluído**. Chave do Google AI Studio configurada, `@google/genai` instalado, `gemini-3.1-flash-lite` (o `gemini-2.5-flash-lite` inicialmente escolhido havia sido descontinuado para novos usuários — corrigido).
2. Testar geração real de oferta end-to-end. — **Concluído**. Copy real gerada para "Galpão ConnectionCyber", tom de voz da marca respeitado, oferta salva com sucesso.
3. Promover `PR-0001` de `Ready` para `Active` (ciclo de vida do STD-0007). — **Concluído**. Versão 0.2.1 do prompt.
4. Expandir `STD-0007` para 0.3.0 com critérios quantitativos reais (custo, latência, segurança), usando a integração real como caso concreto. — **Concluído**. Adicionados: custo de referência (Gemini 3.1 Flash-Lite, US$ 0,25/US$ 1,50 por 1M tokens), latência de referência, tratamento obrigatório de resposta vazia/bloqueada e de falha de rede/cota, campo `model_verified_at`. Código de `generateOfferCopy()`/`generateOfferCopyAction()` atualizado para cumprir esses critérios.
5. Escrever `STD-0006 — Engineering Principles` (dívida de DP-003, nunca escrita). — **Concluído**. 12 princípios, cada um citando o caso real deste projeto que o originou.
6. Escrever o Product Constitution (dívida de DP-003, nunca escrita). — **Concluído**. `FND-0004`, cobrindo propósito, público-alvo, não escopo permanente e 5 princípios não negociáveis do produto.
7. Decidir com o usuário se testes automatizados e CI entram no escopo do MVP ou ficam de fora (hoje em Backlog, sem decisão). — **Concluído**. Decisão do usuário: ficam para depois. Backlog consciente, não esquecimento — ver `cc-commerce-studio/ROADMAP.md`, seção Transversais.
8. Consolidar um Release final do MVP (`releases/Release-000X.md`) marcando o fechamento desta fase. — **Concluído**. `releases/Release-0002.md`.

**Status:** Done — os 8 itens do plano de ação concluídos. MVP fechado: Workspace + Products + Brands + Offer Engine com IA real, STD-0006, FND-0004, decisão consciente sobre testes/CI, Release-0002 consolidado.

## DP-011 — Specification do Landing Page Engine (retomada pós-MVP)

**Objetivo:** especificar o primeiro dos 8 módulos que ficaram fora do escopo do MVP (DP-010), retomando a construção do produto.

### Entregáveis

- SPC-0005 — Landing Page Engine (CS-011) — **Done**. Auditoria revelou dois achados não bloqueantes: (1) rota pública não exige mudança de middleware, só ficar fora de `app/(app)/`; (2) RLS existente não cobre acesso público — nova política de select restrita a `status = 'published'` resolve sem enfraquecer o isolamento multi-tenant.

**Status:** Done

## DP-012 — Ordem de retomada dos módulos restantes (decisão do usuário)

**Objetivo:** registrar a decisão do usuário sobre a ordem de construção dos 7 módulos que ainda restam fora do escopo do MVP (DP-010), após CS-011.

### Decisão (2026-07-11)

Critério proposto por Claude e escolhido pelo usuário: priorizar módulos que não exigem nenhuma conta/API externa nova, reaproveitando só a infraestrutura já ativa (Supabase + Gemini). Ordem confirmada: **Video Script Engine → Creative Engine → Analytics Engine**. Isso altera a ordem original combinada (Creative Engine viria antes de Video Script Engine); Creative Engine provavelmente exige geração de imagem (API separada/billing), por isso foi adiado até ser auditado.

### Entregáveis

- SPC-0006 — Video Script Engine (CS-012) — **Done**. Auditoria confirmou reaproveitamento total da infraestrutura de IA já ativa (`generateOfferCopy()` como referência); nenhuma rota pública necessária (diferente do Landing Page Engine).
- CS-012 testado end-to-end em `cc-commerce-studio`: roteiro real gerado via `PR-0002`/Gemini 3.1 Flash-Lite, salvo com sucesso.
- STD-0007 promovido de 0.3.0 (Draft) para **1.0.0 (Approved)** — critério de aprovação da seção 6 cumprido: PR-0002, de um Engine diferente (Video Script Engine), seguiu o Standard sem exigir nenhuma expansão nova.

**Status:** Done

## DP-013 — Escopo real do Creative Engine (decisão de arquitetura e custo)

**Objetivo:** auditar e decidir o escopo técnico real do Creative Engine (CS-013), a partir do fluxo manual que o usuário já usa hoje (referências de imagem + descrição → imagens de criativo → vídeo).

### Achados da auditoria (pesquisa real, 2026-07-11)

- **Geração de imagem** — modelos "Nano Banana" (`gemini-3.1-flash-image`) e "Nano Banana Pro" (`gemini-3-pro-image-preview`) usam a **Interactions API** (`ai.interactions.create()`), não a `generateContent()` já usada em PR-0001/PR-0002. Nano Banana Pro aceita até 14 imagens de referência — cobre o caso de uso real do usuário. Custo: US$ 0,034–US$ 0,134 por imagem (sem tier gratuito).
- **Geração de vídeo** — modelo Veo 3.1 (`veo-3.1-generate-preview`) usa `ai.models.generateVideos()`, que retorna uma operação assíncrona de longa duração (o próprio exemplo oficial faz polling a cada 10s até `operation.done`). Custo: US$ 0,03–US$ 0,60 por segundo de vídeo.
- **Implicação arquitetural real** — geração de vídeo não cabe numa Server Action síncrona única (o polling do exemplo oficial já passa de 10s, um vídeo real pode levar minutos). É preciso desenho assíncrono: guardar o `operation` e um campo de status na tabela, e o cliente chamar uma Server Action de "verificar status" repetidamente (poll), em vez de bloquear uma única chamada — diferente do padrão síncrono usado em Offer Engine/Video Script Engine.
- **Achado bloqueante** — nem geração de imagem nem de vídeo têm tier gratuito (diferente do texto usado até agora). É necessário habilitar billing na conta Google AI Studio/Vertex antes de qualquer teste real. `@google/genai` instalado é `^2.11.0`; precisa ser auditado/atualizado para confirmar suporte à Interactions API e a `generateVideos()`.

### Decisão do usuário (2026-07-11)

Escopo completo agora: imagem (com referências) + vídeo a partir das imagens, replicando o fluxo manual já usado — não uma versão reduzida só de imagem.

### Entregáveis

- SPC-0007 — Creative Engine (CS-013) — em andamento.

**Status:** In Progress

## DP-014 — Escopo real do Billing/Subscription Engine (achado antecipado, execução adiada)

**Objetivo:** registrar a análise real de um mecanismo de cobrança recorrente para o Commerce Studio, feita a partir de um ativo já validado em produção em outro projeto (VaultMindOS), para não perder o achado — a execução fica para depois do Creative Engine (CS-013), por decisão do usuário.

### Achado (2026-07-11) — ativo reaproveitável real

O repositório `vaultmindos` (connectioncyberos) tem uma integração Mercado Pago **Checkout Pro** validada em sandbox: `fetch` nativo (sem SDK), cria uma "preferência" via `POST /checkout/preferences`, redireciona o comprador para a página hospedada do Mercado Pago, e nunca confia no status vindo do webhook ou da query string — sempre reconsulta `GET /v1/payments/{id}` antes de liberar acesso. RLS: usuário só insere e lê o próprio registro; só a service role (fora de qualquer Server Action disparada por usuário) atualiza o status.

### O que se aplica direto ao Commerce Studio (sem mudança)

Os quatro princípios de segurança: reconsulta obrigatória do status real antes de liberar algo; service role isolada só no reconciliador; RLS sem policy de update para o usuário; registro em audit log só quando o status muda de fato.

### O que muda (Commerce Studio cobra assinatura recorrente, não produto único)

O VaultMindOS cobra uma vez (`Checkout Pro`, `createPreference`). O Commerce Studio precisa de **Assinaturas** do Mercado Pago (endpoint `/preapproval`, não `/checkout/preferences`), com vocabulário de status próprio (`authorized`/`paused`/`cancelled`/`pending`, não `PENDING`/`APPROVED`/`REJECTED`/`CANCELLED`) e tipo de evento de webhook diferente (`subscription_preapproval`, não `payment`). A tabela `payments` vinculada a `user_id`/`course_id` vira `subscriptions` vinculada a `workspace_id` (cobrança por tenant, não por usuário individual — mesmo isolamento multi-tenant já usado em todo o resto do produto). Faltam ainda duas peças novas que o VaultMindOS não precisa: uma tabela `plans` (tiers e limites de uso) e uma medição de uso por Workspace por ciclo de cobrança, para aplicar o limite do plano.

### Decisões do usuário (2026-07-11)

- **Prioridade:** terminar o Creative Engine (CS-013) primeiro; Billing/Subscription Engine fica para depois.
- **Conta Mercado Pago:** o Commerce Studio usa uma aplicação/conta separada da do VaultMindOS — isolamento completo de credenciais entre os dois produtos.

**Status:** Backlog (achado documentado, execução adiada por decisão do usuário — não é dívida esquecida)

## DP-015 — Auditoria arquitetural completa e Playbook de melhoria contínua

**Objetivo:** auditar `cc-commerce-studio` de ponta a ponta (arquitetura, requisitos, documentação, código, segurança, observabilidade, governança) e transformar os achados num programa executável e mensurável, não só num relatório.

### Achados reais (auditoria 2026-07-11)

- Tabelas `projects`/`assets` existem no schema (migração 001) sem nenhum módulo de feature correspondente; o link "Assets" no Sidebar do produto aponta para uma rota inexistente (404 real). **Correção (2026-07-11, execução da Fase 0 do PBK-0001):** durante a execução, um segundo link quebrado foi encontrado — "Dashboard" (`/dashboard`) também não existia (o login redireciona para `/workspace`). Eram 2 links quebrados, não 1. Ambos removidos do Sidebar; tabelas mantidas no schema sem remoção destrutiva.
- Zero CI (`.github/` só tem `PULL_REQUEST_TEMPLATE.md`), zero teste automatizado (`tests/` só tem `.gitkeep`), zero observabilidade estruturada (só `console.error`).
- RLS é a única camada de autorização multi-tenant e não tem nenhum teste automatizado validando isolamento entre Workspaces.
- Fallback silencioso de env var ausente (`placeholder.supabase.co`) no middleware do Supabase, em vez de falha explícita.
- Cada Engine de IA duplica seu próprio tratamento de erro de provedor — risco de triplicar a mesma lógica quando o Creative Engine for construído.
- Governança (Standards, Specifications, Roadmaps reconciliados) está muito acima da maturidade técnica de produção — ponto forte real, não um problema.

### Entregáveis

- `Auditoria-Arquitetural-ConnectionCyber-Commerce-Studio-2026-07-11.md` — relatório completo nos 8 blocos solicitados pelo usuário, entregue como arquivo.
- `PBK-0001` — Quality & Technical Debt Improvement Playbook: decisões justificadas com trade-off, plano de ação em 4 fases (Imediato/Curto/Médio/Longo prazo) com Definition of Done por item, modelo de métricas com baseline e meta, e ritual de governança que reaproveita o próprio ciclo de fechamento de Delivery Package do ROADMAP (sem processo novo).

**Status:** In Progress — auditoria e Playbook entregues; Fase 0 do PBK-0001 concluída (2026-07-11): CI criado, 2 links quebrados corrigidos no Sidebar. Fases 1 a 3 ainda não iniciadas.

## Dependência cruzada

O roadmap de produto (Commerce Studio) é mantido separadamente em `cc-commerce-studio/ROADMAP.md`. Este arquivo cobre apenas o Engineering Framework.

## Histórico de alterações

| Data | Alteração |
|------|-----------|
| 2026-07-07 | Versão inicial (DP-001 a DP-003 planejados) |
| 2026-07-09 | DP-001/DP-002 marcados como concluídos; DP-003 reconciliado com a execução real; DP-004 e DP-005 adicionados |
| 2026-07-10 | DP-006 adicionado: STD-0007 (Prompt & Script Governance Standard) criado, STD-0003 ajustado com tipos PR/SCR |
| 2026-07-10 | DP-005 reconciliado para Done: SPC-0001 confirmada e implementada em CS-007 |
| 2026-07-10 | DP-008 adicionado: SPC-0003 (Brands) escrita, schema novo + migração aditiva em products |
| 2026-07-10 | DP-009 adicionado: SPC-0004 (Offer Engine) escrita, STD-0007 expandido para 0.2.0 |
| 2026-07-10 | DP-007 adicionado: SPC-0002 (Products) escrita; prioridade de módulos CS-008+ decidida por dependência de schema |
| 2026-07-10 | DP-010 adicionado: escopo final do MVP decidido pelo usuário (Workspace+Products+Brands+Offer Engine com IA real + STD-0006 + Product Constitution); demais 8 módulos ficam fora do escopo de "finalizar" |
| 2026-07-10 | STD-0007 expandido para 0.3.0: critérios quantitativos reais (custo, latência, segurança) a partir da integração PR-0001/Gemini 3.1 Flash-Lite |
| 2026-07-10 | STD-0006 (Engineering Principles) escrito — dívida de DP-003 fechada; 12 princípios extraídos da execução real (DP-001 a DP-010) |
| 2026-07-10 | FND-0004 (Product Constitution) escrito — dívida de DP-003 fechada |
| 2026-07-10 | Decisão do usuário: testes automatizados/CI ficam fora do MVP (backlog consciente) |
| 2026-07-10 | DP-010 fechado (Done); Release-0002 consolidado, MVP encerrado |
| 2026-07-10 | DP-011 adicionado: SPC-0005 (Landing Page Engine) escrita, retomando os módulos fora do MVP |
| 2026-07-11 | DP-012 adicionado: ordem de retomada decidida pelo usuário (Video Script Engine → Creative Engine → Analytics Engine); SPC-0006 (Video Script Engine) escrita |
| 2026-07-11 | DP-012 fechado (Done): CS-012 testado end-to-end; STD-0007 promovido de 0.3.0 (Draft) para 1.0.0 (Approved) |
| 2026-07-11 | DP-013 adicionado: escopo real do Creative Engine auditado (Interactions API para imagem, geração de vídeo assíncrona via Veo, custos reais sem tier gratuito); usuário decidiu escopo completo (imagem + vídeo) |
| 2026-07-11 | DP-014 adicionado: análise do ativo Mercado Pago do VaultMindOS para um futuro Billing/Subscription Engine; execução adiada (Creative Engine primeiro), conta Mercado Pago será separada |
| 2026-07-11 | DP-015 adicionado: auditoria arquitetural completa do cc-commerce-studio entregue, e PBK-0001 (Quality & Technical Debt Improvement Playbook) escrito com plano de ação faseado e métricas mensuráveis |
| 2026-07-11 | DP-015: Fase 0 do PBK-0001 executada — CI criado em cc-commerce-studio; 2 links quebrados no Sidebar corrigidos (não 1 — "Dashboard" encontrado durante a execução, corrigindo a auditoria original) |
