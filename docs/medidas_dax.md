# Colunas Calculadas e Medidas DAX

## Visão Geral
Colunas e medidas criadas no modelo (tabelas `produtos` e `_Medidas`). Os números de referência foram conferidos contra o cálculo em Python.

---

## 1. Colunas calculadas (`produtos`)

### Categorias agrupadas (materialidade: mínimo de 20 produtos)
```dax
CATEGORIA_N1_AGRUPADA =
VAR c = produtos[categoria_n1]
RETURN IF(COUNTROWS(FILTER(produtos, produtos[categoria_n1] = c)) >= 20, c, "Outras")
```
`CATEGORIA_N2_AGRUPADA` segue a mesma regra, sobre `categoria_n2`.

### Faixa de desconto
```dax
FAIXA_DESCONTO =
SWITCH(TRUE(),
    produtos[discount_percentage] <= 0.2, "0-20%",
    produtos[discount_percentage] <= 0.4, "20-40%",
    produtos[discount_percentage] <= 0.6, "40-60%",
    produtos[discount_percentage] <= 0.8, "60-80%",
    "80-100%")
```
`FAIXA_DESCONTO_ORDEM` devolve 1 a 5 com os mesmos cortes e ordena a coluna acima.

### Segmento do produto (alerta e oportunidade)
```dax
SEGMENTO_PRODUTO =
SWITCH(TRUE(),
    ISBLANK(produtos[rating]) || ISBLANK(produtos[rating_count]), "Sem dados",
    produtos[rating_count] >= 16020 && produtos[rating] <= 3.9, "Alto engajamento e nota baixa",
    produtos[rating_count] <= 1106 && produtos[rating] >= 4.3, "Nota alta e pouca visibilidade",
    "Demais produtos")
```
Os cortes são percentis da base: 16.020 = P75 e 1.106 = P25 das avaliações; 3,9 = P25 e 4,3 = P75 da nota.

### Grupo de nota
`GRUPO_NOTA` classifica cada produto em **Baixa** (nota ≤ 3,9), **Média** e **Alta** (nota ≥ 4,3), com "Sem nota" para o produto sem rating. `GRUPO_NOTA_ORDEM` ordena Baixa, Média, Alta.

```dax
-- COLE AQUI a fórmula de GRUPO_NOTA e GRUPO_NOTA_ORDEM do modelo
```

---

## 2. Medidas básicas

```dax
TOTAL_PRODUTOS = COUNTROWS(produtos)

TOTAL_AVALIACOES = SUM(produtos[rating_count])

NOTA_MEDIA_SIMPLES = AVERAGE(produtos[rating])

NOTA_MEDIA_PONDERADA =
DIVIDE(
    SUMX(FILTER(produtos, NOT ISBLANK(produtos[rating]) && NOT ISBLANK(produtos[rating_count])),
         produtos[rating] * produtos[rating_count]),
    SUMX(FILTER(produtos, NOT ISBLANK(produtos[rating]) && NOT ISBLANK(produtos[rating_count])),
         produtos[rating_count]))

DESCONTO_MEDIANO = MEDIAN(produtos[discount_percentage])
PRECO_MEDIANO = MEDIAN(produtos[discounted_price])
PRECO_ORIGINAL_MEDIANO = MEDIAN(produtos[actual_price])
AVALIACOES_MEDIANA = MEDIAN(produtos[rating_count])
```

| Medida | Valor de referência |
|---|---|
| `TOTAL_PRODUTOS` | 1.351 |
| `TOTAL_AVALIACOES` | 23.802.423 |
| `NOTA_MEDIA_SIMPLES` | 4,09 |
| `NOTA_MEDIA_PONDERADA` | 4,16 |
| `DESCONTO_MEDIANO` | 49% |
| `PRECO_MEDIANO` | ₹ 899 |

**Por que ponderada?** A nota simples trata igual um produto com 5 avaliações e outro com 400 mil. A ponderada dá peso a quem de fato avaliou.

---

## 3. Participação no portfólio

```dax
% PORTFOLIO_PRODUTOS =
DIVIDE([TOTAL_PRODUTOS], CALCULATE([TOTAL_PRODUTOS], ALLSELECTED(produtos)))

% PORTFOLIO_AVALIACOES =
DIVIDE([TOTAL_AVALIACOES], CALCULATE([TOTAL_AVALIACOES], ALLSELECTED(produtos)))
```
`ALLSELECTED` mantém os filtros de página e segmentadores e remove só o filtro da linha atual, de modo que o percentual sempre é "parte do que está selecionado".

---

## 4. Alerta e oportunidade

```dax
PRODUTOS_ALERTA =
CALCULATE([TOTAL_PRODUTOS], produtos[SEGMENTO_PRODUTO] = "Alto engajamento e nota baixa")

PRODUTOS_OPORTUNIDADE =
CALCULATE([TOTAL_PRODUTOS], produtos[SEGMENTO_PRODUTO] = "Nota alta e pouca visibilidade")
```
Valores de referência: 49 produtos de alerta e 81 de oportunidade.

---

## 5. Customer Insights

`% MENCIONA(AMOSTRA MINIMA)` calcula, para cada tema, a **% de produtos do grupo de nota que mencionam o tema** (tabela `produto_tema`) e devolve vazio quando o grupo tem **menos de 20 produtos**, para evitar percentuais enganosos.

```dax
-- COLE AQUI a fórmula de % MENCIONA(AMOSTRA MINIMA) do modelo
```
