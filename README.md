---
license: mit
language:
  - es
pretty_name: mi-urba-corpus
task_categories:
  - text-classification
  - text-generation
tags:
  - synthetic-data
  - complaints
  - spanish
  - triage
  - property-management
size_categories:
  - 10K<n<100K
---

# mi-urba-corpus

Corpus sintéticos de quejas vecinales en español y sus extracciones de referencia, generados con
[mi-urba](https://github.com/oeg-upm/mi-urba) a partir de escenarios muestreados sobre una taxonomía cerrada del
dominio (comunidades de propietarios y su mantenimiento).

Cada registro lleva su anotación por construcción: el escenario se muestrea antes de redactar el texto, de modo que
la estructura que un clasificador debe recuperar se conoce a priori y no se etiqueta a posteriori.

## Carga

Desde el Hub (una vez publicado), con el corpus final como configuración por defecto:

```python
from datasets import load_dataset

ds = load_dataset("oeg-upm/mi-urba-corpus", "records")   # vista unida, una fila por queja
```

Desde un clon del repositorio, sin depender del Hub:

```python
from datasets import load_dataset

ds = load_dataset("parquet", data_files="dataset/final_10k/parquet/records.parquet")
```

## Configuraciones

| Configuración | Contenido | Filas |
|---|---|---|
| `records` (por defecto) | vista unida: `id`, `user_text`, `timestamp`, `model`, `persona`, `modifier`, `community`, `extractions` | 10.983 |
| `complaints` | la queja como entrada: `id`, `user_text`, `timestamp` | 10.983 |
| `ground_truth` | anotación de referencia: `complaint_id`, `extractions`, `image_descriptions` | 10.983 |
| `metadata` | contexto de generación: `complaint_id`, `persona`, `modifier`, `community`, `model`, `num_pics`, `image_prompts` | 10.983 |

Los tres ficheros por registro se entregan también como JSONL en [`dataset/final_10k/`](dataset/final_10k/), que es la fuente
canónica: los Parquet se generan a partir de ellos y son idénticos registro a registro.

## Datasets

| Carpeta | Contenido |
|---|---|
| [`dataset/final_10k/`](dataset/final_10k/) | 10.983 quejas con su extracción de referencia (ground truth), en JSONL y Parquet |
| [`dataset/iterations/`](dataset/iterations/) | tres corpus anteriores, sin auditar: `5k` (3.897), `20k` (20.000) y `50k` (50.000) |

Los generadores del corpus final son cuatro modelos locales, identificados registro a registro en el campo `model`:
gemma-3-12b-awq (2.356), qwen3.6-27b-awq (2.922), muse-glimmer-30b-awq (2.850) y gpt-oss-20b (2.855).

## Evaluación

Pendiente de publicación en este repositorio: protocolo y resultados de la auditoría de fidelidad del generador al
escenario (juez LLM, rúbrica de dos niveles, 10.983 registros completos).

## Licencia

MIT. Ver [`LICENSE`](LICENSE).
