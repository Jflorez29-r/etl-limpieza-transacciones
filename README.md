# ETL de limpieza y validación de transacciones

Pipeline en Python que lee un CSV de transacciones con datos sucios, valida cinco campos, normaliza los registros válidos y guarda aparte los rechazados con el motivo del rechazo.

![CI](https://github.com/Jflorez29-r/etl-limpieza-transacciones/actions/workflows/ci.yml/badge.svg)

## Qué hace

```text
datos.csv ──► lectura fila por fila (generador)
                   │
                   ├─ validación de 5 campos ──► rejects.csv (fila + motivo)
                   │
                   └─ normalización ───────────► clean.csv
```

| Campo | Validación | Normalización |
|---|---|---|
| `country` | país reconocido (acepta variantes como `COL`, `ch1le`) | código ISO alpha-2 (`Colombia` → `CO`) |
| `category` | categoría soportada | minúscula estándar |
| `amount` | formato de monto válido | número sin símbolos (`$8596.9` → `8596.9`) |
| `quantity` | cantidad válida | entero (`dos` → `2`) |
| `transaction_date` | formato de fecha reconocible | `YYYY-MM-DD` |

Los valores faltantes (`""`, `N/A`, `null`, `none`) se tratan como ausentes y la fila se rechaza.

## Resultado con la muestra incluida

`data/datos.csv` tiene 100 filas, con errores puestos a propósito (montos vacíos, fechas en varios formatos, países mal escritos, correos inválidos):

- 13 filas pasan a `clean.csv`
- 87 filas pasan a `rejects.csv` con su motivo

## Cómo ejecutarlo

Requiere Python 3.12.

```bash
python src/pipeline.py          # genera data/clean.csv y data/rejects.csv
python scripts/generate_mock_data.py 50000   # opcional: datos de prueba más grandes
pip install -r requirements.txt
pytest                          # pruebas unitarias de validaciones y transformaciones
```

## Estructura

- `src/pipeline.py`: orquesta la lectura, validación, transformación y escritura.
- `src/validadores.py`: reglas de validación por campo.
- `src/transformaciones.py`: normalización de los valores válidos.
- `src/explorador.py`: lectura del CSV con un generador (`yield`), sin cargar todo el archivo en memoria.
- `scripts/generate_mock_data.py`: genera datos sucios de prueba.
- `tests/test_pipeline.py`: pruebas con `pytest`.

## Qué aprendí / qué mejoraría

- Aprendí a separar validar de transformar, y a no perder los registros malos: se guardan con su motivo para poder revisarlos.
- Mejoraría: validar también `email` y `phone`, y leer el CSV con `pandas` para comparar.
