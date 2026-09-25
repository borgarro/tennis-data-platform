# Bloque 1 — Esqueleto del repositorio y herramientas de calidad

## Qué hemos construido
Un repositorio Python profesional **antes de escribir una sola línea de lógica de datos**.
En un equipo real esto se hace primero: si la base no tiene calidad, todo lo que se construya
encima la heredará.

## Las piezas y para qué sirve cada una

| Fichero | Qué es | Para qué |
|---|---|---|
| `pyproject.toml` | "Ficha" del proyecto Python | Nombre, versión, dependencias y configuración de herramientas en un único sitio (estándar PEP 621) |
| `uv.lock` | Versiones exactas congeladas | Que cualquiera que clone el repo instale **exactamente** lo mismo → reproducibilidad |
| `.python-version` | Versión de Python (3.12) | `uv` usa automáticamente esa versión; coincide con Databricks serverless |
| `src/tennis_platform/` | El paquete Python ("src layout") | Nuestro código se importa como librería y se empaquetará como *wheel* para Databricks |
| `tests/` | Tests con pytest | Comprobar automáticamente que el código hace lo que debe |
| `.gitignore` | Lo que git ignora | Secretos (`.env`), entorno virtual, datos descargados |
| `.gitattributes` | Reglas de finales de línea | Windows usa CRLF y Linux LF; guardamos todo en LF para evitar diffs "fantasma" |
| `.pre-commit-config.yaml` | Hooks antes de cada commit | Revisiones automáticas (ver abajo) |
| `.env.example` | Plantilla de variables | Dice qué variables hacen falta sin exponer valores |
| `LICENSE` | Licencia MIT del código | Sin licencia, legalmente nadie puede reutilizar tu código |
| `docs/decisiones/` | ADRs | El "porqué" de cada decisión de arquitectura |

## Conceptos clave

### Dependencias de desarrollo vs de producción
`ruff`, `pytest` y `pre-commit` están en el grupo `dev`: sirven para **desarrollar**, pero no
los necesita el código cuando corre en Databricks. Así el paquete final es más ligero.

### Linter y formateador: `ruff`
- **Linter** (`ruff check`): busca errores y malas prácticas sin ejecutar el código — imports
  sin usar, variables no definidas, patrones propensos a bugs.
- **Formateador** (`ruff format`): da a todo el código el mismo estilo (espacios, comillas,
  saltos de línea). Se acaban las discusiones de estilo en el equipo.
- `ruff` está escrito en Rust y sustituye a varias herramientas antiguas (flake8, isort, black).

### Tests: `pytest`
Un test es código que comprueba otro código. Ahora solo tenemos uno trivial (el paquete se
importa), pero en los próximos bloques testearemos el parseo de marcadores, el Elo, la
reconstrucción de cuadros…

### pre-commit: el portero de los commits
Cada vez que haces `git commit`, antes de crearlo se ejecutan automáticamente:
1. Limpieza: espacios al final de línea, salto de línea final.
2. Validación de YAML/TOML y bloqueo de ficheros grandes (>1 MB: los datos no van al repo).
3. `ruff` (lint + formato).
4. **gitleaks**: busca secretos (tokens, claves). Lo probamos con un token falso de Databricks
   y **bloqueó el commit** (regla `databricks-api-token`).

Si algún hook falla, el commit no se crea. Es "calidad por defecto": no depende de acordarse.

### ADR (Architecture Decision Record)
Documento corto por cada decisión importante: contexto, opciones, decisión y consecuencias.
Dentro de 6 meses (o en una entrevista) sabrás **por qué** hiciste cada cosa.

## Errores / aprendizajes del bloque
- El aviso `LF will be replaced by CRLF` apareció en el primer commit → motivó `.gitattributes`.
- `pre-commit run --all-files` solo revisa ficheros que git conoce: hay que hacer `git add` antes.

## 💼 Preguntas de entrevista
- **"¿Cómo aseguras la calidad del código en un equipo?"** → Linter y formateador comunes
  (ruff), tests (pytest), hooks de pre-commit para que se apliquen solos, y CI que lo repite
  en cada cambio (Bloque 16).
- **"¿Cómo garantizas que tu entorno es reproducible?"** → Versión de Python fijada,
  dependencias bloqueadas en un lockfile (`uv.lock`) y servicios en Docker.
- **"¿Cómo evitas subir secretos?"** → `.gitignore`, variables de entorno, gitleaks en
  pre-commit (y en CI), y rotación si algo se filtra.
- **"¿Por qué documentas decisiones?"** → ADRs: el código dice *qué* hace; el ADR dice *por qué*
  y qué alternativas se descartaron.

## Comandos útiles
```bash
uv sync                          # instala/actualiza el entorno según uv.lock
uv add <paquete>                 # añade dependencia de producción
uv add --dev <paquete>           # añade dependencia de desarrollo
uv run pytest                    # ejecuta los tests
uv run ruff check . --fix        # lint (y arregla lo automático)
uv run ruff format .             # formatea
uv run pre-commit run --all-files
```
