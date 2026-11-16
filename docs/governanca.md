# Governança de Dados — Amazon Product Performance & Customer Insights

## Visão Geral
Este documento registra as decisões analíticas, os critérios de exclusão e os tratamentos aplicados ao longo do projeto. O objetivo é garantir transparência, reprodutibilidade e rastreabilidade: cada escolha tem uma **Decisão**, um **Motivo** e um **Impacto**.

**Base analisada:** 1.351 produtos únicos (1.465 linhas brutas) · 23,8 milhões de avaliações · preços em rúpias indianas (₹).

---

## 1. Granularidade: uma linha por produto

**Decisão:** consolidar as 1.465 linhas brutas em 1.351 produtos, mantendo a primeira ocorrência de cada `product_id` (coluna `ordem_original`).

**Motivo:** a auditoria mostrou 114 linhas excedentes em 92 produtos. O Python não encontrou duplicatas exatas (0); o Power Query mostrou 14 porque a remoção havia sido feita antes no Excel. O que se repete são produtos com conteúdo ligeiramente diferente entre linhas (links, textos de review, `rating_count`).

**Impacto:** todas as métricas contam produtos, não linhas. Cada produto tem a coluna `linhas_originais`, que permite auditar quantas linhas ele tinha.

---

## 2. Produtos com preço divergente entre linhas repetidas

**Decisão:** aplicar a regra fixa "primeira ocorrência" e sinalizar os 4 produtos afetados na coluna `preco_divergente`: B096MSW6CT, B09MT84WV5, B0B5B6PQCT e B0B5LVS732.

**Motivo:** não há data nem outro critério para dizer qual linha é a correta. Uma regra fixa é previsível e reproduzível.

**Impacto e teste de sensibilidade:** trocar a regra para "última ocorrência" não altera a mediana de preço (₹ 899), o desconto mediano (49%) nem a nota média ponderada (4,16). O único efeito é que o produto B096MSW6CT sairia da faixa de desconto de 80–100% (de 90% para 80%), ou seja, 1 produto em 1.351.

---

## 3. Reviews: um texto por produto

**Decisão:** manter uma linha por produto com o texto de review dessa linha, sem separar review a review.

**Motivo:** cada linha traz até 8 reviews concatenadas. Em 18 produtos repetidos, o conteúdo de review varia entre linhas: 11 com o mesmo conjunto de `review_id` e 7 com 1 review diferente.

**Impacto:** a análise de Customer Insights vale para o **produto** (ex.: "33% dos produtos de nota baixa mencionam defeito"), não para cada comentário individual.

---

## 4. Limpeza e tipagem

**Decisão:** importar tudo como texto, tratar e só então converter os tipos.

| Coluna | Tratamento |
|---|---|
| `discounted_price`, `actual_price` | remoção de `₹` e do separador de milhar; conversão para número |
| `discount_percentage` | remoção de `%`; conversão para fração (0,64 = 64%) |
| `rating` | 1 produto (B08L12N5H1) tinha o símbolo `\|` no lugar da nota; vira nulo, não zero |
| `rating_count` | remoção da vírgula de milhar; 2 valores vazios permanecem nulos |

**Motivo:** o Power Query adivinha tipos e erra em silêncio. Converter por último, com conferência, evita inflar ou truncar valores.

**Impacto:** o produto sem nota fica fora das médias e aparece como "Sem nota" no agrupamento de notas. Conferências feitas: 0 preços ≤ 0, 0 notas fora de 0–5, 0 descontos fora de 0–100% e 0 divergências entre o desconto declarado e o recalculado a partir dos dois preços.

---

## 5. Moeda: rúpias mantidas

**Decisão:** não converter os preços para reais.

**Motivo:** a base não tem data, e converter exigiria uma taxa de câmbio arbitrária. A análise usa proporções (% de produtos, % de avaliações, faixas de desconto) e medianas, que não dependem da moeda.

**Impacto:** valores de preço aparecem em ₹ e a limitação é dita no vídeo e no relatório.

---

## 6. Hierarquia de categorias

**Decisão:** separar a coluna `category` (separador `|`) em `categoria_n1`, `categoria_n2` e `categoria_n3`.

**Motivo:** a base tem 211 caminhos de 2 a 7 níveis. O nível 1 serve para a visão executiva, o nível 2 para a análise principal e o nível 3 para detalhe. Nomes com vírgula, como "HomeTheater,TV&Video", foram testados para garantir que a separação não os quebra.

**Impacto:** a análise por categoria usa o nível 1 (visão geral) e o nível 2 (filtro de Customer Insights).

---

## 7. Materialidade: mínimo de 20 produtos por grupo

**Decisão:** categorias com menos de 20 produtos viram "Outras" (`CATEGORIA_N1_AGRUPADA` e `CATEGORIA_N2_AGRUPADA`), e a medida de menção a temas só exibe resultado para grupos com pelo menos 20 produtos.

**Motivo:** 5 categorias de nível 1 somam apenas 7 produtos. Percentuais calculados sobre amostras minúsculas enganam; por exemplo, no filtro de fones o grupo de nota alta tem ~1 produto e mostraria 100%.

