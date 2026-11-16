# Dicionário de Dados — Amazon Product Performance & Customer Insights

## Visão Geral
Este documento descreve a base de origem, as colunas criadas no tratamento e no modelo, e as tabelas do modelo do Power BI. As medidas (DAX) estão documentadas em [`medidas_dax.md`](medidas_dax.md).

**Fonte:** Amazon Sales Dataset (Kaggle) · arquivo `amazon.csv` · 1.465 linhas × 16 colunas · 1.351 produtos únicos.

**Modelo no Power BI:**

| Tabela | Linhas | Origem | Função |
|---|---|---|---|
| `produtos` | 1.351 | `amazon.csv` tratado (1 linha por produto) | tabela principal, com colunas tratadas e calculadas |
| `produto_tema` | 4.291 | `produto_tema.csv`, gerado em Python | produtos × temas citados nas reviews (1.319 produtos têm ao menos 1 tema) |
| `_Medidas` | — | criada no Power BI | guarda todas as medidas DAX |

Relação: `produtos[product_id]` 1 → N `produto_tema[product_id]`.

---

## 1. Colunas originais (`amazon.csv`)

Na origem, **todas as colunas são texto**. Os tipos abaixo são os aplicados depois do tratamento.

| Coluna | Tipo (tratado) | Descrição | Observações |
|---|---|---|---|
| `product_id` | Texto | Identificador do produto | 1.351 distintos em 1.465 linhas; é a chave do modelo |
| `product_name` | Texto | Nome do produto | Nunca varia dentro do mesmo produto |
| `category` | Texto | Caminho completo da categoria, com `\|` entre níveis | 211 caminhos, de 2 a 7 níveis; ver colunas `categoria_n1/n2/n3` |
| `discounted_price` | Decimal | Preço com desconto, em ₹ | Origem tinha `₹` e vírgula de milhar; faixa 39 a 77.990 |
| `actual_price` | Decimal | Preço original, em ₹ | Divergente entre linhas em 1 produto (ver `preco_divergente`) |
| `discount_percentage` | Decimal (fração) | Desconto sobre o preço original | Origem em `%`; 0,49 = 49%; máximo 0,94; sem inconsistência com os dois preços |
| `rating` | Decimal | Nota média do produto (0 a 5) | 1 valor inválido (`\|`) virou nulo; nunca varia dentro do produto |
| `rating_count` | Inteiro | Número de avaliações do produto | Faixa 2 a 426.973; 2 vazios; varia pouco entre linhas repetidas (mediana de 1) |
| `about_product` | Texto | Descrição do produto | Varia em 4 produtos repetidos |
| `user_id` | Texto | IDs dos autores das reviews, separados por vírgula | Até 8 por linha; não usado na análise |
| `user_name` | Texto | Nomes dos autores | Dado pessoal; não usado na análise |
| `review_id` | Texto | IDs das reviews, separados por vírgula | Até 8 por linha |
| `review_title` | Texto | Títulos das reviews, separados por vírgula | Até 8 por linha |
| `review_content` | Texto (inglês) | Texto das reviews concatenadas | Base do Customer Insights; ver governança, item 3 |
| `img_link` | Texto | URL da imagem do produto | Varia em 77 produtos; não usado na análise |
| `product_link` | Texto | URL da página do produto | Varia em 92 produtos; não usado na análise |

---

## 2. Colunas criadas no Power Query

| Coluna | Tipo | Descrição |
|---|---|---|
| `ordem_original` | Inteiro | Posição da linha no arquivo original; define a "primeira ocorrência" |
| `linhas_originais` | Inteiro | Quantas linhas o produto tinha na base bruta (92 produtos têm mais de 1) |
| `preco_divergente` | Lógico | Verdadeiro se o preço difere entre as linhas repetidas (4 produtos) |
| `categoria_n1` | Texto | 1º nível do caminho (9 valores); visão executiva |
| `categoria_n2` | Texto | 2º nível (29 valores); análise principal |
| `categoria_n3` | Texto | 3º nível (71 valores); detalhe |

---

## 3. Colunas calculadas no modelo (DAX)

| Coluna | Tipo | Regra |
|---|---|---|
| `CATEGORIA_N1_AGRUPADA` | Texto | `categoria_n1` se a categoria tem 20 produtos ou mais; senão "Outras" |
| `CATEGORIA_N2_AGRUPADA` | Texto | Mesmo critério, para o nível 2 (9 categorias passam de 20 produtos) |
| `GRUPO_NOTA` | Texto | Baixa (nota ≤ 3,9), Média (entre 3,9 e 4,3) e Alta (≥ 4,3); sem nota = "Sem nota". Cortes = P25 e P75 da nota |
| `FAIXA_DESCONTO` | Texto | 0–20%, 20–40%, 40–60%, 60–80%, 80–100% |
| `FAIXA_DESCONTO_ORDEM` | Inteiro | 1 a 5; ordena `FAIXA_DESCONTO` |
| `SEGMENTO_PRODUTO` | Texto | "Alto engajamento e nota baixa" (≥ 16.020 avaliações e nota ≤ 3,9), "Nota alta e pouca visibilidade" (≤ 1.106 avaliações e nota ≥ 4,3), "Demais produtos" ou "Sem dados" |

---

## 4. Tabela `produto_tema`

Cada linha diz que um produto **menciona** um tema em alguma review.

| Coluna | Tipo | Descrição |
|---|---|---|
| `product_id` | Texto | Chave para `produtos` |
| `tema` | Texto | Um dos 11 temas abaixo |
| `tipo` | Texto | Problema, Tópico ou Ponto forte |

**Temas (dicionário de palavras-chave em inglês):**

| Tipo | Temas |
|---|---|
| Problema | Defeito / não funciona · Qualidade ruim / insatisfação · Alerta a compradores |
| Tópico | Áudio (grave, ruído, microfone) · Bateria / autonomia · Garantia / troca / reembolso · Entrega / embalagem |
| Ponto forte | Custo-benefício · Qualidade / durabilidade · Facilidade de uso · Recomendação explícita |

A lista completa de expressões de cada tema está no notebook do projeto. A detecção é por **menção de palavras, não sentimento**.

---

## 5. Medidas (`_Medidas`)

| Medida | O que mede | Formato |
|---|---|---|
| `TOTAL_PRODUTOS` | Quantidade de produtos | inteiro |
| `TOTAL_AVALIACOES` | Soma de `rating_count` | inteiro |
| `NOTA_MEDIA_PONDERADA` | Nota média ponderada pelo número de avaliações | 0,00 |
| `DESCONTO_MEDIANO` | Mediana do desconto | % |
| `PRECO_MEDIANO` | Mediana do preço com desconto (₹) | inteiro |
| `PRECO_ORIGINAL_MEDIANO` | Mediana do preço original (₹) | inteiro |
| `AVALIACOES_MEDIANA` | Mediana de avaliações por produto | inteiro |
| `% PORTFOLIO_PRODUTOS` | Participação nos produtos da seleção | 0,0% |
| `% PORTFOLIO_AVALIACOES` | Participação nas avaliações da seleção | 0,0% |
| `PRODUTOS_ALERTA` | Produtos do segmento "Alto engajamento e nota baixa" | inteiro |
| `PRODUTOS_OPORTUNIDADE` | Produtos do segmento "Nota alta e pouca visibilidade" | inteiro |
| `% MENCIONA(AMOSTRA MINIMA)` | % de produtos do grupo que mencionam o tema; vazio se o grupo tem menos de 20 produtos | % |

Há também `NOTA_MEDIA_SIMPLES` (4,09), usada apenas para comparação com a ponderada (4,16).
