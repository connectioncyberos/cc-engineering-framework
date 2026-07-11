---
id: STD-0006
title: Engineering Principles
category: Standard
version: 0.1.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Engineering Framework
repository: cc-engineering-framework
created_at: 2026-07-10
updated_at: 2026-07-10
---

# STD-0006 — Engineering Principles

## 1. Objetivo

Registrar, como lista prescritiva de regras (não narrativa — ver DP-003.1), os princípios de engenharia que já guiaram, na prática, todo o desenvolvimento do ConnectionCyber Commerce Studio e do próprio Engineering Framework. Cada princípio aqui foi extraído de um caso real já vivido neste projeto, não de teoria — a citação do caso que originou o princípio faz parte da definição de cada um.

## 2. Escopo

Princípios aplicáveis a qualquer repositório do ecossistema ConnectionCyber (`cc-engineering-framework`, `cc-commerce-studio`, `cc-shared-framework` e futuros).

## 3. Não Escopo

- Regras de sintaxe/estilo de código — isso é lint/formatter, não princípio de engenharia.
- Detalhes de arquitetura de um módulo específico — isso é Specification (SPC-).
- Processo de aprovação com múltiplos revisores — nenhum princípio aqui assume isso; hoje há um único mantenedor.

## 4. Conteúdo Principal — Princípios

### P1. Especificação antes de código

Nenhum módulo vira código sem uma Specification (SPC-) auditando o estado real do sistema primeiro. Caso real: SPC-0001 a SPC-0004 (Workspace, Products, Brands, Offer Engine) — todas escritas e confirmadas antes da primeira linha de feature.

### P2. Auditar em vez de assumir

Nunca declarar o estado de um schema, dependência ou arquivo sem ler o arquivo real primeiro. Caso real: a auditoria de SPC-0002 revelou que `products` já existia pronto (sem achado bloqueante), enquanto a de SPC-0003 revelou que `brands` não existia — nenhuma das duas conclusões teria sido confiável por suposição.

### P3. Standards nascem mínimos e crescem com casos reais, nunca com especulação

Um Standard começa cobrindo só o que já foi validado; critérios avançados esperam um caso real que os justifique. Caso real: STD-0007 nasceu em 0.1.0 (só local/formato), subiu para 0.2.0 quando SPC-0004 existiu, e para 0.3.0 só depois da integração real com o Gemini — nunca antes. O mesmo raciocínio evitou reescrever a Engineering Bible como narrativa especulativa (DP-003.1).

### P4. Decisões de arquitetura irreversíveis são apresentadas como opções, não decididas unilateralmente

Quando uma escolha tem custo real de retrabalho se errada, ela é levada ao usuário com opções claras, não assumida. Casos reais: autenticação real vs. placeholder (SPC-0001, Opção A/B); escolha de provedor de IA (Gemini vs. OpenAI vs. Anthropic).

### P5. Toda instrução de terminal/Git vem no mesmo instante da mudança de código

Formalizado em POL-0001: nenhuma atualização de arquivo fica sem os comandos exatos para o usuário aplicar — nunca "depois eu te passo".

### P6. Roadmap é reconciliado ao estado real, não ao estado desejado

Um item só é marcado "Done" quando testado end-to-end — não quando o código é escrito. Caso real: CS-007 ficou "In Progress" mesmo com todo o código pronto, até o teste manual confirmar; roadmaps foram reabertos e corrigidos (DP-005, CS-007) quando o texto não batia com a realidade.

### P7. Diagnosticar com evidência, não com suposição, antes de corrigir

Um bug só é corrigido depois de ter uma causa raiz comprovada — nunca por tentativa e erro às cegas. Caso real: o bug de RLS no insert de `workspaces` só foi resolvido depois de decodificar o JWT, fazer um `fetch` cru direto ao PostgREST e rodar uma função de diagnóstico via RPC — três evidências convergentes antes da correção.

### P8. Migrações preferem ser aditivas

Alterar uma tabela existente para suportar um recurso novo não pode quebrar os dados já existentes. Caso real: `brand_id` foi adicionado a `products` como coluna nula (`on delete set null`), preservando produtos já cadastrados sem marca.

### P9. Isolar dependências externas voláteis atrás de uma única função

Uma integração externa (provedor de IA, API de terceiro) fica isolada num único ponto do código, para que trocá-la não exija mudar o resto do módulo. Caso real: `generateOfferCopy()` foi desenhada assim desde o início — quando o modelo `gemini-2.5-flash-lite` foi descontinuado horas depois de escolhido, a correção foi de uma linha, num único lugar.

### P10. Decisões de custo e risco usam dados reais, não estimativa de memória

Preço, limite de free tier e disponibilidade de modelo são verificados no momento da decisão — nunca assumidos de conhecimento desatualizado. Caso real: a escolha do Gemini Flash-Lite e a posterior correção para `gemini-3.1-flash-lite` vieram de busca de preço/disponibilidade atual, não de suposição.

### P11. Versionamento semântico é literal — nada é 1.0.0 até realmente aprovado

Um artefato em Draft continua em Draft até o critério de aprovação declarado ser cumprido de verdade, mesmo que o conteúdo pareça "pronto". Caso real: STD-0007 já está na sua terceira revisão de conteúdo (0.1 → 0.3) e continua Draft, porque o critério de aprovação (validado por um segundo Engine real) ainda não aconteceu.

### P12. Erros do processo são nomeados, não escondidos

Quando um passo do próprio processo falha (um `git add` incompleto, um `ROADMAP.md` desatualizado, um encoding quebrado num comentário SQL), o erro é reconhecido explicitamente e corrigido — nunca silenciado ou reescrito como se não tivesse acontecido. Casos reais: a correção do `git add -A` depois do esquecimento de `features/auth`; a reconciliação de CS-007/DP-005 quando o roadmap não refletia a realidade; o erro de encoding na migração 003.

## 5. Dependências

- STD-0001 — Documentation & Artifact Standard
- STD-0002 — Versioning Standard
- DP-001 a DP-010 (`cc-engineering-framework/ROADMAP.md`) — fonte primária destes princípios

## 6. Critérios de Aprovação

Permanece em Draft até ser citado/aplicado explicitamente em pelo menos uma Specification futura (fora do escopo do MVP atual, ver DP-010) como referência de decisão — confirmando que os princípios são realmente usados, não só documentados.

## 7. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft — 12 princípios extraídos da execução real do projeto (DP-001 a DP-010) | Claude |
