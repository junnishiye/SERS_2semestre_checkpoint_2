# Checkpoint 02 — Machine Learning com dados de energia

**João Vitor Jun Nishiye de Sousa — RM 572079**

[Abrir o notebook no Google Colab](https://colab.research.google.com/github/junnishiye/SERS_2semestre_checkpoint_2/blob/main/checkpoint2_energia_renovavel.ipynb) · [Ver notebook no GitHub](checkpoint2_energia_renovavel.ipynb)

Este checkpoint usa o [material da atividade do professor André Tritiack](https://github.com/prof-atritiack/CHECKPOINT_02_SERS_1CC_2SEM). São duas tarefas independentes em Python, cada uma com três algoritmos treinados e comparados na mesma divisão dos dados:

1. **Classificação:** prever a categoria da fonte de um empreendimento da ANEEL (Solar, Eólica ou Hidráulica) com potência outorgada e coordenadas.
2. **Regressão:** estimar a radiação solar horária em Petrolina (PE) com condições meteorológicas e hora local.

## Arquivos da entrega

| Arquivo | Conteúdo |
|---|---|
| [`checkpoint2_energia_renovavel.ipynb`](checkpoint2_energia_renovavel.ipynb) | Consultas às APIs, preparação dos dados, exploração, seis modelos, tabelas, gráficos e análise escrita |
| [`aneel_classificacao_orange.csv`](aneel_classificacao_orange.csv) | 3.876 empreendimentos classificados em três fontes |
| [`meteo_regressao_orange.csv`](meteo_regressao_orange.csv) | 1.001 horas diurnas de Petrolina, de 01/04/2025 a 30/06/2025 |

O repositório conserva os arquivos das aulas 6 e 7 anteriores. **O arquivo do checkpoint atual é `checkpoint2_energia_renovavel.ipynb`.**

## Dados e método

### 1. Fonte renovável — ANEEL

O CSV foi preparado a partir do [SIGA da ANEEL](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel), pelo código do notebook de apoio. Os campos de entrada são `potencia_kw` (potência outorgada, não energia produzida), `latitude` e `longitude`. O alvo `fonte` reúne `UFV` em Solar, `EOL` em Eólica e `UHE`, `PCH` e `CGH` em Hidráulica. Há 1.200 solares, 1.200 eólicos e 1.476 hidráulicos. A consulta usa um limite por sigla; essas proporções **não representam** a participação das fontes na matriz elétrica brasileira.

O conjunto não tem valores ausentes. A divisão foi **estratificada**, com 80% para treino e 20% para teste (`random_state=42`). KNN (7 vizinhos) e Regressão Logística usam `StandardScaler` em um `Pipeline`, ajustado apenas no treino. Random Forest usa 200 árvores. Precision, Recall e F1 são calculados com média **macro**; o notebook também exibe os resultados por classe e as três matrizes de confusão.

| Classificador | Accuracy | Precision macro | Recall macro | F1 macro |
|---|---:|---:|---:|---:|
| KNN | 0,9652 | 0,9668 | 0,9633 | 0,9647 |
| Regressão Logística | 0,8247 | 0,8282 | 0,8214 | 0,8197 |
| Random Forest | **0,9755** | **0,9769** | **0,9741** | **0,9753** |

**Conclusão:** Random Forest teve o melhor resultado neste teste. Na sua matriz, os erros mais frequentes foram Solar classificada como Hidráulica (7 casos) ou Eólica (5). Potência e localização apresentam sobreposição entre classes; o modelo não substitui a informação cadastral da tecnologia de um empreendimento.

### 2. Radiação solar — Open-Meteo

O CSV vem da [API histórica Open-Meteo](https://open-meteo.com/en/docs/historical-weather-api) para Petrolina (aproximadamente −9,39, −40,50), entre **01/04/2025 e 30/06/2025**, no fuso `America/Recife`. São horas locais de 7h a 17h. As entradas são `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh` e `hora`; o alvo é `radiacao_w_m2` em W/m². `data_hora` serve para ordenar e separar, sem entrar nas variáveis preditoras.

Não há valores ausentes. Para respeitar o tempo, as **primeiras 800 horas** são treino e as **201 últimas** são teste, sem embaralhamento. Os modelos são Regressão Linear, Árvore de Decisão (`max_depth=6`, `min_samples_leaf=5`) e Random Forest (200 árvores, `min_samples_leaf=2`). O notebook exibe os valores reais e previstos no teste.

| Regressor | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---:|---:|---:|
| Regressão Linear | 145,20 | 30.034,20 | 0,3598 |
| Árvore de Decisão | 87,14 | 14.005,88 | 0,7015 |
| Random Forest | **67,20** | **7.458,54** | **0,8410** |

**Conclusão:** Random Forest teve o menor erro e o maior R² no período de teste. A hora ajuda a descrever o ciclo diário da radiação, mas nuvens e outras condições geram variação. Esses dados históricos são estimativas meteorológicas de radiação horizontal: **prever W/m² não equivale a prever a energia elétrica produzida por painéis solares**. Para estimar geração também seriam necessárias características e perdas do sistema fotovoltaico.

## Como executar

**Google Colab:** abra o link no topo e escolha *Ambiente de execução → Executar tudo*. O notebook baixa os CSVs deste repositório se não estiverem presentes no ambiente. Não é necessário token.

**Localmente:** clone o repositório e execute o notebook na raiz, com Python 3 e os pacotes `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn` e `notebook` instalados:

```bash
python -m pip install pandas numpy matplotlib seaborn scikit-learn notebook
jupyter notebook checkpoint2_energia_renovavel.ipynb
```

Por padrão, `ATUALIZAR_DADOS = False` mantém os CSVs publicados e reproduz as métricas acima. Para realizar novas consultas às APIs e **sobrescrever os CSVs locais**, altere a variável para `True` e execute todas as células; os valores e resultados podem mudar porque a ANEEL atualiza seu cadastro. As duas consultas públicas do notebook não exigem credenciais.
