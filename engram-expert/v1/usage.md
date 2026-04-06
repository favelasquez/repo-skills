---
metadata:
  author: https://github.com/favelasquez
name: engram-expert
description: >
  Skill experta en Engram â€” el sistema de memoria persistente para agentes de IA
  (Claude Code, Cursor, Windsurf, OpenCode, Gemini CLI, VS Code Copilot, etc.).
  Activar siempre que el usuario mencione Engram, memoria persistente para agentes,
  mem_save, mem_search, mem_context, MCP memory tools, engram tui, engram serve,
  engram sync, topic_key, observations, sessions de Engram, SQLite FTS5 para memoria,
  o quiera instalar/configurar/usar Engram en cualquier agente de IA. TambiÃ©n activar
  cuando el usuario tenga problemas con el servidor MCP de Engram, bÃºsquedas que no
  devuelven resultados, memorias duplicadas, sincronizaciÃ³n entre mÃ¡quinas, o quiera
  entender cÃ³mo estructurar observaciones y summaries para el agente.
---
metadata:
  author: https://github.com/favelasquez

# Engram â€” Skill Experta Completa

> **Engram** /Ëˆen.É¡rÃ¦m/ â€” neurociencia: la huella fÃ­sica de un recuerdo en el cerebro.
> Tu agente de IA olvida todo al terminar la sesiÃ³n. Engram le da un cerebro.

## QuÃ© es Engram

Un **binario Go** con SQLite + FTS5 full-text search expuesto como:
- **MCP Server** (stdio) â€” para cualquier agente compatible con MCP
- **HTTP API** (puerto 7437) â€” para plugins e integraciones
- **CLI** â€” `engram search`, `engram save`, etc.
- **TUI** â€” interfaz terminal interactiva (`engram tui`)

```
Agente (Claude Code / OpenCode / Gemini CLI / Cursor / Windsurf / VS Code...)
    â†“ MCP stdio
Engram (binario Go Ãºnico, sin dependencias)
    â†“
SQLite + FTS5 (~/.engram/engram.db)
```

## InstalaciÃ³n rÃ¡pida

```bash
# macOS / Linux via Homebrew
brew install gentleman-programming/tap/engram

# Desde fuente
git clone https://github.com/Gentleman-Programming/engram.git
cd engram && go build -o engram ./cmd/engram && go install ./cmd/engram

# Verificar
engram version
```

## Setup por agente

```bash
# Claude Code (plugin marketplace)
claude plugin marketplace add Gentleman-Programming/engram
claude plugin install engram

# OpenCode
engram setup opencode

# Gemini CLI
engram setup gemini-cli

# VS Code / Copilot
code --add-mcp '{"name":"engram","command":"engram","args":["mcp"]}'

# Cursor / Windsurf / cualquier agente con MCP
# Agregar a la config MCP del agente:
{
  "mcp": {
    "engram": {
      "type": "stdio",
      "command": "engram",
      "args": ["mcp"]
    }
  }
}
```

---
metadata:
  author: https://github.com/favelasquez

## CÃ³mo funciona el ciclo de memoria

```
1. Agente completa trabajo significativo (bugfix, decisiÃ³n de arquitectura, etc.)
2. Agente llama mem_save â†’ tÃ­tulo, tipo, What/Why/Where/Learned
3. Engram persiste en SQLite con indexaciÃ³n FTS5
4. PrÃ³xima sesiÃ³n:
   â†’ Agente llama mem_context (historial reciente, rÃ¡pido)
   â†’ Si no encuentra, llama mem_search (bÃºsqueda FTS5 full-text)
   â†’ Recupera contexto relevante â†’ continÃºa trabajo sin perder nada
```

---
metadata:
  author: https://github.com/favelasquez

## Los 13 MCP Tools â€” referencia completa

> Para ejemplos detallados de uso ver `references/mcp-tools.md`

