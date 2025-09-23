# Análise Detalhada do Modelo

Este documento apresenta uma análise detalhada de um projeto de modelo de churn desenvolvido em um Jupyter Notebook. O objetivo principal é identificar fatores relacionados à evasão (cancelamento) de beneficiários de um plano de saúde e gerar insights para reduzir o churn.

## 1. Objetivo do Projeto

O projeto visa criar e avaliar um modelo estatístico preditivo capaz de estimar quais beneficiários têm maior risco de evasão nos próximos 12 meses.

## 2. Base de Dados

A base de dados utilizada, embora anonimizada, contém as seguintes variáveis:
- `Id_cliente`: Identificação única do cliente (removida na análise).
- `Titularidade`: Se é titular ou dependente do plano (removida na análise por ser quase constante).
- `Cancelado`: Status do cliente (Sim/Não) - Variável resposta.
- `Faixa de Renda`
- `Idade (na adesão)`
- `Tempo de Plano (meses)`
- `Sexo`
- `UF` (Unidade Federativa)
- `Inadimplente` (Sim/Não)
- `Quantidade de consultas (últimos 12 meses)`
- `Quantidade de internações (últimos 12 meses)`
- `Valor da mensalidade`

## 3. Etapas da Análise

O projeto seguiu as seguintes etapas:
1.  **Exploração e Limpeza dos Dados (EDA)**: Análise exploratória, tratamento de valores faltantes e outliers.
2.  **Análise Estatística**: Cálculo da taxa de churn, comparação entre grupos e identificação de variáveis relevantes.
3.  **Visualização**: Geração de gráficos para distribuições e heatmaps.
4.  **Modelagem**: Criação de um modelo preditivo de churn, avaliação de acurácia e interpretação de fatores.
5.  **Recomendação**: Sugestões de ações estratégicas baseadas nos insights obtidos.

## 4. Análise Exploratória de Dados (EDA) e Pré-processamento

### 4.1. Observações Iniciais e Tratamento de Dados

-   **`ID_CLIENTE`**: Removida por não ter utilidade na análise ou modelagem.
-   **Valores Ausentes**: Identificados em `VALOR_MENSALIDADE` (36.81%), `UF` (2.01%) e `SEXO` (1.97%).
-   **`TITULARIDADE`**: Variável quase constante (99.99% 'TITULAR'), removida por não oferecer poder explicativo e evitar problemas de convergência/overfitting.
-   **`CANCELADO` (Variável Resposta)**: 58.82% dos clientes cancelaram, indicando uma alta taxa de churn na base histórica.
-   **`FAIXA_RENDA`**: Predomínio de clientes de baixa renda (64.32%), seguido por média (24.65%) e alta (11.03%).
-   **`IDADENAADESÃO`**: Identificados valores negativos e acima de 150, considerados erros de cadastro e descartados. A variável apresentava forte assimetria e outliers.
-   **`TEMPO_DE_PLANO_MESES`**: Valores negativos descartados como erros de cadastro. Predomínio de planos antigos (acima de 220 meses).
-   **`SEXO`**: Predomínio feminino (55.89%).
-   **`UF`**: Maior concentração de clientes em SP, RJ, BA e MG. Mapeamento por regiões (Norte, Nordeste, Centro-Oeste, Sudeste, Sul) mostrou Sudeste e Nordeste com 76.9% dos clientes.
-   **`INADIMPLENTE`**: 26.83% de inadimplentes, um percentual considerado alto, sugerindo a necessidade de validação com a área de negócios.
-   **`QTD_CONSULTAS_12M`**: Distribuição assimétrica positiva, com alta frequência de valores baixos e uma cauda longa para valores elevados. Muitos outliers.
-   **`QTD_INTERNACOES_12M`**: Concentração em zero, um ou dois eventos, com poucos usuários com mais de duas internações. Forte assimetria positiva.
-   **`VALOR_MENSALIDADE`**: Elevada ausência de dados. Distribuição discreta com picos definidos, sugerindo faixas fixas de mensalidade. Foi sugerido categorizar esta variável e tratar os ausentes como 'OUTROS'.

