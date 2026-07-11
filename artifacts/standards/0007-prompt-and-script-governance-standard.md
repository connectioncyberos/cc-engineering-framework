---
id: STD-0007
title: Prompt & Script Governance Standard
category: Standard
version: 1.0.0
status: Approved
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Engineering Framework
repository: cc-engineering-framework
created_at: 2026-07-10
updated_at: 2026-07-11
---

# STD-0007 — Prompt & Script Governance Standard

## 1. Objetivo

Definir onde prompts de IA e scripts de automação do ecossistema ConnectionCyber são armazenados, seu formato mínimo, os metadados obrigatórios, o contrato de entrada/saída, o ciclo de vida do prompt, e — a partir desta versão (0.3.0) — os critérios quantitativos reais (custo, latência, segurança) medidos contra um provedor de IA em produção. Esta expansão foi feita a partir de um caso real e completo: `PR-0001` (Offer Engine) ligado ao Gemini 3.1 Flash-Lite e testado end-to-end, não especulativamente.

## 2. Escopo

- Prompts de IA usados pelos Engines de produto (Offer Engine, Creative Engine, Video Script Engine, etc.), definidos em `cc-commerce-studio/ROADMAP.md` (CS-008+).
- Scripts de manutenção, automação e apoio ao desenvolvimento dos repositórios do ecossistema.
- Contrato de entrada/saída e checklist de revisão de um prompt antes de uso.
- Critérios quantitativos de custo, latência e segurança de conteúdo, e tratamento de erro/indisponibilidade do provedor (novo nesta versão).

## 3. Não Escopo (ainda nesta versão)

- Ferramentas de teste automatizado de prompts (prompt evals) — segue fora de escopo; os critérios desta versão são de revisão manual, não automatizada.
- Processo de aprovação formal com múltiplos aprovadores (quem assina, quando) — nesta fase, um único mantenedor aprova.
- Critérios específicos para prompts multimodais (imagem/vídeo — Nano Banana, Veo) — o caso real que originou esta versão (`PR-0001`) é só texto; critérios de imagem/vídeo ficam para quando houver um Engine real desse tipo (mesma disciplina já aplicada aqui: não regular sem caso concreto).

Diferente da 0.2.0 (que ainda dependia de uma decisão externa pendente — qual provedor escolher), esta versão fecha o ciclo: há um provedor real, configurado, testado, com dados reais de custo e latência observados.

## 4. Conteúdo Principal

### 4.1 Tipos de artefato

Adicionados à tabela de tipos oficiais do STD-0003:

| Código | Tipo |
|-------|------|
| PR | Prompt |
| SCR | Script |

### 4.2 Local de armazenamento

- Prompt específico de uma feature/Engine: `features/<nome-do-engine>/prompts/`, dentro do repositório do produto (`cc-commerce-studio`) — consistente com a arquitetura de fatia vertical já em uso (`features/workspace/...`).
- Prompt compartilhado entre múltiplos Engines: candidato natural a `cc-shared-framework` (repositório reservado para código reutilizável, ver STD-0004). Decisão adiada até existir um segundo Engine real que confirme a necessidade — evita mover código para um repositório vazio de forma especulativa.
- Script de manutenção/automação: `scripts/` na raiz do repositório do produto (pasta já existente em `cc-commerce-studio`, hoje vazia).

### 4.3 Formato do arquivo de Prompt

Arquivo Markdown (`.md`) com front-matter YAML contendo, no mínimo:

```yaml
id: PR-NNNN
title: <nome do prompt>
target_model: <ex. Gemini 3.1 Flash-Lite, Nano Banana, Veo>
objective: <objetivo do prompt em uma frase>
version: 0.1.0
status: Draft
model_verified_at: <data em que target_model foi confirmado disponível — novo em 0.3.0>
```

O campo `model_verified_at` existe porque provedores descontinuam modelos com pouco aviso — caso real: `PR-0001` foi escrito com `gemini-2.5-flash-lite`, e horas depois, no primeiro teste real, o modelo já não aceitava novos usuários. Revisar esse campo sempre que o prompt for reativado após um período parado.

### 4.4 Versionamento

Git é a fonte de verdade — histórico, diff e rollback nativos, sem ferramenta adicional nesta fase. O versionamento semântico do conteúdo do próprio Standard segue o STD-0002.

### 4.5 Contrato de entrada e saída (novo em 0.2.0)

Todo prompt precisa declarar, além do front-matter mínimo (4.3), duas seções no corpo do arquivo:

- **Variáveis de entrada** — lista explícita dos dados que o prompt exige antes de rodar (ex.: `product.name`, `product.description`, `brand.tone_of_voice`), com indicação do que é obrigatório e do que é opcional.
- **Formato de saída esperado** — descrição do formato que a resposta do modelo deve ter (texto livre, JSON com campos específicos, etc.), para que o código que chama o prompt saiba o que validar/parsear.

Esse contrato existe independente de haver ou não uma chamada real de IA ligada — é isso que separa um prompt "pronto para revisão" de um rascunho.

