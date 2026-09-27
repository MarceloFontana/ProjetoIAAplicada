# Projeto IA Aplicada — Aprendizado Não Supervisionado

**Aluno:** Marcelo Fontana
**Disciplina:** Aprendizado Não Supervisionado

Dois trabalhos de aprendizado não supervisionado, cada um em sua pasta, com notebooks executados
(todas as saídas e gráficos gravados).

---

## 1. [`segmentacao_clientes/`](segmentacao_clientes/) — Segmentação de clientes de e-commerce

Atividade Prática de segmentação de clientes com dados reais de e-commerce
([UCI Online Retail](https://archive.ics.uci.edu/dataset/352/online+retail), 541.909 transações,
dez/2010–dez/2011).

**Resultado principal:** 726 clientes (16,8% da base) concentram **64,9% da receita**.

| Segmento | Clientes | % da base | % da receita | Receita média |
|---|---:|---:|---:|---:|
| Campeões | 726 | 16,8% | **64,9%** | £7.834 |
| Clientes Leais | 1.342 | 31,0% | 23,9% | £1.561 |
| Novos Clientes | 1.173 | 27,1% | 7,0% | £526 |
| Perdidos / Inativos | 1.093 | 25,2% | 4,1% | £331 |

**Técnicas:** RFM estendido, K-Means (Elbow + Silhouette, convergência e estabilidade), Hierárquico
Aglomerativo (ward/complete/average, dendrograma), DBSCAN (k-distance graph), Gaussian Mixture Models
(BIC/AIC, probabilidades de pertencimento), PCA 2D/3D, dashboard Plotly e relatório executivo.

Entregáveis: notebook, `dashboard_segmentacao.html` (abre no navegador), `relatorio_executivo.md`
(1.352 palavras) e `clientes_segmentados.csv` (base pronta para CRM).

---

## 2. [`logs_firewall/`](logs_firewall/) — Análise de logs de firewall

Análise não supervisionada de 200 eventos de log de um firewall de perímetro, sem rótulos de ataque.

**Resultado principal:** o DBSCAN isolou como ruído **todos os 4 eventos negados** do dataset — dois
`Invalid Traffic` (cabeçalho IP inválido e flags TCP inválidas, típicas de *scan*) e dois bloqueios da regra
`BlockedIP` — sem usar nenhum rótulo, funcionando como filtro de triagem (200 → 36 eventos para inspeção).

**Técnicas:** estatística descritiva completa, seleção de características (VarianceThreshold, filtro de
correlação, informação mútua), PCA, K-Means, Hierárquico (Ward), DBSCAN, avaliação de clusters
(Silhouette, Davies-Bouldin, Calinski-Harabasz, ARI/NMI), t-SNE, Apriori e FP-Growth.

---

## Como reproduzir

```bash
pip install pandas numpy scikit-learn matplotlib seaborn plotly openpyxl scipy mlxtend jupyter
```

O notebook de segmentação baixa o dataset da UCI automaticamente (~23 MB, não versionado aqui).
O notebook de logs usa o `new_logs.csv` incluído na pasta.

## Notas metodológicas

Os dois notebooks documentam explicitamente as decisões em que o "ótimo" das métricas foi **recusado** em
favor do resultado utilizável — por exemplo, escolher k pelo joelho da inércia em vez do máximo do
Silhouette, e preferir a ligação Ward apesar de `average` ter coeficiente cofenético maior. As razões estão
escritas nas respectivas seções, junto das limitações de cada análise.
