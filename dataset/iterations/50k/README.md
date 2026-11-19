# 50k — corpus de quejas vecinales en español

50.000 registros de quejas de vecinos de comunidades de propietarios.

## Ficheros

| Fichero | Contenido |
|---|---|
| `complaints.jsonl` | la queja: `id`, `user_text`, `user_pics`, `timestamp` |
| `ground_truth.jsonl` | la extracción de referencia, por `complaint_id`: lista `extractions` con `category`, `affected_elements`, `location`, `risks` y `urgency` |
| `metadata.jsonl` | `complaint_id`, `community`, `persona`, `modifier`, `num_pics`, `image_prompts` |

Los tres ficheros van alineados por línea y por id. No hay imágenes: `user_pics` e `image_descriptions` van vacíos. El formato es el mismo que el de [`final_10k`](../../final_10k/).

## Cifras

| | |
|---|---|
| Registros | 50.000, 10.000 por cada uno de los cinco generadores |
| Extracciones por queja | 1 → 42.708, 2 → 5.467, 3 → 1.825 |
| Generadores | gemma-3-12b, gpt-oss-20b, muse-glimmer-30b, olmo-3.1-32b, qwen3.6-27b |

## Cómo se generó

Con [mi-urba](https://github.com/oeg-upm/mi-urba), antes de la versión que produjo `final_10k`. El escenario se muestrea de una taxonomía cerrada y el texto lo escribe uno de cinco LLM locales. Cada registro pasa por un juez LLM que lo rechaza y pide reescritura si no respeta el escenario.

## Problemas

- **Sin auditoría independiente**: no pasó por el panel de 6 votos, así que no hay medida de su calidad.
- **1.231 extracciones (2,1 %) con `ninguno` junto a un riesgo concreto**, del fallo de muestreo ya corregido en el generador.
- **Sin rastro del juez**: no hay `rejected.jsonl` ni `conversations.jsonl` por modelo, así que no se puede reconstruir el veredicto.
- **Los ids coinciden con los de `final_10k`**: los dos empiezan en `Q0001`.
- El texto lo escribe un modelo: puede contener fórmulas raras o detalles inverosímiles. No es lenguaje de usuarios reales.
