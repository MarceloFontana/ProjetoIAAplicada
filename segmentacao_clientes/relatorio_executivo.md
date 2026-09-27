# Relatório Executivo — Segmentação de Clientes

**Aluno:** Marcelo Fontana  
**Disciplina:** Aprendizado Não Supervisionado  
**Base:** UCI Online Retail | 4,334 clientes identificados | £8,761,067 de receita identificada  
**Modelo:** K-Means (k=4) sobre atributos RFM estendidos

---
### 1. Sumário Executivo

A base de 4.334 clientes identificados do e-commerce foi segmentada por comportamento de compra
(dez/2010–dez/2011, £8,76 milhões de receita identificada). Principais descobertas:

- **A receita é extremamente concentrada.** Os 10% maiores clientes respondem por **61,3%** de todo o
  faturamento; os 20% maiores, por **74,5%**; e o 1% maior, por **31,9%**. A empresa opera, na prática, como um
  negócio de contas-chave disfarçado de varejo de massa.
- **Existem quatro perfis de cliente estatisticamente distintos** (p < 0,001 em todos os atributos), e um
  único deles — os **Campeões**, cerca de 17% da base — concentra em torno de **65% da receita**.
- **O maior ponto de fuga é a segunda compra.** Cerca de 35% dos clientes fizeram **um único pedido**. O
  segmento "Novos Clientes" tem ticket médio próximo ao da base, ou seja, **não é um problema de poder de
  compra, é um problema de recompra**.
- **Há receita significativa em risco silencioso:** clientes de alto valor (acima do percentil 75) que já
  passaram da recência mediana representam uma fatia relevante do faturamento e não estão sendo tratados como
  prioridade de retenção.
- **Um grupo pequeno de contas tem comportamento de atacado** (identificado pelo DBSCAN como ruído estatístico)
  e distorce qualquer média de segmento. Devem sair da campanha de massa e entrar em gestão individual.

**Top 3 recomendações prioritárias**

1. **Blindar os Campeões** com programa de retenção dedicado e alerta automático de queda de frequência. É o
   investimento de maior retorno: a perda de 10% desse grupo equivale a perder mais receita do que todo o
   segmento "Perdidos" gera.
2. **Instituir um fluxo de segunda compra** para Novos Clientes (cupom com prazo curto, disparado em D+15 da
   primeira compra). Converter uma fração dos ~1.500 clientes de pedido único é o maior ganho incremental
   disponível sem custo de aquisição.
3. **Realocar verba dos Perdidos para os Leais.** O segmento Perdidos consome atenção e entrega receita
   marginal; os Leais estão a poucos pedidos de virar Campeões e respondem melhor a cross-sell.

---

### 2. Metodologia

**Dados e preparação.** Partimos de 541.909 transações do UCI Online Retail. A limpeza é auditada
linha a linha na seção 1 e removeu 26,9% dos registros: cancelamentos, códigos que não são produto (frete,
taxas, ajustes), quantidades e preços não positivos e — a maior perda, 24,3% — vendas **sem identificação de
cliente**. Consequência que a diretoria precisa conhecer: a segmentação cobre a base identificável, e os
percentuais de receita referem-se à **receita identificada**, não ao faturamento total.

Cada cliente foi descrito pelo framework **RFM** (recência, frequência, valor monetário), estendido com
variedade de produtos e tempo de relacionamento. Como os atributos têm assimetria extrema (poucos atacadistas
compram centenas de vezes), aplicamos transformação logarítmica antes da padronização — sem isso, meia dúzia de
clientes dominaria a segmentação. Ticket médio e itens por pedido foram deliberadamente mantidos **fora** do
modelo, porque são funções aritméticas dos demais e contariam a dimensão "valor" em duplicidade; entraram no
perfil e na validação.

**Escolha do algoritmo final: K-Means com k = 4.** Foram implementados e comparados quantitativamente quatro
algoritmos (seção 8): K-Means, Hierárquico Aglomerativo (Ward/complete/average), DBSCAN e Gaussian Mixture
Models. O K-Means venceu pelo conjunto de índices internos, pela estabilidade e por um critério operacional
decisivo: possui `predict()`, o que permite classificar novos clientes no CRM sem retreinar o modelo.

A definição de k = 4 foi um **trade-off explícito**. O Silhouette Score é máximo em k = 2, mas uma bipartição
"ativo vs. inativo" não permite ação de marketing diferenciada. Adotamos o joelho da curva de inércia
(k = 4), o menor valor que produz segmentos acionáveis, e validamos que os grupos diferem significativamente
em todos os atributos.