### 4.2. Análise Bivariada

-   **Correlação entre variáveis numéricas**: Nenhuma correlação muito elevada (acima de 0.75) foi encontrada, indicando baixa colinearidade inicial.
-   **`Cancelado` x `Valor da Mensalidade` x `Faixa de Renda`**: O teste de Mann-Whitney não encontrou diferença significativa na distribuição do `VALOR_MENSALIDADE` entre clientes cancelados e não cancelados. No entanto, o teste Qui-quadrado, após categorizar `VALOR_MENSALIDADE`, indicou uma associação estatisticamente significativa entre o valor da mensalidade e o cancelamento. Clientes de diferentes faixas de valor de mensalidade têm diferentes taxas de cancelamento.
-   **Churn por `UF`**: Gráficos mostraram que a maioria das UFs apresenta mais cancelamentos do que clientes ativos. UFs como ES, PI, MG e PR têm churn acima de 69%, enquanto DF e PE têm as menores taxas (abaixo de 47%). Isso sugere que fatores geográficos e socioeconômicos podem influenciar o churn.
-   **Churn por `Região`**: Sul, Sudeste e Norte apresentaram churn acima de 60%, enquanto Nordeste e Centro-Oeste ficaram em torno de 50%. Isso reforça a necessidade de estratégias regionais.
-   **Churn por `Sexo`**: O cancelamento é ligeiramente maior para o sexo masculino (60%) do que para o feminino (58%).
-   **Amplitude do Impacto Financeiro**: A perda mínima total (1 mês) foi estimada em R$ 34.875.730,00 e a perda máxima (12 meses) em R$ 418.508.760,00. As UFs de SP, RJ, BA e MG são as que mais contribuem para essa perda.

### 4.3. Divisão Treino e Teste

Os dados foram divididos em 70% para treino e 30% para teste, com estratificação pela variável `CANCELADO` para manter as proporções. Variáveis como `regiao` e `TITULARIDADE` foram removidas antes da divisão.

### 4.4. Regras de Ajuste na Base de Treino

-   **Imputação de Valores**: Para `UF` e `SEXO`, foi utilizado `KNNImputer`. Para `VALOR_MENSALIDADE`, devido à alteração na distribuição com KNN, optou-se por categorizar a variável e tratar os valores ausentes como 'OUTROS'.
-   **Tratamento de Outliers/Valores Inválidos**: Valores de `IDADENAADESÃO` (negativos ou > 150) e `TEMPO_DE_PLANO_MESES` (negativos) foram descartados.
-   **Linearidade com Logito de P**: Verificou-se que as variáveis numéricas (`IDADENAADESÃO`, `TEMPO_DE_PLANO_MESES`, `QTD_CONSULTAS_12M`, `QTD_INTERNACOES_12M`) não apresentavam linearidade com o logito de p, o que levou à decisão de categorizá-las para o modelo de regressão logística.
    -   `TEMPO_DE_PLANO_MESES`: Categorizada em 'Até 280 meses' e 'Acima de 280 meses'.
    -   `IDADENAADESÃO`: Categorizada em '0-25', '25-50' e 'Acima de 50'.
    -   `QTD_CONSULTAS_12M`: Categorizada em '0-12', '12-30' e 'Acima de 30'.
    -   `QTD_INTERNACOES_12M`: Categorizada em '0', '1', '2' e 'acima de 2'.

## 5. Modelagem (Regressão Logística)

### 5.1. Primeiro Modelo

Um modelo de Regressão Logística (GLM com família Binomial) foi criado utilizando as variáveis categorizadas e transformadas. O resumo do modelo (`modelo_churn.summary()`) foi apresentado, mostrando os coeficientes, erros padrão, valores Z e p-valores.

**Observações do Primeiro Modelo:**
-   Algumas variáveis categóricas (e suas respectivas dummies) apresentaram p-valores próximos de 1, indicando que não eram significativas para o modelo (ex: `FAIXA_RENDA_Média renda`, `UF_RO`, `TEMPO_FAIXA_Acima de 280 meses`, `qtdconsultas12m_cat_12-30`, `qtdinter12m_cat_1`, `idadenaadesao_cat_Acima de 50`).

