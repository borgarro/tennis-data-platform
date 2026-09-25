# ADR-0003: Orquestación con Airflow local (frente a Databricks Workflows)

- **Estado:** Aceptada
- **Fecha:** 2026-09-25

## Contexto
El pipeline tiene pasos fuera de Databricks (descargar fuentes, convertir Excel, exportar el
snapshot para Streamlit) y pasos dentro (jobs Spark, dbt, entrenamiento). Necesitamos
coordinarlos: orden, reintentos, calendario y backfills.

## Opciones consideradas
1. **Databricks Workflows (Lakeflow Jobs):** nativo, sin infraestructura extra. Pero solo ve lo
   que ocurre dentro de Databricks.
2. **Apache Airflow en Docker local:** el orquestador más demandado en ofertas de empleo;
   coordina sistemas distintos (local + Databricks + dbt).

## Decisión
**Airflow 3 en Docker**, que lanza los jobs de Databricks con el provider oficial y los modelos
dbt con Astronomer Cosmos.

Por qué la **descarga se hace fuera de Databricks** aunque la prueba del Bloque 0 mostró que
Databricks sí llega a GitHub:
- tennis-data.co.uk necesita un tratamiento especial (user-agent, conversión de Excel).
- La lógica de ingesta se testea en local con pytest sin gastar cómputo ni cuota.
- Todo el "E" del ELT queda en un único sitio.

## Consecuencias
- ✅ Demuestra Airflow y la integración entre sistemas.
- ⚠️ Airflow solo corre cuando el PC está encendido. En producción se usaría un Airflow
  gestionado (MWAA, Astronomer, Cloud Composer).
- ⚠️ Consume RAM (~4–6 GB en Docker).
