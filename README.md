# Plataforma de BI para Draft de NBA Fantasy

## 1\. Descrição do Projeto

Este projeto, desenvolvido para a disciplina de **Mineração de Dados**, é uma solução de Business Intelligence de ponta-a-ponta projetada para otimizar a tomada de decisão em drafts de ligas de NBA Fantasy.

O processo de draft é complexo e exige a análise de dezenas de fontes de dados (estatísticas passadas, projeções futuras, percepção do mercado, etc.). Esta plataforma automatiza a coleta, processamento, análise e visualização desses dados, entregando ao usuário final um dashboard interativo que identifica jogadores subvalorizados (`sleepers`) e supervalorizados (`busts`) com base em métricas de mineração de dados.

## 2\. Principais Funcionalidades

  * **Pipeline de Dados Automatizado:** Um pipeline ETL robusto que transforma dados brutos de múltiplas fontes em tabelas de análise limpas e otimizadas.
  * **Mineração de KPIs Estratégicos:** Cálculo de métricas de negócio (`Value_vs_ADP`) que não são encontradas em plataformas tradicionais.
  * **Dashboard Interativo:** Uma interface em Power BI que permite ao usuário explorar perfis de jogadores, comparar atletas e visualizar os principais alvos de draft.
  * **Arquitetura Escalável:** Uso da arquitetura Medallion (Bronze, Silver, Gold) no Databricks, garantindo governança e qualidade dos dados.

## 3\. Arquitetura de Dados (Medallion)

A solução é estruturada em três camadas lógicas dentro do Databricks:

### 🥉 Camada Bronze

  * **Propósito:** Ingestão dos dados brutos, sem tratamento.
  * **Fontes:**
      * `players.csv`: Dados dimensionais (biografia, altura, peso) e IDs dos jogadores (Kaggle).
      * `stats.csv`: Estatísticas da temporada passada.
      * `projections.csv`: Projeções para a próxima temporada.
      * `adp.csv`: Posição média no draft (Average Draft Position).
  * **Tecnologia:** Databricks Volumes, SQL (`read_files`).

### 🥈 Camada Silver

  * **Propósito:** Limpeza, padronização, filtragem e enriquecimento dos dados.
  * **Processo:**
    1.  Criação de uma chave de junção universal (`Player_Key`) através da normalização dos nomes dos jogadores (ex: `nikola jokic`).
    2.  Conversão de tipos de dados (String para Double, etc.).
    3.  Criação de novas colunas (ex: `Image_URL` a partir do `playerid`).
    4.  Criação de duas tabelas limpas: `nba_silver_player_data` (fatos) e `dim_jogadores_silver` (dimensões).
  * **Tecnologia:** PySpark, SQL.

### 🥇 Camada Gold

  * **Propósito:** Camada final de consumo, com dados agregados e KPIs de negócio.
  * **Processo:**
    1.  Criação da **`dim_jogadores`**: Tabela de dimensão final, enriquecida com dados biográficos e imagens.
    2.  Criação da **`fct_analise_draft`**: Tabela de fatos principal, onde a mineração de dados ocorre.
    3.  Criação da **`player_metrics_unpivoted`**: Tabela desdinamizada (formato longo) otimizada para visuais de comparação no Power BI.
  * **Tecnologia:** PySpark (para KPIs), Delta Lake.

## 4\. Lógica de Negócio (Mineração de Dados)

O núcleo do projeto está no cálculo de dois KPIs na camada Gold:

1.  **`Fantasy_Score_Projected`**: Uma pontuação de projeção personalizada baseada em uma fórmula ponderada (ex: `(PTS*1) + (TRB*1.2) + (AST*1.5) + (STL*3) + (BLK*3) - (TOV*2)`), que define o ranking interno do nosso modelo.

2.  **`Value_vs_ADP` (KPI Principal)**: O verdadeiro "valor" de um jogador, calculado pela fórmula `(ADP_Rank - Fantasy_Rank)`.

      * Um **valor positivo alto** significa que o jogador é subvalorizado pelo mercado (um `sleeper`).
      * Um **valor negativo alto** significa que o jogador é supervalorizado (um `bust`).

## 5\. Visualização (Power BI)

O Power BI se conecta às três tabelas da Camada Gold (`dim_jogadores`, `fct_analise_draft`, `player_metrics_unpivoted`) usando o conector Databricks no modo **Importar**.

### Páginas do Dashboard:

  * **Overview:** Apresenta os KPIs de alto nível (Total de Jogadores, Top Sleeper, Mais Supervalorizado) e a tabela principal de "Top Draft Targets", com formatação condicional divergente no `Value_vs_ADP`.
  * **Player Search:** Uma página de busca detalhada com filtros (Slicers) por Posição, Rank de ADP e Nome.
  * **Player Comparisons:** Permite ao usuário selecionar dois ou mais jogadores e compará-los lado a lado usando Gráficos de Radar (para perfil) e Gráficos de Barras (para estatísticas absolutas).

## 6\. Tecnologias Utilizadas

  * **Plataforma de Dados:** Databricks
  * **Linguagens:** PySpark (para transformações) e SQL
  * **Armazenamento e Formato:** Databricks Volumes, Delta Lake
  * **Visualização (BI):** Power BI
  * **Fonte de Dados:** Arquivos CSV (Kaggle e outras fontes estáticas)

## 7\. Como Executar

1.  **Bronze:** Faça o upload dos arquivos CSV brutos para o diretório `/Volumes/workspace/nba_lakehouse_bronze/raw/`.
2.  **Silver:** Execute o notebook `02_silver_transfomation.ipynb` para carregar os dados brutos, limpá-los e criar as tabelas da camada Silver (`nba_silver_player_data`, `dim_jogadores_silver`).
3.  **Gold:** Execute o notebook `03_gold.ipynb` para consumir os dados da camada Silver, aplicar a lógica de negócio (KPIs) e criar as três tabelas finais da camada Gold.
4.  **Power BI:** Abra o arquivo `NBA_Fantasy_Draft.pbix` e atualize os dados. O modelo de dados está configurado para se conectar às tabelas Gold do Databricks.