### 5.2. Interpretação da Odds Ratio (Primeiro Modelo)

-   **`FAIXA_RENDA_Baixa renda`**: OR = 21.1650. Isso significa que a chance de cancelamento para a faixa de baixa renda é aproximadamente 2016.50% maior do que para a faixa de alta renda (categoria de referência), mantendo outras variáveis constantes.
-   **Comparação entre UFs**: Clientes no RJ têm chances de cancelamento aproximadamente 14.3% maiores que os de SP. Clientes em MG têm chances de cancelamento aproximadamente 44% maiores que os de SP.

### 5.3. Reagrupamento de Variáveis Categóricas e Refatoração do Modelo

Com base nos p-valores e na análise de PCA para `UF` (que mostrou SP, RJ e BA com perfis mais distintos), algumas variáveis foram reagrupadas para simplificar o modelo e melhorar a significância:
-   `UF`: Reagrupadas em 'BA', 'RJ', 'SP' e 'OUTRAS'.
-   `FAIXA_RENDA`: 'Média renda' e 'Alta renda' foram agrupadas em 'Média/Alta'.
-   `QTD_CONSULTAS_12M`: '0-12' e '12-30' foram agrupadas em '0-30'.
-   `QTD_INTERNACOES_12M`: '0' e '1' foram agrupadas em '0/1'.
-   `IDADENAADESÃO`: '0-25' e 'Acima de 50' foram agrupadas em '0-25 e 50+'.
-   `TEMPO_FAIXA`: Removida do modelo após o reagrupamento.

### 5.4. Segundo Modelo (Refatorado)

Um novo modelo de Regressão Logística foi ajustado com as variáveis reagrupadas. O resumo do modelo (`modelo_final.summary()`) foi apresentado.

**Observações do Segundo Modelo:**
-   O número de observações e graus de liberdade mudou devido aos reagrupamentos.
-   O Pseudo R-squared (CS) foi de 0.5201, indicando uma capacidade explicativa razoável do modelo.
-   A maioria das variáveis reagrupadas agora apresenta p-valores significativos (próximos de 0), como `FAIXA_RENDA_Média/Alta`, `SEXO_M`, `grupo_UFs_OUTRAS`, `grupo_UFs_RJ`, `grupo_UFs_SP`, `INADIMPLENTE_SIM`, `VALOR_MENSALIDADE_1870.0`, `VALOR_MENSALIDADE_550.0`, `VALOR_MENSALIDADE_720.0`, `VALOR_MENSALIDADE_OUTROS`, `idadenaadesao_cat_25-50`, `idadenaadesao_cat_Acima de 50`.

## 6. Métricas de Qualidade do Modelo (TREINO)

As métricas foram calculadas para o conjunto de treino:

-   **MSE (Mean Squared Error)**: 0.0912. Considerado relativamente alto, pois o erro quadrático médio está em 9% da faixa total (0 a 1).
-   **RMSE (Root Mean Squared Error)**: 0.3020.
-   **Precisão, Recall, Acurácia, F1-Score e Lift por Threshold**:
    -   Para um threshold de 0.5, as métricas foram: Precisão: 0.8696, Recall: 0.9506, Acurácia: 0.8871, F1-Score: 0.9083, Lift: 1.4781.
    -   Precisão e Recall acima de 0.80 para faixas intermediárias de threshold, indicando bom equilíbrio.
-   **Curva ROC e AUC**: A AUC (Area Under the Curve) para o conjunto de treino foi de 0.75 (valor não explícito no texto, mas inferido pelo gráfico e contexto).

## 7. Avaliação do Modelo com Dados de TESTE

As mesmas regras de pré-processamento e categorização aplicadas ao conjunto de treino foram replicadas para o conjunto de teste.

### 7.1. Métricas de Qualidade (TESTE)

