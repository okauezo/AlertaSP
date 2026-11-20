
# AlertaSP — Inteligência contra Alagamentos

**Global Solution FIAP 2026.2 | CitizenTech**  
Curso: Engenharia de Software  
Disciplina: Inteligência Artificial & Machine Learning

| Integrante | RM |
|---|---|
| Ricardo Di Tilia | RM555155 |
| Bento Rangel | RM559124 |
| Eric Yuji | RM554869 |
| Kauê Pires | RM554403 |
| Higor Batista | RM558907 |

**Vídeo de IA & Machine Learning:** https://www.youtube.com/watch?v=aoErw4pQP54


## Sobre o projeto

O AlertaSP é uma proposta de plataforma inteligente para monitoramento e prevenção de riscos de alagamentos na cidade de São Paulo.

A solução prevê a utilização de Machine Learning, dados meteorológicos e registros históricos de ocorrências para auxiliar cidadãos a tomar decisões sobre seus deslocamentos.

Como evolução, o aplicativo poderá permitir que usuários cadastrem trajetos favoritos, como casa-trabalho, e recebam alertas relevantes às regiões monitoradas.

## Objetivo do Machine Learning

Desenvolver um modelo de classificação supervisionada capaz de estimar a possibilidade de haver pelo menos um registro de alagamento ou inundação no município de São Paulo no dia seguinte.

O modelo utiliza dados meteorológicos observados até o final do dia atual e informações de sazonalidade.

**Importante:** o modelo atual realiza uma classificação diária para o município, não uma previsão por rua ou um sistema de alertas em tempo real.

## Fontes dos dados

### 1. INMET — Instituto Nacional de Meteorologia

Dados históricos de estações meteorológicas automáticas.

Link oficial:
https://portal.inmet.gov.br/dadoshistoricos

Estações utilizadas:
- A701 — São Paulo / Mirante
- A771 — São Paulo / Interlagos

Variáveis utilizadas:
- Precipitação (mm)
- Temperatura média (°C)
- Umidade relativa (%)

### 2. GeoSampa — Prefeitura de São Paulo

Catálogo oficial dos registros de alagamentos da Defesa Civil.

Link oficial:
https://metadados.geosampa.prefeitura.sp.gov.br/geonetwork/srv/api/records/432c06b1-03a1-4b1f-9210-423e1b58e869

### 3. Base consolidada de ocorrências

CSV público derivado dos dados da Defesa Civil/GeoSampa, utilizado neste experimento.

Repositório:
https://github.com/23216886alunounivesp/sp-alagamentos-inundacoes-defesa-civil

O arquivo contém 12.487 registros de alagamentos e inundações de 2013 a 2026.

A base é mantida por terceiros. Sua correspondência integral com os registros oficiais e a cobertura de 2026 ainda precisam de auditoria independente.

Mais informações: `ml/dados/README.md`.

## Tecnologias

- Python
- Pandas e NumPy
- Scikit-learn
- Random Forest Classifier
- Matplotlib
- Jupyter Notebook
- Joblib
- Requests

## Estrutura do projeto

O repositório está organizado para facilitar a reprodução dos experimentos de Machine Learning, desde a coleta dos dados até a avaliação do modelo.

```text
AlertaSP/
│
├── README.md
├── RELATORIO_ML.md
├── .gitignore
│
└── ml/
    │
    ├── requirements.txt
    ├── requirements-lock.txt
    │
    ├── notebooks/
    │   └── AlertaSP_ML.ipynb        ← notebook executado (entrega)
    │
    ├── scripts/
    │   ├── 01_inspecionar_inmet.py
    │   ├── 02_preparar_inmet.py
    │   ├── 03_preparar_alagamentos.py
    │   ├── 04_cruzar_dados.py
    │   ├── 05_analisar_dados.py
    │   ├── 06_consolidar_historico.py
    │   ├── 07_preparar_features.py
    │   ├── 08_treinar_random_forest.py
    │   └── 09_avaliar_modelo.py
    │
    ├── dados/
    │   ├── README.md
    │   ├── brutos/                  (gerada pelos scripts, não versionada)
    │   ├── processados/
    │   └── modelagem/
    │       ├── treino_2022_2024.csv
    │       ├── teste_2025.csv
    │       └── teste_2026_PROVISORIO.csv
    │
    ├── modelos/
    │   └── random_forest_alagamentos.joblib
    │
    └── resultados/
        ├── 01..03_*.png             (análise exploratória)
        ├── 08_comparacao_hiperparametros.csv
        ├── 08_metricas_modelo.csv
        ├── 08_matriz_confusao.png
        ├── 08_importancia_features.png
        ├── 09_comparacao_modelos.csv
        ├── 09_curvas_precision_recall.png
        ├── 09_importancia_permutacao.png
        └── 10_metricas_notebook.csv
```

