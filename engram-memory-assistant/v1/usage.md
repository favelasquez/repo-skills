---
metadata:
  version: "1.0"
  author: https://github.com/favelasquez
name: engram-memory-assistant
description: >
  Skill para interactuar con Engram directamente desde la conversación con Claude.
  Activar siempre que el usuario quiera guardar algo en Engram, buscar memorias,
  ver qué recuerda Engram de un proyecto, hacer un summary de sesión, o cuando diga
  frases como "guarda esto en Engram", "busca en mis memorias", "¿qué sé de X?",
  "haz un summary de lo que hicimos", "muéstrame mis memorias recientes",
  "¿Engram tiene algo sobre JWT/auth/performance/etc?". También activar cuando el
  usuario quiera guardar decisiones, bugs, patterns o preferencias personales en
  Engram desde Claude.ai sin abrir la terminal.
---
metadata:
  author: https://github.com/favelasquez

# Engram Memory Assistant � Interacción directa desde Claude

Esta skill permite a Claude interactuar con Engram a través de su HTTP API
(`http://localhost:7437`) directamente desde la conversación.

## Prerequisito

Engram debe estar corriendo localmente:
```bash
engram serve          # inicia en puerto 7437
# o como servicio en background (ver engram-install-setup skill)
```

---
metadata:
  author: https://github.com/favelasquez

## Qué puede hacer esta skill

| El usuario dice | Claude hace |
|---|---|
| "guarda esto en Engram: [decisión]" | Construye y guarda observación estructurada |
| "busca en mis memorias sobre JWT" | Llama GET /search y muestra resultados |
| "¿qué recuerda Engram de este proyecto?" | Llama GET /context y resume el contexto |
| "muéstrame mis memorias recientes" | Llama GET /observations/recent |
| "haz un summary de lo que trabajamos hoy" | Genera y guarda un session summary |
| "¿cuántas memorias tengo?" | Llama GET /stats |
| "exporta todas mis memorias" | Llama GET /export |

---
metadata:
  author: https://github.com/favelasquez

## Cómo guardar una memoria � flujo correcto

Cuando el usuario quiere guardar algo, Claude debe:

1. **Identificar el tipo**: ¿es una decisión, un bugfix, un pattern, una preferencia?
2. **Extraer los 4 campos**: What / Why / Where / Learned
3. **Sugerir un topic_key** si es algo evolutivo (decisiones de arquitectura, patterns)
4. **Confirmar con el usuario** antes de guardar si hay ambigüedad
5. **Llamar la API** y confirmar el ID guardado

```
Tipos válidos:
  decision      �  elección técnica o de diseño
  architecture  �  estructura del sistema
  bugfix        �  causa raíz y solución de un bug
  pattern       �  convención o forma de hacer algo
  config        �  configuración de entorno
  discovery     �  hallazgo no obvio
  learning      �  aprendizaje general
```

---
metadata:
  author: https://github.com/favelasquez

## Formato de observación a guardar

```
POST http://localhost:7437/observations  (o usar CLI: engram save)

{
  "title":     "Verbo + qué � corto y buscable",
  "type":      "decision|architecture|bugfix|pattern|config|discovery|learning",
  "content":   "**What**: ...\n**Why**: ...\n**Where**: ...\n**Learned**: ...",
  "project":   "nombre-del-proyecto (si aplica)",
  "scope":     "project | personal",
  "topic_key": "categoria/subtema (solo para topics evolutivos)"
}
```

**scope personal** � usar cuando el usuario guarda preferencias propias que aplican a todos sus proyectos, no a uno específico. Ejemplo: "prefiero funciones de máximo 20 líneas", "siempre usar TypeScript strict mode".

---
metadata:
  author: https://github.com/favelasquez

## Instrucciones para Claude al usar esta skill

### Al buscar memorias
1. Hacer GET /health primero � si falla, Engram no está corriendo �  indicar al usuario
2. Llamar GET /context primero (más rápido, historial reciente)
3. Si no hay resultados relevantes, llamar GET /search con keywords
4. Presentar los resultados de forma clara, NO en JSON crudo

### Al guardar una memoria
1. Nunca guardar sin que el usuario haya confirmado el contenido
2. Si el usuario da información parcial, completar What/Why/Where/Learned inteligentemente
3. Sugerir `topic_key` para decisions de arquitectura o patterns que evolucionan
4. Confirmar con el ID devuelto: "�S& Guardado con ID #42"

### Al generar un session summary
Usar la plantilla oficial de Engram:
```
## Goal
[Qué se estaba trabajando]

## Instructions
[Preferencias del usuario descubiertas]

## Discoveries
- [Hallazgos técnicos, gotchas]

## Accomplished
- [Lo completado]

## Next Steps
- [Lo que queda]

## Relevant Files
- path/to/file � descripción
```

---
metadata:
  author: https://github.com/favelasquez

## Referencia de API para Claude

Ver `references/api-calls.md` para los llamados curl exactos por operación.

