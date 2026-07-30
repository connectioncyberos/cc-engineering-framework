---
id: PBK-0001
title: Quality & Technical Debt Improvement Playbook
category: Playbook
version: 0.1.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Engineering Framework
repository: cc-engineering-framework
created_at: 2026-07-11
updated_at: 2026-07-11
---

# PBK-0001 — Quality & Technical Debt Improvement Playbook

## 1. Objetivo

Transformar os achados da auditoria arquitetural de 2026-07-11 (`cc-commerce-studio`) em um programa executável, com decisões justificadas, plano de ação priorizado e faseado, e um modelo de medição contínua — para que a melhoria de qualidade não dependa de lembrança, e sim de um processo repetível, com metas numéricas e cadência de revisão.

## 2. Escopo

Cobre qualidade de código, requisitos não funcionais, documentação, segurança, observabilidade, escalabilidade e governança do `cc-commerce-studio`, a partir do estado real encontrado na auditoria. Cada item tem: decisão, justificativa técnica, trade-off, fase de execução, Definition of Done e métrica associada.

## 3. Não Escopo

Não cobre a construção de novas features de produto (Creative Engine, Billing Engine — esses continuam em suas próprias Specifications, SPC-0007 em diante). Não introduz processo pesado (não propõe Scrum, não propõe ferramenta de gestão nova) — o mecanismo de acompanhamento reaproveita o que já existe e funciona (ROADMAP.md reconciliado a estado real).

## 4. Conteúdo Principal

### 4.1 Decisões, justificativas e trade-offs

| # | Achado (auditoria 2026-07-11) | Decisão | Justificativa técnica | Trade-off aceito |
|---|---|---|---|---|
| D1 | Zero CI (`.github/` só tem PR template) | Adicionar workflow de GitHub Actions rodando `tsc --noEmit` + `next build` a cada push/PR | Falha de tipo ou de build hoje só é pega se o usuário rodar manualmente; CI pega isso antes do merge, sem exigir mudança de arquitetura | Nenhum custo real (Actions grátis para repositório privado dentro do limite de minutos); único trade-off é manutenção do workflow em si |
| D2 | Tabelas `projects`/`assets` no schema sem feature correspondente, link "Assets" quebrado no Sidebar | Decidir explicitamente entre (a) construir os módulos ou (b) remover as tabelas via migração aditiva de desativação + remover o link do Sidebar | Rastreabilidade Specification→schema→código, que o resto do projeto mantém, está quebrada aqui; um cliente pagante pode clicar no link quebrado no primeiro dia | Se optar por (b), perde-se o schema já criado (mitigável: nunca houve dado real nessas tabelas, custo de remoção é zero) |
| D3 | Zero teste automatizado sobre isolamento de RLS entre Workspaces | Escrever ao menos um teste de integração por tabela, confirmando que um usuário do Workspace A não lê/escreve dado do Workspace B | RLS é a única camada de autorização do sistema; um erro de política numa tabela nova hoje só seria descoberto por acidente ou incidente real | Exige instalar um framework de teste (Vitest é o mais leve para Next.js) — primeira dependência de teste do projeto, decisão pequena mas real |
| D4 | Fallback silencioso de `NEXT_PUBLIC_SUPABASE_URL`/chave ausente no middleware (`placeholder.supabase.co`) | Trocar por falha explícita (throw) em build/boot quando a env var estiver ausente em produção | Fail-fast é mais seguro que mascarar; hoje, uma env var esquecida na Vercel derruba o app de forma confusa em vez de falhar no boot com mensagem clara | Nenhum — é estritamente uma melhoria, sem custo de reversibilidade |
| D5 | Cada Engine de IA (`offer.service.ts`, `video-script.service.ts`, e o futuro Creative Engine) reimplementa o mesmo tratamento de erro de provedor | Extrair um cliente de IA compartilhado (`lib/ai/gemini-client.ts`) antes de escrever o código do Creative Engine | Evita triplicar a mesma lógica de erro de rede/cota/bloqueio pela terceira vez (STD-0007 §4.8 já formaliza esse tratamento como obrigatório) | Exige refatorar os dois módulos já existentes para usar o cliente compartilhado — risco baixo (mesma lógica, só reorganizada), mas é retrabalho real, não trabalho novo |
| D6 | Zero observabilidade estruturada (só `console.error`) | Adotar um padrão mínimo de log estruturado (`workspace_id`, `action`, `status`, `duration_ms`) em toda Server Action que chama IA ou processa pagamento, usando os logs nativos da Vercel (sem ferramenta paga nova) | Hoje, todo problema é descoberto pelo usuário testando manualmente na UI; não escala para clientes que não são o próprio dono do produto | Nenhum custo de ferramenta; custo é só disciplina de adicionar o log em cada Server Action nova daqui para frente |

