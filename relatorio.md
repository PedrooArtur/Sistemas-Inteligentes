# Relatório – Problema 1 – Regressão Linear para Previsão de Produção de Energia em uma Usina de Ciclo Combinado

**Disciplina:** PAM0466 – Sistemas Inteligentes

Código: `projeto01.ipynb` (todos os gráficos e saídas estão no notebook).

**Dados:** UCI CCPP, 9.568 amostras, 4 variáveis independentes (AT, V, AP, RH) e a variável alvo PE. Não há valores ausentes. Divisão 80% treino (7.654 amostras) e 20% teste (1.914), `random_state=42`. Padronização Z-score com média e desvio calculados só no treino.

## Pré-processamento

> Realize o pré-processamento dos dados, incluindo a normalização ou padronização das variáveis independentes, tratamento de valores ausentes e divisão dos dados em conjuntos de treino e teste.

**Resposta.** Valores ausentes: nenhum (0 nulos em todas as colunas). Divisão treino/teste feita **antes** da padronização; o `StandardScaler` aprendeu média e desvio padrão apenas no treino e foi aplicado ao teste, evitando vazamento de informação.

## Tarefa 1

> Calcule a correlação entre as variáveis independentes (AT, V, AP, RH) e a variável dependente (PE). Quais variáveis parecem ter maior influência na produção de energia?

**Resposta.**

| Variável | Pearson com PE | Spearman com PE |
|:---|---:|---:|
| AT | −0,948 | −0,944 |
| V | −0,870 | −0,884 |
| AP | +0,518 | +0,543 |
| RH | +0,390 | +0,390 |

As variáveis de maior influência são **AT** (temperatura) e **V** (vácuo de escape), com correlação negativa e forte: quanto maiores, menor a produção de energia. Fisicamente, ar mais quente é menos denso e reduz a eficiência da combustão. **AP** e **RH** têm correlação positiva e moderada. Pearson e Spearman são quase iguais, o que indica relação aproximadamente linear.

## Tarefa 2

> Calcule a correlação entre as variáveis independentes (AT, V, AP, RH) entre si. Existe alguma relação forte entre elas? Se sim, isso pode indicar multicolinearidade.

**Resposta.**

| | AT | V | AP | RH |
|:---|---:|---:|---:|---:|
| AT | 1,000 | **0,844** | −0,508 | −0,543 |
| V | 0,844 | 1,000 | −0,414 | −0,312 |
| AP | −0,508 | −0,414 | 1,000 | 0,100 |
| RH | −0,543 | −0,312 | 0,100 | 1,000 |

VIF: AT = 5,98; V = 3,94; AP = 1,45; RH = 1,71.

Sim. **AT e V** têm correlação forte entre si (r = 0,844), e AT também se relaciona de forma moderada com RH (−0,543) e AP (−0,508). Isso indica **multicolinearidade moderada**: o VIF de AT passa de 5, mas fica abaixo de 10, então não é grave. A consequência é que os coeficientes individuais de AT e V ficam menos estáveis e mais difíceis de interpretar, mas a capacidade de previsão quase não é afetada: ao retirar V, o R² no teste cai apenas de 0,9301 para 0,9219.

## Tarefa 3

> Construa um modelo de regressão linear múltipla utilizando as variáveis independentes (AT, V, AP, RH) para prever a variável dependente (PE). Divida os dados em treino (80%) e teste (20%).

**Resposta.**

Modelo treinado com as variáveis padronizadas (80% dos dados). Equação na unidade original:

$$ \hat{PE} = 454{,}5691 - 1{,}9859\,AT - 0{,}2321\,V + 0{,}0622\,AP - 0{,}1581\,RH $$

| Variável | Coeficiente padronizado | Coeficiente na unidade original |
|:---|---:|---:|
| AT | −14,7639 | −1,9859 MW/°C |
| V | −2,9496 | −0,2321 MW/cmHg |
| AP | +0,3698 | +0,0622 MW/mbar |
| RH | −2,3122 | −0,1581 MW/% |

Os coeficientes coincidem com os da equação normal $\theta = (X^TX)^{-1}X^Ty$, e o modelo padronizado e o modelo em unidades originais produzem as mesmas previsões. Mantidas as demais variáveis fixas, +1 °C em AT reduz a produção em cerca de 2 MW. AT domina o modelo na escala padronizada. RH tem correlação positiva com PE isoladamente, mas coeficiente negativo no modelo múltiplo: RH é maior em dias frios (correlação negativa com AT) e dias frios produzem mais energia; descontado o efeito de AT, o efeito próprio de RH é levemente negativo. É um exemplo de inversão de sinal causada por multicolinearidade.

## Tarefa 4

> Avalie o desempenho do modelo utilizando métricas como R², RMSE e MAE. O modelo é adequado para prever a produção de energia? Justifique sua resposta.

**Resposta.**

| Conjunto | R² | RMSE (MW) | MAE (MW) |
|:---|---:|---:|---:|
| Treino | 0,9283 | 4,5709 | 3,6300 |
| Teste | **0,9301** | **4,5026** | **3,5959** |
| Baseline (sempre a média) | −0,0004 | 17,0343 | 14,8067 |

Validação cruzada (5 folds) no treino: R² = 0,9282 ± 0,0031.

**Sim, o modelo é adequado.** O R² de 0,93 no teste significa que ele explica cerca de 93% da variação de PE (valor acima de 0,8 é considerado muito bom). R² de treino (0,928) e de teste (0,930) são praticamente iguais, então **não há overfitting**, e a validação cruzada confirma a estabilidade. O erro típico é de ≈ 4,5 MW (RMSE) e ≈ 3,6 MW (MAE), contra ≈ 17 MW do baseline, numa variável que vai de 420 a 496 MW (média de 454 MW): erro inferior a 1% da média. Limitação: o maior erro absoluto no teste chega a 28 MW, e modelos não lineares poderiam reduzir os erros maiores. Para o despacho energético, a precisão obtida é adequada.
