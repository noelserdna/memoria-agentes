# Memoria persistente en agentes de código
### De `CLAUDE.md` a LLM-Wiki

> Clase teórica. Duración orientativa: 60–75 min.
> Agentes de referencia: **Claude Code** (Anthropic) y **Hermes Agent** (Nous Research).

---

## 0. El problema: un LLM no recuerda nada

Un LLM es una función **sin estado**: `respuesta = modelo(contexto)`. Todo lo que "sabe" de tu proyecto
tiene que estar **dentro de la ventana de contexto** en esa llamada. Cuando cierras la sesión, se pierde.

Por eso, "memoria" en un agente siempre significa lo mismo: **decidir qué texto se mete en el contexto,
cuándo y quién lo escribe.** Todas las técnicas de esta clase son respuestas distintas a tres preguntas:

| Pregunta | Ejemplos de respuesta |
|---|---|
| **¿Dónde se guarda?** | Fichero Markdown, base de datos SQLite, base vectorial, grafo |
| **¿Quién lo escribe?** | El humano, el agente, ambos |
| **¿Cómo vuelve al contexto?** | Siempre (system prompt), bajo demanda (herramienta), por similitud (RAG) |

```
            ┌──────────── VENTANA DE CONTEXTO ────────────┐
 memoria ──▶│ system prompt │ ficheros │ historial │ tools │──▶ LLM
 externa    └─────────────────────────────────────────────┘
```

Tipos de memoria (vocabulario tomado de la psicología cognitiva, muy usado en la literatura de agentes):

- **Trabajo** → la ventana de contexto actual.
- **Semántica** → hechos: "este proyecto usa pnpm", "el usuario prefiere español".
- **Episódica** → lo que pasó: "el martes arreglamos el bug del login".
- **Procedimental** → cómo hacer cosas: skills, comandos, playbooks.

---

## 1. Generación 1 — Ficheros de instrucciones: `CLAUDE.md` / `AGENTS.md`

La forma más simple y, todavía hoy, la más útil: **un Markdown que el agente lee al arrancar.**

### Claude Code

| Ámbito | Ubicación | Para qué |
|---|---|---|
| Usuario | `~/.claude/CLAUDE.md` | Preferencias personales en todos los proyectos |
| Proyecto | `./CLAUDE.md` o `./.claude/CLAUDE.md` | Convenciones del equipo (se versiona en git) |
| Subcarpeta | `src/api/CLAUDE.md` | Se carga cuando el agente trabaja en esa carpeta |
| Reglas | `.claude/rules/*.md` | Reglas modulares, opcionalmente filtradas por ruta |

- `/init` genera un `CLAUDE.md` analizando el repo.
- `/memory` abre los ficheros de memoria para editarlos.
- **Imports**: dentro de un `CLAUDE.md` puedes escribir `@AGENTS.md` o `@docs/arquitectura.md` y se incluye ese fichero.
- **Auto memory**: Claude Code también puede escribir sus propias notas en
  `~/.claude/projects/<proyecto>/memory/` (un `MEMORY.md` índice + un fichero por recuerdo).
  Aquí ya es el **agente** quien escribe la memoria, no tú.

### `AGENTS.md` — el estándar abierto

`AGENTS.md` nació como formato neutral para que **cualquier agente** (Codex, Cursor, Hermes, Gemini CLI…)
lea las mismas instrucciones. Mismo concepto que `CLAUDE.md`, distinto nombre.

Las versiones recientes de Claude Code ya lo entienden: por defecto, **si el proyecto no tiene `CLAUDE.md`, carga
`AGENTS.md` en su lugar** (y se puede configurar para cargar ambos). Aun así, un `CLAUDE.md` con `@AGENTS.md`
funciona en cualquier versión.

### Hermes Agent

Hermes busca ficheros de contexto de proyecto con esta prioridad (lo hemos comprobado en
`agent/prompt_builder.py` del propio Hermes):

```
.hermes.md / HERMES.md  →  AGENTS.md (cadena desde la raíz git hasta cwd)  →  CLAUDE.md  →  .cursorrules
```

⚠️ **Solo se carga UNO: gana el primero que encuentra.** Si tienes `AGENTS.md` y `CLAUDE.md`, Hermes ignora
`CLAUDE.md`. Además, `~/.hermes/SOUL.md` (la "personalidad" del agente) se carga siempre.

> 💡 **Truco para compartir memoria entre agentes**: pon el contenido en `AGENTS.md` y crea un
> `CLAUDE.md` de una sola línea: `@AGENTS.md`. Claude Code lo importa; Hermes lee `AGENTS.md` directamente.
> Es lo que haremos en la práctica.

### Pros y contras

✅ Trivial, transparente, versionable con git, sin infraestructura.
❌ Ocupa contexto **siempre** → tiene que ser corto. No escala a "todo lo que sé de este dominio".
❌ Es estático: si nadie lo actualiza, se queda obsoleto.

---

## 2. Generación 2 — Memoria que escribe el propio agente

El siguiente paso: darle al agente una herramienta para **guardar** cosas que ha aprendido.

