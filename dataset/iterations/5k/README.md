# 5k — corpus de quejas vecinales en español

3.897 registros de quejas de vecinos de comunidades de propietarios.

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
| Registros | 3.897, repartidos entre cinco generadores: gemma-3-12b 817, gpt-oss-20b 976, muse-glimmer-30b 968, olmo-3.1-32b 174, qwen3.6-27b 962 |
| Extracciones por queja | 1 → 3.371, 2 → 393, 3 → 133 |
| Aceptación | 66 %: 5.926 intentos para 3.897 registros |

## Cómo se generó

Con [mi-urba](https://github.com/oeg-upm/mi-urba), antes de la versión que produjo `final_10k`. El escenario se muestrea de una taxonomía cerrada y el texto lo escribe uno de cinco LLM locales. Cada registro pasa por un juez LLM que lo rechaza y pide reescritura si no respeta el escenario.

## Problemas

- **Sin auditoría independiente**: no pasó por el panel de 6 votos, así que no hay medida de su calidad.
- **95 extracciones (2,1 % de las extracciones) con `ninguno` junto a un riesgo concreto**, del fallo de muestreo ya corregido en el generador.
- **Un generador arrastra el conjunto**: olmo-3.1-32b aporta solo 174 registros, con 1.303 rechazos sobre 1.477 intentos (12 % de aceptación).
- **Los ids coinciden con los de `final_10k`**: los dos empiezan en `Q0001`.
- El texto lo escribe un modelo: puede contener fórmulas raras o detalles inverosímiles. No es lenguaje de usuarios reales.
