# Submissão — Hoffman — Challenge 002

## Sobre mim

- **Nome:** Hoffman
- **LinkedIn:** https://www.linkedin.com/in/mateushoffman/
- **Challenge escolhido:** 002 — Redesign de Suporte (`challenges/process-002-support`)

---

## Executive Summary

No Dataset 1, 67% do volume está Open/Pending; entre fechados com timestamps válidos, a mediana de pós-primeira-resposta é 6,3 h (p90: 16 h), com piores combinações em Chat/Social + refund. CSAT (~3/5) é puxado sobretudo por **canal** (Phone/Email/Social vs Chat), não por tempo de resolução. Cruzando com o Dataset 2, proponho classificação + **roteamento** (~86% no hold-out), com ~21 h/mês ilustrativas economizadas em triagem. Entreguei diagnóstico reprodutível (`diagnostico.py`), proposta de fluxo e protótipo CLI (`prototipo.py`).

---

## Solução

Datasets (CC0, Kaggle): [Dataset 1](https://www.kaggle.com/datasets/suraj520/customer-support-ticket-dataset) → `solution/data/customer_support_tickets.csv` · [Dataset 2](https://www.kaggle.com/datasets/adisongoh/it-service-ticket-classification-dataset) → `solution/data/all_tickets_processed_improved_v3.csv`.

Métricas abaixo geradas por `python solution/diagnostico.py` (out/2026).

### Abordagem

Dois scripts em `solution/`: **`diagnostico.py`** (Dataset 1, métricas e ROI) e **`prototipo.py`** (treino no Dataset 2 + roteador do fluxo proposto). Dependências: `pip install pandas scikit-learn`.

### Resultados / Findings

#### 1. Diagnóstico operacional (obrigatório)

Com base no **Dataset 1** (8.469 tickets). Tempos: |`Time to Resolution` − `First Response Time`| em horas; **2.769** fechados com ambos timestamps, **1.404** com ordem cronológica válida (≥0 h) — métricas de tempo usam este subconjunto.

**Onde o fluxo trava?** Médias de pós-FRT (h): canal — Social media (7,9), Email (7,6), Chat (7,5), Phone (7,3); prioridade — High (8,1), Low (7,9), Medium (7,2), Critical (7,1); tipo — Refund (8,1), Cancellation (7,7), Product inquiry (7,7), Technical (7,4). Mediana global 6,3 h, p90 16 h (caudas longas explicam médias “planas” entre canais). Piores combinações (n≥20): Chat + High + Refund (11,4 h); Social + High + Refund (10,6 h); Email + Low + Cancellation (9,7 h); Social + Low + Product inquiry (9,1 h).

**O que impacta satisfação?** CSAT médio 2,99/5. Pior tipo: Refund request (2,93). Pior canal: Phone (2,95); melhor: Chat (3,08). Correlação CSAT × pós-FRT ≈ 0. **O que mais move CSAT:** regressão com dummies — efeitos negativos mais fortes em Phone (−0,13), Email (−0,12) e Social media (−0,11) vs referência; tempo de resolução sozinho não explica satisfação.

**Quanto estamos desperdiçando?** 5.700 tickets Open/Pending (67%). Upper bound ilustrativo: mediana pós-FRT × backlog ≈ **36.148 h**; custo @ R$ 120/h ≈ **R$ 4,34 mi**. Maior recuperação: reduzir backlog em refund/cancelamento e canais assíncronos (Social/Email).

#### 2. Proposta de automação com IA (obrigatório)

Com base nos **dois datasets**.

**O que automatizar** — Classificação + roteamento (Dataset 2, ~86% acurácia hold-out). Triagem de prioridade quando Dataset 1 indica `High`. Respostas sugeridas a partir de `Ticket Description` + `Resolution` (Dataset 1) — fase 2. Detecção de duplicatas em social — fase posterior.

**O que NÃO automatizar** — Refund request e Cancellation request (Dataset 1: pior CSAT e maiores tempos). HR Support e Administrative rights (Dataset 2: ~26,5% do volume, risco de acesso). Confiança < 70% no classificador.

**Como funcionaria na prática** — Ticket entra → se tipo Refund/Cancel (Dataset 1) → agente sênior → senão classificação (IA, Dataset 2) → se HR/Admin ou confiança baixa → humano → se prioridade High → fila prioritária → senão L1 + resposta sugerida opcional → humano valida.

**Cruzamento Dataset 2 → operação** (destino padrão por categoria, antes de regras DS1):

| Topic_group (DS2) | % volume | Destino |
|-------------------|----------|---------|
| Hardware | 28,5% | L1 |
| HR Support | 22,8% | HUMANA |
| Access | 14,9% | L1 |
| Miscellaneous | 14,8% | L1 |
| Storage | 5,8% | L1 |
| Purchase | 5,2% | L1 |
| Internal Project | 4,4% | L1 |
| Administrative rights | 3,7% | HUMANA |

**ROI triagem (ilustrativo):** ~23 tickets/dia (DS1/365), 4 min → 0,5 min por ticket elegível (70%) → **~21 h/mês** de esforço de triagem evitável, antes de ganhos de resolução.

#### 3. Protótipo funcional (diferencial)

Roteador: classifica texto (Dataset 2) e aplica regras do fluxo; metadados opcionais `--tipo` / `--prioridade` (Dataset 1).

```bash
pip install pandas scikit-learn
python solution/diagnostico.py
python solution/prototipo.py treinar
python solution/prototipo.py rotear "VPN password reset cannot login"
python solution/prototipo.py rotear "I want a refund" --tipo "Refund request" --prioridade High
```

Saída: `categoria`, `confianca`, `fila` (`AGENTE_SENIOR` | `HUMANA` | `PRIORITARIA` | `L1`), `motivo`.

### Recomendações

1. Implementar roteador na entrada; fila humana para HR/Admin, baixa confiança e exceções comerciais.
2. Ataque simultâneo a backlog (67%) e squads em refund + Social/Email (tempo e CSAT).
3. Fase 2: respostas sugeridas com pares descrição/resolução no Dataset 1.

### Limitações

- Métricas de tempo só em 1.404/2.769 fechados com pós-FRT ≥0 h (metade dos fechados tem timestamps invertidos no CSV).
- Protótipo: classificação + roteamento; sem respostas sugeridas, duplicatas ou dashboard.
- Backlog, custo e ROI com premissas explícitas no output de `diagnostico.py`.

---

## Process Log — Como usei IA

> **Este bloco é obrigatório.** Sem ele, a submissão é desclassificada. Detalhes em [process-log/narrativa.md](process-log/narrativa.md).

### Ferramentas usadas

| Ferramenta | Para que usei |
|------------|----------------|
| Cursor (IA) | Diagnóstico, proposta, `diagnostico.py` e `prototipo.py` |
| Kaggle | Download dos datasets CC0 |

### Workflow

1. Decomposição alinhada ao challenge (diagnóstico, proposta, protótipo roteador, process log).
2. Script reprodutível de diagnóstico; validação de timestamps e subconjunto cronológico.
3. Protótipo iterado (hold-out + `classification_report` + regras de fila).

### Onde a IA errou e como corrigi

Interpretação errada dos campos de tempo; filtro ≥0 h e documentação do recorte. Ajuste da fronteira humano/IA em refund/cancelamento e categorias sensíveis.

### O que eu adicionei que a IA sozinha não faria

Regras de exceção humanas ancoradas em CSAT/tempo; ordem de decisão no roteador; premissas de ROI explícitas; reprodutibilidade via scripts.

---

## Evidências

_Anexe ou linke as evidências do processo:_

- [ ] Screenshots das conversas com IA
- [ ] Screen recording do workflow
- [ ] Chat exports
- [ ] Git history (se construiu código)
- [x] Outro: narrativa escrita — [process-log/narrativa.md](process-log/narrativa.md)

---

_Submissão enviada em: outubro/2026_
