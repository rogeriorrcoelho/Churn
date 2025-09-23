# Modelo de Churn
Projeto de modelo de churn desenvolvido em um Jupyter Notebook. 
O objetivo principal é identificar fatores relacionados à evasão (cancelamento) de beneficiários de um plano de saúde e gerar insights para reduzir o churn.

## 1. Objetivo do Projeto
O projeto visa criar e avaliar um modelo estatístico preditivo capaz de estimar quais beneficiários têm maior risco de evasão nos próximos 12 meses.

## 2. Base de Dados
A base de dados utilizada, embora anonimizada, contém as seguintes variáveis:

* Id_cliente: Identificação única do cliente (removida na análise).
* Titularidade: Se é titular ou dependente do plano (removida na análise por ser quase constante).
* Cancelado: Status do cliente (Sim/Não) - Variável resposta.
* Faixa de Renda
* Idade (na adesão)
* Tempo de Plano (meses)
* Sexo
* UF (Unidade Federativa)
* Inadimplente (Sim/Não)
* Quantidade de consultas (últimos 12 meses)
* Quantidade de internações (últimos 12 meses)
* Valor da mensalidade

## 3. Etapas da Análise
O projeto seguiu as seguintes etapas:

Exploração e Limpeza dos Dados (EDA): Análise exploratória, tratamento de valores faltantes e outliers.
Análise Estatística: Cálculo da taxa de churn, comparação entre grupos e identificação de variáveis relevantes.
Visualização: Geração de gráficos para distribuições e heatmaps.
Modelagem: Criação de um modelo preditivo de churn, avaliação de acurácia e interpretação de fatores.
Recomendação: Sugestões de ações estratégicas baseadas nos insights obtidos.