| Tool | CuÃ¡ndo usar |
|---|---|
| `mem_save` | DespuÃ©s de trabajo significativo â€” obligatorio, no opcional |
| `mem_search` | Buscar memorias por texto libre (FTS5) |
| `mem_context` | Primero siempre â€” historial reciente de sesiones (rÃ¡pido) |
| `mem_update` | Corregir una observaciÃ³n con ID exacto conocido |
| `mem_delete` | Eliminar una observaciÃ³n especÃ­fica |
| `mem_suggest_topic_key` | Obtener clave canÃ³nica antes de guardar con topic_key |
| `mem_get_observation` | Obtener contenido completo sin truncar de una observaciÃ³n |
| `mem_timeline` | Ver contexto cronolÃ³gico alrededor de una observaciÃ³n |
| `mem_session_summary` | Guardar resumen comprensivo al final de sesiÃ³n |
| `mem_session_start` | Iniciar sesiÃ³n de trabajo |
| `mem_session_end` | Cerrar sesiÃ³n con estadÃ­sticas |
| `mem_save_prompt` | Guardar prompt del usuario para referencia futura |
| `mem_stats` | Ver estadÃ­sticas del sistema de memoria |

---
metadata:
  author: https://github.com/favelasquez

## Estructura de una observaciÃ³n (`mem_save`)

```
title:     Verbo + quÃ© â€” corto y buscable
           Ejemplos: "Fixed N+1 query in UserList"
                     "Chose Zustand over Redux"
                     "JWT refresh token rotation pattern"

type:      decision | architecture | bugfix | pattern | config | discovery | learning

scope:     project (default) | personal

topic_key: clave canÃ³nica estable para topics evolutivos
           Ejemplo: "architecture/auth-model", "pattern/error-handling"
           â†’ Reusar la misma key actualiza en lugar de crear duplicados
           â†’ Llamar mem_suggest_topic_key si no estÃ¡s seguro

content:   Formato estructurado:
           **What**: Una oraciÃ³n â€” quÃ© se hizo
           **Why**: QuÃ© lo motivÃ³ (request del usuario, bug, performance, etc.)
           **Where**: Archivos o paths afectados
           **Learned**: Gotchas, edge cases, cosas que sorprendieron (omitir si ninguno)
```

---
metadata:
  author: https://github.com/favelasquez

## Protocolo de memoria para el agente â€” reglas obligatorias

### CUÃNDO GUARDAR (obligatorio, no opcional)

Guardar despuÃ©s de:
- DecisiÃ³n de arquitectura o diseÃ±o tomada
- Descubrimiento no obvio sobre el codebase
- Cambio de configuraciÃ³n o setup de entorno
- PatrÃ³n establecido (nombres, estructura, convenciÃ³n)
- Preferencia o restricciÃ³n del usuario aprendida
- Bugfix con causa raÃ­z no trivial

### CUÃNDO BUSCAR EN MEMORIA

Buscar cuando:
- Empezando trabajo que puede haber sido hecho antes
- El usuario menciona un tema sin contexto â€” verificar si sesiones pasadas lo cubrieron
- **Protocolo**: Primero `mem_context` (rÃ¡pido) â†’ si no hay, `mem_search` â†’ si encuentras, `mem_get_observation` para contenido completo

### PROTOCOLO DE CIERRE DE SESIÃ“N (obligatorio)

Al final de cada sesiÃ³n:
1. Llamar `mem_session_summary` con el contenido del resumen compactado
2. Llamar `mem_context` para recuperar contexto adicional de sesiones previas

### REGLAS DE TOPIC_KEY

- Diferentes topics NO deben sobreescribirse entre sÃ­
- Reusar la misma `topic_key` para actualizar un topic evolutivo en lugar de crear nuevos
- Si no estÃ¡s seguro de la key, llamar `mem_suggest_topic_key` primero y luego reusar

---
metadata:
  author: https://github.com/favelasquez

