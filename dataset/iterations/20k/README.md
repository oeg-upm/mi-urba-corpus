# 20k — corpus de quejas vecinales en español

20.000 registros de quejas de vecinos de comunidades de propietarios.

## Ficheros

| Fichero | Contenido |
|---|---|
| `complaints.jsonl` | la queja: `id`, `user_text`, `user_pics`, `timestamp` |
| `ground_truth.jsonl` | la extracción de referencia, por `complaint_id`: lista `extractions` con `category`, `affected_elements`, `location`, `risks` y `urgency` |
| `metadata.jsonl` | `complaint_id`, `community`, `persona`, `modifier`, `num_pics`, `image_prompts` |

Los tres ficheros van alineados por línea y por id. No hay imágenes: `user_pics` e `image_descriptions` van vacíos. Una diferencia con [`final_10k`](../../final_10k/): aquí `complaints.jsonl` conserva el campo `user_pics` (siempre vacío) y `metadata.jsonl` no incluye el campo `model`.

## Cifras

| | |
|---|---|
| Registros | 20.000, 10.000 por cada uno de los dos generadores |
| Extracciones por queja | 1 → 17.175, 2 → 2.148, 3 → 677 |
| Generadores | gemma-3-12b, gpt-oss-20b |

## Cómo se generó

Con [mi-urba](https://github.com/oeg-upm/mi-urba), antes de la versión que produjo `final_10k`. Es una variante de la campaña del `50k` de la que solo se conservaron dos generadores. El escenario se muestrea de una taxonomía cerrada y el texto lo escribe uno de dos LLM locales. Cada registro pasa por un juez LLM que lo rechaza y pide reescritura si no respeta el escenario.

## Problemas

- **Sin auditoría independiente**: no pasó por el panel de 6 votos, así que no hay medida de su calidad.
- **500 extracciones (2,1 % de las extracciones) con `ninguno` junto a un riesgo concreto**, del fallo de muestreo ya corregido en el generador.
- **Sin rastro del juez**: no hay `rejected.jsonl` ni `conversations.jsonl` por modelo, así que no se puede reconstruir el veredicto.
- **Los ids coinciden con los de `final_10k`**: los dos empiezan en `Q0001`.
- El texto lo escribe un modelo: puede contener fórmulas raras o detalles inverosímiles. No es lenguaje de usuarios reales.
