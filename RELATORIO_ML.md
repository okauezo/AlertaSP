
# Relatório de Machine Learning — AlertaSP

**Global Solution FIAP 2026.2 — CitizenTech**  
**Disciplina:** Inteligência Artificial & Machine Learning  
**Algoritmo:** Random Forest Classifier  
**Vídeo de apresentação:** https://www.youtube.com/watch?v=aoErw4pQP54

## 1. Problema e objetivo

Os alagamentos e inundações representam um desafio para a mobilidade e segurança dos cidadãos de São Paulo, especialmente durante períodos de chuva intensa.

O AlertaSP propõe a utilização de Machine Learning para analisar padrões meteorológicos relacionados a ocorrências históricas e auxiliar na identificação de períodos de maior risco.

A pergunta de pesquisa é:

**É possível estimar a ocorrência de registros de alagamentos ou inundações no dia seguinte, no município de São Paulo, a partir de dados meteorológicos e sazonais?**

O modelo desenvolvido realiza uma classificação binária, identificando dias com e sem ocorrências registradas.

## 2. Dados utilizados

Foram utilizadas duas fontes principais:

**INMET:** séries meteorológicas oficiais das estações São Paulo/Mirante (A701) e São Paulo/Interlagos (A771), contendo precipitação, temperatura e umidade.

Fonte:
https://portal.inmet.gov.br/dadoshistoricos

**GeoSampa/Defesa Civil:** registros históricos de alagamentos e inundações, acessados por meio de uma base consolidada publicamente por terceiros.

Catálogo oficial:
https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/432c06b1-03a1-4b1f-9210-423e1b58e869

Base consolidada utilizada:
https://github.com/23216886alunounivesp/sp-alagamentos-inundacoes-defesa-civil

O arquivo de ocorrências contém 12.487 registros entre 2013 e 2026. O experimento principal utiliza informações de 2022 a 2025.

Os dados de 2026 foram preparados separadamente, mas não utilizados na avaliação final devido à necessidade de confirmar a cobertura da fonte de ocorrências.

## 3. Preparação dos dados

O processamento foi realizado em Python, utilizando Pandas e NumPy.

As etapas incluíram:

1. Download e leitura dos datasets.
2. Normalização de datas, horários e valores numéricos.
3. Conversão de UTC para o horário de São Paulo.
4. Identificação de valores ausentes e registros duplicados.
5. Agregação diária das medições meteorológicas.
6. Cruzamento com ocorrências históricas pela data.
7. Criação de 12 variáveis meteorológicas e sazonais.
8. Separação cronológica dos conjuntos.

A variável-alvo `alvo_registro_amanha` identifica se existe pelo menos um registro no dia seguinte.

Foram utilizadas variáveis de precipitação diária, chuva acumulada em três e sete dias, temperatura, umidade e sazonalidade representada por seno e cosseno do dia do ano.

O tratamento de valores ausentes foi realizado por imputação de mediana dentro do pipeline, ajustada apenas aos dados de treinamento.

### Divisão temporal

| Conjunto | Período | Amostras |
|---|---|---:|
| Treino | 2022–2024 | 1.093 |
| Teste | 2025 | 364 |
| Provisório (não usado) | 2026 | 183 |

A divisão temporal evita o embaralhamento aleatório de observações de diferentes períodos. O ano de 2025 funciona como **conjunto de teste**: não participa do treinamento nem da escolha dos hiperparâmetros, e só é usado uma vez, na avaliação final.

## 4. Modelo e treinamento

O algoritmo escolhido foi o Random Forest Classifier, devido à capacidade de modelar relações não lineares entre variáveis meteorológicas e a ocorrência de eventos.

Também foram consideradas suas possibilidades de regularização e análise de importância das variáveis.

A otimização utilizou GridSearchCV com TimeSeriesSplit de quatro divisões temporais, aplicado somente ao conjunto de treinamento.

Foram avaliadas 12 combinações de três hiperparâmetros:

- n_estimators: 100, 200 e 300
- max_depth: 5 e 10
- min_samples_leaf: 2 e 5

A métrica principal para selecionar a configuração foi Average Precision (AP), adequada à avaliação de classificação com classes desbalanceadas.

### Melhores hiperparâmetros

- n_estimators = 300
- max_depth = 5
- min_samples_leaf = 5

A Average Precision média na validação cruzada foi **0,6463** (0,6466 na reexecução do notebook com scikit-learn 1.8; a diferença vem da versão da biblioteca e não altera a escolha nem as métricas finais).

### Comparação das 12 combinações (validação cruzada no treino)