### Organização dos diretórios

| Diretório | Finalidade |
|---|---|
| `scripts/` | Coleta, tratamento, treinamento e avaliação |
| `dados/brutos/` | Arquivos originais baixados |
| `dados/processados/` | Dados limpos e consolidados |
| `dados/modelagem/` | Datasets de treino (2022–2024), teste (2025) e provisório (2026) |
| `notebooks/` | Notebook executado com o ciclo de treino e avaliação |
| `modelos/` | Modelo Random Forest treinado |
| `resultados/` | Gráficos e métricas dos experimentos |

Os dados processados, os datasets de modelagem e o modelo treinado estão versionados para que o notebook rode sem baixar nada. Os arquivos brutos (ZIPs do INMET) são baixados pelos scripts e não são versionados.

## Requisitos para execução

- Python 3.9 ou superior
- Git
- Acesso à internet para baixar os dados públicos
- Ambiente virtual Python (venv)

O desenvolvimento original foi realizado em macOS.

## 1. Clonar o repositório

No terminal:

```bash
git clone <URL_DO_REPOSITORIO>
cd AlertaSP
```

Substitua `<URL_DO_REPOSITORIO>` pela URL HTTPS exibida em Code → HTTPS no GitHub.

## 2. Criar e ativar o ambiente virtual

### macOS / Linux

Na raiz do repositório:

```bash
python3 -m venv ml/.venv
source ml/.venv/bin/activate
cd ml
```

### Windows — PowerShell

Na raiz do repositório:

```powershell
py -3 -m venv ml\.venv
.\ml\.venv\Scripts\Activate.ps1
cd ml
```

Caso a ativação seja bloqueada pelo PowerShell, é possível executar diretamente `ml\.venv\Scripts\python.exe` a partir da raiz, sem ativar o ambiente.

## 3. Instalar dependências

Os comandos seguintes devem ser executados dentro da pasta `ml`, com o ambiente virtual ativo:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Verificar a instalação:

```bash
python -c "import pandas, numpy, sklearn, matplotlib, requests, joblib; print('Ambiente configurado!')"
```

## 4. Executar os scripts de 01 a 09

Execute na ordem apresentada abaixo.

### Script 01 — Inspeção do INMET

Baixa o histórico meteorológico de 2025 e identifica as estações utilizadas no projeto.

```bash
python scripts/01_inspecionar_inmet.py
```

### Script 02 — Preparação do INMET

Extrai os dados das estações A701 e A771, converte valores e gera o CSV meteorológico de 2025.

```bash
python scripts/02_preparar_inmet.py
```

### Script 03 — Preparação dos alagamentos

Baixa o CSV consolidado de ocorrências, padroniza datas, identifica tipos de eventos e prepara os registros de 2025.

```bash
python scripts/03_preparar_alagamentos.py
```

### Script 04 — Cruzamento inicial

Relaciona os registros de alagamentos de 2025 às medições meteorológicas, agrupadas por data local.

```bash
python scripts/04_cruzar_dados.py
```

### Script 05 — Análise exploratória

Analisa distribuição de ocorrências, valores ausentes e precipitação, gerando gráficos.

```bash
python scripts/05_analisar_dados.py
```

### Script 06 — Consolidação histórica

Baixa os dados meteorológicos dos anos necessários, cruza-os com o histórico de alagamentos e gera uma base diária de 2022 a 2026.

```bash
python scripts/06_consolidar_historico.py
```

Os dados de 2026 são limitados a uma data provisória, definida conforme as fontes disponíveis. A data da última ocorrência não comprova a cobertura completa da fonte.

### Script 07 — Feature Engineering

Calcula as 12 variáveis de entrada e separa os dados em treino (2022–2024), teste (2025) e conjunto provisório (2026).

```bash
python scripts/07_preparar_features.py
```

### Script 08 — Treinamento Random Forest

Treina o modelo, testa 12 combinações de hiperparâmetros com validação cruzada temporal, calcula métricas e salva o modelo.

```bash
python scripts/08_treinar_random_forest.py
```

### Script 09 — Avaliação complementar

