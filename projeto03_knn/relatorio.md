# Relatório – Problema 3 – Classificação de Qualidade de Vinhos Utilizando k-NN

**Disciplina:** PAM0466 – Sistemas Inteligentes

Código: `projeto03.ipynb` (todos os gráficos e saídas estão no notebook).

**Dados:** `winequality-red.csv`, 1.599 vinhos tintos, 11 atributos físico-químicos. Classes: ruim (nota ≤ 5), médio (6–7) e bom (≥ 8). Normalização Z-score (aprendida no treino), divisão estratificada 80/20 (1.279 treino, 320 teste), `random_state=42`.

## Tarefa 1

> Preparação dos dados: carregue o dataset e recodifique a variável quality em três classes: notas ≤ 5 como ruim, 6–7 como médio e ≥ 8 como bom. Em seguida, normalize todos os atributos utilizando Min-Max ou Z-score. Justifique por que a normalização é uma etapa obrigatória para o k-NN.

**Resposta.**

Recodificação da nota em três classes:

| Classe | Quantidade | Proporção |
|:---|---:|---:|
| ruim (≤ 5) | 744 | 46,5% |
| médio (6–7) | 837 | 52,4% |
| bom (≥ 8) | 18 | 1,1% |

Foi usado o **Z-score**, aprendido apenas no treino (depois da divisão), dentro de um `Pipeline`.

**Por que a normalização é obrigatória no k-NN?** O k-NN classifica pelos vizinhos mais próximos usando a distância euclidiana, que soma as diferenças ao quadrado dos atributos **nas unidades originais**. Sem normalizar, atributos de escala grande (como `total sulfur dioxide`, variância ≈ 1082) dominam a distância, enquanto atributos de escala pequena (como `density`, variância ≈ 0) quase não influenciam, mesmo que sejam informativos. Normalizar coloca todos na mesma escala. Evidência (k = 5, validação cruzada de 5 folds): acurácia de **0,630 sem normalizar** e **0,696 com Z-score**.

## Tarefa 2

> Divisão treino/teste: divida os dados em 80% para treino e 20% para teste, utilizando divisão estratificada para garantir que a proporção das três classes seja mantida em ambos os conjuntos.

**Resposta.**

Divisão 80/20 com `stratify=y`, mantendo as proporções das três classes:

| Classe | Geral | Treino | Teste |
|:---|---:|---:|---:|
| ruim | 46,5% | 46,5% (595) | 46,6% (149) |
| médio | 52,4% | 52,4% (670) | 52,2% (167) |
| bom | 1,1% | 1,1% (14) | 1,3% (4) |

Sem estratificação, a classe *bom* (18 amostras no total) poderia ficar quase ausente de um dos lados. Mesmo estratificado, o teste tem só **4 vinhos "bom"**, então qualquer métrica dessa classe é muito instável.

## Tarefa 3

> Treinamento e escolha do k: treine o modelo para diferentes valores de k (ex: 1, 3, 5, 7, 11) utilizando validação cruzada de 5 folds sobre o conjunto de treino. Plote a acurácia média em função de k e escolha o valor com melhor desempenho. O que acontece com o modelo para valores muito baixos e muito altos de k?

**Resposta.**

Acurácia média na validação cruzada de 5 folds (só no treino):

| k | 1 | 3 | 5 | 7 | 9 | 11 | 15 | 21 | 31 | 51 | 101 |
|:---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Acurácia | 0,725 | 0,689 | 0,701 | 0,706 | 0,717 | 0,712 | 0,719 | 0,714 | 0,715 | 0,716 | **0,726** |

O gráfico está no notebook. O maior valor ocorre em **k = 101 (0,7263)**, escolhido como melhor k. Contudo **a diferença entre os k é pequena**: o desvio entre os folds (0,02 a 0,04) é maior que a diferença entre quase todos os valores, então a partir de k = 7 o desempenho é praticamente o mesmo.

**Valores muito baixos e muito altos de k:**
- **k muito baixo (k = 1):** o modelo é sensível demais aos dados de treino e aprende o ruído (**overfitting**, alta variância). Ele acerta 100% do próprio treino, pois cada ponto é seu próprio vizinho, mas na validação cai para ≈ 0,72.
- **k muito alto:** o modelo considera vizinhos demais e perde os padrões locais (**underfitting**, alto viés). No limite, k = N prevê sempre a classe mais frequente ("médio", 52%). Com k = 101, a acurácia no próprio treino é 0,732 e a classe rara *bom* nunca é prevista.

## Tarefa 4

> Avaliação do modelo: utilizando o conjunto de teste, calcule a matriz de confusão e as métricas de acurácia, precisão, recall e F1-score para cada classe. Responda: Entre quais classes o modelo erra com mais frequência?

**Resposta.**

Modelo final: k-NN com k = 101, Z-score, treinado em todo o treino e avaliado nas 320 amostras de teste. **Acurácia = 0,7469** (linha de base, prever sempre "médio": 0,5219).

| | Previsto ruim | Previsto médio | Previsto bom |
|:---|---:|---:|---:|
| **Real ruim** | 107 | 42 | 0 |
| **Real médio** | 35 | 132 | 0 |
| **Real bom** | 0 | 4 | 0 |

| Classe | Precisão | Recall | F1 | Suporte |
|:---|---:|---:|---:|---:|
| ruim | 0,7535 | 0,7181 | 0,7354 | 149 |
| médio | 0,7416 | 0,7904 | 0,7652 | 167 |
| bom | 0,0000 | 0,0000 | 0,0000 | 4 |
| **Acurácia** | | | **0,7469** | 320 |
| Média macro | 0,4984 | 0,5028 | 0,5002 | |

**Entre quais classes o modelo erra mais?** Entre **ruim e médio**: 42 vinhos ruins foram classificados como médios e 35 médios como ruins, ou seja, 77 dos 81 erros (95,1%). Faz sentido, porque a fronteira fica entre as notas 5 e 6, avaliações humanas subjetivas de vinhos quimicamente muito parecidos.

**Observação:** a classe **bom** tem precisão, recall e F1 iguais a zero: o modelo nunca a prevê (só 18 vinhos na base e 4 no teste). A acurácia global esconde essa falha, e por isso o F1 macro (0,50) é muito menor que a acurácia. Para detectar vinhos "bom" seria preciso balancear as classes ou usar outra técnica.
