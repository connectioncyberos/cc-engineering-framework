---
id: STD-0007
title: Prompt & Script Governance Standard
category: Standard
version: 0.2.0
status: Draft
author: Joaquim Mario Soares Coelho
architect: Claude
project: ConnectionCyber Engineering Framework
repository: cc-engineering-framework
created_at: 2026-07-10
updated_at: 2026-07-10
---

# STD-0007 — Prompt & Script Governance Standard

## 1. Objetivo

Definir onde prompts de IA e scripts de automação do ecossistema ConnectionCyber são armazenados, seu formato mínimo, os metadados obrigatórios, e — a partir desta versão (0.2.0) — o contrato de entrada/saída e o checklist de revisão que todo prompt precisa cumprir antes de ser considerado pronto para uso. Esta expansão foi feita a partir de um caso real (SPC-0004 — Offer Engine, `PR-0001`), não especulativamente.

## 2. Escopo

- Prompts de IA usados pelos Engines de produto (Offer Engine, Creative Engine, Video Script Engine, etc.), definidos em `cc-commerce-studio/ROADMAP.md` (CS-008+).
- Scripts de manutenção, automação e apoio ao desenvolvimento dos repositórios do ecossistema.
- Contrato de entrada/saída e checklist de revisão de um prompt antes de uso (novo nesta versão).

## 3. Não Escopo (ainda nesta versão)

- Critérios quantitativos de custo, latência e segurança medidos contra um modelo real — dependem de um provedor de IA já escolhido e configurado, o que ainda não aconteceu (ver achado bloqueante em SPC-0004, seção 4.1). Ficam para uma futura 0.3.0, quando houver um provedor real em produção para medir contra.
- Ferramentas de teste automatizado de prompts (prompt evals).
- Processo de aprovação formal com múltiplos aprovadores (quem assina, quando) — nesta fase, um único mantenedor aprova.

Diferente da 0.1.0 (que adiou tudo por falta de caso real), esta versão já incorpora o que o caso real (SPC-0004) permitiu aprender. O que ainda falta depende de uma decisão externa (provedor de IA), não de falta de caso de uso.

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
target_model: <ex. Gemini, Nano Banana, Veo>
objective: <objetivo do prompt em uma frase>
version: 0.1.0
status: Draft
```

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

Critérios quantitativos (custo por chamada, latência, filtros de segurança de conteúdo) entram como uma seção adicional deste checklist somente quando o prompt avançar de `Ready` para `Active` — ou seja, quando houver um provedor real para medir contra (ver Não Escopo, seção 3).

## 5. Dependências

- STD-0001 — Documentation & Artifact Standard
- STD-0002 — Versioning Standard
- STD-0003 — Naming Convention Standard
- STD-0004 — Repository Organization Standard
- SPC-0004 — Offer Engine (caso real que originou esta expansão)

## 6. Critérios de Aprovação (desta versão do Standard)

Permanece em Draft até `PR-0001` (SPC-0004) alcançar o status `Active` com um provedor de IA real configurado.

## 7. Roadmap de evolução

- **0.3.0** — quando um provedor de IA real for escolhido e configurado, adicionar critérios quantitativos de custo, latência e segurança, e formalizar a transição `Ready` → `Active`.
- **1.0.0** — quando aprovado.

## 8. Histórico de Alterações

| Versão | Data | Alteração | Autor |
|--------|------|-----------|-------|
| 0.1.0 | 2026-07-10 | Primeira versão Draft — local, formato e metadados mínimos | Joaquim Mario Soares Coelho |
| 0.2.0 | 2026-07-10 | Contrato de entrada/saída, ciclo de vida do prompt e checklist de revisão, a partir do caso real SPC-0004 (Offer Engine, PR-0001) | Claude |
