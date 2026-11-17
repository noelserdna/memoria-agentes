---
description: Ingiere una fuente de raw/ en la wiki (operación INGEST de AGENTS.md)
argument-hint: <ruta en raw/ o "pendientes">
---
Ejecuta la operación **INGEST** definida en AGENTS.md sobre: $ARGUMENTS

Si el argumento es "pendientes", detecta qué ficheros de `raw/` todavía no tienen página en
`wiki/fuentes/` (compáralo con `wiki/log.md`) e ingiérelos uno a uno en modo batch, sin preguntar.
