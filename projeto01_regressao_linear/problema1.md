# Problema 1 — Regressão Linear para Previsão de Produção de Energia em uma Usina de Ciclo Combinado

*PAM0466 – Sistemas Inteligentes (UFERSA) · 1ª Lista de Projetos · Prof. Pedro Thiago Valério de Souza*

Uma usina termoelétrica de ciclo combinado (UTCC) é uma instalação de alta eficiência que combina turbinas a gás e a vapor para gerar eletricidade. Ela reaproveita o calor dos gases de exaustão da turbina a gás (ciclo Brayton) para produzir vapor em uma caldeira, acionando uma segunda turbina (ciclo Rankine), aumentando significativamente o rendimento. A principal vantagem é a eficiência energética, que pode chegar a 60%, reduzindo as emissões de gases de efeito estufa.

Embora apresente alta eficiência, o principal desafio técnico no uso de uma usina UTCC reside na integração, sincronização e operação conjunta de dois ciclos termodinâmicos distintos (ciclo Brayton e ciclo Rankine) para funcionar de forma eficiente. Assim sendo, a potência elétrica de saída (PE) é altamente sensível às condições ambientais.

Neste projeto, você deve construir um modelo matemático de regressão linear múltipla para prever a produção de energia baseando-se em sensores ambientais, permitindo a otimização do despacho energético.

Para realizar o treinamento do nosso modelo, utilizamos o dataset UCI CCPP (disponível em https://archive.ics.uci.edu/ml/datasets/Combined+Cycle+Power+Plant). O dataset contém 9.568 amostras coletadas ao longo de 6 anos. Cada amostra possui as seguintes variáveis:

- AT (Ambient Temperature): Temperatura em °C. Afeta a densidade do ar e a eficiência da combustão.
- V (Exhaust Vacuum): Vácuo de escape em cm Hg. Relacionado à pressão na turbina a vapor.
- AP (Ambient Pressure): Pressão atmosférica em mbar.
- RH (Relative Humidity): Umidade relativa em percentagem.
- PE (Net Hourly Electrical Energy Output): A variável alvo em MW (MegaWatts).

Realize as seguintes tarefas utilizando o dataset fornecido:

1. Calcule a correlação entre as variáveis independentes (AT, V, AP, RH) e a variável dependente (PE). Quais variáveis parecem ter maior influência na produção de energia?
2. Calcule a correlação entre as variáveis independentes (AT, V, AP, RH) entre si. Existe alguma relação forte entre elas? Se sim, isso pode indicar multicolinearidade.
3. Construa um modelo de regressão linear múltipla utilizando as variáveis independentes (AT, V, AP, RH) para prever a variável dependente (PE). Divida os dados em treino (80%) e teste (20%).
4. Avalie o desempenho do modelo utilizando métricas como R², RMSE e MAE. O modelo é adequado para prever a produção de energia? Justifique sua resposta.

Realize o pré-processamento dos dados, incluindo a normalização ou padronização das variáveis independentes, tratamento de valores ausentes e divisão dos dados em conjuntos de treino e teste.
