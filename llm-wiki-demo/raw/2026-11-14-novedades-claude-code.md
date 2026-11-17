---
origen: Apuntes del profesor tras revisar Claude Code 2.1 (fuente de ejemplo escrita para el curso CodeCrypto)
fecha: 2026-11-14
url_referencia: https://docs.claude.com/en/docs/claude-code/memory
---

# Novedades: Claude Code y AGENTS.md

Revisando la versión 2.1 de Claude Code:

- **Claude Code ya lee `AGENTS.md` por sí solo** cuando en la carpeta no hay `CLAUDE.md`. Ya no hace
  falta crear un `CLAUDE.md` con `@AGENTS.md` para que lo tenga en cuenta.
- Si existen los dos, manda `CLAUDE.md`. Por eso sigue siendo útil un `CLAUDE.md` que contenga
  `@AGENTS.md`: así se cargan las reglas comunes y puedes añadir debajo lo específico de Claude.
- Los comandos de `.claude/commands/` y las skills de `.claude/skills/` siguen funcionando igual.

Conclusión para el curso: los apuntes del 10 de noviembre decían que hacía falta el `@AGENTS.md`;
ahora es opcional.
