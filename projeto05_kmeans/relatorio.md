# Relatório – Problema 5 – Segmentação de Clientes Utilizando k-Means

**Disciplina:** PAM0466 – Sistemas Inteligentes

Código: `projeto05.ipynb` (todos os gráficos e saídas estão no notebook).

**Dados:** `Wholesale customers data.csv`, 440 clientes, seis categorias de gasto. `Channel` e `Region` ficam fora do agrupamento e são usados só na validação externa. Semente 42.

## Tarefa 1

> Preparação dos dados: carregue o dataset e separe Channel e Region do restante. Para cada uma das seis categorias de gasto, calcule média, mediana, desvio padrão e valores extremos, e construa um boxplot ou histograma. Você observará distribuições fortemente assimétricas à direita, com poucos clientes de gasto muito elevado — em Fresh, por exemplo, os valores vão de dezenas a mais de cem mil unidades monetárias. Aplique então a transformação logarítmica x' = log(1 + x) e, em seguida, padronize os atributos por Z-score. Justifique as duas etapas separadamente: (a) por que a transformação logarítmica é recomendável diante da assimetria e da presença de outliers, e (b) por que a padronização é determinante para o k-Means, relacionando sua resposta ao uso da distância euclidiana na atribuição dos pontos aos centróides. Calcule também a matriz de correlação entre as seis categorias e comente o par Grocery–Detergents Paper.

**Resposta.**

Estatísticas descritivas (escala original):

| Categoria | Média | Mediana | Desvio | Mín | Máx | Assimetria |
|:---|---:|---:|---:|---:|---:|---:|
| Fresh | 12.000 | 8.504 | 12.647 | 3 | 112.151 | 2,56 |
| Milk | 5.796 | 3.627 | 7.380 | 55 | 73.498 | 4,05 |
| Grocery | 7.951 | 4.756 | 9.503 | 3 | 92.780 | 3,59 |
| Frozen | 3.072 | 1.526 | 4.855 | 25 | 60.869 | 5,91 |
| Detergents_Paper | 2.881 | 817 | 4.768 | 3 | 40.827 | 3,63 |
| Delicassen | 1.525 | 966 | 2.820 | 3 | 47.943 | 11,15 |

Em todas as categorias a média é maior que a mediana e a assimetria é positiva e alta: poucos clientes gastam muito mais que o típico (histogramas e boxplots no notebook). Depois do `log(1+x)` as assimetrias ficam entre −1,58 e −0,22.

**(a) Por que o log.** O k-Means minimiza uma soma de **quadrados** e o centróide é uma **média**. Um cliente 10 vezes maior pesa 100 vezes mais no erro quadrático e puxa o centróide, e os grupos acabam definidos pelos outliers. O log comprime a cauda (passa a contar a ordem de grandeza) e deixa a distribuição mais simétrica. Na prática, sem o log (só Z-score) o k-Means com k = 3 gera grupos de tamanho **2, 45 e 393**; com o log, **80, 147 e 213**.

**(b) Por que o Z-score.** Cada ponto é atribuído ao centróide de menor distância euclidiana, $\lVert x-\mu\rVert^2 = \sum_j (x_j-\mu_j)^2$. Uma variável de escala grande domina essa soma e as demais quase não influenciam (a variância de Fresh é ≈ 160 milhões, a de Delicassen ≈ 8 milhões). O Z-score deixa todas com média 0 e desvio 1, para que contem igual. O log corrige a **forma** (assimetria) e o Z-score corrige a **escala**, por isso as duas etapas são justificadas separadamente.

**Correlações.** O par **Grocery–Detergents_Paper** tem r = **0,92** (0,80 em log), o mais forte de todos. Milk também se correlaciona com os dois (0,73 e 0,66). Isso reflete o perfil de varejo, que compra esses itens juntos. Para o k-Means é informação **redundante**: o eixo "mercearia/limpeza" acaba com peso dobrado no cálculo da distância.

## Tarefa 2

> Ausência de divisão treino/teste: diferentemente dos projetos anteriores, aqui não se separa um conjunto de teste. Explique por quê, contrastando o objetivo do aprendizado supervisionado (estimar o erro de generalização sobre rótulos conhecidos) com o do não supervisionado. Discuta, então, qual é a dificuldade central da validação em agrupamento: como afirmar que uma partição é “boa” sem dispor de um gabarito?

**Resposta.**

No aprendizado **supervisionado** existe um rótulo e o objetivo é **generalizar**: estimar o erro em dados novos comparando a previsão com o rótulo conhecido. Por isso se reserva um conjunto de teste. No **não supervisionado** não há rótulo, logo não existe "erro de generalização" para estimar; o objetivo é **descobrir estrutura** nos dados disponíveis. Aqui, os 440 clientes são toda a população do distribuidor que se quer segmentar, então não faz sentido separar uma parte deles.

**Dificuldade central da validação:** sem gabarito, "partição boa" não tem uma definição única.
- Os **índices internos** (inércia, silhueta) medem coesão e separação, mas favorecem grupos esféricos e podem discordar entre si.
- Os **índices externos** (como o ARI) precisam de rótulos, que normalmente não existem.
- Há ainda o critério prático: a área comercial precisa saber o que fazer com cada segmento.

