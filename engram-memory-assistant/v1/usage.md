---
metadata:
  author: https://github.com/favelasquez
name: engram-memory-assistant
description: >
  Skill para interactuar con Engram directamente desde la conversaciÃ³n con Claude.
  Activar siempre que el usuario quiera guardar algo en Engram, buscar memorias,
  ver quÃ© recuerda Engram de un proyecto, hacer un summary de sesiÃ³n, o cuando diga
  frases como "guarda esto en Engram", "busca en mis memorias", "Â¿quÃ© sÃ© de X?",
  "haz un summary de lo que hicimos", "muÃ©strame mis memorias recientes",
  "Â¿Engram tiene algo sobre JWT/auth/performance/etc?". TambiÃ©n activar cuando el
  usuario quiera guardar decisiones, bugs, patterns o preferencias personales en
  Engram desde Claude.ai sin abrir la terminal.
---
metadata:
  author: https://github.com/favelasquez

# Engram Memory Assistant â€” InteracciÃ³n directa desde Claude

Esta skill permite a Claude interactuar con Engram a travÃ©s de su HTTP API
(`http://localhost:7437`) directamente desde la conversaciÃ³n.

## Prerequisito

Engram debe estar corriendo localmente:
```bash
engram serve          # inicia en puerto 7437
# o como servicio en background (ver engram-install-setup skill)
```

---
metadata:
  author: https://github.com/favelasquez

## QuÃ© puede hacer esta skill

| El usuario dice | Claude hace |
|---|---|
| "guarda esto en Engram: [decisiÃ³n]" | Construye y guarda observaciÃ³n estructurada |
| "busca en mis memorias sobre JWT" | Llama GET /search y muestra resultados |
| "Â¿quÃ© recuerda Engram de este proyecto?" | Llama GET /context y resume el contexto |
| "muÃ©strame mis memorias recientes" | Llama GET /observations/recent |
| "haz un summary de lo que trabajamos hoy" | Genera y guarda un session summary |
| "Â¿cuÃ¡ntas memorias tengo?" | Llama GET /stats |
| "exporta todas mis memorias" | Llama GET /export |

---
metadata:
  author: https://github.com/favelasquez

## CÃ³mo guardar una memoria â€” flujo correcto

Cuando el usuario quiere guardar algo, Claude debe:

1. **Identificar el tipo**: Â¿es una decisiÃ³n, un bugfix, un pattern, una preferencia?
2. **Extraer los 4 campos**: What / Why / Where / Learned
3. **Sugerir un topic_key** si es algo evolutivo (decisiones de arquitectura, patterns)
4. **Confirmar con el usuario** antes de guardar si hay ambigÃ¼edad
5. **Llamar la API** y confirmar el ID guardado

```
Tipos vÃ¡lidos:
  decision      â†’ elecciÃ³n tÃ©cnica o de diseÃ±o
  architecture  â†’ estructura del sistema
  bugfix        â†’ causa raÃ­z y soluciÃ³n de un bug
  pattern       â†’ convenciÃ³n o forma de hacer algo
  config        â†’ configuraciÃ³n de entorno
  discovery     â†’ hallazgo no obvio
  learning      â†’ aprendizaje general
```

---
metadata:
  author: https://github.com/favelasquez

## Formato de observaciÃ³n a guardar

```
POST http://localhost:7437/observations  (o usar CLI: engram save)

{
  "title":     "Verbo + quÃ© â€” corto y buscable",
  "type":      "decision|architecture|bugfix|pattern|config|discovery|learning",
  "content":   "**What**: ...\n**Why**: ...\n**Where**: ...\n**Learned**: ...",
  "project":   "nombre-del-proyecto (si aplica)",
  "scope":     "project | personal",
  "topic_key": "categoria/subtema (solo para topics evolutivos)"
}
```

**scope personal** â€” usar cuando el usuario guarda preferencias propias que aplican a todos sus proyectos, no a uno especÃ­fico. Ejemplo: "prefiero funciones de mÃ¡ximo 20 lÃ­neas", "siempre usar TypeScript strict mode".

---
metadata:
  author: https://github.com/favelasquez

## Instrucciones para Claude al usar esta skill

### Al buscar memorias
1. Hacer GET /health primero â€” si falla, Engram no estÃ¡ corriendo â†’ indicar al usuario
2. Llamar GET /context primero (mÃ¡s rÃ¡pido, historial reciente)
3. Si no hay resultados relevantes, llamar GET /search con keywords
4. Presentar los resultados de forma clara, NO en JSON crudo

### Al guardar una memoria
1. Nunca guardar sin que el usuario haya confirmado el contenido
2. Si el usuario da informaciÃ³n parcial, completar What/Why/Where/Learned inteligentemente
3. Sugerir `topic_key` para decisions de arquitectura o patterns que evolucionan
4. Confirmar con el ID devuelto: "âœ… Guardado con ID #42"

### Al generar un session summary
Usar la plantilla oficial de Engram:
```
## Goal
[QuÃ© se estaba trabajando]

## Instructions
[Preferencias del usuario descubiertas]

## Discoveries
- [Hallazgos tÃ©cnicos, gotchas]

## Accomplished
- [Lo completado]

## Next Steps
- [Lo que queda]

## Relevant Files
- path/to/file â€” descripciÃ³n
```

---
metadata:
  author: https://github.com/favelasquez

## Referencia de API para Claude

Ver `references/api-calls.md` para los llamados curl exactos por operaciÃ³n.

