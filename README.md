# 🏀 NBA Salary Predictor

## Sobre o Projeto
Pipeline preditivo de ponta a ponta desenvolvido em Python para 
estimar o salário anual de jogadores da NBA com base em suas 
estatísticas de desempenho da temporada 2022-23.

O modelo auxilia times e agentes esportivos a identificar jogadores
sub ou supervalorizados no mercado, apoiando decisões estratégicas
de contratação baseadas em dados.

## Problema Preditivo
**Variável alvo:** `Salary` — salário anual do jogador em dólares (valor numérico contínuo)

**Questão de negócio:** Com base nas estatísticas de desempenho 
da temporada 2022-23, é possível prever o salário anual de um 
jogador da NBA? Isso é útil para:
- Times que precisam tomar decisões estratégicas de contratação
- Agentes esportivos que negociam contratos com base em desempenho real
- Análise de jogadores sub ou supervalorizados no mercado

## Dataset
- **Nome:** NBA Players Stats and Salaries — Temporada 2022-23
- **Fonte:** [Kaggle](https://www.kaggle.com)
- **Registros:** 467 jogadores
- **Colunas originais:** 52
- **Colunas utilizadas na modelagem:** 24
  
## Glossário de Métricas da NBA

Para facilitar o entendimento do projeto, explicamos abaixo
as principais siglas utilizadas no dataset:

### Estatísticas Básicas
| Sigla | Nome | Descrição |
|---|---|---|
| `GP` | Games Played | Jogos disputados na temporada |
| `GS` | Games Started | Jogos como titular |
| `MP` | Minutes Per Game | Minutos jogados por jogo |
| `PTS` | Points Per Game | Pontos marcados por jogo |
| `AST` | Assists Per Game | Assistências por jogo |
| `TRB` | Total Rebounds | Rebotes totais por jogo |
| `STL` | Steals Per Game | Roubos de bola por jogo |
| `BLK` | Blocks Per Game | Bloqueios por jogo |
| `TOV` | Turnovers | Perdas de bola por jogo |
| `PF` | Personal Fouls | Faltas pessoais por jogo |
| `FG` | Field Goals | Cestas convertidas por jogo |
| `3P` | Three Pointers | Cestas de 3 pontos por jogo |
| `FT` | Free Throws | Lances livres convertidos por jogo |

### Métricas de Percentual
| Sigla | Nome | Descrição |
|---|---|---|
| `TS%` | True Shooting % | Percentual real de aproveitamento considerando cestas de 2, 3 pontos e lances livres |
| `3PAr` | 3-Point Attempt Rate | Taxa de tentativas de 3 pontos em relação ao total de arremessos |
| `FTr` | Free Throw Rate | Taxa de lances livres em relação ao total de arremessos |
| `USG%` | Usage Rate | Percentual de jogadas do time que passam pelo jogador |

### Métricas Avançadas
| Sigla | Nome | Descrição |
|---|---|---|
| `PER` | Player Efficiency Rating | Índice geral de eficiência por minuto. Média da liga = 15.0. Acima de 25 indica candidato ao MVP |
| `WS` | Win Shares | Estimativa de quantas vitórias o jogador contribuiu para o time na temporada |
| `BPM` | Box Plus/Minus | Impacto do jogador em quadra comparado à média da liga. Zero = jogador médio |
| `VORP` | Value Over Replacement Player | Valor do jogador comparado a um substituto médio disponível no mercado |

### Posições
| Sigla | Nome | Descrição |
|---|---|---|
| `PG` | Point Guard | Armador — o menor e mais ágil, responsável pela organização do jogo |
| `SG` | Shooting Guard | Ala-armador — especialista em arremessos |
| `SF` | Small Forward | Ala — equilíbrio entre agilidade e força |
| `PF` | Power Forward | Ala-pivô — mais físico que os alas |
| `C` | Center | Pivô — o maior e mais físico, dominante próximo à cesta |

## Pipeline — 6 Fases

### Fase 1 — Análise Exploratória de Dados (EDA)
- Estatísticas descritivas do dataset
- Análise da distribuição do salário — assimetria positiva identificada
- Gráficos de dispersão entre estatísticas e salário
- Mapa de calor de correlação de Pearson

### Fase 2 — Tratamento e Limpeza (Data Prep)
- Remoção de 18 colunas desnecessárias (baixa correlação e redundância)
- Simplificação de posições combinadas (ex: PG-SG → PG)
- Verificação e remoção de duplicatas
- Imputação de valores ausentes pela **mediana** — justificativa: distribuição assimétrica do salário torna a mediana mais robusta que a média
- Detecção e tratamento de outliers por **winsorização (IQR)**

### Fase 3 — Feature Engineering
Criação de 4 novas variáveis:
- `eficiencia`: PTS + AST + TRB + STL + BLK — índice geral de desempenho
- `minutos_totais`: MP × GP — volume total de participação na temporada
- `aproveitamento_geral`: média entre TS% e FTr
- `tier_time`: agrupamento dos 30 times em 3 tiers por poder financeiro

### Fase 4 — Preparação para Modelagem
- **Label Encoding** na variável categórica `Position`
- Análise de **multicolinearidade** — remoção de 12 colunas com correlação acima de 0.85
- Divisão amostral **80/20** (373 treino / 94 teste)
- **Escalonamento:** testamos Padronização (StandardScaler) e Normalização (MinMaxScaler) — métricas idênticas para Regressão Linear, mantivemos StandardScaler

### Fase 5 — Modelagem e Diagnóstico
Treinamos e comparamos 2 modelos:

| Modelo | MAE | RMSE | R² | Overfitting |
|---|---|---|---|---|
| Regressão Linear | US$ 4,921,851 | US$ 6,371,145 | 0.6943 | ✅ Não |
| KNN Regressor | US$ 4,319,213 | US$ 6,637,371 | 0.6682 | ⚠️ Sim |

**Modelo campeão:** Regressão Linear — menor RMSE no teste e melhor generalização

### Otimização do KNN
Testamos diferentes valores de K para encontrar o número ideal
de vizinhos para o dataset de 467 jogadores:

| K | RMSE | R² |
|---|---|---|
| 3 | US$ 7,080,808 | 0.6223 |
| 5 | US$ 6,637,371 | 0.6682 |
| 7 | US$ 6,500,433 | 0.6817 |
| **10** | **US$ 6,115,352** | **0.7183** |
| 15 | US$ 6,152,925 | 0.7148 |
| 20 | US$ 6,243,159 | 0.7064 |
| 30 | US$ 6,256,959 | 0.7051 |

K=10 apresentou o melhor resultado superando inclusive a
Regressão Linear. Identificado como melhoria para a v2.

### Fase 6 — Avaliação e Versionamento
- Métricas técnicas: MAE, MSE, RMSE e R²
- Gráfico de dispersão: valores reais vs previstos
- Distribuição dos resíduos — concentrados próximos de zero ✅
- Veredito de negócios documentado
- Modelo salvo em `models/v1/`

## Métricas do Modelo v1
| Métrica | Valor |
|---|---|
| MAE | US$ 4,921,851 |
| RMSE | US$ 6,371,145 |
| R² | 0.6943 |
| Data de treinamento | 2026-07-14 |

## Experimento — Transformação Logarítmica
Testamos a transformação logarítmica no salário para melhorar
a Regressão Linear — o R² caiu de 0.69 para 0.39, resultado
pior que o modelo original. Mantivemos o modelo sem transformação.

## Estrutura do Projeto
nba-salary-predictor/
├── data/
│   ├── raw/                        # Dataset bruto original
│   ├── processed/                  # Datasets limpos e tratados
│   └── final/                      # Dataset final para modelagem
├── models/
│   └── v1/
│       ├── modelo_regressao_v1.pkl # Modelo treinado
│       ├── scaler_v1.pkl           # Scaler para novos dados
│       └── metricas_v1.json        # Métricas e metadados
├── notebooks/
│   └── nba_salary_predictor.ipynb  # Notebook principal
├── outputs/
│   └── figures/                    # Gráficos exportados
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md

## Melhorias Identificadas para Versões Futuras
- Testar modelos não lineares como **Random Forest** e **Gradient Boosting**
- Incluir dados de popularidade e marketing dos jogadores
- Analisar o impacto de histórico de lesões no salário
- Explorar dados de múltiplas temporadas para capturar tendências
- Implementar validação cruzada para avaliação mais robusta

## Tecnologias Utilizadas
- Python 3.10+
- Google Colab
- Pandas, NumPy, Matplotlib, Seaborn
- Scikit-learn (LinearRegression, KNeighborsRegressor, StandardScaler, MinMaxScaler, LabelEncoder)
- Joblib, JSON
- GitHub para versionamento com branches

## Como Executar
### No Google Colab (recomendado)
1. Acesse [Google Colab](https://colab.research.google.com)
2. Faça upload do arquivo `notebooks/nba_salary_predictor.ipynb`
3. Faça upload do dataset `nba_2022-23_all_stats_with_salary.csv`
4. Execute as células na ordem, de cima para baixo

### Localmente
1. Instale as dependências:
2. pip install -r requirements.txt
3. 2. Abra o arquivo `notebooks/nba_salary_predictor.ipynb`
3. Execute as células na ordem

## Versionamento
O projeto utilizou **Git Flow** com branches organizadas por fase:
- `feature/eda` → Fase 1
- `feature/data-prep` → Fase 2
- `feature/feature-engineering` → Fase 3
- `feature/modeling` → Fase 4
- `feature/avaliacao` → Fases 5 e 6

## Vídeo de Demonstração
*(link será adicionado após a gravação)*