-   **MSE (Mean Squared Error)**: 0.2744.
-   **RMSE (Root Mean Squared Error)**: 0.5238.
-   **Precisão, Recall, Acurácia, F1-Score e Lift por Threshold**:
    -   Para um threshold de 0.5, as métricas foram: Precisão: 0.5884, Recall: 1.0000, Acurácia: 0.5884, F1-Score: 0.7409, Lift: 1.0000.
    -   Observa-se uma queda significativa nas métricas de teste em comparação com as de treino, especialmente na Precisão e Acurácia.
-   **Curva ROC e AUC**: A AUC para o conjunto de teste foi de 0.39. Este valor é consideravelmente baixo e indica que o modelo tem pouca capacidade de discriminação entre as classes no conjunto de teste.

## 8. Resumo e Parecer Final sobre o Modelo

O autor do projeto conclui que o modelo **não está pronto/finalizado** devido às diferenças significativas entre os resultados de treino e teste. Os principais pontos de atenção são:

-   **Vazamento de Dados/Aplicação de Regras**: Possibilidade de vazamento de informações do treino para o teste, ou aplicação incorreta das regras de pré-processamento no conjunto de teste.
-   **Overfitting (no treino)**: As probabilidades no treino apresentaram picos muito próximos de 0 e 1, indicando alta confiança do modelo, enquanto no teste as probabilidades se concentraram entre 0.6 e 1.0, com menos previsões extremas. A alta performance nas curvas de treino (precisão, recall, f1-score) e a AUC de 0.75 no treino contrastam fortemente com a AUC de 0.39 no teste, sugerindo um sobreajuste severo.
-   **Complexidade do Modelo/Multicolinearidade**: O modelo inicial com muitas variáveis e dummies de UF com alto VIF (Variance Inflation Factor) pode ter contribuído para o sobreajuste. A análise de VIF mostrou valores muito altos para algumas UFs (ex: SP, RJ, BA), indicando multicolinearidade.

**Sugestões para Melhoria (do autor do projeto):**
-   Revisar a aplicação das regras de pré-processamento no conjunto de teste.
-   Reduzir a dimensionalidade, especialmente para a variável `UF`, utilizando a variável `regiao` ou aplicando PCA de forma mais eficaz, ou agrupando UFs de forma mais estratégica (como feito no segundo modelo, mas que ainda não resolveu o problema de generalização).
-   Remover algumas dummies de UF para reduzir o VIF.

## 9. Insights Obtidos

Mesmo com as limitações do modelo final, a análise exploratória e bivariada forneceu insights valiosos:

-   **Perfil do Cliente Churn**: Clientes de baixa renda têm uma probabilidade significativamente maior de cancelar. Há uma alta taxa de inadimplência na base, que pode ser um forte preditor de churn.
-   **Fatores Geográficos**: O churn varia consideravelmente por UF e região, com Sul, Sudeste e Norte apresentando taxas mais altas. Isso indica a necessidade de estratégias de retenção regionalizadas.
-   **Impacto Financeiro**: O churn representa uma perda financeira substancial, concentrada em UFs específicas, o que direciona os esforços de mitigação.
-   **Tempo de Plano**: O comportamento de churn varia com o tempo de relacionamento do cliente, sugerindo que estratégias de fidelização devem ser adaptadas ao ciclo de vida do cliente.
-   **Variáveis de Uso do Plano**: `QTD_CONSULTAS_12M` e `QTD_INTERNACOES_12M` são assimétricas e podem indicar padrões de uso que, quando categorizados, podem ajudar a identificar clientes em risco.

## 10. Conclusão

O projeto demonstrou um processo robusto de análise de dados e modelagem, desde a exploração inicial até a construção e avaliação de um modelo preditivo. Embora o modelo final tenha apresentado problemas de generalização (overfitting), os insights da análise exploratória são cruciais para entender os fatores de churn. A recomendação é focar na melhoria da generalização do modelo, possivelmente através de uma seleção de variáveis mais rigorosa ou técnicas de regularização, e na validação das regras de pré-processamento entre os conjuntos de treino e teste. A comunicação com as áreas de negócio é fundamental para validar as premissas e os tratamentos aplicados aos dados.



