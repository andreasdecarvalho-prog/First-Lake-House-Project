# Bike Lakehouse - Medallion Architecture Data Pipeline

Um projeto de engenharia de dados implementando a **Medallion Architecture (Bronze-Silver-Gold)** com **Databricks**, focado em transformação e análise de dados de vendas de bicicletas.

## Sobre

Este projeto demonstra uma abordagem profissional e escalável para engenharia de dados, aplicando best practices da indústria como Medallion Architecture, 
DRY principles e Star Schema. É um exemplo de como um engenheiro de dados junior pode estruturar pipelines que são fáceis de manter, escalar e compreender.

---

### Por que Medallion?
-  **Separação clara de responsabilidades**: cada camada tem um propósito definido
-  **Rastreabilidade**: é sempre possível voltar à origem dos dados
-  **Qualidade progressiva**: dados melhoram conforme passam pelas camadas
-  **Escalabilidade**: fácil adicionar novas transformações sem quebrar pipelines existentes

---

##  Estrutura do Projeto

bike_lakehouse/ 

├── Bronze (1)/# Camada de dados brutos 
│ └── Bronze_code.ipynb # Ingestão automatizada de arquivos CSV 
├── Silver/ # Camada de dados limpos e transformados 
│ ├── helper_functions.ipynb # Funções reutilizáveis para limpeza 
│ ├── run_silver_pipeline.ipynb # Orquestrador do pipeline 
│ ├── silver_cust_info.ipynb 
│ ├── silver_cust_az12.ipynb 
│ ├── silver_loc_a101.ipynb 
│ ├── silver_prd_info.ipynb  
│ ├── silver_px_cat_g1v2.ipynb 
│ └── silver_sales_details.ipynb 
└── gold/ # Camada de dados otimizados para analytics 
└── gold.ipynb # Criação de tabelas dimensionais e de fatos

---

##  Fluxo de Dados por Camada

### ** Bronze Layer - Ingestão de Dados Brutos**

A camada Bronze é o ponto de entrada do data lake. Sua responsabilidade é:
- **Ler arquivos** de sistemas de origem (CSVs neste caso)
- **Ingerir minimamente processados** mantendo os dados o mais próximo possível da fonte
- **Criar tabelas gerenciadas** no Databricks


### ** Silver Layer - Transformação e Limpeza**

A camada Silver transforma dados brutos em dados confiáveis através de:
- **Limpeza de dados: remoção de espaços em branco, tratamento de nulos**
- **Validação de tipos: conversão de tipos de dados**
- **Enriquecimento: normalização de valores (ex: "M" → "male")**
- **Remoção de duplicatas: garantir unicidade**




### ** Gold Layer - Otimização para Analytics**

A camada Gold cria estruturas de dados otimizadas para análise e dashboards usando o padrão Star Schema:

#### Tabelas Dimensionais:
Dimensão: Customers

SQL
CREATE TABLE workspace.gold.customers AS
SELECT 
    id, key,
    firstname || ' ' || lastname AS full_name,
    gender,
    marital_status
FROM workspace.silver.cust_info

- Armazena atributos do cliente

Dimensão: Products

SQL
CREATE TABLE workspace.gold.products AS
SELECT 
    prd_id AS id, 
    prd_key AS key,
    prd_name AS name, 
    prd_cost AS cost,
    category, 
    subcategory
FROM workspace.silver.prd_info t1
JOIN workspace.silver.px_cat_g1v2 t2 
    ON LEFT(t1.prd_key, 5) = t2.id
    
- Reúne informações de produtos e categorias 
- Faz join inteligente usando os primeiros 5 caracteres da chave


#### Tabela de Fatos

Sales

SQL
CREATE TABLE workspace.gold.sales AS
SELECT 
    order_id, 
    product_id, 
    customer_id,
    order_date, 
    ship_date, 
    due_date,
    price
FROM workspace.silver.sales_details

- Armazena transações de vendas
- Contém chaves estrangeiras para tabelas dimensionais
- Otimizada para análises de vendas


##  Como Executar a Pipeline
Pré-requisitos:
- Databricks workspace com acesso a Volumes
- Python com PySpark
- Arquivos CSV de origem em /Volumes/workspace/bronze/source_systems

A execução toda da pipiline é automatizada pelas ferramentas dentro do Databricks como, Jobs e Pipelines, logo não sendo necessário executar cada notebook manualmente.

** Por: Andreas de Carvalho, Engenheiro de dados Jr **