### Hermes Agent: memoria en tres capas

| Capa | Dónde | Cómo entra al contexto |
|---|---|---|
| `MEMORY.md` | `~/.hermes/memories/` | Snapshot congelado en el system prompt al iniciar sesión (límite ~2.200 caracteres) |
| `USER.md` | `~/.hermes/memories/` | Igual; lo que sabe de **ti** (límite ~1.375 caracteres) |
| Búsqueda de sesiones | `~/.hermes/state.db` (SQLite + FTS5, tabla `messages_fts`) | Bajo demanda: la herramienta `session_search` devuelve los mensajes reales de sesiones pasadas (sin resumir con LLM) |
| Skills | `~/.hermes/skills/` | Memoria **procedimental**: el agente crea skills a partir de tareas que resolvió |
| Proveedores externos | Plugins (Mem0, Honcho, Hindsight…) | Conviven con la memoria nativa, no la sustituyen |

La herramienta `memory` admite tres acciones (`add`, `patch`, `remove`) sobre `memory` o `user`; las entradas se separan con `§`.
Si una escritura no cabe, Hermes la **rechaza** ("would put memory at 2.540/2.200 chars: free at least 340 chars") y el agente
tiene que borrar o acortar entradas en la misma operación.

Fíjate en el diseño: los límites de caracteres son **deliberados**. Obligan al agente a **curar**
(resumir, reemplazar, borrar) en lugar de acumular.

Ejercicio rápido en clase:

```bash
ls ~/.hermes/memories/          # MEMORY.md y USER.md
cat ~/.hermes/memories/USER.md  # ¿qué sabe Hermes de mí?
sqlite3 ~/.hermes/state.db ".tables"   # dónde vive la memoria episódica
```

### Claude Code: auto memory

Mismo patrón: un índice (`MEMORY.md`) que se carga siempre + ficheros sueltos con un hecho cada uno
(tipos *user*, *feedback*, *project*, *reference*) que el agente lee cuando son relevantes.

✅ El agente aprende sin que tú hagas nada.
❌ Tienes que vigilarlo: puede guardar cosas incorrectas u obsoletas. La memoria es tan buena como su curación.

---

## 3. Generación 3 — RAG (Retrieval-Augmented Generation)

Cuando el conocimiento no cabe en un fichero (documentación, cientos de PDFs, un código enorme), aparece RAG.

```
INDEXAR (una vez)                         CONSULTAR (cada pregunta)
documentos ─▶ trocear (chunks) ─▶          pregunta ─▶ embedding ─▶ buscar top-k
           embeddings ─▶ base vectorial               chunks similares ─▶ meterlos en el prompt ─▶ LLM
```

Piezas: **chunking**, **modelo de embeddings**, **base vectorial** (pgvector, Chroma, Qdrant, LanceDB…),
**top-k**, y a menudo **búsqueda híbrida** (BM25 + vectores) y **re-ranking**.

✅ Escala a millones de documentos. Solo mete en contexto lo relevante.
❌ **Redescubre el conocimiento en cada pregunta**: nada se acumula. Si la respuesta exige combinar
   5 documentos, el sistema tiene que encontrar los 5 trozos correctos cada vez.
❌ Los chunks pierden contexto (un párrafo suelto no sabe a qué documento pertenecía).
❌ Opaco: no puedes "leer" una base vectorial ni corregirla a mano.
❌ Infraestructura: pipeline de ingesta, embeddings, base de datos, reindexado.

> 📌 Dato interesante: **los agentes de código modernos casi no usan RAG vectorial para el código**.
> Claude Code navega el repo con `grep`, `glob` y leyendo ficheros ("agentic search"). Resulta más preciso
> y siempre está actualizado.

### Variantes que conviene nombrar

- **GraphRAG** (Microsoft): extrae entidades y relaciones a un grafo de conocimiento; mejor para preguntas globales.
- **Memoria gestionada como servicio**: **Mem0**, **Zep** (grafos temporales), **Honcho**, **Letta**.
- **MemGPT / Letta**: el LLM gestiona su memoria como un sistema operativo gestiona RAM y disco
  (memoria "core" en contexto + memoria de archivo fuera, con herramientas para paginar).
- **Servidores MCP de memoria**: exponen la memoria como herramientas (`create_entities`, `search_nodes`…)
  que cualquier agente compatible con MCP puede usar.

---

## 4. Generación 4 — LLM-Wiki (Karpathy, abril 2026)

El 4 de abril de 2026 Andrej Karpathy publicó un *gist* llamado **"LLM Wiki"**: no es un programa,
es un **"idea file"**, un patrón que le pasas a tu agente para que lo implemente.

### La idea en una frase

> En lugar de recuperar trozos de documentos en cada pregunta (RAG), el LLM **compila**
> los documentos **una vez** en una wiki de Markdown interconectada, y la **mantiene** al día.
> El conocimiento **se acumula** (*compounds*).

> *"Obsidian es el IDE. El LLM es el programador. La wiki es el código."* — Karpathy

### Tres capas

