# Problema 2 – Regressão Logística para Previsão de Doenças Cardíacas

*PAM0466 – Sistemas Inteligentes · 1ª Lista de Projetos*

Doenças cardiovasculares são a principal causa de morte no mundo, responsáveis por cerca de 18 milhões de óbitos por ano segundo a OMS. O diagnóstico precoce é determinante para o sucesso do tratamento. Existe, portanto, uma necessidade real de ferramentas capazes de estimar o risco de um paciente com base em dados clínicos simples, coletados em consulta de rotina. É nesse contexto que o aprendizado de máquina pode contribuir. Um modelo de classificação treinado com dados históricos de pacientes diagnosticados é capaz de aprender quais combinações de variáveis clínicas estão associadas à presença de doença cardíaca e aplicar esse aprendizado a novos pacientes.

Neste projeto, você é convidado a construir um modelo utilizando regressão logística para prever a presença de doença cardíaca com base em um conjunto de dados clínicos.

O projeto utiliza o dataset Heart Disease Cleveland (UCI Machine Learning Repository – disponível em https://archive.ics.uci.edu/dataset/45/heart+disease), com registros de 303 pacientes e 13 variáveis clínicas. Os diagnósticos foram confirmados por cateterismo cardíaco, o que garante rótulos confiáveis. As variáveis incluem:

- age — idade do paciente em anos;
- sex — sexo biológico (1 = masculino, 0 = feminino);
- cp — tipo de dor no peito: angina típica (1), atípica (2), não anginosa (3) ou assintomático (4);
- trestbps — pressão arterial em repouso em mmHg;
- chol — colesterol sérico total em mg/dl;
- fbs — glicemia em jejum acima de 120 mg/dl (1 = sim, 0 = não);
- restecg — resultado do ECG em repouso: normal (0), alteração na onda ST-T (1) ou hipertrofia ventricular (2);
- thalach — frequência cardíaca máxima atingida no teste de esforço;
- exang — angina induzida por exercício (1 = sim, 0 = não);
- oldpeak — depressão do segmento ST durante o esforço, indicador de isquemia;
- slope — inclinação do segmento ST no pico do esforço: ascendente (1), plano (2) ou descendente (3);
- ca — número de artérias coronárias com obstrução visível na fluoroscopia (0 a 3);
- thal — cintilografia de tálio: normal (3), defeito fixo / infarto prévio (6) ou defeito reversível / isquemia ativa (7);

A saída é a variável target, que indica o diagnóstico original de 0 a 4, binarizado para 0 (sem doença) e 1 (com doença) antes do treinamento.

Realize as seguintes tarefas utilizando o dataset fornecido:

1. Treinar um modelo de LogisticRegression do scikit-learn utilizando os dados de treino, com os seguintes parâmetros: solver='liblinear', random_state=42 e max_iter=1000. Indique quantas iterações foram necessárias para a convergência do modelo.
2. Avaliar o modelo utilizando os dados de teste e calcular as seguintes métricas: Acurácia, Precisão, Recall, F1-score e Curva ROC com valor de AUC.

Realize o pré-processamento dos dados, incluindo a normalização ou padronização das variáveis independentes, tratamento de valores ausentes, divisão dos dados em conjuntos de treino e teste e binarização das variáveis alvo.
