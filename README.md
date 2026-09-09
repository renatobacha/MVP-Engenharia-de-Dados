# MVP-Engenharia-de-Dados

Repositório da Pos Graduação - Matéria Engenharia de Dados
# 🩺 MVP Diabetes Analytics Pipeline (Databricks & Unity Catalog)

Este repositório contém a documentação técnica e os scripts ETL do pipeline preditivo *end-to-end* desenvolvido na plataforma **Databricks** para análise de risco epidemiológico de *diabetes mellitus* em uma população de 50.000 pacientes.

---

## 📄 Sumário do Projeto

- [1. Contexto de Negócios e Perguntas](#1-contexto-de-negócios-e-perguntas)
- [2. Carga dos Dados](#2-carga-dos-dados)
- [3. Modelagem e Catálogo de Dados](#3-modelagem-e-catálogo-de-dados)
- [4. Pipeline de Dados ETL](#4-pipeline-de-dados-etl)
- [5. Qualidade de Dados](#5-qualidade-de-dados)
- [6. Análise de Dados](#6-análise-de-dados)
- [7. Autoavaliação](#7-autoavaliação)

---

## 1. Contexto de Negócios e Perguntas

### 🎯 Problema de Negócio e Contexto
O *diabetes mellitus* representa um grave desafio para a gestão de saúde pública e suplementar. Por se tratar de uma condição silenciosa, o acompanhamento tardio gera complicações graves e internações de alto custo. Este projeto visa transformar registros clínicos brutos em inteligência preditiva para identificar perfis de risco e suportar decisões médicas preventivas.

### ❓ Perguntas de Negócio
1. Qual a taxa de prevalência de diabetes por faixa etária?
2. Como a combinação entre IMC elevado e alteração glicêmica impacta o diagnóstico?
3. Qual o peso do histórico familiar associado ao nível de atividade física no desenvolvimento da doença?
4. Quais as médias de glicemia e IMC que diferenciam pacientes diagnosticados dos não diagnosticados?

### 📂 Estrutura e Licença dos Dados Brutos
- **Dataset:** *Diabetes Risk Prediction Dataset* (50.000 registros populacionais)
- **Estrutura Bruta (Colunas):** `Age`, `BMI`, `FastingBloodSugar`, `FamilyHistoryDiabetes`, `PhysicalActivityLevel`, `Diabetes` (Target).
- **Licença dos Dados:** Dados de domínio público/sintéticos disponibilizados para fins acadêmicos e educacionais (Open Database License / CC0).

---

## 2. Carga dos Dados

A carga inicial foi realizada via upload manual do arquivo `.csv` bruto para o ambiente de nuvem do Databricks através da funcionalidade de **Unity Catalog Volumes**.

- **Caminho de Destino no Volume:** `/Volumes/mvp/staging/diabetes/diabetes_risk_prediction_dataset.csv`.
- **Script de Ingestão:** Ver o notebook [`01_bronze_ingestion.py`](./scripts/01_bronze_ingestion.py) no repositório.

---

## 3. Modelagem e Catálogo de Dados

Foi implementado um modelo dimensional **Snowflake Schema** sob a governança do Unity Catalog no esquema `mvp.staging`.

### 📖 Catálogo de Dados (Transcrito)

- **`mvp.staging.fato_diabetes_snowflake` (Tabela Fato):**
  - `id_paciente` (BIGINT) [FK]: Identificador do paciente.
  - `id_glicemia` (BIGINT) [FK]: Chave da dimensão de glicemia.
  - `bmi` (DOUBLE): IMC contínuo.
  - `fasting_blood_sugar` (DOUBLE): Glicemia em jejum (mg/dL).
  - `flag_diabetes` (INT): Diagnóstico ($1 = \text{Sim}$, $0 = \text{Não}$).

- **`mvp.staging.dim_paciente` (Dimensão):**
  - `id_paciente` (BIGINT) [PK]: Chave surrogate do paciente.
  - `age` (INT): Idade em anos.
  - `faixa_etaria` (STRING): Agrupamento etário (`Jovem (<30)`, `Adulto (30-59)`, `Idoso (60+)`).
  - `family_history_diabetes` (INT): Histórico familiar ($1/0$).
  - `physical_activity_level` (STRING): Nível de atividade (`low`, `moderate`, `high`).
  - `id_categoria_bmi` (BIGINT) [FK]: Chave da sub-dimensão de IMC.

- **`mvp.staging.dim_categoria_imc` (Sub-Dimensão Snowflake):**
  - `id_categoria_bmi` (BIGINT) [PK]: Chave da faixa de IMC.
  - `descricao_categoria` (STRING): `1. Normal`, `2. Sobrepeso`, `3. Obesidade`.
  - `classificacao_risco_metabolico` (STRING): Classificação de risco epidemiológico.

- **`mvp.staging.dim_glicemia` (Dimensão):**
  - `id_glicemia` (BIGINT) [PK]: Chave do grupo glicêmico.
  - `faixa_glicemica` (STRING): `1. Normal`, `2. Alterada`, `3. Elevada`.

*(Adicione aqui os Screenshots da aba 'Data' e 'Catalog Explorer' do Databricks evidenciando a estrutura criada)*

---

## 4. Pipeline de Dados ETL

O pipeline foi organizado de forma modularizada sob a **Arquitetura Medallion** em notebooks PySpark e SQL separados para garantir governança, rastreabilidade e facilidade de manutenção:

1. **`01_bronze_ingestion.py`:** Leitura da fonte CSV bruta no Volume, adição de metadados de auditoria (`_ingestion_datetime`, `_source_file`) e gravação na tabela Delta `bronze_diabetes_raw`.
2. **`02_silver_cleaning.py`:** Padronização dos nomes de colunas (*lowercase*), casting explícito de tipos, binarização de categóricas e remoção de duplicatas/nulos, gerando a tabela `silver_diabetes_clean`.
3. **`03_gold_dimensional.py`:** Criação de colunas derivadas (faixas etárias, classificações clínicas) na `gold_fato_diabetes`.
4. **`04_snowflake_modeling.sql`:** Script SQL DDL construindo o *Snowflake Schema* com chaves surrogate e relacionamentos.

### 🔗 Links para Scripts no Repositório
- Notebooks PySpark/SQL disponíveis na pasta [`/scripts`](./scripts) deste repositório.

*(Adicione aqui os Screenshots comprovando a persistência das tabelas no ambiente Delta/Databricks)*

---

## 5. Qualidade de Dados

Durante a auditoria da camada Bronze, os seguintes problemas foram detectados e corrigidos no ETL Silver:

| Problema Detectado | Causa | Solução Aplicada no Pipeline |
| :--- | :--- | :--- |
| **Nomes de Colunas Despadronizados** | Espaços e caracteres maiúsculos | Conversão automatizada para *lowercase* e remoção de espaços nas extremidades. |
| **Tipagem Inadequada** | Leitura genérica de strings no CSV | *Casting* explícito para `INT` (`age`) e `DOUBLE` (`bmi`, `fasting_blood_sugar`). |
| **Valores Categóricos Inconsistentes** | Strings heterogêneas (`Yes`/`true`/`1`) | Mapeamento lógico padronizando para binário `1` e `0`. |
| **Registros Duplicados / Ausentes** | Inconsistência na coleta | Filtro `.dropDuplicates()` e remoção de nulos em colunas vitais (`age`, `bmi`). |

---

## 6. Análise de Dados

Análise consolidada baseada nas consultas do modelo dimensional Snowflake:

- **Prevalência por Faixa Etária:** A taxa de incidência cresce linearmente com o avanço da idade, registrando pico crítico na população idosa ($60+$ anos).
- **Matriz IMC x Glicemia:** A combinação de **Obesidade ($\text{IMC} \ge 30$)** com **Glicemia Elevada ($>125\text{ mg/dL}$)** concentra o maior volume de casos positivos da base.
- **Genética x Sedentarismo:** O histórico familiar positivo associado a baixos níveis de atividade física eleva expressivamente o risco, enquanto o exercício moderado/intenso demonstrou efeito atenuante.
- **Perfis Médios:** Pacientes diagnosticados apresentaram glicemia média acima de $100\text{ mg/dL}$ e IMC médio $\ge 27\text{ kg/m}^2$ (faixa de sobrepeso/obesidade).

---

## 7. Autoavaliação

### 🏁 Atingimento dos Objetivos
O projeto atingiu com êxito todos os objetivos propostos. Foi possível construir um pipeline funcional sob a arquitetura Medallion e estruturar um modelo dimensional pronto para responder a perguntas estratégicas de negócio.

### 💡 Dificuldades Encontradas
- **Modelagem de Chaves no Databricks:** Ajustar os relacionamentos `JOIN` no SQL para evitar produtos cartesianos e garantir que o modelo Snowflake não gerasse registros nulos.
- **Governança no Unity Catalog:** Configurar e gerenciar o escopo do Catálogo e Esquema para garantir a correta persistência dos Volumes e Tabelas Delta.

### 🔮 Trabalhos Futuros
1. **Orquestração:** Implementar a execução agendada das rotinas ETL através do *Databricks Workflows*.
2. **Camada de Machine Learning:** Treinar um modelo preditivo (ex: XGBoost via MLflow) na camada Gold para fornecer *scores* de risco preditivo no momento da ingestão.
