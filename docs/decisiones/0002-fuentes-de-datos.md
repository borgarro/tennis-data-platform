# ADR-0002: Fuentes de datos

- **Estado:** Sustituida por [ADR-0004](0004-fuentes-de-datos-revisadas.md) (TML dejó de actualizarse en enero de 2026)
- **Fecha:** 2026-09-25

## Contexto
El dataset de referencia en analítica de tenis era `JeffSackmann/tennis_atp` en GitHub.
Al verificarlo (septiembre de 2026) el repositorio **devuelve 404: ya no existe**.
Necesitamos partidos ATP históricos (desde 1968), actualizados, y cuotas de apuestas para
evaluar el modelo.

## Opciones consideradas
1. **TML-Database** (`Tennismylife/TML-Database`): mismo esquema que Sackmann (+ columna
   `indoor`), 1968–2026, **actualizado a diario**, incluye `ongoing_tourneys.csv`. IDs de jugador
   = códigos ATP alfanuméricos. CC BY-NC-SA.
2. **tennis-sackmann-archive** (GitHub + Hugging Face): copia congelada de Sackmann (junio 2026),
   incluye WTA y punto a punto. No se actualiza. IDs numéricos.
3. APIs de pago (Sportradar, etc.): descartadas por coste.

## Decisión
- **Partidos y jugadores ATP:** TML-Database (fuente principal).
- **Cuotas:** tennis-data.co.uk (ATP 2000–2026).
- **Archivo congelado:** reserva y fases futuras (WTA, punto a punto).

## Consecuencias
- ✅ Al actualizarse a diario, la **ingesta incremental** tiene sentido real.
- ⚠️ Los IDs de TML no cruzan con el archivo de Sackmann → en fase 1 no se mezclan.
- ⚠️ tennis-data.co.uk no comparte IDs (nombres tipo "Nadal R.") → hará falta
  **entity resolution** (Bloque 10).
- ⚠️ tennis-data.co.uk tiene una ruta de descarga ofuscada y **bloquea el user-agent de Python**
  (403). La URL será configurable y el descargador enviará un user-agent de navegador.
- ⚠️ Licencia no comercial: los datos no se suben al repo; se atribuyen en README y app.
