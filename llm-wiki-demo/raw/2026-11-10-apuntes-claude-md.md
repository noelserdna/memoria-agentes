---
origen: Apuntes de clase (fuente de ejemplo escrita para el curso CodeCrypto) basados en la documentación de Claude Code
fecha: 2026-11-10
url_referencia: https://docs.claude.com/en/docs/claude-code/memory
---

# Apuntes: CLAUDE.md, AGENTS.md y la memoria de Claude Code

## CLAUDE.md

Claude Code (Anthropic) lee ficheros `CLAUDE.md` al arrancar y los mete en el contexto:

- `~/.claude/CLAUDE.md` — preferencias del usuario para todos los proyectos.
- `./CLAUDE.md` — instrucciones del proyecto, compartidas con el equipo vía git.
- `CLAUDE.md` en subcarpetas — se cargan cuando el agente trabaja en esa parte del repo.
- `.claude/rules/*.md` — reglas modulares.

Comandos útiles: `/init` genera un CLAUDE.md inicial analizando el repo; `/memory` lo abre para editar.
Un CLAUDE.md puede **importar** otros ficheros con `@ruta/al/fichero.md`.

Buenas prácticas: ser concreto ("usa pnpm, no npm"), mantenerlo corto porque ocupa contexto en
**todas** las sesiones, y revisarlo cuando el proyecto cambia.

## AGENTS.md

Formato abierto y neutral equivalente a CLAUDE.md, pensado para que distintos agentes (Codex, Cursor,
Hermes, Gemini CLI…) lean las mismas instrucciones. Para usarlo también con Claude Code basta un
`CLAUDE.md` que contenga `@AGENTS.md`.

## Auto memory

Además, Claude Code puede escribir su propia memoria en `~/.claude/projects/<proyecto>/memory/`:
un índice `MEMORY.md` que se carga en cada sesión y un fichero por recuerdo, clasificados como
*user*, *feedback*, *project* o *reference*. Es memoria que escribe el **agente**, no el humano,
y conviene revisarla de vez en cuando porque puede quedar obsoleta.

## Skills y comandos

Las **skills** (`.claude/skills/`) y los **slash commands** (`.claude/commands/`) son memoria
procedimental: instrucciones reutilizables que el agente carga cuando las necesita.