**Validação.** A solução é estável: ARI ≈ 1,0 entre sementes diferentes e ARI alto sob reamostragem bootstrap
de 80% da base, indicando que os segmentos não são artefato de inicialização nem de amostra. O método
hierárquico, de princípio distinto, chegou a uma partição fortemente concordante — evidência independente de
que a estrutura é real. A significância das diferenças entre segmentos foi confirmada por Kruskal-Wallis.

**Limitações identificadas e mitigações.** (i) *Cobertura*: 24,3% da receita não tem cliente identificado —
mitigação: tratar os percentuais como proporção da base identificada e priorizar a captura de identificação no
checkout. (ii) *Janela de 12 meses*: não permite separar sazonalidade de tendência; um cliente "perdido" pode
ser comprador anual — mitigação: recalcular a segmentação trimestralmente e observar a migração. (iii)
*Sobreposição entre segmentos*: o comportamento do cliente é um continuum, e o GMM mostra que parte da base tem
atribuição ambígua — mitigação: usar a probabilidade de pertencimento para priorizar quem está na fronteira.
(iv) *Ausência de margem*: só temos receita, não lucratividade; um segmento de alta receita e alto custo de
serviço pode ser menos valioso do que parece.

---

### 3. Resultados e Insights

**Os quatro segmentos identificados** (valores exatos nas tabelas da seção 10):

**Campeões** — cerca de 17% da base e ~65% da receita. Compraram há poucos dias, com frequência mediana
próxima a 10 pedidos e a maior variedade de produtos. Receita média por cliente muito acima da média geral.
São a base instalada que sustenta o negócio.

**Clientes Leais** — aproximadamente 31% da base e ~24% da receita. Ativos, com frequência intermediária e
relacionamento longo. É o segmento com maior potencial de crescimento: já compram, já confiam na marca e estão
a poucos pedidos de migrar para Campeões.

**Novos Clientes** — cerca de 27% da base e ~7% da receita. Relacionamento recente, tipicamente um único
pedido, mas ticket comparável ao da base. O gargalo é a recompra, não o valor.

**Perdidos / Inativos** — cerca de 25% da base e apenas ~4% da receita. Compra única, antiga e de valor baixo.

**Insights comportamentais principais**

1. **A dimensão dominante do comportamento é única.** A análise de componentes principais mostra que ~60% de
   toda a variação entre clientes cabe em um só eixo, que combina frequência, valor e variedade contra
   recência. Traduzido: não existem muitos "tipos" de cliente independentes — existe essencialmente um espectro
   de engajamento. A consequência prática é que **mover o cliente ao longo desse espectro** (fazê-lo comprar
   mais vezes) é o que gera valor, mais do que tentar mudar o que ele compra.
2. **O segundo eixo é a maturidade do relacionamento**, independente do valor. É o que separa um cliente novo
   de um cliente antigo com gasto semelhante — e é por isso que as duas populações exigem campanhas diferentes
   apesar de números parecidos.
3. **Pedido único é a regra, não a exceção**: 35% da base. Isso posiciona o *onboarding* como a maior alavanca
   de crescimento disponível.
4. **Contas de atacado estão misturadas ao varejo.** O DBSCAN isolou como ruído estatístico um grupo de
   clientes com ticket e volume por pedido ordens de magnitude acima do normal. São contas corporativas dentro
   de uma base de consumo. Tratá-las com régua de varejo é errar duas vezes: a campanha não faz sentido para
   elas e a média que elas produzem distorce o planejamento dos outros segmentos.
5. **Clientes em transição são a fila de prioridade natural.** O GMM aponta a parcela da base com atribuição
   ambígua entre dois segmentos — são clientes se movendo, para cima ou para baixo. Agir neles tem retorno
   marginal maior do que agir no centro de um segmento estável.

**Oportunidades de negócio descobertas**

- **Retenção de concentração:** com ~65% da receita em um segmento de ~17% dos clientes, um sistema de alerta
  de queda de frequência nesse grupo protege mais receita do que qualquer campanha de aquisição de igual custo.
- **Conversão da segunda compra:** o volume de clientes de pedido único é alto e o ticket deles não é baixo.
  Um fluxo automatizado de recompra atua sobre a maior massa disponível sem custo de mídia de aquisição.
- **Promoção de Leais a Campeões:** cross-sell por categoria não comprada, usando a variedade de produtos como
  indicador de amplitude do relacionamento.
- **Segmentação de atendimento:** separar as contas de atacado em gestão individual, com condições comerciais
  próprias, e retirá-las da base de cálculo das campanhas de massa.
- **Eficiência de verba:** limitar o investimento no segmento Perdidos a canais de custo marginal
  (e-mail), com corte automático após ausência de resposta.