Compara o modelo completo com versões baseadas apenas em sazonalidade ou meteorologia, além de gerar gráficos e importância por permutação.

```bash
python scripts/09_avaliar_modelo.py
```

### Executar toda a sequência de uma vez

No macOS/Linux, dentro de `ml`:

```bash
set -e

for n in 01 02 03 04 05 06 07 08 09; do
    arquivo=$(find scripts -maxdepth 1 -name "${n}_*.py" -print -quit)
    test -n "$arquivo" || {
        echo "Script ${n} não encontrado"
        exit 1
    }
    python "$arquivo"
done
```

O comando interrompe a sequência caso algum script apresente erro.

## 5. Executar o notebook

Abra o notebook `ml/notebooks/AlertaSP_ML.ipynb` no VS Code ou JupyterLab. Ele já está salvo com as saídas da execução.

Para iniciar o JupyterLab:

```bash
python -m jupyter lab
```

Selecione o interpretador da pasta `ml/.venv` e execute as células de cima para baixo.

O notebook apresenta:
- Pergunta de ML, fontes de dados e divisão temporal
- Carregamento dos dados preparados
- Escolha justificada do Random Forest
- Busca de 12 combinações de hiperparâmetros com validação cruzada temporal (só no treino)
- Métricas de treino e teste lado a lado e discussão de overfitting
- Matriz de confusão e curva Precision-Recall
- Importância das variáveis
- Conclusão para o cidadão

## Metodologia de treinamento

Períodos utilizados:

| Período | Finalidade |
|---|---|
| 2022–2024 | Treinamento e validação cruzada |
| 2025 | Teste (avaliação final, fora da busca de hiperparâmetros) |
| 2026 | Conjunto provisório, não utilizado nas métricas finais |

Algoritmo: Random Forest Classifier.

Hiperparâmetros testados:
- `n_estimators`: 100, 200 e 300
- `max_depth`: 5 e 10
- `min_samples_leaf`: 2 e 5

Total: 12 combinações, com 4 divisões temporais, totalizando 48 ajustes na busca.

Métrica principal: Average Precision (AP).

Os valores ausentes são tratados com SimpleImputer, ajustado apenas aos dados de treinamento de cada etapa.

## Resultados obtidos

Melhores hiperparâmetros:
- `n_estimators=300`
- `max_depth=5`
- `min_samples_leaf=5`

| Métrica | Treino | Teste 2025 |
|---|---:|---:|
| Accuracy | 0,8161 | 0,7527 |
| Precision | 0,6427 | 0,4803 |
| Recall | 0,8549 | 0,7176 |
| F1-score | 0,7338 | 0,5755 |
| Average Precision | 0,8341 | 0,6237 |
| ROC-AUC | 0,9132 | 0,8315 |

A Average Precision da referência sem discriminação em 2025 é aproximadamente 0,2335.

Na avaliação de 2025, o modelo identificou 61 dos 85 dias com ocorrências registradas.

## Limitações

1. A previsão é diária e municipal, não por rua.
2. Apenas duas estações meteorológicas representam as condições climáticas utilizadas.
3. Ausência de registro não garante ausência de alagamento.
4. O CSV de ocorrências é derivado de fontes públicas e publicado por terceiros.
5. A cobertura dos dados de 2026 ainda não foi validada.
6. O modelo apresenta perda de desempenho entre treino e teste.
7. O experimento não deve ser utilizado isoladamente para emitir alertas oficiais de emergência.

## Reprodução dos resultados

Os números apresentados foram obtidos com os arquivos consultados durante o desenvolvimento, em novembro de 2026.

Como as fontes públicas podem ser atualizadas, novos downloads podem produzir resultados diferentes.

Para reprodução exata, é necessário preservar as versões dos arquivos de entrada e das dependências utilizadas.

## Documentação adicional

- `RELATORIO_ML.md`: metodologia, resultados e conclusões
- `ml/dados/README.md`: origem e tratamento dos dados
- `ml/resultados/`: gráficos, métricas e comparações
- Notebook executado: demonstração do ciclo completo de ML

## Objetivos de Desenvolvimento Sustentável

- ODS 11 — Cidades e Comunidades Sustentáveis
- ODS 13 — Ação Contra a Mudança Global do Clima

## Aviso

O AlertaSP é um protótipo acadêmico de pesquisa. Seus resultados não substituem alertas da Defesa Civil, do CGE ou de outros órgãos oficiais.
