---
origen: Apuntes de clase (fuente de ejemplo escrita para el curso CodeCrypto)
fecha: 2026-11-01
---

# Apuntes: RAG (Retrieval-Augmented Generation)

RAG es la técnica de **recuperar** fragmentos de documentos relevantes para una pregunta y
**añadirlos al prompt** antes de que el LLM genere la respuesta. Se popularizó a partir del paper de
Lewis et al. (Facebook AI, 2020).

## Pipeline típico

1. **Ingesta**: se trocean los documentos en *chunks* (por ejemplo, 500–1.000 tokens con solapamiento).
2. **Embeddings**: cada chunk se convierte en un vector numérico con un modelo de embeddings.
3. **Almacenamiento**: los vectores se guardan en una base vectorial (pgvector, Chroma, Qdrant, LanceDB).
4. **Consulta**: la pregunta también se convierte en vector y se buscan los *k* chunks más cercanos
   (similitud coseno).
5. **Generación**: los chunks recuperados se pegan en el prompt y el LLM responde.

Mejoras habituales: **búsqueda híbrida** (combinar BM25, que busca palabras exactas, con vectores),
**re-ranking** con un segundo modelo, y añadir a cada chunk el título del documento al que pertenece.

## Limitaciones

- El conocimiento **no se acumula**: cada pregunta vuelve a buscar desde cero. Si la respuesta requiere
  conectar cinco documentos, hay que recuperar los cinco trozos correctos cada vez.
- Un chunk aislado pierde el contexto del documento.
- La base vectorial no es legible por humanos: es difícil auditar o corregir lo que "sabe".
- Requiere infraestructura y reindexar cuando cambian los documentos.

## Observación sobre agentes de código

Los agentes de código actuales (Claude Code, por ejemplo) apenas usan RAG vectorial sobre el código:
prefieren la **búsqueda agéntica** con herramientas como `grep` y `glob`, leyendo los ficheros reales.
Así la información nunca está desactualizada.

Ver también: GraphRAG (Microsoft, 2024), que construye un grafo de entidades y relaciones en lugar de
trabajar solo con chunks.
