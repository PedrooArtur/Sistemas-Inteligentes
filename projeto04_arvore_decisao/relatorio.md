# Relatório – Problema 4 – Classificação de Risco de Crédito Utilizando Árvores de Decisão

**Disciplina:** PAM0466 – Sistemas Inteligentes

Código: `projeto04.ipynb` (todos os gráficos e saídas estão no notebook).

**Dados:** `german.data` (Statlog German Credit), 1.000 solicitações, 20 atributos (7 numéricos e 13 categóricos); classe positiva = mau pagador. Divisão estratificada 80/20 (800 treino, 200 teste), `random_state=42`.

## Tarefa 1

> Preparação dos dados: carregue o dataset, identifique quais atributos são numéricos e quais são categóricos. Verifique se existe necessidade de codificação adequada às variáveis categóricas (one-hot encoding ou ordinal encoding). Avalie o balanceamento entre as classes e comente o impacto de um conjunto desbalanceado (70%/30%) sobre a interpretação da acurácia.

**Resposta.**

- **Numéricos (7):** `duration`, `amount`, `installment_rate`, `residence_since`, `age`, `num_credits`, `people_liable`.
- **Categóricos (13):** `status`, `credit_history`, `purpose`, `savings`, `employment`, `personal_status`, `other_debtors`, `property`, `other_installment_plans`, `housing`, `job`, `telephone`, `foreign_worker`. Não há valores ausentes.
- **Codificação:** é necessária, porque o scikit-learn não aceita texto. Foi usado **one-hot** (61 colunas no final): a maioria dos atributos é nominal e mesmo os quase ordinais (conta corrente, poupança, emprego) têm categoria fora da escala ("sem conta corrente", "desconhecida"); além disso, o one-hot gera regras fáceis de ler na árvore.
- **Balanceamento:** 700 bons (70%) e 300 maus (30%). Um modelo que **aprova todo mundo** acerta 70% sem aprender nada, então a acurácia sozinha engana: 70% é o piso, não um bom desempenho. Além disso, a acurácia trata igual dois erros de custos muito diferentes (aprovar um mau pagador custa 5 vezes mais que recusar um bom, segundo o `german.doc`), por isso é preciso olhar também precisão e recall da classe "mau".

## Tarefa 2

> Justifique por que a normalização dos atributos não é necessária para uma árvore de decisão.

**Resposta.**

A árvore compara **um atributo por vez** com um limiar (por exemplo `duration ≤ 15,5`) e só depende da **ordem** dos valores, não da escala. Padronizar (Z-score), Min-Max ou aplicar log são transformações monótonas: mudam o valor numérico do limiar, mas não a ordem dos exemplos, então a árvore encontra as mesmas divisões. Diferentemente do k-NN ou do k-Means, ela não calcula distâncias entre atributos, então uma variável de escala grande (`amount`) não domina as demais. Verificação: a árvore (profundidade 5) com e sem Z-score gerou **previsões idênticas** (acurácia 0,645 nos dois casos).

## Tarefa 3

> Divisão treino/teste: divida os dados em 80% para treino e 20% para teste, utilizando divisão estratificada para garantir que a proporção das duas classes seja mantida em ambos os conjuntos.

**Resposta.**

Divisão 80/20 com `stratify=y`:

| Conjunto | Amostras | Maus pagadores |
|:---|---:|---:|
| Treino | 800 | 240 (30%) |
| Teste | 200 | 60 (30%) |

A estratificação mantém a proporção 70/30 nos dois conjuntos, importante porque "mau" é a classe minoritária.

## Tarefa 4

> Treinamento e controle de complexidade: treine árvores com diferentes valores de profundidade máxima (ex: 2, 3, 5, 7, 10 e sem limite), utilizando validação cruzada de 5 folds sobre o conjunto de treino. Plote, em um mesmo gráfico, a acurácia média de treino e a de validação em função da profundidade, e escolha o valor com melhor desempenho. O que acontece com o modelo para profundidades muito baixas e muito altas? Relacione sua resposta aos conceitos de underfitting e overfitting. Repita o experimento trocando o critério de divisão dos nós de Gini para entropia e comente se houve diferença relevante.

**Resposta.**

Validação cruzada de 5 folds no treino (o gráfico treino × validação está no notebook):

| max_depth | Gini treino | Gini validação | Entropia treino | Entropia validação |
|:---|---:|---:|---:|---:|
| 2 | 0,708 | 0,689 | 0,707 | 0,694 |
| 3 | 0,745 | 0,699 | 0,736 | 0,699 |
| 5 | 0,804 | 0,693 | 0,794 | **0,715** |
| 7 | 0,879 | **0,713** | 0,859 | 0,714 |
| 10 | 0,958 | 0,685 | 0,934 | 0,689 |
| sem limite | 1,000 | 0,675 | 1,000 | 0,686 |

**Profundidade escolhida (Gini): 7**, com a melhor acurácia de validação (71,25%).

- **Profundidade muito baixa (2):** a árvore é simples demais e erra tanto no treino quanto na validação (≈ 69–71%). É **underfitting** (viés alto).
- **Profundidade muito alta / sem limite:** a acurácia de treino chega a 100% (a árvore memoriza os dados, inclusive o ruído), mas a de validação cai para ≈ 67,5%. É **overfitting** (variância alta). A distância entre treino e validação cresce com a profundidade.

