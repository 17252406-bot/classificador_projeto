# Projeto SETA - Classificador Supervisionado (Previsão de Cancelamento Hoteleiro)

Este repositório contém a **2ª Etapa do Projeto SETA** (Aprendizado de Máquina Supervisionado), desenvolvido para a disciplina de *Eletiva*. O objetivo central é implementar, otimizar e comparar modelos preditivos para classificar a probabilidade de cancelamento de reservas hoteleiras a partir da base de dados *Hotel Booking Demand*.

---

## 📌 1. Visão Geral do Problema

O cancelamento imprevisível de reservas gera ociosidade nos leitos, reduz a receita operacional e prejudica o planejamento financeiro da rede hoteleira. 

A partir da base tratada na 1ª Etapa (Análise Exploratória de Dados - EDA), este projeto formula a solução como um **problema de classificação binária supervisionada**:
* **Variável Alvo (`y`):** `is_canceled`
  * `0`: Reserva mantida (Não Cancelada - 63% do histórico).
  * `1`: Reserva cancelada pelo cliente (Cancelada - 37% do histórico).

---

## 🛠️ 2. Pipeline de Pré-processamento e Tratamento dos Dados

A base de dados original conta com **119.390 registros**. O pipeline de preparação automatizado seguiu as seguintes etapas:

1. **Limpeza e Eliminação de Ruídos:**
   * Remoção de colunas com alta vacuidade ou que causariam vazamento de dados (*data leakage*): `company`, `agent`, `reservation_status_date` e `reservation_status`.
2. **Imputação de Valores Ausentes:**
   * Variáveis numéricas: Imputação automatizada pela mediana.
   * Variáveis categóricas: Imputação automatizada pela moda (`mode()[0]`).
3. **Codificação Categórica (Feature Encoding):**
   * Aplicação de *One-Hot Encoding* (`pd.get_dummies(drop_first=True)`), expandindo as *features* para **245 colunas preditoras**.
4. **Divisão Estratificada (Treino e Teste):**
   * **Treino (70%):** 83.573 amostras.
   * **Teste (30%):** 35.817 amostras.
   * Manutenção estrita da proporção das classes original via `stratify=y` (`random_state=42`).

---

## 🤖 3. Modelos Implementados e Metodologia

Foram implementados e avaliados dois classificadores com abordagens matemáticas distintas em três fases de maturidade (**Baseline**, **Versão 1** e **Tuning**):

### **Método 1: K-Nearest Neighbors (KNN)**
* **Baseline:** Classificador KNN padrão com K=5 sem normalização de escala.
* **Versão 1:** Aplicação de padronização z-score (`StandardScaler`), essencial para métricas de distância euclidiana.
* **Tuning:** Otimização do hiperparâmetro K avaliando o intervalo K em {3, 7, 11, 15}.

### **Método 2: Árvore de Decisão (Decision Tree Classifier)**
* **Baseline:** Árvore de decisão sem restrição de profundidade (`max_depth=None`).
* **Versão 1:** Introdução do critério de entropia (`criterion='entropy'`) e limitação inicial de profundidade (`max_depth=10`).
* **Tuning:** Grade de otimização para profundidade máxima `max_depth` em {8, 12, 16, 20} com suporte a divisão mínima de folhas (`min_samples_split=10`).

---

## 📊 4. Matriz Comparativa de Resultados

Os experimentos foram avaliados sobre o conjunto de teste não visto (**35.817 amostras**), com foco no **F1-Score** da classe de cancelamento (`1`):

| Modelo | Configuração / Hiperparâmetros | Acurácia Geral | Precisão (Classe 1) | Recall (Classe 1) | F1-Score (Classe 1) | Tempo de Execução |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **KNN (Baseline)** | K=5, Sem escala | 77% | 0.71 | 0.63 | 0.67 | ~1 min |
| **KNN (Versão 1)** | K=5, Com `StandardScaler` | 82% | 0.77 | 0.74 | 0.76 | ~1 min |
| **KNN (Tunado)** | **K=3**, Com `StandardScaler` | **82%** | **0.78** | **0.74** | **0.7613** | **~5 min** |
| **Árvore (Baseline)** | Profundidade livre | 85% | 0.79 | 0.80 | 0.80 | ~2 s |
| **Árvore (Versão 1)** | `entropy`, `max_depth=10` | 84% | 0.81 | 0.73 | 0.77 | ~1 s |
| **Árvore (Tunada)** | **`max_depth=20`**, `min_samples_split=10` | **85%** | **0.80** | **0.79** | **0.7963** | **~9 s** |

---

## 🏆 5. Escolha do Modelo para Produção e Justificativas

O modelo selecionado para implantação em produção foi a **Árvore de Decisão Tunada (`max_depth=20`)**.

### **Justificativas Técnicas e de Negócio:**
1. **Desempenho Balanceado:** Alcançou a maior acurácia global (**85%**) e um **F1-Score de 79,63%** na classe crítica de cancelamento.
2. **Eficiência Computacional Extrema:** A Árvore de Decisão realizou a busca de hiperparâmetros e inferência em apenas **9 segundos**, enquanto o KNN consumiu cerca de **5 minutos** devido ao custo de cálculo de distâncias contra 83.573 pontos de treino.
3. **Interpretabilidade Operacional:** A Árvore permite extrair a importância relativa das *features* (como `deposit_type`, `lead_time` e `adr`), fornecendo justificativas transparentes para que a equipe hoteleira adote estratégias de *overbooking* preventivo.

### **Riscos e Mitigações:**
* **Data Drift (Mudança de Comportamento):** Sazonalidade hoteleira pode alterar o perfil das reservas. Mitigação via retreino periódico.
* **Overfitting Residual:** Controlado pelo ajuste de `max_depth=20` e `min_samples_split=10`.

---

## 📂 6. Estrutura do Repositório

```text
classificador_projeto/
├── README.md                            # Documentação executiva do projeto
├── classificador_hotel_bookings.ipynb   # Notebook Jupyter/Colab com o código e análises
└── hotel_bookings.csv                   # Base de dados original (Kaggle)
```

---

## 🚀 7. Como Executar o Projeto

1. Crie o repositório `classificador_projeto` no GitHub.
2. Abra o arquivo `classificador_hotel_bookings` no **Google Colab**.
3. Faça o upload do arquivo `hotel_bookings.csv` na pasta de arquivos do Colab.
4. Execute todas as células em sequência (`Ambiente de execução -> Executar tudo`).

---

## 🎓 Autor e Créditos

Desenvolvido como projeto prático da disciplina **Eletiva**.
* **Dataset:** Hotel Booking Demand (Kaggle)
* **Orientação:** Prof. Kelvin Frade Ribeiro
* **Alunos:** Luiz Claudio Martins / 
              Gabriel Normandio / 
              Pethrus Souza