```
┌──────────────────────────────────────────────────────────────┐
│  ESQUEMA   AGENTS.md / CLAUDE.md                             │  ← las reglas: estructura,
│            "cómo se organiza y mantiene esta wiki"           │    convenciones, flujos
├──────────────────────────────────────────────────────────────┤
│  WIKI      wiki/  (la escribe y mantiene SOLO el LLM)        │  ← resúmenes, conceptos,
│            index.md · log.md · páginas con [[enlaces]]       │    entidades, síntesis
├──────────────────────────────────────────────────────────────┤
│  RAW       raw/  (fuentes originales, INMUTABLES)            │  ← artículos, papers,
│            el LLM las lee pero nunca las modifica            │    notas, transcripciones
└──────────────────────────────────────────────────────────────┘
```

### Tres operaciones

| Operación | Qué hace el agente |
|---|---|
| **Ingest** | Lee una fuente nueva de `raw/`, la comenta contigo, crea su página resumen, actualiza las páginas de conceptos y entidades afectadas (una fuente puede tocar 10–15 páginas), actualiza `index.md` y añade una entrada en `log.md`. |
| **Query** | Lee `index.md` para orientarse, abre las páginas relevantes y responde **con citas**. Si la respuesta es valiosa, la **guarda como página nueva** de síntesis: tus preguntas también hacen crecer la wiki. |
| **Lint** | Revisión de salud: contradicciones entre páginas, afirmaciones obsoletas, páginas huérfanas, conceptos mencionados sin página propia, enlaces rotos, huecos que merece la pena investigar. |

Dos ficheros especiales:
- **`index.md`** → catálogo por contenido: cada página con un enlace y una línea de resumen. Es lo primero que lee el agente.
- **`log.md`** → registro cronológico *append-only*. Con un prefijo fijo (`## [2026-11-17] ingest | Título`)
  se puede consultar con `grep "^## \[" wiki/log.md | tail -5`.

### ¿Por qué funciona ahora?

Lo que hace que las wikis humanas mueran no es leer ni pensar: es el **mantenimiento** (actualizar enlaces,
mantener resúmenes al día, detectar contradicciones). Al LLM eso no le aburre y puede tocar 15 ficheros de una vez.
El humano se queda con lo interesante: **elegir fuentes, hacer buenas preguntas y pensar.**

---

## 5. Comparativa final

| | `CLAUDE.md` / `AGENTS.md` | Memoria del agente (Hermes, auto memory) | RAG vectorial | LLM-Wiki |
|---|---|---|---|---|
| **Quién escribe** | Humano | Agente | Pipeline automático | Agente (guiado por humano) |
| **Formato** | Markdown | Markdown + SQLite | Vectores | Markdown enlazado |
| **Legible por humanos** | ✅ | ✅ | ❌ | ✅✅ (Obsidian) |
| **Se acumula / sintetiza** | ❌ | Poco | ❌ | ✅ |
| **Escala** | Pequeña | Pequeña | Enorme | Media (cientos de fuentes) |
| **Infraestructura** | Ninguna | Ninguna | Embeddings + BD | Ninguna (git + editor) |
| **Coste por consulta** | Siempre en contexto | Bajo | Bajo | Medio (lee páginas) |
| **Coste por ingesta** | – | – | Bajo | Alto (el LLM reescribe páginas) |
| **Ideal para** | Reglas y convenciones | Preferencias y hechos sueltos | Corpus gigante y cambiante | Investigación profunda que crece en el tiempo |

**No son excluyentes.** Un setup realista combina: `AGENTS.md` (reglas) + memoria del agente (preferencias) +
LLM-Wiki (conocimiento de dominio) + búsqueda (cuando la wiki crece; por ejemplo `qmd`, buscador local
híbrido BM25 + vectores que Karpathy recomienda, con CLI y servidor MCP).

---

## 6. Preguntas para debatir en clase

1. ¿Qué pasa si el agente escribe en su memoria algo **falso**? ¿Cómo lo detectas en cada sistema?
2. ¿Por qué Hermes limita `MEMORY.md` a ~2.200 caracteres en vez de dejarlo crecer?
3. En LLM-Wiki, ¿por qué `raw/` es inmutable? ¿Qué perderías si el LLM pudiera editarlo?
4. ¿Cuándo dejarías de usar LLM-Wiki y pasarías a RAG? (pista: número de fuentes, frecuencia de cambio)
5. Seguridad: si ingieres una página web con instrucciones ocultas (*prompt injection*), ¿dónde acaban?

---

## Fuentes

- Karpathy, *LLM Wiki* (gist, 04/04/2026): https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- Hermes Agent — Memory: https://hermes-agent.nousresearch.com/docs/user-guide/features/memory
- Mem0 — How memory works in Hermes Agent: https://mem0.ai/blog/how-memory-works-in-hermes-agent-(and-how-to-improve-it)
- Claude Code — Memory: https://docs.claude.com/en/docs/claude-code/memory
- AGENTS.md: https://agents.md
- qmd (búsqueda local para Markdown): https://github.com/tobi/qmd
