# 10K — corpus de quejas vecinales en español

11.276 registros de quejas de vecinos de una comunidad de propietarios.

## Ficheros

| Fichero | Contenido |
|---|---|
| `complaints.jsonl` | la queja: `id`, `user_text`, `user_pics`, `timestamp` |
| `ground_truth.jsonl` | la extracción de referencia, por `complaint_id`: lista `extractions` con `category`, `affected_elements`, `location`, `risks` y `urgency` |
| `metadata.jsonl` | `complaint_id`, `community`, `persona`, `modifier`, `num_pics`, `image_prompts` |

Los tres ficheros van alineados por línea y por id. No hay imágenes: `user_pics` e `image_descriptions` van vacíos.

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
