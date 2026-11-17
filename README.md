# Memoria persistente en agentes de código

Material de clase: cómo hacer que un agente (Claude Code, Hermes Agent) **recuerde** entre sesiones,
desde `CLAUDE.md` / `AGENTS.md` hasta el patrón **LLM-Wiki** de Karpathy.

🌐 **Página interactiva con historia y laboratorios:** https://noelserdna.github.io/memoria-agentes/

| Fichero | Contenido |
|---|---|
| [`01-teoria-memoria-agentes.md`](01-teoria-memoria-agentes.md) | Teoría: ficheros de instrucciones → memoria del agente → RAG → LLM-Wiki, con comparativa |
| [`02-practica-llm-wiki.md`](02-practica-llm-wiki.md) | Práctica guiada: instalar Obsidian, montar el vault, ingest / query / lint, ejercicios |
| [`llm-wiki-demo/`](llm-wiki-demo/) | Vault de partida listo para usar con Claude Code y Hermes |
| [`docs/index.html`](docs/index.html) | La página interactiva (se publica con GitHub Pages) |

## Arranque rápido

```bash
brew install --cask obsidian
git clone https://github.com/noelserdna/memoria-agentes.git && cd memoria-agentes
cp -R llm-wiki-demo ~/mi-llm-wiki && cd ~/mi-llm-wiki && git init && git add -A && git commit -m init
claude            # o: hermes
> /ingest raw/2026-11-01-apuntes-rag.md
```

Abre `~/mi-llm-wiki` como vault en Obsidian y mira cómo crece el grafo.