### 4.6 Ciclo de vida do prompt (novo em 0.2.0)

O campo `status` do front-matter (4.3) passa a seguir um ciclo fechado:

| Status | Significado |
|--------|--------------|
| `Draft` | Escrito, ainda não revisado. |
| `Ready` | Contrato de entrada/saída preenchido, revisado manualmente pelo checklist (4.7), mas ainda sem chamada real a um modelo. |
| `Active` | Ligado a um provedor de IA real e em uso em produção. |
| `Deprecated` | Substituído por uma versão mais nova; mantido só para histórico. |

### 4.7 Checklist de revisão antes de `Ready` (novo em 0.2.0)

- [ ] Objetivo do prompt está declarado em uma frase clara (`objective`).
- [ ] Todas as variáveis de entrada estão listadas e mapeadas para uma fonte de dado real no código (ex.: `Product`, `Brand`).
- [ ] O formato de saída esperado está descrito.
- [ ] Existe pelo menos um exemplo manual de entrada/saída (mesmo que a saída tenha sido escrita à mão, sem IA real, só para validar o formato).
- [ ] Se o prompt depende de tom de voz de marca, o texto do prompt referencia `brand.tone_of_voice` explicitamente, não um tom fixo hardcoded.

Critérios quantitativos (custo por chamada, latência, filtros de segurança de conteúdo) entram como uma seção adicional deste checklist somente quando o prompt avançar de `Ready` para `Active` — ver 4.8.

### 4.8 Critérios quantitativos para `Active` (novo em 0.3.0)

Baseado na integração real de `PR-0001` com Gemini 3.1 Flash-Lite:

- **Custo de referência** — um prompt de copywriting típico (500–800 tokens de entrada, 300–500 de saída) no nível "Flash-Lite" custa entre US$ 0,0005 e US$ 0,001 por chamada. Se um novo prompt ultrapassar 10x essa referência por chamada (ex.: por usar um modelo "Pro" sem necessidade, ou por um contrato de saída longo demais), revisar o motivo antes de promover a `Active`.
- **Latência de referência** — a chamada real observada levou poucos segundos (aceitável para uso síncrono numa UI de formulário, com estado de carregamento). Acima de ~10 segundos, considerar resposta assíncrona ou streaming em vez de bloquear a UI.
- **Segurança de conteúdo** — todo prompt `Active` deve tratar explicitamente o caso de resposta vazia ou bloqueada pelo filtro de segurança do provedor. Nunca devolver uma string vazia silenciosa ao usuário quando o motivo real é um bloqueio de conteúdo — o erro precisa ser visível e diferenciável de "o modelo não teve nada a dizer".
- **Tratamento de erro/indisponibilidade** — toda Server Action que chama um prompt `Active` deve capturar falhas de rede, limite de cota (HTTP 429) e indisponibilidade do provedor, devolvendo um erro tratável pela UI — nunca deixar a exceção subir sem tratamento até o usuário.

## 5. Dependências

- STD-0001 — Documentation & Artifact Standard
- STD-0002 — Versioning Standard
- STD-0003 — Naming Convention Standard
- STD-0004 — Repository Organization Standard
- SPC-0004 — Offer Engine (caso real que originou esta expansão)

## 6. Critérios de Aprovação (desta versão do Standard)

Cumprido em 2026-07-11: `PR-0002` (Video Script Engine, CS-012) seguiu este Standard do início ao fim — mesmo formato de front-matter, mesmo ciclo de vida (Draft → Active), mesmo checklist de revisão (§4.7) e mesmos critérios quantitativos de custo/latência/segurança (§4.8) — sem exigir nenhuma expansão nova do Standard. Isso confirma que os critérios generalizam entre Engines diferentes (Offer Engine e Video Script Engine) e não eram um caso particular do primeiro caso real. Standard promovido de `0.3.0` (Draft) para `1.0.0` (Approved).

## 7. Roadmap de evolução

- **1.0.0** — Concluído (2026-07-11): PR-0002 (Video Script Engine) seguiu este Standard sem necessidade de expansão.
- Critérios para prompts multimodais (imagem/vídeo) ficam para quando houver um Engine real desse tipo (ver Não Escopo, seção 3) — candidato natural: Creative Engine, próximo módulo da ordem decidida em DP-012. Não antes.

## 8. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft — local, formato e metadados mínimos | Joaquim Mario Soares Coelho |
| 0.2.0 | 2026-07-10 | Contrato de entrada/saída, ciclo de vida do prompt e checklist de revisão, a partir do caso real SPC-0004 (Offer Engine, PR-0001) | Claude |
| 0.3.0 | 2026-07-10 | Critérios quantitativos de custo/latência/segurança, tratamento de erro e campo `model_verified_at`, a partir da integração real de PR-0001 com Gemini 3.1 Flash-Lite | Claude |
| 1.0.0 | 2026-07-11 | Promovido a Approved — PR-0002 (Video Script Engine) seguiu o Standard sem exigir nova expansão, confirmando que os critérios generalizam entre Engines diferentes | Claude |
