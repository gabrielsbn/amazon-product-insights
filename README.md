# Amazon Product Performance & Customer Insights

## Sobre o Projeto
Análise de produtos de um catálogo da Amazon, desenvolvida como Tech Challenge da pós-tech em Data Analytics (FIAP). O objetivo é transformar dados de preço, desconto, nota e avaliações em uma narrativa executiva para a liderança: **onde está a atenção dos clientes, se o desconto funciona e o que as avaliações dizem sobre o que melhorar**, terminando em recomendações acionáveis.

## Dataset
- **Fonte:** Amazon Sales Dataset (Kaggle);
- **Volume:** 1.465 linhas brutas, consolidadas em **1.351 produtos únicos**, com **23,8 milhões de avaliações**;
- **Moeda:** preços em rúpias indianas (₹), sem conversão — a análise usa proporções e medianas;
- **Limitação central:** a base **não tem vendas, lucro nem datas**. O número de avaliações é usado como medida de engajamento (atenção), não de receita.

## Ferramentas Utilizadas
- **Power BI** — Power Query, modelo, medidas DAX e dashboard executivo;
- **Python (pandas)** — auditoria da base, análises estatísticas e dicionário de temas das reviews;
- **GitHub** — documentação e versionamento do projeto.

## Estrutura do Repositório

📁 **amazon-product-insights**
- 📄 `README.md` — Visão geral do projeto, links do dashboard e do vídeo
- 📁 **docs/**
  - 📄 `governanca.md` — Decisões analíticas e critérios de qualidade
  - 📄 `dicionario_dados.md` — Tabelas, colunas e medidas
  - 📄 `medidas_dax.md` — Colunas calculadas e medidas DAX
  - 📄 `relatorio_executivo.pdf` — Relatório executivo do projeto
- 📁 **notebooks/**
  - 📄 `auditoria.ipynb` — Auditoria da base, exploração e dicionário de temas
- 📁 **data/**
  - 📄 `produto_tema.csv` — Produtos × temas citados nas reviews (gerado em Python)

> O `amazon.csv` original não é versionado aqui; baixe-o no Kaggle (link acima).

## Como Reproduzir
1. Baixe o `amazon.csv` no Kaggle e salve em `data/amazon.csv`;
2. Execute `notebooks/auditoria.ipynb` do início ao fim; ele gera o `produto_tema.csv` na própria pasta `notebooks/` (a versão já publicada está em `data/`; copie o arquivo novo para lá se quiser substituí-la);
3. Abra o `.pbix` (link abaixo) no Power BI Desktop; o modelo usa `amazon.csv` tratado no Power Query e `produto_tema.csv`.

## Principais Indicadores

| Indicador | Valor | Leitura |
|---|---|---|
| Produtos | 1.351 | Um por `product_id` |
| Avaliações | 23,8 milhões | Proxy de engajamento |
| Nota média ponderada | 4,16 | Ponderada pelo nº de avaliações |
| Desconto mediano | 49% | Mediana, por causa da assimetria |
| Preço mediano | ₹ 899 | Com desconto |

## Principais Achados

1. **A atenção está concentrada.** Electronics tem 36% dos produtos e ~60% das avaliações; Home & Kitchen tem 33% dos produtos e só ~13% das avaliações.
2. **Há riscos e oportunidades claros.** 49 produtos têm alto engajamento e nota baixa (risco de reputação em escala); 81 têm nota alta e pouca visibilidade. A categoria de fones tem a menor nota (4,03).
3. **Desconto não compra engajamento nem satisfação.** A correlação de Spearman com avaliações e com a nota é fraca e negativa (≈ −0,13 e −0,15). Essa é uma correlação, não uma causa.
4. **A nota baixa tem a ver com o produto, não com o preço.** Defeito/"não funciona" aparece em 33% dos produtos de nota baixa contra 17% nos de nota alta; "custo-benefício" aparece em 51% contra 56% — não separa os grupos.

## Estrutura do Dashboard
O dashboard tem cinco páginas, navegáveis pelas abas do cabeçalho:

1. **Visão Executiva** — indicadores principais e participação de cada categoria nos produtos e nas avaliações
2. **Produtos e Categorias** — mapa de produtos (engajamento × nota, em escala logarítmica) e nota por categoria
3. **Preços e Estratégia Comercial** — nota, engajamento e preço por faixa de desconto
4. **Customer Insights** — temas das reviews em produtos de nota baixa, média e alta
5. **Recomendações** — ações priorizadas, mensagens-chave e limitações

## Decisões Analíticas
As decisões metodológicas (granularidade, tratamento de preços, materialidade, cortes dos segmentos, escalas) estão em [docs/governanca.md](docs/governanca.md). As colunas e medidas estão em [docs/dicionario_dados.md](docs/dicionario_dados.md) e [docs/medidas_dax.md](docs/medidas_dax.md).

## Relatório Executivo
O relatório com o contexto, os achados e as recomendações está em [docs/relatorio_executivo.pdf](docs/relatorio_executivo.pdf).

## Dashboard Power BI
O arquivo `dashboard_final.pbix` está hospedado externamente por conta da limitação de tamanho do GitHub. Abra no Power BI Desktop; ele tem 5 páginas, navegáveis pelas abas do cabeçalho, com filtros por categoria, faixa de desconto e grupo de nota.

[Download do Dashboard — Google Drive](https://drive.google.com/file/d/15KEfWRGRh8WhzZtXD9UEXHaU9X2dPgjt/view?usp=sharing)

## Vídeo Executivo
[Assistir ao vídeo (até 7 minutos)](https://youtu.be/ElUIhzJ9VUQ)

## Autor
Gabriel da Silva Brandão Nascimento [RM374018] — Pós-Tech em Data Analytics FIAP 2026.
