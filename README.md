# Coin Recognition

Projeto de segmentação e reconhecimento de moedas de Euro utilizando técnicas clássicas de Visão Computacional e Machine Learning.

## Pipeline

1. Pré-processamento das imagens
2. Segmentação das moedas
3. Avaliação da segmentação com Intersection over Union (IoU)
4. Data Augmentation e balanceamento das classes
5. Extração de características
   * Histogram of Oriented Gradients (HOG)
   * Local Binary Pattern (LBP)
   * Histogramas de cor
   * Fusão de features
6. Classificação
   * Support Vector Machine (SVM)
   * Random Forest (RF)
7. Avaliação dos resultados

## *Dataset*

Foi utilizado o dataset [EuroCoins](https://www.kaggle.com/datasets/janstaffa/euro-coins-dataset), contendo 336 imagens de moedas e 7 classes.

## Pré-processamento e segmentação

`Grayscale → Median Filter → Hough Circle Transform`

As imagens foram convertidas para escala grayscale de modo a reduzir a dimensionalidade dos dados e eliminar informações de cor não necessárias para a detecção circular das moedas. Em seguida, aplicou-se o Median Filter por conseguir conciliar a redução e suavização de ruídos e preservação de bordas na imagem. Importante pela próxima etapa depender da identificação de contornos. Uma suavização excessiva poderia enfraquecer essas bordas e prejudicar a detecção, enquanto a ausência de filtragem poderia aumentar a ocorrência de bordas causadas por ruído, reflexos ou texturas do fundo.

Para a detecção de círculos, utilizou-se o Hough Circles, que já aplica informação de gradiente e detecção de bordas (com Canny), uma técnica do OpenCV que é comumente aplicada para identificar formas circulares geométricas conhecidas em imagens.

Do total de 336 moedas, 334 (99.4%) foram segmentadas corretamente (considerando IoU >= 0.5) com um IoU médio correspondente a 0.9143.

## *Data augmentation* e balanceamento

Para aumentar a diversidade dos dados de treinamento e auxiliar no balanceamento das classes, foram geradas novas versões das moedas.

As transformações utilizadas foram:
* Espelhamento horizontal
* Rotações (em 15°, -15°, 90°, 180° e 270°)
* Variações de brilho de ±25

As transformações foram aplicadas apenas aos dados de **treinamento**, evitando vazamento de dados (*data leakage*) para os conjuntos utilizados na avaliação.

## *Feature Extraction*
Foram avaliadas diferentes formas de representação das moedas:

* **HOG**: representa principalmente informações relacionadas às bordas, formas e direções dos gradientes da imagem, sendo usado para descrever a estrutura visual das moedas.
* **LBP**: descreve padrões locais de textura, permitindo capturar diferenças nos detalhes presentes na superfície das moedas.
* **Histograma de cor**: representa a distribuição das cores, considerando as diferenças de material e tonalidade das moedas.
* **Fusão de features**: combina os três descritores (`Fusion = HOG + LBP + Color`), utilizando simultaneamente as informações de forma, textura e cor.

## Classificação
Foram selecionados o **SVM** e o **Random Forest**, dois algoritmos clássicos que utilizam estratégias distintas de classificação. 

O SVM busca encontrar fronteiras de decisão capazes de separar as diferentes classes a partir das características extraídas das imagens, enquanto o Random Forest combina múltiplas árvores de decisão para realizar a classificação. Dessa forma, foi possível comparar o comportamento de diferentes estratégias sobre as características das moedas.

O projeto utilizou **Nested Cross-Validation** para selecionar os hiperparâmetros e avaliar os modelos de forma mais confiável. A validação cruzada interna (inner CV - 3 folds) foi responsável por selecionar a melhor combinação de descritor, classificador e hiperparâmetros com base no *F1-Score*. Ao passo que a validação cruzada externa (outer CV - 5 folds) foi utilizada para estimar o desempenho dessa estratégia de seleção em dados não utilizados na escolha do modelo.

Além disso, foi utilizado **Stratified Group K-Fold** para que amostras provenientes de uma mesma imagem original permanecessem no mesmo grupo. Assim, evitando que moedas originadas da mesma imagem apareçam simultaneamente nos conjuntos de treinamento e validação, reduzindo o risco de *data leakage*. A estratificação também busca preservar a distribuição das classes entre os folds.

## Avaliação Final

Após a etapa de seleção e validação dos modelos, a melhor combinação foi utilizada para realizar a avaliação final no conjunto de teste.

A configuração final foi:
- Feature: HOG
- Classificador: SVM
- Hiperparâmetros: {'C': 10.0, 'gamma': 0.01}

Foram obtidos os seguintes resultados:

| Métrica | Resultado |
| :-------- | -------- |
|Acurácia | 0.90|
|*F1-Score* ponderado|0.90|
|*Precision* ponderado|0.92|
|*Recall* ponderado|0.90|
