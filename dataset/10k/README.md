# 10K — corpus de quejas vecinales en español

11.276 registros de quejas de vecinos de una comunidad de propietarios.

## Ficheros

| Fichero | Contenido |
|---|---|
| `complaints.jsonl` | la queja: `id`, `user_text`, `user_pics`, `timestamp` |
| `ground_truth.jsonl` | la extracción de referencia, por `complaint_id`: lista `extractions` con `category`, `affected_elements`, `location`, `risks` y `urgency` |
| `metadata.jsonl` | `complaint_id`, `community`, `persona`, `modifier`, `num_pics`, `image_prompts` |

Los tres ficheros van alineados por línea y por id. No hay imágenes: `user_pics` e `image_descriptions` van vacíos.

## Ejemplo

Registro `Q0008`: una queja por cortes de suministro eléctrico en el patio de luces. Las tres líneas que siguen son el mismo registro en los tres ficheros.

### `complaints.jsonl`

| Campo | Valor |
|---|---|
| `id` | `Q0008`, mismo valor que `complaint_id` en los otros dos ficheros |
| `user_text` | el texto de la queja, tal como lo escribiría un vecino |
| `user_pics` | lista de ficheros de imagen; siempre vacía en este corpus |
| `timestamp` | fecha sintética en formato ISO 8601 |

```json
{"id": "Q0008", "user_text": "Hola, ¿qué tal? Quería avisaros porque aquí en la zona exterior, concretamente en el patio de luces, tenemos cortes frecuentes en el suministro eléctrico. Es importante y me preocupa, deberían revisarlo pronto. Además, se ha afectado la instalación eléctrica, concretamente el cuadro eléctrico está haciendo ruidos raros.", "user_pics": [], "timestamp": "2024-03-15T12:00:15+0000"}
```

### `ground_truth.jsonl`

Es la extracción de referencia: lo que el texto *dice* y el modelo debería recuperar.

| Campo | Valor | Significado |
|---|---|---|
| `complaint_id` | `Q0008` | enlaza con la queja |
| `extractions` | 1 elemento | un problema distinto por elemento |
| `extractions[].category` | `electricidad` / `cortes_suministro` | categoría y subcategoría de la taxonomía |
| `extractions[].affected_elements` | `instalación_eléctrica` / `cuadro` | objeto dañado (principal y subelemento) |
| `extractions[].location` | `exterior` / `patio de luces` | zona de la taxonomía y detalle libre |
| `extractions[].risks` | `[ninguno]` | riesgos asociados; `ninguno` o una lista de riesgos |
| `extractions[].urgency` | `alta` | una de: `inmediata`, `urgente`, `alta`, `media`, `baja`, `planificable` |
| `image_descriptions` | `[]` | reservado para descripciones de fotos; vacío |

```json
{"complaint_id": "Q0008", "extractions": [{"category": {"principal": "electricidad", "subcategory": "cortes_suministro"}, "affected_elements": [{"principal": "instalación_eléctrica", "subcategory": "cuadro"}], "location": {"zone": "exterior", "detail": "patio de luces"}, "risks": ["ninguno"], "urgency": "alta"}], "image_descriptions": []}
```

### `metadata.jsonl`

Contexto de generación, útil para análisis y no necesario para entrenar.

| Campo | Valor | Significado |
|---|---|---|
| `complaint_id` | `Q0008` | enlaza con la queja |
| `persona` | `vecino_informal_amable` | perfil de escritura pedido al generador |
| `modifier` | `null` | variante de estilo aplicada, o `null` |
| `community` | ver línea | comunidad de vecinos simulada: id, nombre, tipo, viviendas, año, ascensor, piscina, garaje |
| `num_pics` | `0` | número de fotos; siempre 0 |
| `image_prompts` | `[]` | reservado para prompts de imagen; vacío |

```json
{"complaint_id": "Q0008", "persona": "vecino_informal_amable", "modifier": null, "community": {"id": "C005", "name": "Adosados La Dehesa", "type": "adosados", "num_viviendas": 32, "year": 2015, "has_elevator": false, "has_pool": true, "has_garage": true}, "num_pics": 0, "image_prompts": []}
```

## Cifras

| | |
|---|---|
| Registros | 11.276, de 13.398 intentos |
| Extracciones por queja | 1 → 9.773, 2 → 1.123, 3 → 380 |
| Generadores | gemma-3-12b 2.427, qwen3.6-27b 2.997, gpt-oss-20b 2.932, muse-glimmer-30b 2.920 |

## Cómo se generó

Con [mi-urba](https://github.com/oeg-upm/mi-urba). El escenario se muestrea de una taxonomía cerrada y el texto lo escribe uno de cuatro LLM locales. Cada registro pasa por un juez LLM (gemma4-12b) que lo rechaza y pide reescritura si no respeta el escenario. Los timestamps son sintéticos (marzo de 2024).

## Limitaciones conocidas

- 289 registros (2,6 %) tienen un escenario contradictorio en `risks`: `ninguno` junto a un riesgo concreto. El muestreo ya está corregido, pero estos se generaron antes.
- El texto lo escribe un modelo: puede contener fórmulas raras o detalles inverosímiles. No es lenguaje de usuarios reales.
