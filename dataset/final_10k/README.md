# final_10k — corpus de quejas vecinales en español

10.983 registros de quejas de vecinos de una comunidad de propietarios.

## Ficheros

| Fichero | Contenido |
|---|---|
| `complaints.jsonl` | la queja: `id`, `user_text`, `timestamp` |
| `ground_truth.jsonl` | la extracción de referencia, por `complaint_id`: lista `extractions` con `category`, `affected_elements`, `location`, `risks` y `urgency` |
| `metadata.jsonl` | `complaint_id`, `persona`, `modifier`, `community`, `num_pics`, `image_prompts`, `model` |

Los tres ficheros van alineados por línea y por id. No hay imágenes: `user_pics` ya no forma parte del registro y `num_pics`/`image_prompts`/`image_descriptions` quedan vacíos, reservados para una futura extensión multimodal.

## Formatos

| Formato | Ficheros |
|---|---|
| JSONL (canónico) | `complaints.jsonl`, `ground_truth.jsonl`, `metadata.jsonl` |
| Parquet (HuggingFace) | `parquet/complaints.parquet`, `parquet/ground_truth.parquet`, `parquet/metadata.parquet`, `parquet/records.parquet` |

Los Parquet se generan a partir de los JSONL y son idénticos registro a registro (mismos campos, mismo orden de
registros, tipos anidados preservados: `community` como struct y `extractions` como lista de structs).

`parquet/records.parquet` es una vista unida por identificador, una fila por queja, con `id`, `user_text`,
`timestamp`, `model`, `persona`, `modifier`, `community` y `extractions`: es la configuración que carga por defecto
`load_dataset` y la más cómoda para entrenar o evaluar un clasificador.

```python
from datasets import load_dataset

ds = load_dataset("parquet", data_files="parquet/records.parquet")
```

El generador de cada registro se identifica en `metadata.jsonl`, no en `complaints.jsonl`: el texto de la queja es la entrada del corpus y no debe arrastrar la procedencia de generación.

## Ejemplo

Registro `Q0008`. Lo relevante es el texto: es la **entrada** del corpus. Lo demás es la referencia que debería recuperarse (`ground_truth`) y el contexto de generación (`metadata`).

> Hola, ¿qué tal? Quería avisaros porque aquí en la zona exterior, concretamente en el patio de luces, tenemos cortes frecuentes en el suministro eléctrico. Es importante y me preocupa, deberían revisarlo pronto. Además, se ha afectado la instalación eléctrica, concretamente el cuadro eléctrico está haciendo ruidos raros.

La tarea asociada: dado ese `user_text`, producir las `extractions` de `ground_truth.jsonl`.

### La entrada: `complaints.jsonl`

| Campo | Valor |
|---|---|
| **`user_text`** | **el texto de la queja; es lo que se usa como entrada** |
| `id` | `Q0008`, mismo valor que `complaint_id` en los otros dos ficheros |
| `timestamp` | fecha sintética en formato ISO 8601 |

```json
{"id": "Q0008", "user_text": "Hola, ¿qué tal? Quería avisaros porque aquí en la zona exterior, concretamente en el patio de luces, tenemos cortes frecuentes en el suministro eléctrico. Es importante y me preocupa, deberían revisarlo pronto. Además, se ha afectado la instalación eléctrica, concretamente el cuadro eléctrico está haciendo ruidos raros.", "timestamp": "2024-03-15T12:00:15+0000"}
```

### La referencia: `ground_truth.jsonl`

Es la extracción esperada: lo que el texto *dice* y un extractor debería recuperar.

| Campo | Valor | Significado |
|---|---|---|
| `complaint_id` | `Q0008` | enlaza con la queja |
| `extractions` | 1 elemento | un problema distinto por elemento |
| `extractions[].category` | `electricidad` / `cortes_suministro` | categoría y subcategoría de la taxonomía |
| `extractions[].affected_elements` | `instalación_eléctrica` / `cuadro` | objeto dañado (principal y subelemento) |
| `extractions[].location` | `exterior` / `patio de luces` | zona de la taxonomía y detalle propio de esa zona (vocabulario cerrado) |
| `extractions[].risks` | `[ninguno]` | riesgos asociados; `ninguno` o una lista de riesgos |
| `extractions[].urgency` | `alta` | una de: `inmediata`, `urgente`, `alta`, `media`, `baja`, `planificable` |
| `image_descriptions` | `[]` | reservado para descripciones de fotos; vacío |

```json
{"complaint_id": "Q0008", "extractions": [{"category": {"principal": "electricidad", "subcategory": "cortes_suministro"}, "affected_elements": [{"principal": "instalación_eléctrica", "subcategory": "cuadro"}], "location": {"zone": "exterior", "detail": "patio de luces"}, "risks": ["ninguno"], "urgency": "alta"}], "image_descriptions": []}
```

### El contexto: `metadata.jsonl`

Contexto de generación, útil para analizar el corpus y no necesario para entrenar.

| Campo | Valor | Significado |
|---|---|---|
| `complaint_id` | `Q0008` | enlaza con la queja |
| `persona` | `vecino_informal_amable` | perfil de escritura pedido al generador |
| `modifier` | `null` | variante de estilo aplicada, o `null` |
| `community` | ver línea | comunidad de vecinos simulada: id, nombre, tipo, viviendas, año, ascensor, piscina, garaje |
| `model` | `gemma-3-12b-awq` | **modelo generador del texto** |
| `num_pics` | `0` | número de fotos; siempre 0 |
| `image_prompts` | `[]` | reservado para prompts de imagen; vacío |

```json
{"complaint_id": "Q0008", "persona": "vecino_informal_amable", "modifier": null, "community": {"id": "C005", "name": "Adosados La Dehesa", "type": "adosados", "num_viviendas": 32, "year": 2015, "has_elevator": false, "has_pool": true, "has_garage": true}, "num_pics": 0, "image_prompts": [], "model": "gemma-3-12b-awq"}
```

## Cifras

| | |
|---|---|
| Registros | 10.983, de 13.398 intentos |
| Extracciones por queja | 1 → 9.543, 2 → 1.085, 3 → 355 |
| Generadores | gemma-3-12b-awq 2.356, qwen3.6-27b-awq 2.922, muse-glimmer-30b-awq 2.850, gpt-oss-20b 2.855 |

Cada generador ocupa un bloque contiguo de identificadores (`gemma` hasta `Q3000`, `qwen3.6` hasta `Q6000`, `muse-glimmer` hasta `Q9000`, `gpt-oss` en adelante), pero el campo `model` de `metadata.jsonl` es la fuente de verdad: los bloques no están completos porque los registros rechazados por el juez no se publicaron.

## Cómo se generó

Con [mi-urba](https://github.com/oeg-upm/mi-urba). El escenario se muestrea de una taxonomía cerrada y el texto lo escribe uno de cuatro LLM locales. Cada registro pasa por un juez LLM (gemma4-12b) que lo rechaza y pide reescritura si no respeta el escenario. Los timestamps son sintéticos (2024–2026).
