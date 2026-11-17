---
origen: Resumen de lectura (fuente de ejemplo escrita para el curso CodeCrypto)
fecha: 2026-11-15
referencias: Park et al. 2023 "Generative Agents" (arXiv 2304.03442); Packer et al. 2023 "MemGPT" (arXiv 2310.08560)
---

# Lectura: dos papers que inventaron la memoria de los agentes

## Generative Agents (Stanford y Google, abril de 2023)

Joon Sung Park y su equipo metieron **25 agentes** en un pueblo virtual llamado Smallville. Cada
agente guardaba un **flujo de memoria** (*memory stream*): una lista de todo lo que observaba, en
lenguaje natural. Para decidir qué recordar en cada momento puntuaban cada recuerdo por tres
criterios: **reciente**, **importante** y **relevante** para la situación.

Cada cierto tiempo los agentes hacían **reflexión**: releían sus recuerdos y escribían conclusiones de
más alto nivel ("Klaus está obsesionado con su investigación"). Es memoria episódica convertida en
memoria semántica. El experimento famoso: un agente decidió organizar una fiesta de San Valentín y,
sin que nadie lo programara, la invitación se fue difundiendo y varios agentes acudieron.

## MemGPT (UC Berkeley, octubre de 2023)

Charles Packer y su equipo trataron la ventana de contexto como la **RAM** de un ordenador y el
almacenamiento externo como el **disco**. El propio LLM decide, con llamadas a funciones, qué sube a
la ventana y qué baja al disco: lo llamaron **paginación de memoria**. Distinguían:

- **Memoria núcleo** (*core memory*): pocos datos clave siempre visibles, que el agente edita.
- **Memoria de recuerdo** (*recall memory*): el historial de conversaciones, buscable.
- **Memoria de archivo** (*archival memory*): un almacén grande consultado con búsqueda.

En 2024 el proyecto se convirtió en la empresa **Letta**.

## Por qué importa hoy

La memoria núcleo de MemGPT es casi lo mismo que `MEMORY.md` y `USER.md` de Hermes (pocos caracteres,
editados por el agente), y la memoria de recuerdo se parece a `session_search`. La reflexión de
Generative Agents es la misma idea que la operación de ingesta de una LLM-Wiki: no guardar lo bruto,
sino reescribirlo en conocimiento.
