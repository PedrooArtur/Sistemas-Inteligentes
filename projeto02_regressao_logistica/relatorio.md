# Relatório – Problema 2 – Regressão Logística para Previsão de Doenças Cardíacas

**Disciplina:** PAM0466 – Sistemas Inteligentes

Código: `projeto02.ipynb` (todos os gráficos e saídas estão no notebook).

**Dados:** Heart Disease Cleveland, 303 pacientes, 13 variáveis clínicas e o `target` (0 a 4) binarizado. Divisão estratificada 80/20 (242 treino, 61 teste), `random_state=42`.

## Pré-processamento

> Realize o pré-processamento dos dados, incluindo a normalização ou padronização das variáveis independentes, tratamento de valores ausentes, divisão dos dados em conjuntos de treino e teste e binarização das variáveis alvo.

**Resposta.**

- **Valores ausentes:** `ca` (4) e `thal` (2), representados por `?` no arquivo. Foram preenchidos pela **mediana do treino** (`ca` = 0 e `thal` = 3), aplicada aos dois conjuntos.
- **Binarização do alvo:** `target` > 0 vira 1 (com doença). Resultado: 164 sem doença e 139 com doença.
- **Divisão:** 80% treino e 20% teste, com `stratify=y` (46% de doentes nos dois conjuntos).
- **Padronização:** Z-score (`StandardScaler`) nas variáveis contínuas `age`, `trestbps`, `chol`, `thalach` e `oldpeak`, com média e desvio calculados só no treino. As variáveis binárias e categóricas não foram alteradas.

## Tarefa 1

> Treinar um modelo de LogisticRegression do scikit-learn utilizando os dados de treino, com os seguintes parâmetros: solver='liblinear', random_state=42 e max_iter=1000. Indique quantas iterações foram necessárias para a convergência do modelo.

**Resposta.**

O modelo foi treinado com `LogisticRegression(solver='liblinear', random_state=42, max_iter=1000)`. O atributo `n_iter_` indicou **5 iterações** até a convergência, muito abaixo do limite de 1000, portanto o algoritmo convergiu sem problemas.

Coeficientes aprendidos (variáveis contínuas padronizadas), com a razão de chances $e^{\theta}$:

| Variável | θ | Razão de chances |
|:---|---:|---:|
| ca | +0,9957 | 2,71 |
| sex | +0,8015 | 2,23 |
| exang | +0,7985 | 2,22 |
| thalach | −0,5284 | 0,59 |
| oldpeak | +0,4028 | 1,50 |
| fbs | −0,3968 | 0,67 |
| thal | +0,2879 | 1,33 |
| trestbps | +0,2226 | 1,25 |
| cp | +0,2024 | 1,22 |
| restecg | +0,1712 | 1,19 |
| chol | +0,1459 | 1,16 |
| slope | −0,1400 | 0,87 |
| age | −0,1093 | 0,90 |

Intercepto: θ₀ = −3,2881. Razão de chances maior que 1 aumenta a chance de doença; menor que 1 diminui. As variáveis de maior efeito são `ca` (mais artérias obstruídas, mais risco), `sex` (masculino) e `exang` (angina no exercício); `thalach` (frequência cardíaca máxima alta) é protetora.

## Tarefa 2

> Avaliar o modelo utilizando os dados de teste e calcular as seguintes métricas: Acurácia, Precisão, Recall, F1-score e Curva ROC com valor de AUC.

**Resposta.**

Matriz de confusão no teste (61 pacientes; positivo = com doença):

| | Previsto sem doença | Previsto com doença |
|:---|---:|---:|
| **Real sem doença** | 27 (VN) | 6 (FP) |
| **Real com doença** | 2 (FN) | 26 (VP) |

| Métrica | Valor |
|:---|---:|
| Acurácia | **0,8689** |
| Precisão | **0,8125** |
| Recall | **0,9286** |
| F1-score | **0,8667** |
| AUC (curva ROC) | **0,9545** |

O modelo acerta ≈ 87% dos pacientes. O **recall de 0,93** significa que, de 28 pacientes doentes, só 2 foram classificados como saudáveis, o que é desejável em diagnóstico, onde deixar passar um doente é o erro mais grave. A **precisão de 0,81** indica que, dos 32 pacientes apontados como doentes, 26 realmente eram (6 alarmes falsos). O **F1 de 0,87** equilibra precisão e recall. O **AUC de 0,955** está na faixa excelente (0,9 a 1,0): o modelo separa bem as duas classes em praticamente qualquer limiar. A curva ROC está no notebook.

Ressalva: o teste tem só 61 pacientes, então cada paciente muda a acurácia em ≈ 1,6 ponto percentual; o resultado deve ser lido com essa margem.
