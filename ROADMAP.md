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
