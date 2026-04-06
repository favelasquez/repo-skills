---
metadata:
  author: https://github.com/favelasquez
name: engram-expert
description: >
  Skill experta en Engram � el sistema de memoria persistente para agentes de IA
  (Claude Code, Cursor, Windsurf, OpenCode, Gemini CLI, VS Code Copilot, etc.).
  Activar siempre que el usuario mencione Engram, memoria persistente para agentes,
  mem_save, mem_search, mem_context, MCP memory tools, engram tui, engram serve,
  engram sync, topic_key, observations, sessions de Engram, SQLite FTS5 para memoria,
  o quiera instalar/configurar/usar Engram en cualquier agente de IA. También activar
  cuando el usuario tenga problemas con el servidor MCP de Engram, búsquedas que no
  devuelven resultados, memorias duplicadas, sincronización entre máquinas, o quiera
  entender cómo estructurar observaciones y summaries para el agente.
---
metadata:
  author: https://github.com/favelasquez

# Engram � Skill Experta Completa

> **Engram** /��en.ɡræm/ � neurociencia: la huella física de un recuerdo en el cerebro.
> Tu agente de IA olvida todo al terminar la sesión. Engram le da un cerebro.

## Qué es Engram

Un **binario Go** con SQLite + FTS5 full-text search expuesto como:
- **MCP Server** (stdio) � para cualquier agente compatible con MCP
- **HTTP API** (puerto 7437) � para plugins e integraciones
- **CLI** � `engram search`, `engram save`, etc.
- **TUI** � interfaz terminal interactiva (`engram tui`)

```
Agente (Claude Code / OpenCode / Gemini CLI / Cursor / Windsurf / VS Code...)
    �  MCP stdio
Engram (binario Go único, sin dependencias)
    � 
SQLite + FTS5 (~/.engram/engram.db)
```

## Instalación rápida

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

## Cómo funciona el ciclo de memoria

```
1. Agente completa trabajo significativo (bugfix, decisión de arquitectura, etc.)
2. Agente llama mem_save �  título, tipo, What/Why/Where/Learned
3. Engram persiste en SQLite con indexación FTS5
4. Próxima sesión:
   �  Agente llama mem_context (historial reciente, rápido)
   �  Si no encuentra, llama mem_search (búsqueda FTS5 full-text)
   �  Recupera contexto relevante �  continúa trabajo sin perder nada
```

---
metadata:
  author: https://github.com/favelasquez

## Los 13 MCP Tools � referencia completa

> Para ejemplos detallados de uso ver `references/mcp-tools.md`

| Tool | Cuándo usar |
|---|---|
| `mem_save` | Después de trabajo significativo � obligatorio, no opcional |
| `mem_search` | Buscar memorias por texto libre (FTS5) |
| `mem_context` | Primero siempre � historial reciente de sesiones (rápido) |
| `mem_update` | Corregir una observación con ID exacto conocido |
| `mem_delete` | Eliminar una observación específica |
| `mem_suggest_topic_key` | Obtener clave canónica antes de guardar con topic_key |
| `mem_get_observation` | Obtener contenido completo sin truncar de una observación |
| `mem_timeline` | Ver contexto cronológico alrededor de una observación |
| `mem_session_summary` | Guardar resumen comprensivo al final de sesión |
| `mem_session_start` | Iniciar sesión de trabajo |
| `mem_session_end` | Cerrar sesión con estadísticas |
| `mem_save_prompt` | Guardar prompt del usuario para referencia futura |
| `mem_stats` | Ver estadísticas del sistema de memoria |

---
metadata:
  author: https://github.com/favelasquez

## Estructura de una observación (`mem_save`)

```
title:     Verbo + qué � corto y buscable
           Ejemplos: "Fixed N+1 query in UserList"
                     "Chose Zustand over Redux"
                     "JWT refresh token rotation pattern"

type:      decision | architecture | bugfix | pattern | config | discovery | learning

scope:     project (default) | personal

topic_key: clave canónica estable para topics evolutivos
           Ejemplo: "architecture/auth-model", "pattern/error-handling"
           �  Reusar la misma key actualiza en lugar de crear duplicados
           �  Llamar mem_suggest_topic_key si no estás seguro

content:   Formato estructurado:
           **What**: Una oración � qué se hizo
           **Why**: Qué lo motivó (request del usuario, bug, performance, etc.)
           **Where**: Archivos o paths afectados
           **Learned**: Gotchas, edge cases, cosas que sorprendieron (omitir si ninguno)
```

---
metadata:
  author: https://github.com/favelasquez

## Protocolo de memoria para el agente � reglas obligatorias

### CUÁNDO GUARDAR (obligatorio, no opcional)