| max_depth | min_samples_leaf | n_estimators | AP treino | AP validação cruzada | Ranking |
|---:|---:|---:|---:|---:|---:|
| 5 | 5 | 300 | 0,8966 | 0,6463 | 1 |
| 5 | 5 | 100 | 0,8950 | 0,6453 | 2 |
| 10 | 5 | 100 | 0,9461 | 0,6452 | 3 |
| 10 | 5 | 300 | 0,9485 | 0,6441 | 4 |
| 5 | 5 | 200 | 0,8957 | 0,6433 | 5 |
| 10 | 5 | 200 | 0,9479 | 0,6423 | 6 |
| 5 | 2 | 300 | 0,9200 | 0,6421 | 7 |
| 5 | 2 | 200 | 0,9203 | 0,6377 | 8 |
| 5 | 2 | 100 | 0,9181 | 0,6371 | 9 |
| 10 | 2 | 100 | 0,9952 | 0,6207 | 10 |
| 10 | 2 | 200 | 0,9965 | 0,6202 | 11 |
| 10 | 2 | 300 | 0,9960 | 0,6184 | 12 |

Fonte: `ml/resultados/08_comparacao_hiperparametros.csv`.

As árvores mais profundas e com folhas pequenas (`max_depth=10`, `min_samples_leaf=2`) praticamente decoram o treino (AP ≈ 0,99), mas ficam nas últimas posições da validação cruzada. O `min_samples_leaf` foi o hiperparâmetro mais decisivo, e o `n_estimators` teve efeito pequeno.

## 5. Resultados

| Métrica | Treinamento | Teste 2025 |
|---|---:|---:|
| Accuracy | 0,8161 | 0,7527 |
| Precision | 0,6427 | 0,4803 |
| Recall | 0,8549 | 0,7176 |
| F1-score | 0,7338 | 0,5755 |
| Average Precision | 0,8341 | 0,6237 |
| ROC-AUC | 0,9132 | 0,8315 |

A referência sem capacidade de discriminação apresentou AP de aproximadamente 0,2335 em 2025.

O modelo completo alcançou AP de 0,6237, indicando capacidade de priorizar dias que tiveram registros.

### Matriz de confusão — 2025

- Verdadeiros negativos: 213
- Falsos positivos: 66
- Falsos negativos: 24
- Verdadeiros positivos: 61

O modelo identificou 61 dos 85 dias com ocorrências registradas, correspondendo a um recall de aproximadamente 71,76%.

Entretanto, a precision de 48,03% indica quantidade relevante de falsos positivos.

![Matriz de confusão — Teste 2025](ml/resultados/08_matriz_confusao.png)

![Curvas Precision-Recall — 2025](ml/resultados/09_curvas_precision_recall.png)

### Comparação entre conjuntos de variáveis

| Modelo | Average Precision |
|---|---:|
| Completo | 0,6237 |
| Apenas meteorologia | 0,5783 |
| Apenas sazonalidade | 0,5177 |

O modelo completo apresentou melhor desempenho, sugerindo que a combinação de informações meteorológicas e sazonais acrescenta capacidade preditiva.

A importância por permutação destacou a sazonalidade, a temperatura em Interlagos e a precipitação acumulada no Mirante.

![Importância por permutação — 2025](ml/resultados/09_importancia_permutacao.png)

## 6. Overfitting e limitações

A Average Precision caiu de 0,8341 no treinamento para 0,6237 no teste de 2025.

Essa diferença indica perda de generalização, compatível com algum grau de overfitting e possíveis diferenças na distribuição dos dados entre os períodos.

A limitação da profundidade das árvores e o uso de min_samples_leaf=5 ajudam a controlar a complexidade do modelo.

Outras limitações incluem:

- Cobertura meteorológica restrita a duas estações.
- Previsões agregadas para o município, sem detalhamento por rua.
- Possibilidade de registros incompletos de ocorrências.
- Ausência de horários completos na base de alagamentos.
- Base consolidada por terceiros, ainda sem auditoria integral contra a origem oficial.
- Dados de 2026 ainda não validados para teste final.

A ausência de registro não comprova ausência de alagamento. Além disso, as previsões experimentais não devem substituir alertas oficiais.

## 7. Conclusão e impacto para o cidadão

O experimento demonstrou que é possível identificar padrões relacionados a registros futuros de alagamentos utilizando Machine Learning e dados históricos reais.

O Random Forest apresentou desempenho superior às referências avaliadas, com ROC-AUC de 0,8315 e Average Precision de 0,6237 no teste temporal de 2025.

Apesar dos resultados promissores, a quantidade de falsos positivos e falsos negativos evidencia a necessidade de melhorias e validação adicional.

Para o cidadão, o AlertaSP representa uma proposta de apoio à prevenção e à tomada de decisão. Futuramente, o projeto poderá incorporar medições hidrológicas locais, sensores IoT e informações de trajetos para produzir alertas mais específicos.

A solução contribui conceitualmente para os ODS 11 (Cidades e Comunidades Sustentáveis) e 13 (Ação Contra a Mudança Global do Clima).

**Conclusão:** o modelo demonstrou capacidade preditiva experimental, mas ainda não está validado para operação em situações reais de emergência.