### 4.2 Plano de ação priorizado (fases)

**Fase 0 — Imediato (antes de qualquer novo módulo de receita), custo baixo, risco zero — Concluída em 2026-07-11**

| Item | Ação | Definition of Done | Métrica |
|---|---|---|---|
| D1 | Criar `.github/workflows/ci.yml` rodando `npm run typecheck` e `npm run build` | **Feito e confirmado.** Workflow criado em `cc-commerce-studio/.github/workflows/ci.yml`; primeiro run real (commit `0a3585a`) passou verde em 55s. | `CI status`: de "não existe" para "verde no primeiro push real" |
| D2 | Decidir destino de `projects`/`assets` e corrigir o Sidebar | **Feito.** Um segundo link quebrado foi encontrado durante a execução (não capturado na auditoria original): "Dashboard" (`/dashboard`) também não existia — o login redireciona para `/workspace`. Ambos os itens ("Dashboard" e "Assets") foram removidos do Sidebar. Decisão sobre as tabelas: manter no schema sem remoção destrutiva, documentado aqui e no DP-015 — não construir os módulos agora, sem apagar dado/estrutura. | `Links quebrados no Sidebar`: de 2 (não 1 — achado corrigido) para 0 |

**Fase 1 — Curto prazo (antes do Creative Engine)**

| Item | Ação | Definition of Done | Métrica |
|---|---|---|---|
| D5 | Extrair `lib/ai/gemini-client.ts`, migrar Offer Engine e Video Script Engine para usá-lo | Os dois módulos existentes usam o cliente compartilhado; nenhuma regressão nos testes manuais já validados | `Duplicação de tratamento de erro de IA`: de 2 implementações para 1 |
| D4 | Falha explícita de env var ausente em produção | Deploy sem `NEXT_PUBLIC_SUPABASE_URL` falha no boot com mensagem clara, não silenciosamente | `Falhas silenciosas de config`: 0 |

**Fase 2 — Médio prazo (antes do primeiro cliente pagante real / antes do Billing Engine ir ao ar)**

| Item | Ação | Definition of Done | Métrica |
|---|---|---|---|
| D3 | Instalar Vitest; escrever teste de isolamento de RLS para cada tabela multi-tenant | 100% das tabelas com RLS (`workspaces`, `products`, `brands`, `offers`, `landing_pages`, `video_scripts`, + novas) têm ao menos 1 teste de isolamento | `Cobertura de teste de RLS`: 0/6 → 6/6 (e 1/1 a cada tabela nova daqui em diante) |
| D6 | Log estruturado em toda Server Action de IA/pagamento | Todo Server Action que chama Gemini ou (futuramente) Mercado Pago loga `workspace_id`/`action`/`status`/`duration_ms` | `% de Server Actions críticas com log estruturado`: 0% → 100% |

**Fase 3 — Longo prazo (quando houver volume real de uso)**