Por isso a validação combina várias evidências.

## Tarefa 3

> Escolha do número de grupos: execute o k-Means para k variando de 2 a 10. Plote a inércia (soma dos quadrados intra-cluster) em função de k e aplique o método do cotovelo; em seguida, plote o coeficiente de silhueta médio em função de k. Escolha o valor de k justificando sua decisão à luz dos dois critérios e da interpretabilidade comercial dos grupos resultantes — um segmento só é útil se a área comercial souber o que fazer com ele. Responda: por que a inércia decresce monotonicamente com k e por que, portanto, ela não pode ser usada isoladamente como critério de escolha? O que ocorre no caso limite em que k é igual ao número de amostras?

**Resposta.**

| k | Inércia | Queda % | Silhueta |
|:---|---:|---:|---:|
| 2 | 1844,1 | – | **0,290** |
| 3 | 1553,4 | 15,8 | 0,259 |
| 4 | 1386,9 | 10,7 | 0,188 |
| 5 | 1266,1 | 8,7 | 0,192 |
| 6 | 1174,8 | 7,2 | 0,201 |
| 7 | 1086,4 | 7,5 | 0,196 |
| 8 | 1028,5 | 5,3 | 0,185 |
| 9 | 973,1 | 5,4 | 0,197 |
| 10 | 932,0 | 4,2 | 0,191 |

**Escolha: k = 3.**
- **Cotovelo:** a curva é suave, sem joelho nítido; a maior queda marginal é de k = 2 para k = 3 (15,8%).
- **Silhueta:** é máxima em k = 2 (0,290), mas k = 3 mantém 0,259 e **cai para ≈ 0,19 a partir de k = 4**.
- **Interpretação comercial:** k = 2 só reproduz Horeca × Retail, que já conhecemos; k = 3 entrega um terceiro segmento acionável; k ≥ 4 cria grupos pequenos e difíceis de distinguir.
- **Ressalva:** a silhueta de 0,26 indica estrutura **fraca**; os grupos não são bem separados, os dados formam uma nuvem contínua.

**Por que a inércia decresce monotonicamente?** Dividir um grupo em dois nunca aumenta a soma dos quadrados (dois centróides representam um conjunto ao menos tão bem quanto um), então a inércia ótima com k + 1 grupos é menor ou igual à de k. Se a minimizássemos, escolheríamos sempre o maior k possível, por isso ela não serve sozinha como critério.

**Caso k = n:** cada cliente vira o seu próprio centróide e a inércia vale **0** (verificado: 0,000000). É uma partição perfeita e inútil, e a silhueta nem é definida. Para k = 1 a inércia é a total, 2640.

## Tarefa 4

> Inicialização e convergência: o k-Means converge para um mínimo local, e não necessariamente global. Execute o algoritmo dez vezes com init='random' e n_init=1, usando sementes distintas, e registre a inércia final de cada execução. Repita o procedimento com init='k-means++'. Compare a dispersão dos resultados nos dois casos e explique qual problema a inicialização k-means++ busca mitigar e de que forma.

**Resposta.**

Inércia final em 10 execuções com `n_init=1` (k = 3, sementes 0 a 9):

| Método | Mínimo | Máximo | Média | Desvio padrão | Amplitude |
|:---|---:|---:|---:|---:|---:|
| random | 1553,39 | 1626,55 | 1569,21 | 24,11 | 73,16 |
| k-means++ | 1553,34 | 1582,23 | 1564,79 | 14,55 | 28,89 |

Aparecem mínimos locais diferentes (≈ 1553, ≈ 1580–1582 e ≈ 1627). O k-means++ teve menor dispersão (desvio 14,55 contra 24,11).

**Que problema o k-means++ mitiga?** O k-Means converge para um mínimo **local**, que depende dos centróides iniciais; a inicialização aleatória pode colocar vários centróides na mesma região densa e deixar outras sem centróide. O **k-means++** sorteia o primeiro centróide ao acaso e cada novo com probabilidade proporcional a $D(x)^2$ (o quadrado da distância ao centróide mais próximo já escolhido), o que **espalha** os centróides iniciais e reduz a chance de partir de uma configuração ruim.

**Cuidado com a conclusão.** Com 100 execuções, a diferença some para k = 3 (desvio ≈ 24 nos dois; 54% das execuções aleatórias e 48% das k-means++ chegam ao melhor ótimo). Para k = 8, o k-means++ reduz só um pouco o pior caso (1093,6 contra 1122,5). Ou seja, ajuda, mas não elimina o problema: convém usar também `n_init` maior (várias inicializações, guardando a melhor).

## Tarefa 5