**Impacto:** o nível 1 fica com Electronics (490), Home&Kitchen (448), Computers&Accessories (375), OfficeProducts (31) e Outras (7). Na página Customer Insights, grupos abaixo do mínimo aparecem vazios.

---

## 8. Mediana em vez de média

**Decisão:** usar mediana para preço, desconto e avaliações; usar nota **ponderada pelo número de avaliações** como nota geral.

**Motivo:** preço (₹ 39 a ₹ 77.990) e avaliações (2 a 426.973) são muito assimétricos; poucos produtos gigantes distorceriam a média. A nota simples (4,09) trata igualmente um produto com 5 avaliações e outro com 400 mil; a ponderada (4,16) reflete a experiência de quem de fato avaliou.

**Impacto:** os cartões do painel mostram mediana (desconto 49%, preço ₹ 899) e nota ponderada (4,16).

---

## 9. Avaliações como medida de engajamento

**Decisão:** usar `rating_count` como proxy de atenção/engajamento.

**Motivo:** a base não tem vendas, lucro, estoque nem datas.

**Impacto:** nenhuma conclusão fala de receita, margem ou recompra; o painel mostra atenção dos clientes. Afirmações sobre o efeito de campanhas de desconto não são possíveis.

---

## 10. Segmentos de alerta e oportunidade

**Decisão:** definir os cortes por percentis, sem número escolhido a olho.

| Segmento | Regra | Origem dos cortes | Produtos |
|---|---|---|---|
| Alto engajamento e nota baixa (alerta) | ≥ 16.020 avaliações **e** nota ≤ 3,9 | P75 das avaliações; P25 da nota | 49 |
| Nota alta e pouca visibilidade (oportunidade) | ≤ 1.106 avaliações **e** nota ≥ 4,3 | P25 das avaliações; P75 da nota | 81 |
| Demais produtos | restante | — | — |

**Motivo:** percentis são um critério objetivo e reproduzível; "quartil superior de atenção com quartil inferior de nota" é uma definição fácil de explicar.

**Impacto:** os cortes mudam se a base mudar. Os segmentos são para priorização, não classificação definitiva.

---

## 11. Faixas de desconto

**Decisão:** agrupar o desconto em faixas de 20 pontos (0–20%, 20–40%, 40–60%, 60–80%, 80–100%).

**Motivo:** faixas permitem comparar nota e engajamento sem forçar uma linha reta. O desconto máximo na base é 94%.

**Impacto:** a faixa de 80–100% tem só 51 produtos e deve ser lida com cautela. O resultado de nota ≈ 3,99 nessa faixa se mantém ao retirar os 3 maiores produtos (3,995).

---

## 12. Correlação, não causa

**Decisão:** medir a relação entre desconto e nota/avaliações com correlação de Spearman (ρ ≈ −0,15 e −0,13) e declarar que é correlação.

**Motivo:** Spearman não exige relação linear e aguenta bem a assimetria dos dados. Os valores são fracos; eles não confirmam a hipótese inicial de que mais desconto traz mais engajamento.

**Impacto:** a conclusão é "desconto não compra engajamento nem satisfação nesta base". Provar causa exigiria experimento (teste A/B) ou dados de vendas e datas.

---

## 13. Customer Insights: dicionário de temas

**Decisão:** detectar 11 temas por **palavras-chave** em inglês (as reviews estão em inglês), organizados em Problema, Tópico e Ponto forte, e comparar produtos de nota baixa (≤ 3,9; 340 produtos) com os de nota alta (≥ 4,3; 419 produtos).

**Motivo:** um dicionário de palavras é transparente e auditável tema a tema. Análise de sentimento exigiria modelo e validação que não cabiam no prazo.

**Validação:** o dicionário foi lido contra trechos reais. A primeira versão gerou falsos positivos (negações, "please don't raise prices", som de TV, bateria de celular) e foi reescrita; os números do painel vêm da versão corrigida e batem com o cálculo em Python.

**Impacto:** o método detecta **menção de palavras**, não sentimento. Um produto "menciona defeito" quando alguma review usa a expressão, mesmo que o produto seja bom. Os resultados são evidência de associação, não de causa.

---

## 14. Escalas dos gráficos

**Decisão:** (a) escala logarítmica no eixo de avaliações do gráfico de dispersão; (b) eixo da nota iniciando em 3,0 no gráfico por categoria.

**Motivo:** (a) as avaliações vão de 2 a mais de 400 mil; em escala linear quase todos os produtos ficariam espremidos num canto. (b) as notas se concentram entre 4 e 4,3; um eixo curto torna a diferença visível.

**Impacto:** como o eixo truncado amplia visualmente as diferenças, todos os valores aparecem escritos nas barras e há aviso no rodapé da página.

---

## 15. Limitações conhecidas

- Sem vendas, lucro, estoque, datas ou histórico de preço: não é possível medir receita nem efeito de campanhas.
- Preços em rúpias indianas, sem conversão.
- Notas concentradas entre 4 e 4,3: diferenças pequenas na nota podem ser ruído.
- Faixa de desconto de 80–100% com apenas 51 produtos.
- Temas por palavras-chave, em inglês, sem análise de sentimento.
- Preço divergente em 4 produtos, tratado por regra fixa (ver item 2).
