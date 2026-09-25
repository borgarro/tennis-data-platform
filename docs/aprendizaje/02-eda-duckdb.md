# Bloque 2 — Exploración de datos (EDA) con DuckDB

## Qué hemos hecho
Antes de construir el pipeline hemos **interrogado a los datos** con SQL en local:
[`notebooks/02_eda.ipynb`](../../notebooks/02_eda.ipynb). Resultado: 12 "trampas" documentadas, un
diccionario de datos y un **cambio de fuente principal** (ADR-0004) porque TML había dejado de
actualizarse.

> **Idea clave para la entrevista:** "Nunca construyo sobre una fuente sin perfilarla antes.
> En este proyecto el EDA me hizo cambiar la fuente principal: la que había elegido llevaba 8 meses
> sin actualizarse y tenía ficheros corruptos. Lo detecté antes de escribir una línea de pipeline."

## Cómo usar el notebook
1. Abre `notebooks/02_eda.ipynb` en VS Code.
2. Arriba a la derecha: **Select Kernel → Python Environments → `.venv`** (Python 3.12 del proyecto).
3. Ejecuta celda a celda con `Shift+Enter` leyendo cada explicación.
   La primera celda descarga los datos (~58 MB) la primera vez; después usa la caché.

## Conceptos aprendidos

### DuckDB
- Base de datos **analítica (OLAP)**, **por columnas**, que corre **dentro de Python** (sin servidor).
- Lee ficheros directamente: `FROM read_csv('ruta/*.csv')`, `read_xlsx(...)` (extensión `excel`).
- **Globs**: `atp_matches_[0-9][0-9][0-9][0-9].csv` lee 59 ficheros de una vez.
- `union_by_name=true`: une ficheros por nombre de columna, no por posición.
- **`DESCRIBE`** (esquema) y **`SUMMARIZE`** (perfilado automático: nulos, mín/máx, distintos).
- **`TRY_CAST`**: convierte y devuelve `NULL` si no puede, en vez de fallar.
- **`store_rejects=true`**: aparta las filas inválidas en `reject_errors` → patrón de
  **cuarentena / dead-letter**.
- **Macros** (`CREATE MACRO`): funciones SQL reutilizables.
- ⚠️ Escribe **siempre `AS`** en los alias (`weeks`, `years`, `start` rompen la consulta sin él).

### Perfilado de datos (data profiling)
Las preguntas que siempre hay que hacerse ante una fuente nueva:
1. **Grano**: ¿qué representa una fila?
2. **Clave**: ¿qué columnas la identifican de forma única? ¿Hay duplicados?
3. **Tipos y formatos**: fechas, números guardados como texto, codificación.
4. **Dominios**: valores posibles de cada categoría, valores "sucios".
5. **Nulos**: cuántos, dónde y **por qué** (por época, por tipo de torneo...).
6. **Integridad referencial**: ¿existen todos los jugadores de los partidos en la tabla de jugadores?
7. **Frescura**: ¿hasta cuándo llegan los datos? ¿Se siguen actualizando?
8. **Coherencia con la realidad**: ¿reproducen hechos conocidos?

### Gaps and islands
Técnica para encontrar **rachas** (secuencias consecutivas). Con
`sum(es_derrota) OVER (ORDER BY fecha)` se crea un contador que solo sube en las derrotas: todas las
victorias entre dos derrotas comparten valor → forman una "isla". Agrupando por ese valor se cuentan
las rachas. La usamos para la racha de 81 victorias de Nadal en tierra.

### Entity resolution
Unir registros de dos fuentes **sin ID común** (`"Nadal R."` vs `Rafael Nadal`, id 104745):
clave normalizada (minúsculas, sin acentos, solo letras) + ventana de fechas. Una regla simple ya
cruza el **96,7 %**; los fallos (apellidos compuestos, orden de nombres asiático) se tratarán en el
Bloque 10.

## Las 12 trampas encontradas
Resumen en la sección 11 del notebook. Las 3 más importantes:
1. **`tourney_date` es el lunes de inicio del torneo**, no el día del partido → riesgo de
   *data leakage*. Solución: ordenar por (fecha, torneo, ronda, nº partido).
2. **El ganador siempre va en `winner_*`** → hay que simetrizar para el modelo.
3. **Fuentes que se degradan** → TML parada desde enero y con ficheros rotos.

## Errores cometidos (y aprendizajes)
- `store_rejects` no se puede combinar con `union_by_name` en DuckDB.
- `LIMIT` dentro de un `UNION` necesita paréntesis.
- Editar un fichero UTF-8 con `Get-Content`/`Set-Content` de **PowerShell 5.1** estropeó las tildes
  ("invÃ¡lidas"): PowerShell lo leyó como ANSI. Mismo tipo de problema que la codificación mixta de
  TML. **Lección:** especifica siempre la codificación al leer y escribir ficheros.

## 💼 Preguntas de entrevista
- **"¿Qué haces cuando recibes una fuente de datos nueva?"** → Perfilarla: grano, clave, tipos,
  dominios, nulos, integridad, frescura y comprobaciones con hechos conocidos. Documentarlo en un
  diccionario de datos.
- **"¿Qué es el grano de una tabla?"** → Qué representa exactamente una fila. Mezclar granos en un
  JOIN multiplica filas sin avisar.
- **"¿Cómo gestionas filas corruptas en la ingesta?"** → No se tira todo el fichero ni se ignoran en
  silencio: se apartan a una zona de cuarentena (tabla de rechazos), se cuentan y se alerta si
  superan un umbral.
- **"Una fuente deja de actualizarse, ¿cómo lo detectas?"** → Tests de frescura (`dbt source
  freshness`): alerta si el dato más reciente tiene más de N días.
- **"¿OLTP vs OLAP?" / "¿Por qué el formato por columnas es más rápido en analítica?"** → Ver la
  explicación de DuckDB.

## ✏️ Ejercicios para ti
Añade celdas al final del notebook (o usa uno nuevo) y resuélvelos con SQL sobre la vista `matches`.
Luego me pasas tus consultas y las corregimos juntos.

1. **Rey del ace:** ¿qué jugador ha hecho más aces en una sola temporada (desde 1991)? Muestra
   jugador, año y aces.
   *Pista:* un jugador aparece como `winner_*` o como `loser_*` → necesitas juntar las dos
   perspectivas con `UNION ALL` antes de agrupar.
2. **Alcaraz por superficie:** porcentaje de victorias de Carlos Alcaraz en cada superficie
   (partidos jugados, ganados y %). Excluye los walkovers.
   *Pista:* `count(*) FILTER (WHERE ...)` o `sum(CASE WHEN ... THEN 1 ELSE 0 END)`.
3. **Sorpresas por superficie:** desde el año 2000, ¿en qué superficie gana más a menudo el jugador
   **peor clasificado** (ranking más alto)? Ignora los partidos donde falte algún ranking.
   *Pista:* cuidado con los `NULL` en `winner_rank`/`loser_rank`.

## Comandos útiles
```bash
uv run --with jupyterlab jupyter lab       # alternativa a VS Code para abrir el notebook
uv run python -c "import duckdb; print(duckdb.sql('SELECT 42'))"
```