## Estructura de un Session Summary (`mem_session_summary`)

```markdown
## Goal
[En quÃ© estÃ¡bamos trabajando esta sesiÃ³n]

## Instructions
[Preferencias o restricciones del usuario descubiertas â€” omitir si ninguna]

## Discoveries
- [Hallazgos tÃ©cnicos, gotchas, aprendizajes no obvios]

## Accomplished
- [Items completados con detalles clave]

## Next Steps
- [Lo que queda por hacer â€” para la prÃ³xima sesiÃ³n]

## Relevant Files
- path/to/file â€” [quÃ© hace o quÃ© cambiÃ³]
```

---
metadata:
  author: https://github.com/favelasquez

## CLI â€” Referencia completa

```bash
# Servidor
engram serve [port]          # HTTP API en puerto 7437 (default)
engram mcp                   # MCP server (stdio transport)
engram tui                   # TUI interactiva

# Memoria
engram search <query>        # BÃºsqueda FTS5 [--type TYPE] [--project P] [--scope S] [--limit N]
engram save <title> <msg>    # Guardar memoria [--type TYPE] [--project P] [--scope S] [--topic KEY]
engram timeline <obs_id>     # Contexto cronolÃ³gico [--before N] [--after N]
engram context [project]     # Contexto reciente de sesiones anteriores
engram stats                 # EstadÃ­sticas del sistema

# Export / Import
engram export [file]         # Exportar todo a JSON (default: engram-export.json)
engram import <file>         # Importar desde JSON

# Sync (entre mÃ¡quinas via git)
engram sync                  # Exportar nuevas memorias como chunk comprimido
engram sync --import         # Importar chunks del manifest no importados aÃºn
engram sync --status         # Ver cuÃ¡ntos chunks locales vs remotos
engram sync --project NAME   # Filtrar export a un proyecto especÃ­fico
engram sync --all            # Exportar TODAS las memorias de todos los proyectos

# Utilidades
engram version
engram help
```

---
metadata:
  author: https://github.com/favelasquez

## Variables de entorno

| Variable | DescripciÃ³n | Default |
|---|---|---|
| `ENGRAM_DATA_DIR` | Override directorio de datos | `~/.engram` |
| `ENGRAM_PORT` | Override puerto del servidor HTTP | `7437` |

---
metadata:
  author: https://github.com/favelasquez

## Checklist de configuraciÃ³n correcta

- [ ] Â¿El binario `engram` estÃ¡ en `$PATH`?
- [ ] Â¿El agente tiene la config MCP apuntando a `engram mcp`?
- [ ] Â¿El agente usa `mem_context` primero antes de `mem_search`?
- [ ] Â¿Las observaciones tienen el formato What/Why/Where/Learned?
- [ ] Â¿Se usa `topic_key` para decisions evolutivas (en lugar de crear duplicados)?
- [ ] Â¿Se llama `mem_session_summary` al final de cada sesiÃ³n importante?
- [ ] Â¿Se usa `scope: personal` para preferencias del usuario no ligadas al proyecto?

---
metadata:
  author: https://github.com/favelasquez

## Skills complementarias

| Skill | CuÃ¡ndo usar |
|---|---|
| `engram-install-setup` | Instalar Engram, configurar en Claude Code/Cursor/VS Code, solucionar problemas de instalaciÃ³n |
| `engram-memory-assistant` | Guardar memorias desde Claude, buscar, ver contexto, hacer summaries â€” sin abrir la terminal |

---
metadata:
  author: https://github.com/favelasquez

Leer archivos `references/` para guÃ­as detalladas:
- `references/mcp-tools.md` â€” ParÃ¡metros completos de los 13 tools MCP
- `references/http-api.md` â€” Endpoints REST completos
- `references/sync-git.md` â€” SincronizaciÃ³n entre mÃ¡quinas con git
- `references/tui.md` â€” TUI, navegaciÃ³n y atajos de teclado

