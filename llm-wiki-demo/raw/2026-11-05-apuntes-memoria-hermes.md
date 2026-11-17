---
origen: Apuntes de clase (fuente de ejemplo escrita para el curso CodeCrypto) basados en la documentación de Hermes Agent
fecha: 2026-11-05
url_referencia: https://hermes-agent.nousresearch.com/docs/user-guide/features/memory
---

# Apuntes: cómo recuerda Hermes Agent

Hermes Agent es un agente open source de **Nous Research**. Su memoria combina varias capas.

## Memoria declarativa en Markdown

- `~/.hermes/memories/MEMORY.md`: notas del propio agente (hechos del entorno, convenciones,
  peculiaridades de herramientas). Límite aproximado: **2.200 caracteres**.
- `~/.hermes/memories/USER.md`: lo que el agente sabe del usuario (estilo, preferencias).
  Límite aproximado: **1.375 caracteres**.
- Ambos se inyectan en el system prompt como un **snapshot congelado** al empezar la sesión: los cambios
  que haga el agente durante la sesión se verán en la siguiente.
- Los límites obligan al agente a **curar** la memoria (resumir, sustituir) en lugar de acumular.

## Memoria episódica: búsqueda de sesiones

Todas las conversaciones (CLI y mensajería) se guardan en SQLite (`~/.hermes/state.db`) con un índice de
texto completo **FTS5** (tabla `messages_fts`). Con la herramienta `session_search` el agente busca "¿hablamos de X la semana pasada?"
y recibe los mensajes reales de esas sesiones. No ocupa contexto hasta que se usa.

## Memoria procedimental: skills

Hermes puede crear **skills** (carpetas con instrucciones) a partir de tareas que ha resuelto, y reutilizarlas.

## Ficheros de contexto de proyecto

Al arrancar en una carpeta, Hermes carga **un único** fichero de contexto de proyecto, con esta prioridad:
`.hermes.md` → `AGENTS.md` → `CLAUDE.md` → `.cursorrules`. Además siempre carga `~/.hermes/SOUL.md`
(identidad/personalidad del agente).

## Proveedores externos

Hermes admite plugins de memoria externa (Honcho, Mem0, Hindsight, Supermemory, entre otros), que
funcionan **junto a** la memoria nativa, no en su lugar.