> Interpretação e validação dos grupos: com o k escolhido, recupere os centróides, reverta-os para a escala original de gasto (desfazendo a padronização e a transformação logarítmica) e apresente-os em uma tabela e em um gráfico de barras agrupadas por categoria. Atribua a cada agrupamento um nome descritivo de natureza comercial, baseado nas categorias em que ele se destaca. Em seguida, realize uma validação externa: construa tabelas de contingência cruzando o agrupamento de cada cliente com Channel e com Region, e calcule o índice Rand ajustado (Adjusted Rand Index) entre a partição obtida e a variável Channel. Responda: o algoritmo, sem jamais ter visto o canal de venda, foi capaz de reconstruí-lo? Qual conjunto de categorias de produto mais separa Horeca de Retail? E a região geográfica — ela se reflete nos agrupamentos? Por fim, projete os dados em duas dimensões por PCA, colorindo os pontos por agrupamento, e discuta duas limitações do k-Means evidenciadas nesse gráfico: a tendência a formar grupos de geometria esférica e de tamanhos semelhantes e a sensibilidade a outliers.

**Resposta.**

**Centróides na escala original** (desfazendo o Z-score e depois o log com `expm1`; o gráfico de barras agrupadas está no notebook):

| Grupo | n | Fresh | Milk | Grocery | Frozen | Det. Paper | Delicassen |
|:---|---:|---:|---:|---:|---:|---:|---:|
| Varejo de mercearia e limpeza | 80 | 1.095 | 5.635 | 9.851 | 344 | 3.392 | 318 |
| Food service (frescos e congelados) | 213 | 7.869 | 1.529 | 1.985 | 1.976 | 246 | 602 |
| Grandes compradores diversificados | 147 | 11.287 | 7.972 | 10.520 | 2.168 | 2.762 | 1.906 |

O centróide revertido do log é uma média geométrica, por isso é menor que a média aritmética real (que também é calculada no notebook).

**Nomes dos grupos:**
- **Varejo de mercearia e limpeza:** gasto muito baixo em Fresh e Frozen e alto em Milk, Grocery e Detergents_Paper.
- **Food service (frescos e congelados):** é o maior grupo; gasto baixo em Milk, Grocery e Detergents_Paper e gasto em Fresh e Frozen próximo ou acima do típico (perfil de restaurantes, hotéis e cafés).
- **Grandes compradores diversificados:** acima da mediana em todas as categorias, inclusive Delicassen (≈ 2 vezes): contas grandes, que merecem atendimento e limite de crédito diferenciados.

**Validação externa.** Tabelas de contingência:

| Grupo | Horeca | Retail |
|:---|---:|---:|
| Varejo de mercearia e limpeza | 32 | 48 |
| Food service | 210 | 3 |
| Grandes compradores | 56 | 91 |

| Grupo | Lisboa | Porto | Outras |
|:---|---:|---:|---:|
| Varejo de mercearia e limpeza | 14 | 10 | 56 |
| Food service | 40 | 26 | 147 |
| Grandes compradores | 23 | 11 | 113 |

**ARI (grupos, Channel) = 0,3504** (qui-quadrado p ≈ 8·10⁻⁴⁰); com k = 2, o ARI sobe para 0,5135 (≈ 86% de concordância). ARI (grupos, Region) = −0,0064 (qui-quadrado p = 0,49).

- **O algoritmo reconstruiu o canal sem vê-lo? Em parte.** O grupo *Food service* é 98,6% Horeca e os outros dois grupos reúnem 139 dos 142 clientes Retail (misturados com 88 Horeca de alto gasto). O ARI está bem acima de 0, mas com k = 3 ele cai mecanicamente em relação a uma variável binária.
- **Categorias que mais separam Horeca de Retail** (d de Cohen, em log): Detergents_Paper (2,43), Grocery (2,02) e Milk (1,59) são maiores no Retail; Frozen (−0,54) e Fresh (−0,36) são maiores no Horeca. Ou seja, **detergentes/papel, mercearia e laticínios**.
- **Região:** **não se reflete** nos grupos: ARI ≈ 0 e cada grupo tem aproximadamente a proporção geral (Lisboa 18%, Porto 11%, Outras 72%).

**PCA** (gráfico no notebook): PC1 explica 44,1% e PC2 27,2% da variância (71,3% no total). PC1 ≈ Milk/Grocery/Detergents_Paper e PC2 ≈ Fresh/Frozen/Delicassen.

**Duas limitações do k-Means evidenciadas no gráfico:**
1. **Geometria esférica e tamanhos semelhantes.** O k-Means cria regiões separadas por fronteiras retas e favorece grupos compactos de variância parecida. Os pontos formam uma **nuvem contínua**, sem lacunas, e as fronteiras cortam essa nuvem (silhueta de 0,26; no grupo de varejo, 25% dos pontos têm silhueta negativa). Estruturas alongadas ou curvas seriam cortadas ao meio.
2. **Sensibilidade a outliers.** Mesmo depois do log restam clientes isolados (por exemplo Fresh = 3, ou Frozen = 15.601 com Fresh = 3). Como o centróide é uma média e o erro é quadrático, eles pesam demais: retirando só 10 clientes (2,3%), o centróide do varejo se desloca ≈ 0,48 desvio padrão e a concordância com a partição original cai para ARI ≈ 0,89. Alternativas: k-medoids, tratar os outliers antes, ou DBSCAN.
