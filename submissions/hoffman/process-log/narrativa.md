# Process log — narrativa

## Ferramentas usadas

| Ferramenta | Para que usei |
|------------|----------------|
| Cursor (IA) | Explorar CSVs, gerar `diagnostico.py` e `prototipo.py`, redigir README |
| Kaggle | Download dos datasets CC0 (links no README) |

## Workflow

1. Li o README do challenge 002 e o [Guia de Submissão](../../submission-guide.md); separei entrega em diagnóstico (Dataset 1), proposta cruzando os dois datasets e protótipo de roteamento (Dataset 2 + regras Dataset 1).
2. Baixei os CSVs para `solution/data/` e validei colunas e volume (8.469 tickets no Dataset 1; ~47,8 mil no Dataset 2).
3. Criei `diagnostico.py` para reproduzir métricas (gargalos, p50/p90, CSAT, regressão de drivers, backlog, ROI, cruzamento DS2). Descobri que metade dos fechados tem pós-FRT negativo no CSV e restrigi análises de tempo a ≥0 h.
4. Estendi `prototipo.py`: subcomando `treinar` com `classification_report`; subcomando `rotear` com flags `--tipo` / `--prioridade` e filas do fluxo proposto.
5. Rodei os scripts e alinhei o README aos outputs (sem números “soltos”).

## Onde a IA errou e como corrigi

- A IA tratou `First Response Time` e `Time to Resolution` como durações em horas. No CSV são **timestamps**; usei diferença em horas e filtrei ordem cronológica válida.
- Sugestões genéricas de “automatizar tudo” foram descartadas; mantive refund/cancelamento e categorias sensíveis (HR/admin) fora da automação plena.

## O que eu adicionei que a IA sozinha não faria

- **Reprodutibilidade:** diagnóstico via script, não só texto no README.
- **Ordem de regras no roteador:** tipo comercial (DS1) antes da classificação; confiança e categorias sensíveis como gate humano.
- **Drivers de CSAT** além de correlação com tempo (regressão com canal/tipo/prioridade).
- **ROI de triagem** com premissas impressas no terminal (`diagnostico.py`).

## Iterações

Várias rodadas para enxugar escopo (dois scripts, sem dashboard), refinar combinações n≥20, e validar README ↔ output dos comandos.
