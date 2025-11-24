# 🧠 Projeto 2 — Previsão de Séries Temporais: Deep Learning vs. Estatística

📢 **Apresentação (Slides):** [Clique aqui para visualizar (PDF)](./time-series.pdf)

## 🎯 Objetivo
Atendendo aos requisitos da disciplina, o objetivo central deste projeto é realizar um **estudo comparativo** de desempenho na previsão de séries temporais entre duas abordagens distintas:
1.  **Modelagem Estatística (Baseline):** utilizando o modelo **ARIMA**, já explorado no Projeto 1.
2.  **Machine Learning (Deep Learning):** implementando uma rede neural recorrente do tipo **LSTM (Long Short-Term Memory)**.

O foco é analisar se a capacidade da LSTM de capturar não-linearidades e dependências de longo prazo supera a abordagem linear clássica na previsão de emissões globais.

## 🗂️ Dataset
Foi utilizado o mesmo conjunto de dados base do projeto anterior, **CO₂ Emissions** ([Kaggle](https://www.kaggle.com/datasets/ulrikthygepedersen/co2-emissions-by-country)), contendo a série anual de emissões globais (`total_ghg`) de **1850 a 2023**.

## ⚙️ Pré-processamento e Engenharia de Features
Diferente da abordagem estatística, as redes neurais exigem uma preparação específica dos dados:

- **Normalização (StandardScaler):**
  a série foi padronizada (média 0, desvio padrão 1) para facilitar a convergência do gradiente durante o treinamento da rede neural.
- **Janela Deslizante (Sliding Window):**
  a série temporal foi transformada em um problema de aprendizado supervisionado.
  - **Window Size (Lag):** 3 anos (os 3 valores passados são usados para prever o próximo).
  - Isso gerou pares de entrada ($X$) e saída ($y$) para o modelo.
- **Divisão de Dados:**
  - **Treino:** ~80% (139 amostras).
  - **Teste:** ~20% (35 amostras), respeitando a ordem cronológica para evitar *data leakage*.
- **Tensores:** conversão dos dados para tensores do **PyTorch**.

## 🧠 Arquitetura da Rede (LSTM)
Optou-se pela **LSTM** em vez de uma RNN simples (Vanilla) para mitigar o problema do *Vanishing Gradient* e permitir que o modelo aprendesse tendências seculares (desde 1850) sem "esquecer" o passado distante.

| Camada | Configuração |
|---------|-------------|
| **Entrada** | Input Size = 1 (univariado) |
| **LSTM** | Hidden Size = 32, Layers = 1 |
| **Fully Connected** | Linear (32 $\to$ 1) para a previsão final |
| **Otimizador** | Adam (Learning Rate = 1e-3) |
| **Função de Perda** | MSE (Mean Squared Error) |
| **Épocas** | 1000 |

## 📊 Avaliação de Desempenho
Para validar a eficácia do Machine Learning, os resultados da LSTM foram comparados diretamente com o modelo **ARIMA** (baseline estatístico) no mesmo conjunto de teste (últimos 32 anos alinhados).

| Modelo | MSE | RMSE | MAE | R² |
|:--|--:|--:|--:|--:|
| **LSTM** (Deep Learning) | **2.48 × 10⁶** | **1576.1** | **1319.5** | **0.92** |
| **ARIMA** (Estatístico) | 3.02 × 10⁷ | 5497.8 | 4655.3 | 0.02 |

### 📈 Comparativo de Melhoria
A abordagem com LSTM apresentou um ganho de performance massivo sobre o modelo estatístico:
- **Redução do MSE:** 91.78% ⬇️
- **Redução do RMSE:** 71.33% ⬇️
- **Aumento do R²:** De 0.02 para **0.92** ⬆️

## 🧪 Análise dos Resultados

- **Captura de Tendência:** enquanto o ARIMA tendeu a projetar uma linha mais rígida (superestimando o crescimento linear e falhando em capturar a curvatura exata), a **LSTM** conseguiu aprender a dinâmica não linear e a inércia da série.
- **Aderência aos Dados:** o gráfico de teste revelou que a LSTM acompanhou muito bem as oscilações recentes, resultando em um coeficiente de determinação (**R²**) de **0.92**, o que indica que o modelo explica 92% da variabilidade dos dados de teste.
- **Suavização:** o modelo de ML suavizou alguns picos extremos, mas manteve a direção geral da série com muito mais precisão do que os métodos clássicos.

## 🧭 Conclusão

Este projeto demonstrou a superioridade das **Redes Neurais Recorrentes (LSTM)** para esta série temporal específica em comparação aos métodos estatísticos tradicionais.

> **Em síntese:**
> A capacidade da LSTM de reter memórias de sequências passadas e modelar não-linearidades permitiu reduzir o erro quadrático médio em mais de **90%** em relação ao ARIMA. Isso reforça a ideia de que, para séries temporais complexas e com padrões não lineares, as abordagens de Machine Learning podem oferecer vantagens sobre os métodos estatísticos convencionais.

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

### 4) ⚠️ Organização dos Arquivos (Importante)
Para que o notebook execute corretamente as comparações, certifique-se de que os seguintes arquivos estejam **no mesmo diretório (pasta)** do arquivo `lstm.ipynb`:

* `lstm.ipynb`: o notebook principal.
* `co2_emissions_processed.csv`: o dataset pré-processado.
* `fc_arima_log.npy`: o arquivo com as previsões salvas do modelo ARIMA (necessário para o gráfico comparativo).

> **Nota:** Não coloque estes arquivos em subpastas (como `data/`), pois o código espera encontrá-los na raiz de execução.

### 5) **Instalar as dependências a partir do `requirements.txt`:**
O comando a seguir instalará todas as bibliotecas necessárias de uma só vez.
```bash
pip install -r requirements.txt
```

### 6) Execução do Notebook
Com o ambiente preparado, você pode optar pelo **VSCode**, **Google Colab** ou **Jupyter Notebook/Lab** para a execução do projeto.