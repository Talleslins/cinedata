# CineData Analytics - Pipeline de Dados

Este repositório contém a implementação de um pipeline de Engenharia de Dados completo construído na infraestrutura Data Lakehouse do **Databricks**. O projeto estrutura dados fragmentados do setor audiovisual (TMDB/IMDb) e os transforma em Data Marts analíticos e bases vetorizáveis para Inteligência Artificial, seguindo rigorosamente a **Arquitetura Medalhão (Medallion Architecture)**.

## 🛠️ Tecnologias Utilizadas
* **Databricks** (Data Lakehouse)
* **PySpark & Spark SQL** (Processamento Distribuído)
* **Python** (Integração com APIs)
* **Databricks Workflows** (Orquestração de Jobs)

## 📂 Estrutura do Repositório

* `Landing_to_Bronze.ipynb`: Notebook responsável pela ingestão dos dados brutos (CSV) e consumo da API do Banco Central.
* `Bronze_to_Silver.ipynb`: Notebook dedicado à limpeza, tratamento, deduplicação, normalização e tradução dos dados.
* `Silver_to_Gold.ipynb`: Notebook de modelagem dimensional (Star Schema) e criação da tabela de contexto para o assistente de IA.
* `Analitycs.ipynb`: Notebook com as resoluções do Desafio de Analytics (consultas de negócio).
* `job.yaml`: Arquivo de configuração exportado do Databricks Workflow contendo a orquestração do pipeline.
* `Prints/`: Diretório contendo as evidências (screenshots) da execução com sucesso do Job e suas dependências.

## 🏗️ Arquitetura do Pipeline

O fluxo de dados foi dividido em três camadas principais:

### 🥉 Camada Bronze (Ingestão)
* Leitura de 5 arquivos CSV originais da pasta `Inputs` (`movies_info`, `financials`, `metrics`, `credits_and_tags`, `reviews`).
* Ingestão de dados da **API do Banco Central do Brasil (BCB)** para obter a cotação histórica do Dólar (USD para BRL).
* Inserção da coluna de auditoria `ingestion_datetime`.
* Armazenamento em formato **Delta** (modo Append) sem alterações estruturais no dado bruto.

### 🥈 Camada Silver (Tratamento e Normalização)
* **Limpeza e Tradução:** Padronização de strings, remoção de caracteres especiais e tradução de status (ex: "Released" para "Lançado").
* **Tratamento de Tipos e Nulos:** Correção de colunas com *Column Shift*, conversão segura de datas multiformato e sanitização de métricas financeiras (remoção de símbolos de moeda).
* **Regras de Negócio:** Tratamento de notas fora da escala (0 a 10), preenchimento de comentários vazios, cálculo de orçamentos e receitas para BRL utilizando a cotação consumida via API (com técnica de *Forward Fill* para finais de semana).
* **Explosão de Arrays:** Separação de strings delimitadas para normalizar as entidades de Gêneros, Atores, Diretores, Roteiristas e Produtoras.

### 🥇 Camada Gold (Consagração e Modelagem)
* **Modelagem Dimensional (Star Schema):**
  * **Fato:** `fact_movies_performance` (Métricas financeiras e de engajamento).
  * **Dimensões:** `dim_movies`, `dim_genres`, `dim_people`, `dim_companies`, `dim_reviews`.
  * **Bridge Tables (Tabelas-Ponte):** `bridge_movie_genre`, `bridge_movie_person`, `bridge_movie_company` para resolver relacionamentos N:N sem duplicar os grãos da tabela Fato. Todas construídas utilizando Surrogate Keys.
* **Integração GenAI:** Criação da tabela `gold_genai_movies_context` contendo a coluna `llm_context_document`. Essa tabela concatena as informações dimensionais e métricas de forma descritiva e inteligente (com tratamento de valores nulos via *fallback*) para alimentar um Banco de Dados Vetorial utilizado em arquiteturas RAG.

## ⏱️ Orquestração
O pipeline é executado sequencialmente através de um Job no Databricks, garantindo as seguintes dependências:
`Task_Landing_to_Bronze` ➔ `Task_Bronze_to_Silver` ➔ `Task_Silver_to_Gold`.
A configuração detalhada do agendamento e dos clusters encontra-se no arquivo `job.yaml`.

## 📈 Desafio de Analytics
No arquivo `Analitycs.ipynb`, estão consolidadas as respostas para as perguntas de negócio propostas, explorando a camada Gold. O relatório responde a perguntas como:
1. Receita total de todos os filmes.
2. Top 5 filmes mais populares.
3. Contagem de filmes por gênero.
4. Ranking dos 10 filmes de maior receita.
5. Ator com mais participações nos últimos 2 anos.
6. Produtora com maior lucro nos últimos 5 anos.
