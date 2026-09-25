# ADR-0001: Databricks + dbt como plataforma (frente a Snowflake + dbt)

- **Estado:** Aceptada
- **Fecha:** 2026-09-25

## Contexto
Necesitamos una plataforma donde almacenar y procesar los datos (lakehouse o data warehouse).
Las dos opciones más demandadas en el mercado son Databricks y Snowflake. dbt funciona con
ambas, así que la decisión no es "dbt sí o no", sino **dónde viven y se procesan los datos**.

Requisitos del proyecto:
- Coste 0 € y que siga funcionando meses después (un recruiter puede mirarlo en cualquier momento).
- Procesamiento con Spark (Monte Carlo distribuido y, en fase 2, streaming).
- Machine Learning con seguimiento de experimentos.

## Opciones consideradas
| | Databricks Free Edition | Snowflake (trial) |
|---|---|---|
| Coste | Gratis, sin caducidad | 30 días / ~400 $ de créditos |
| Spark | Nativo | No incluido |
| Formato de tablas | Delta Lake | Propietario (admite Iceberg) |
| ML | MLflow + Model Registry integrados | Snowpark ML |
| Mercado en España | Muy alto (sobre todo con Azure) | Alto |

## Decisión
**Databricks Free Edition + dbt (`dbt-databricks`).**

## Consecuencias
- ✅ Una sola plataforma cubre Spark, Delta Lake, SQL, dbt y MLflow.
- ✅ Sin caducidad: el proyecto sigue vivo.
- ⚠️ Limitaciones de Free Edition: solo serverless, un SQL warehouse 2X-Small, salida a internet
  restringida, no se pueden crear catálogos por API (usamos el catálogo `workspace`).
- ⚠️ Uso no comercial (compatible con un portfolio).
- 🔁 Si hiciera falta Snowflake, los modelos dbt se migrarían con cambios menores.
