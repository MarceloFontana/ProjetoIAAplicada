# Segmentação de Clientes com Aprendizado Não Supervisionado

**Aluno:** Marcelo Fontana
**Disciplina:** Aprendizado Não Supervisionado
**Professor:** Cassiano Ricardo Neubauer Moralles

Segmentação comportamental de 4.334 clientes de e-commerce (UCI Online Retail, dez/2010–dez/2011) usando
RFM estendido e quatro algoritmos de clusterização, com dashboard interativo e relatório executivo.

## Resultado principal

| Segmento | Clientes | % da base | % da receita | Receita média |
|---|---:|---:|---:|---:|
| Campeões | 726 | 16,8% | **64,9%** | £7.834 |
| Clientes Leais | 1.342 | 31,0% | 23,9% | £1.561 |
| Novos Clientes | 1.173 | 27,1% | 7,0% | £526 |
| Perdidos / Inativos | 1.093 | 25,2% | 4,1% | £331 |

Modelo final: **K-Means (k=4)** sobre atributos RFM log-transformados e padronizados.

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `segmentacao_clientes_rfm.ipynb` | notebook completo e executado (Itens 1, 2 e 3) |
| `dashboard_segmentacao.html` | dashboard Plotly interativo — abre em qualquer navegador |
| `relatorio_executivo.md` | relatório executivo (1.352 palavras, limite 1.500) |
| `clientes_segmentados.csv` | base com o segmento de cada cliente, pronta para o CRM |
| `dados/` | dataset da UCI (baixado automaticamente na primeira execução) |

## Como reproduzir

```bash
pip install pandas numpy scikit-learn matplotlib seaborn plotly openpyxl scipy jupyter
jupyter notebook segmentacao_clientes_rfm.ipynb
```

O notebook baixa o dataset da UCI automaticamente (~23 MB) se ele não estiver em `dados/`.
Execução completa: cerca de 4 minutos.

## Algoritmos implementados (Item 1)

| Algoritmo | Seleção de hiperparâmetros | Papel no entregável |
|---|---|---|
| **K-Means** | Elbow (Kneedle) + Silhouette; análise de convergência e estabilidade (ARI entre sementes e bootstrap) | **modelo final** — tem `predict()` para novos clientes |
| **Hierárquico** | ward / complete / average + coeficiente cofenético; dendrograma e corte ótimo | validação independente da estrutura |
| **DBSCAN** | `eps` pelo k-distance graph, `min_samples = 2·dim` | detecção de contas de atacado (ruído) |
| **GMM** | BIC / AIC por tipo de covariância | probabilidade de pertencimento → clientes em transição |

## Decisões metodológicas documentadas

- **k = 4 e não k = 2**, apesar de o Silhouette ser máximo em k=2: bipartição "ativo vs. inativo" não é
  acionável. Trade-off explicitado no notebook.
- **Ward e não average**, apesar do coeficiente cofenético maior de `average`: os outros linkages produzem
  *chaining* (um cluster com 98% dos clientes).
- **DBSCAN não serve como segmentador aqui** — a base é uma nuvem contínua unimodal, sem vales de densidade.
  Reportado como achado, e reaproveitado como detector de outliers de negócio.
- **BIC do GMM não tem mínimo interior**: usá-lo para escolher k produziria segmentos sem contraparte comercial.
- `TicketMedio` e `ItensPorPedido` ficam **fora** do modelo (são funções dos demais atributos) e entram no
  perfil e na validação.
