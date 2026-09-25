# ADR-0004: Fuentes de datos revisadas

- **Estado:** Aceptada
- **Fecha:** 2026-09-25
- **Sustituye a:** [ADR-0002](0002-fuentes-de-datos.md)

## Contexto
El ADR-0002 eligió TML-Database como fuente principal porque "se actualizaba a diario".
Al preparar la exploración de datos (Bloque 2) se comprobó que **esa premisa ya no es cierta**, y
se evaluaron las tres fuentes con datos en [`notebooks/02_eda.ipynb`](../../notebooks/02_eda.ipynb).

## Evidencias (2026-09-25)
| | Archivo Sackmann | TML-Database | tennis-data.co.uk |
|---|---|---|---|
| Último dato | 25-may-2026 (Roland Garros 2026, completo) | 17-ene-2026 | **13-sep-2026** |
| Actualización | Congelado (junio 2026) | **Parado** (último commit 27-ene-2026) | Viva |
| Calidad de ficheros | ✅ Sin errores | ❌ Última fila de `2025.csv` truncada; `ATP_Database.csv` con codificación mixta | ⚠️ Fechas serie Excel, cuotas `'-'` |
| IDs de jugador | Numéricos, coherentes en partidos/jugadores/rankings | Alfanuméricos (ATP), **no cruzan** con Sackmann | No tiene |
| Extras | Challengers, qualys, rankings semanales, WTA | — | Cuotas de apuestas |
| Verificación con hechos reales | ✅ H2H Djokovic–Nadal 31–29, 24 Slams, racha 81 de Nadal | — | — |

En el periodo común (enero 2026), Sackmann y TML tienen **exactamente los mismos partidos**:
TML no aporta nada que el archivo no tenga.

## Decisión
1. **Archivo Sackmann = fuente principal histórica** (partidos 1968 → RG 2026, challengers,
   jugadores, rankings).
2. **tennis-data.co.uk = fuente viva e incremental**:
   - Cuotas para evaluar el modelo (Bloque 12).
   - Resultados posteriores al fin del archivo (~980 partidos, RG 2026 → sep 2026) para mantener
     el **Elo al día**, tras resolver los nombres a `player_id` (Bloque 10; una regla simple ya
     cruza el 96,7 %).
3. **TML-Database: descartada.**

## Consecuencias
- ✅ La ingesta incremental sigue teniendo sentido: el fichero del año en curso de tennis-data
  cambia cada semana.
- ✅ Una sola familia de IDs (Sackmann) en todo el modelo.
- ⚠️ Los partidos posteriores a junio 2026 **no tienen estadísticas de saque**: cuentan para el Elo
  y la forma, pero las features de saque de esos jugadores se congelan en junio 2026.
- ⚠️ Jugadores que debuten después de junio 2026 no existen en `atp_players` → se registrarán
  como "no resueltos" en una tabla de auditoría.
- ⚠️ Pinnacle desaparece en 2026 → la referencia principal del mercado será la **media de casas**.
- 🔁 Si el archivo o tennis-data desaparecieran, el diseño (landing inmutable + bronze) permite
  seguir trabajando con lo ya ingerido.

## Lección
Una fuente de datos externa puede degradarse sin avisar. Por eso: verificar frescura y calidad
**antes** de construir, y tener tests de frescura en el pipeline (se añadirán en dbt: `source freshness`).