**Gini × entropia.** As melhores validações foram 71,25% (Gini, profundidade 7) e 71,50% (entropia, profundidade 5). A diferença de 0,25 ponto percentual é menor que o desvio entre os folds, então **não há diferença relevante**. Observação: todas as acurácias de validação ficam entre 67% e 72%, perto da linha de base de 70%, o que mostra que o problema é difícil para uma árvore única.

## Tarefa 5

> Interpretação do modelo: usando a árvore com a profundidade escolhida, gere a visualização gráfica da árvore e a lista de importância dos atributos. Responda: quais são os três atributos mais determinantes para a classificação? Escreva, em linguagem natural, o conjunto de regras correspondente ao caminho que leva da raiz até a folha com maior número de amostras de mau risco.

**Resposta.**

Árvore final: critério Gini, profundidade 7, **59 folhas** (a árvore completa e a versão com os 3 primeiros níveis estão no notebook).

**Três atributos mais determinantes** (importância somada das colunas one-hot de cada atributo):

| # | Atributo | Importância |
|:---|:---|---:|
| 1 | `status` (saldo da conta corrente) | 0,188 |
| 2 | `duration` (duração em meses) | 0,125 |
| 3 | `amount` (valor solicitado) | 0,114 |

`status` é a raiz da árvore: a pergunta sobre a conta corrente é a que mais separa bons e maus pagadores.

**Folha com mais maus pagadores:** 44 maus e 53 bons no treino (97 amostras, 45% maus). Como os bons são maioria nela, a folha prevê *bom*, mas é a que concentra mais maus em números absolutos. As regras do caminho da raiz até ela, em linguagem natural:

> SE o cliente **tem conta corrente** (status diferente de "sem conta corrente") **E** poupança **< 100 DM** **E** duração do empréstimo **entre 15,5 e 47,5 meses** **E** valor solicitado **entre 2.321 e 8.015,5 DM** → 97 amostras, 44 maus pagadores (45%).

Entre as folhas que **preveem "mau"**, a de maior número de maus (24 maus e 14 bons, 63%) tem as regras:

> SE tem conta corrente **E** poupança < 100 DM **E** outros planos de parcelamento = "nenhum" **E** duração entre 15,5 e 47,5 meses **E** valor ≤ 2.321 DM → **mau pagador**.

## Tarefa 6

> Avaliação do modelo: utilizando o conjunto de teste, calcule a matriz de confusão e as métricas de acurácia, precisão, recall e F1-score para cada classe. Responda: qual tipo de erro o modelo comete com mais frequência e por que ele é o mais grave nesta aplicação? Que ajuste você faria no treinamento para reduzi-lo?

**Resposta.**

Matriz de confusão no teste (classe positiva = mau pagador):

| | Previsto bom | Previsto mau |
|:---|---:|---:|
| **Real bom** | VN = 112 | FP = 28 |
| **Real mau** | FN = 36 | VP = 24 |

| Classe | Precisão | Recall | F1 | Suporte |
|:---|---:|---:|---:|---:|
| bom pagador | 0,7568 | 0,8000 | 0,7778 | 140 |
| mau pagador | 0,4615 | 0,4000 | 0,4286 | 60 |
| **Acurácia** | | | **0,6800** | 200 |

A acurácia (68%) fica **abaixo da linha de base** de 70% (aprovar todo mundo). O valor do modelo só aparece no recall da classe "mau", que é baixo.

**Erro mais frequente e mais grave:** o **falso negativo** (FN = 36, contra 28 FP): um mau pagador classificado como bom, ou seja, **crédito aprovado para quem não vai pagar**. O modelo deixa passar 60% dos maus (recall de 40%). O FN é o mais grave porque gera prejuízo direto, enquanto o FP gera só perda de receita (o `german.doc` define que errar um mau pagador custa 5 vezes mais que recusar um bom).

**Ajuste proposto:** ponderar a classe "mau" no treino (`class_weight`). O peso foi escolhido por validação cruzada **no treino**, pelo custo 5·FN + FP (sem peso: 686; peso 2: 654; peso 3: 573; **peso 5: 514**; balanced: 611). Resultado no teste:

| Configuração | Acurácia | Precisão (mau) | Recall (mau) | F1 (mau) | FN | FP | Custo 5FN+FP |
|:---|---:|---:|---:|---:|---:|---:|---:|
| sem peso | 0,680 | 0,462 | 0,400 | 0,429 | 36 | 28 | 208 |
| peso 5 | 0,600 | 0,412 | 0,783 | 0,540 | 13 | 67 | **132** |

O peso 5 reduz os falsos negativos de 36 para 13 (custo 37% menor), ao preço de mais falsos positivos e menor acurácia: é o *trade-off* entre precisão e recall, que compensa em crédito porque um FN custa 5 vezes mais. Outras opções: reduzir o limiar de decisão em `predict_proba` ou controlar a complexidade com `min_samples_leaf` / poda (`ccp_alpha`).