Guardar después de:
- Decisión de arquitectura o diseño tomada
- Descubrimiento no obvio sobre el codebase
- Cambio de configuración o setup de entorno
- Patrón establecido (nombres, estructura, convención)
- Preferencia o restricción del usuario aprendida
- Bugfix con causa raíz no trivial

### CUÁNDO BUSCAR EN MEMORIA

Buscar cuando:
- Empezando trabajo que puede haber sido hecho antes
- El usuario menciona un tema sin contexto � verificar si sesiones pasadas lo cubrieron
- **Protocolo**: Primero `mem_context` (rápido) �  si no hay, `mem_search` �  si encuentras, `mem_get_observation` para contenido completo

### PROTOCOLO DE CIERRE DE SESI�N (obligatorio)

Al final de cada sesión:
1. Llamar `mem_session_summary` con el contenido del resumen compactado
2. Llamar `mem_context` para recuperar contexto adicional de sesiones previas

### REGLAS DE TOPIC_KEY

- Diferentes topics NO deben sobreescribirse entre sí
- Reusar la misma `topic_key` para actualizar un topic evolutivo en lugar de crear nuevos
- Si no estás seguro de la key, llamar `mem_suggest_topic_key` primero y luego reusar

---
metadata:
  author: https://github.com/favelasquez

## Estructura de un Session Summary (`mem_session_summary`)

```markdown
## Goal
[En qué estábamos trabajando esta sesión]

## Instructions
[Preferencias o restricciones del usuario descubiertas � omitir si ninguna]

## Discoveries
- [Hallazgos técnicos, gotchas, aprendizajes no obvios]

## Accomplished
- [Items completados con detalles clave]

## Next Steps
- [Lo que queda por hacer � para la próxima sesión]

## Relevant Files
- path/to/file � [qué hace o qué cambió]
```

---
metadata:
  author: https://github.com/favelasquez

## CLI � Referencia completa

```bash
# Servidor
engram serve [port]          # HTTP API en puerto 7437 (default)
engram mcp                   # MCP server (stdio transport)
engram tui                   # TUI interactiva

# Memoria
engram search <query>        # Búsqueda FTS5 [--type TYPE] [--project P] [--scope S] [--limit N]
engram save <title> <msg>    # Guardar memoria [--type TYPE] [--project P] [--scope S] [--topic KEY]
engram timeline <obs_id>     # Contexto cronológico [--before N] [--after N]
engram context [project]     # Contexto reciente de sesiones anteriores
engram stats                 # Estadísticas del sistema

# Export / Import
engram export [file]         # Exportar todo a JSON (default: engram-export.json)
engram import <file>         # Importar desde JSON

# Sync (entre máquinas via git)
engram sync                  # Exportar nuevas memorias como chunk comprimido
engram sync --import         # Importar chunks del manifest no importados aún
engram sync --status         # Ver cuántos chunks locales vs remotos
engram sync --project NAME   # Filtrar export a un proyecto específico
engram sync --all            # Exportar TODAS las memorias de todos los proyectos

# Utilidades
engram version
engram help
```

---
metadata:
  author: https://github.com/favelasquez

## Variables de entorno

| Variable | Descripción | Default |
|---|---|---|
| `ENGRAM_DATA_DIR` | Override directorio de datos | `~/.engram` |
| `ENGRAM_PORT` | Override puerto del servidor HTTP | `7437` |

---
metadata:
  author: https://github.com/favelasquez

## Checklist de configuración correcta

- [ ] ¿El binario `engram` está en `$PATH`?
- [ ] ¿El agente tiene la config MCP apuntando a `engram mcp`?
- [ ] ¿El agente usa `mem_context` primero antes de `mem_search`?
- [ ] ¿Las observaciones tienen el formato What/Why/Where/Learned?
- [ ] ¿Se usa `topic_key` para decisions evolutivas (en lugar de crear duplicados)?
- [ ] ¿Se llama `mem_session_summary` al final de cada sesión importante?
- [ ] ¿Se usa `scope: personal` para preferencias del usuario no ligadas al proyecto?

---
metadata:
  author: https://github.com/favelasquez

## Skills complementarias

| Skill | Cuándo usar |
|---|---|
| `engram-install-setup` | Instalar Engram, configurar en Claude Code/Cursor/VS Code, solucionar problemas de instalación |
| `engram-memory-assistant` | Guardar memorias desde Claude, buscar, ver contexto, hacer summaries � sin abrir la terminal |

---
metadata:
  author: https://github.com/favelasquez

Leer archivos `references/` para guías detalladas:
- `references/mcp-tools.md` � Parámetros completos de los 13 tools MCP
- `references/http-api.md` � Endpoints REST completos
- `references/sync-git.md` � Sincronización entre máquinas con git
- `references/tui.md` � TUI, navegación y atajos de teclado

