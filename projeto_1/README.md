# 📈 Projeto 1 — Previsão de Emissões Globais de Gases de Efeito Estufa

🎥 **Apresentação em vídeo:** [Assista no YouTube](https://youtu.be/y9fnmKNXnbA?si=0qkUUrP7y_ANGfwi)

## 🎯 Objetivo
O objetivo deste projeto foi prever a evolução das emissões globais de gases de efeito estufa (GEE) utilizando métodos estatísticos clássicos de séries temporais. A escolha do tema está alinhada ao contexto de mudanças climáticas, em que a compreensão do comportamento histórico de emissões auxilia na formulação de políticas ambientais, atendendo também ao escopo descrito no projeto.

## 🗂️ Dataset
Foi utilizado o conjunto de dados público **CO₂ Emissions** disponível no Kaggle ([link para o dataset](https://www.kaggle.com/datasets/ulrikthygepedersen/co2-emissions-by-country)). O arquivo contém séries anuais de **emissões de dióxido de carbono e outros gases** para diferentes países e setores, cobrindo o período de **1850 a 2023**.

## 🔍 Filtragem
- **Categoria:** filtrou-se a coluna `Name` para o valor *World*, removendo emissões regionais/sectoriais.  
  Cada observação representa o total global por ano.  
- **Variável alvo:** selecionou-se a coluna `total_ghg`.  
  As demais variáveis foram descartadas para manter uma **modelagem univariada**.  
- **Tratamento de nulos:** a série `total_ghg` para *World* estava praticamente completa, não sendo necessária interpolação ou outro tipo de preenchimento.

## ⚙️ Pré-processamento
- **Conversão para série temporal:**  
  A coluna `year` foi usada como índice cronológico, e os valores de `total_ghg` formaram a série *y*.  
  Divisão em treino (≈80 %) e teste (≈20 %), preservando a ordem temporal.  
- **Estacionarização:**  
  Testes ADF e KPSS indicaram não estacionaridade.  
  Três transformações foram testadas:
  1. Δ₁ – diferença de primeira ordem  
  2. log – atenuação do crescimento exponencial  
  3. log + Δ₁ – combinação com melhor resultado  
- **Autocorrelação:**  
  Gráficos ACF/PACF sugeriram ordens baixas para AR e MA.

## 🧮 Modelos testados
| Modelo | Observações |
|---------|-------------|
| **SES** | Suavização exponencial simples, sem tendência e sem sazonalidade |
| **Holt** | Modelo com tendência aditiva, mas sem componente sazonal |
| **Holt-Winters** | Modelo com tendência e sazonalidade aditivas (período sazonal = 5) |
| **ARIMA(1,1,1)** | Modelo autorregressivo integrado de média móvel; resíduos bem comportados |
| **SARIMA(1,1,1)(1,1,4)[15]** | Extensão sazonal do ARIMA, com ciclo de longo prazo (~15 anos) |

Todos ajustados sobre a série transformada (*log + Δ₁*) e reconvertidos à escala original para comparação.

## 📊 Avaliação de Desempenho
Métricas utilizadas:
- **MSE**, **RMSE**, **MAE** – erros absolutos e quadráticos  
- **R²** – fração da variância explicada  
- **AIC**, **BIC** – critérios de informação  

| Modelo | MSE (teste) | RMSE | MAE | R² | AIC | BIC |
|:--|--:|--:|--:|--:|--:|--:|
| Holt-Winters | 2.33 × 10⁶ | 1527.4 | 1224.4 | 0.933 | 1657.4 | 1683.8 |
| Holt | 2.34 × 10⁶ | 1530.1 | 1225.7 | 0.933 | 1646.8 | 1658.6 |
| SARIMA | 1.58 × 10⁷ | 3974.6 | 3069.1 | 0.547 | –531.9 | –509.4 |
| ARIMA | 2.77 × 10⁷ | 5259.7 | 4303.9 | 0.207 | –640.5 | –631.7 |
| SES | 1.17 × 10⁸ | 10833.2 | 9081.5 | –2.364 | 1706.7 | 1712.6 |

**Interpretação:**  
- **Holt-Winters** e **Holt** apresentaram **os menores erros (RMSE ≈ 1527 e 1530)** e **R² ≈ 0,93**, o que indica excelente capacidade de previsão na escala original.  
  Holt se destacou levemente por ter **AIC/BIC menores** (1646.8 / 1658.6), representando o melhor compromisso entre precisão e simplicidade.  

- **ARIMA(1,1,1)** e **SARIMA(1,1,1)(1,1,4)[15]** mostraram **erros de previsão mais altos** (RMSE ≈ 5259 e 3975, respectivamente), mas obtiveram **AIC/BIC mais baixos (negativos)**, o que sugere **bom ajuste estatístico na escala modelada** e **resíduos bem comportados** (sem autocorrelação).

- **SES** teve o **pior desempenho geral**, com erros elevados e R² negativo, refletindo a ausência de tendência no modelo.


## 🧪 Diagnóstico dos Resíduos
- **ARIMA / SARIMA:** resíduos ≈ ruído branco (p-values Ljung–Box ≈ 1.0).  
- **Holt / Holt-Winters:** resíduos com autocorrelação (p ≈ 0.03), ainda assim melhor RMSE/MAE.  

## 🧭 Conclusão

Os resultados mostram que **não há um modelo único superior em todas as métricas**, mas há diferenças importantes quanto à forma de modelar a tendência e a dependência temporal da série.

- **Holt** → apresentou melhor desempenho preditivo na escala original, com os menores erros (RMSE e MAE) e boa capacidade explicativa (R² ≈ 0,93). Ainda assim, exibiu autocorrelação residual, indicando que parte da estrutura temporal permaneceu não modelada.  
- **ARIMA(1,1,1)** → demonstrou bom ajuste segundo os critérios de informação (AIC e BIC), além de resíduos bem distribuídos e sem autocorrelação, caracterizando um modelo estatisticamente consistente. Apesar de erros ligeiramente maiores na escala original, representa o melhor equilíbrio entre simplicidade, ajuste e diagnóstico residual.  
- **SARIMA(1,1,1)(1,1,4)[15]** → capturou possíveis ciclos de longo prazo, mas não superou o Holt/Holt-Winters em desempenho preditivo.  
- **SES** → mostrou-se inadequado, por não incorporar tendência.

> **Em síntese:**  
> - **Holt/Holt-Winters:** menores erros, mas resíduos correlacionados;
> - **ARIMA/SARIMA:** resíduos bem comportados, sem autocorrelação;
> - **ARIMA(1,1,1)** se destaca como o **modelo mais adequado** para representar o comportamento da série analisada.

## 🚀 Como rodar

### 1) Pré-requisitos
- Python 3.10+ (recomendado 3.11)
- Git (opcional para clonar o repositório)

### 2) Clonar o repositório
Clone o repositório oficial do projeto a partir do GitHub:

```bash
git clone https://github.com/thaisaraujom/PPGEEC2327-time-series
cd PPGEEC2327-time-series
```

### 3) Criar e ativar o ambiente virtual
```bash
# macOS / Linux
python -m venv .venv
source .venv/bin/activate

# Windows (PowerShell)
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 4) Baixar o dataset ([CO₂ Emissions](https://www.kaggle.com/datasets/ulrikthygepedersen/co2-emissions-by-country))

### 5) **Instalar as dependências a partir do `requirements.txt`:**
O comando a seguir instalará todas as bibliotecas necessárias de uma só vez.
```bash
pip install -r requirements.txt
```

### 6) Execução do Notebook
Com o ambiente preparado, você pode optar pelo **VSCode**, **Google Colab** ou **Jupyter Notebook/Lab** para a execução do projeto.
