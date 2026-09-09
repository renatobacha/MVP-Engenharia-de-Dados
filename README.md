# MVP-Engenharia-de-Dados
Repositório da Pos Graduação - Matéria Engenharia de Dados
# 🩺 MVP Diabetes Analytics Pipeline (Databricks & Unity Catalog)

Pipeline preditivo end-to-end para análise de risco de *diabetes mellitus* em uma base de 50.000 pacientes, construído com PySpark, SQL e Delta Lake no Databricks.

## 🏗️ Arquitetura
- **Ingestão & Auditoria (Bronze):** Ingestão em Delta Lake via Databricks Volumes.
- **Limpeza & Qualidade (Silver):** Normalização de esquemas, cating e remoção de duplicatas/nulos.
- **Modelagem Analítica (Gold):** Enriquecimento e construção de *Snowflake Schema* (`fato_diabetes`, `dim_paciente`, `dim_glicemia`, `dim_categoria_imc`).
- **Governança:** Estruturado sob o Unity Catalog (`mvp.staging`).

## 🛠️ Tecnologias
- **Linguagens:** PySpark, SQL
- **Plataforma:** Databricks, Delta Lake, Unity Catalog

## 📊 Principais Insights
- Prevalência crescente com a idade (pico em $60+$ anos).
- Forte correlação entre obesidade ($\text{IMC} \ge 30$) e glicemia alterada.
- Histórico familiar e sedentarismo como principais fatores de risco agregados.