| Item | Ação | Definition of Done | Métrica |
|---|---|---|---|
| — | Dashboard de custo de IA por Workspace/mês | Visível antes do Billing Engine cobrar de terceiros, para validar margem real vs. estimada | `Custo de IA por Workspace/mês` vs. `Receita por Workspace/mês` |
| — | Avaliar extração do job de vídeo (Creative Engine) para função dedicada, se o volume justificar | Decisão revisitada com dado real de uso, não antecipada sem necessidade | `Tempo médio de geração de vídeo` e `taxa de timeout` |

### 4.3 Modelo de medição contínua

Métricas de baseline (capturadas na auditoria de 2026-07-11) e meta:

| Métrica | Baseline (2026-07-11) | Estado atual (2026-07-11, pós-Fase 0) | Meta | Onde é observada |
|---|---|---|---|---|
| CI status por push | Não existe | Confirmado verde no primeiro push real (commit `0a3585a`, 55s) | 100% dos pushes com `tsc --noEmit` + `next build` verdes antes de merge | GitHub Actions |
| Links quebrados no Sidebar | 2 (`/assets` e `/dashboard` — o segundo só foi encontrado durante a execução da Fase 0, não estava na auditoria original) | 0 | 0 | Revisão manual a cada módulo novo (checklist de Specification) |
| Tabelas com schema órfão (sem feature ou decisão explícita) | 2 (`projects`, `assets`) | 0 | Comparação migração ↔ `features/` a cada Specification |
| Cobertura de teste de isolamento de RLS | 0/6 tabelas | 6/6 hoje, +1 a cada tabela nova | Suíte de testes (Vitest) |
| Duplicação de tratamento de erro de IA | 2 implementações independentes | 1 cliente compartilhado | Revisão de código a cada novo Engine de IA |
| Server Actions críticas com log estruturado | 0% | 100% das que chamam IA ou pagamento | Revisão de código a cada Server Action nova |
| Custo de IA por Workspace/mês | Não medido (não há cobrança ainda) | Medido antes do Billing Engine cobrar de terceiros | A definir junto com SPC do Billing Engine |

### 4.4 Ritual de governança contínua

Este Playbook não cria um processo novo de acompanhamento — reaproveita o que já funciona neste projeto: toda vez que um Delivery Package (`DP-XXX`) for fechado no `ROADMAP.md` do `cc-engineering-framework`, a tabela de métricas da seção 4.3 deve ser revisada e atualizada na mesma entrada de histórico, exatamente como já acontece com o estado de cada módulo de produto. Isso significa: nenhuma reunião nova, nenhuma ferramenta nova — a métrica vira parte do mesmo "fechar com estado real, não aspiracional" que já rege o resto do projeto (STD-0006, Princípio P5).

## 5. Dependências

- STD-0006 — Engineering Principles (P5, roadmap reconciliado a estado real; P9, isolar dependência externa volátil)
- STD-0007 — Prompt & Script Governance Standard (1.0.0, tratamento de erro de IA já formalizado, base do item D5)
- Auditoria arquitetural de 2026-07-11 (`Auditoria-Arquitetural-ConnectionCyber-Commerce-Studio-2026-07-11.md`) — fonte de todos os achados deste Playbook

## 6. Critérios de Aprovação

- [x] Fase 0 concluída e confirmada (2026-07-11): CI verde no primeiro push real (`0a3585a`, 55s), nenhum link quebrado no Sidebar (2 encontrados e corrigidos, não 1).
- [ ] Fase 1 concluída: cliente de IA compartilhado em uso pelos dois Engines existentes; falha explícita de env var.
- [ ] Fase 2 concluída: 100% das tabelas com RLS cobertas por teste de isolamento; log estruturado em Server Actions críticas.
- [ ] Tabela de métricas da seção 4.3 revisada e atualizada a cada fechamento de Delivery Package no ROADMAP.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-11 | Primeira versão Draft, a partir da síntese da auditoria arquitetural de 2026-07-11 | Claude |
| 0.1.0 | 2026-07-11 | Fase 0 executada e reconciliada: CI criado; 2 links quebrados corrigidos no Sidebar (não 1 — "Dashboard" foi encontrado durante a execução, não estava na auditoria original) | Claude |
