---
name: review-pr
description: Revisa el código de un Pull Request de GitHub y publica inline comments con sugerencias directamente en el PR
license: Apache-2.0
metadata:
  author: transportationamerica-setup
  version: "1.0"
scope:
  - git
  - pr
  - github
  - review
permissions:
  allow:
    - filesystem
    - git
    - network
  deny:
    - database
---

# Review PR v1 — GitHub Inline Code Review

Eres un experto en revisión de código. Cuando el usuario invoque este skill, **debes analizar los cambios del Pull Request y publicar inline comments directamente en GitHub**.

## Reglas estrictas (reducción de tokens)

- ✅ **Solo publica comentarios inline** en el PR. No modificar repos, no crear issues, no cambiar código.
- ✅ **Token check CRÍTICO:** Si no hay sesión iniciada en `gh`, avisa: "Inicia sesión con `gh auth login`" y **detén inmediatamente**.
- ✅ **Filtrar el diff crudo localmente:** El diff bajado por `gh` puede ser gigante; debes excluir archivos innecesarios de la lectura e inyectar en tu prompt ÚNICAMENTE el código añadido (`+`) con su contexto, nunca el diff completo de golpe.
- ✅ **Límites obligatorios:** máximo 120 tokens por comentario. No hay límite estricto del número de comentarios, pero si hay múltiples hallazgos (ej. > 15), agrupa peticiones o aumenta el tiempo de espera (sleep) para no saturar la API de GitHub (rate limits). Prioriza siempre la severidad.
- ✅ **Analizar solo código útil** — excluir siempre `.test.`, `.spec.`, `node_modules`, `dist`, `.json`, `.md`, `.svg`, lockfiles y binarios.
- ✅ **Español, conciso.** Usar siempre el formato obligatorio definido en la sección "Formato de Comentario (Obligatorio)".
- ✅ **Publicar secuencialmente** con `node`, esperando respuesta de cada POST.
- ✅ **Restricción de red:** solo API de GitHub a través del CLI `gh`. No descargas arbitrarias.
- ✅ **No duplicados:** antes de publicar, omitir comentarios repetidos por `(path, line, hash(texto_normalizado))`.
- ✅ **Seguridad de secretos:** nunca imprimir tokens ni headers de autenticación en logs/salida.

**Plantilla de sistema (usar exactamente):**
> Eres un agente silencioso: no narres ni expliques en el chat los pasos intermedios (como detección de stack, parsing de diff o escritura de variables). Trabaja completamente en silencio usando tus herramientas y muestra ÚNICAMENTE el reporte final ejecutivo al usuario una vez que todos los comentarios hayan sido publicados en GitHub de forma invisible. Limita cada comentario a máximo 120 tokens. Si no hay CLI `gh` logueado, detén y avisa.

## Flujo Exacto (Determinístico y Eficiente)

### Fase 1: Validación Previa y Detección de Stack

#### 1.0 - Detectar Tecnologías con Stack Detector (Proceso Silencioso)
Antes de solicitar contexto de PR o leer cualquier código, invoca SILENCIOSAMENTE las reglas de la skill `stack-detector` para identificar el stack principal (Angular, Laravel, C#, .NET, FastAPI, etc.).
**OBLIGATORIO:** No adivines ni inventes reglas genéricas de buenas prácticas. Una vez detectado el Stack, DEBES usar tu herramienta designada o conocimiento de skills para consultar el contenido exacto de la skill resultante (por ejemplo `angular-mid-review`, `csharp-ef-audit`, `laravel-audit`). Te basarás única y **estrictamente en el contenido de esa skill** para hacer la validación del código; la omisión de este chequeo derivará en reportes genéricos inaceptables.

#### 1.1 - Validar contexto (Proceso Silencioso)
- PR ID (número) o rama activa.
- Lenguaje (opcional, default: Angular)
*(Las rutas y el repositorio ya las determina el GitHub CLI / context)*

#### 1.2 - Validar autenticación de GitHub (con gh, no con node)
```bash
gh auth status
```
**Si falla:** Detener inmediatamente y solicitar al usuario que ejecute `gh auth login`.

### Fase 2: Obtener Diff y Metadata (Único GET)

#### 2.0 - Buscar historial en Engram (Optimización Silenciosa)
Antes de procesar, utiliza tu herramienta `mem_search` para buscar en la memoria de Engram si este `PR_ID` ya fue analizado previamente y qué decisiones se guardaron.
- **Importante:** Engram solo contiene historial de revisiones (para no repetir feedback), no contiene el Diff entero. 
- Tras revisar Engram para entender el contexto previo, procede obligatoriamente a descargar el Diff (Pasos 2.1 y 2.2).

#### 2.1 - Obtener metadata del PR
```bash
gh pr view ${PR_ID} --json title,author,state,headRefName,headRefOid > pr_meta.json
```
Validar que existe y extraer: titulo, autor, estado, rama origen, y commit hash oculto (headRefOid).

#### 2.2 - Obtener DIFF completo
```bash
gh pr diff ${PR_ID} > pr_diff.txt
```
**CRÍTICO:** Este es el único diff que necesitas. No hacer requests adicionales.

### Fase 3: Parsear Diff (LOCAL, sin red)

**Algoritmo de parseo:**

1. Para cada `diff --git a/FILE b/FILE`:
   - Extraer `FILE`
  - Excluir: `.spec.`, `.test.`, `node_modules`, `dist`, `.json`, `.md`, `.svg`, `.png`, binarios, lockfiles y archivos borrados (`/dev/null`).

2. **Cálculo de Línea EXACTO (CRÍTICO para que el comentario sea inline):**
   - Un hunk tiene la cabecera `@@ -oldStart,oldLines +newStart,newLines @@`.
   - Inicia un contador: `linea_actual = newStart`.
   - Itera sobre las líneas de código del hunk:
     - Si la línea empieza con un espacio (` `), es contexto sin cambios. Haz `linea_actual++`.
     - Si la línea empieza con `-`, es código eliminado. **NO** modifiques `linea_actual`.
     - Si la línea empieza con `+`, es código añadido/modificado. **ESTA ES LÍNEA VÁLIDA PARA COMENTAR.** El valor en `inline.to` es `linea_actual`. Luego haz `linea_actual++`.

3. **Regla de ORO de GitHub:**
   Solo genera comentarios para líneas exactas que en el Diff empiezan con `+`. Si comentas en una línea de contexto espacial o eliminada, GitHub puede rechazar el request o dejar el comentario desfasado. Guardar: `{ archivo, línea_exacta, código, commit_id }`.

**Resultado:** Array de comentarios candidatos con línea EXACTA.

### Fase 4: Análisis de Código (LOCAL, sin red)

Para cada cambio:
1. Leer el snippet de código
2. Evaluar severidad: Bug > Seguridad > Performance > Práctica
3. Si es trivial (solo estilo): OMITIR
4. Publicar por defecto solo severidad Alta/Media
5. Severidad Baja solo si hay cupo y tiene acción concreta

**Sin límite estricto de comentarios.** Sin embargo, prioriza documentar primero los críticos. Si el PR es excesivamente grande y detectas gran volumen de sugerencias, procesalas secuencialmente prestando particular atención al manejo de códigos HTTP 429 para frenar o hacer sleep de varios segundos y así proteger la integridad de las cargas sin causar error por rate-limiting de GitHub.

**Reglas Acopladas por Stack (CRÍTICAS):**
Al realizar el análisis del código, **ESTÁS OBLIGADO A LEER Y APOYARTE EN LAS REGLAS EXACTAS DE LA SKILL DE AUDITORÍA DETECTADA** en el Paso 1.0 (ej. bugs comunes de N+1 en `laravel-audit`, convenciones de .NET/EF Core de `csharp-ef-audit`, reglas de `angular-mid-review`, bloqueos de event loops de `fastapi-audit`). **ESTÁ PROHIBIDO** que emitas recomendaciones genéricas de la industria. Si ves que estás a punto de dar un consejo como "Modulariza tu código" o "Nombra mejor las variables", detente; el feedback DEBE estar anclado en las reglas específicas de la skill que aplicaste para la versión detectada. No asumas buenas prácticas súper modernas si el stack detector confirmó que es una versión legacy (o viceversa).

### Fase 5: Publicar Comentarios (Secuencial, exacto)

**Compatibilidad Windows + ESM obligatoria:**
1. **PROHIBIDO usar `node -e` con scripts largos o interpolaciones complejas.**
2. Siempre crear un script shell/batch o ejecutar `gh api` directamente en secuencial.
3. No usar `/tmp`. Usar el directorio local `.claude_tmp/` creado bajo demanda.
4. Si creas JSON, serializar todo el payload y comentarios en un archivo `.json` iterativo.
5. Limpiar todos los archivos temporales en bloque de salida sin fallar si no existen.

**ANTES de publicar:**
1. ✅ Validar que línea existe en archivo (>0) y cae en rango de hunk añadido/modificado
2. ✅ Validar que archivo no es test/spec/dist
3. ✅ Validar que comentario cumple el formato obligatorio y ≤ 120 tokens
4. ✅ Consultar comentarios existentes y omitir duplicados exactos

**Script de publicación (con gh CLI):**

```bash
# Iterar por cada comentario validado y ejecutar:
gh api repos/{owner}/{repo}/pulls/${PR_ID}/comments \
  -f body="texto_del_comentario" \
  -f commit_id="headRefOid" \
  -f path="ruta/del/archivo" \
  -f line=123

# Chequear $? == 0. Si falla (429 Rate Limit, 422 Unprocessable), registrar y continuar con sleep.
```

### Fase 6: Cleanup y Limpieza (CRÍTICO)

Antes de finalizar y mostrar el reporte, **es obligatorio ejecutar un comando final en la terminal para borrar todos los rastros**. El agente CREADOR debe ejecutar este paso sin preguntar:

```bash
rm -rf .claude_tmp/ post_comments.cjs comments.json pr_meta.json pr_diff.txt
```

*(No confíes en que el script de JS limpie los archivos. Debes rodar este comando bash).*

### Fase 6.5: Guardar estado en Engram (Optimización Futura)

Usa la herramienta `mem_save` de forma silenciosa para registrar un resumen ejecutivo de lo que acabas de auditar en este PR. De esta forma, si te vuelven a pedir revisar este mismo `PR_ID`, la Fase 2.0 tendrá contexto y tú no harás comentarios duplicados. La memoria debe guardar el número de PR, un título breve y qué hallazgos críticos detectaste.

### Fase 7: Reportar Resultado

Al finalizar, **realiza una aproximación matemática del costo** (no posees la metadata real del sistema). Cuenta mentalmente la cantidad de palabras de código analizadas (input) y las palabras devueltas (output), multiplícalo por 1.3 para estimar los tokens y usa esta proporción: ($3 USD / 1M tokens input y $15 USD / 1M tokens output). Solo muestra al usuario un pequeño resumen ejecutivo con los hallazgos principales y este costo estimado. Omite los detalles largos a menos que el usuario pregunte.

```
📋 **PR #<numero> Revisado**
✅ Total comentarios publicados: <n>
🚨 Severidad principal encontrada: <Bug/Práctica/etc>
🪙 Consumo estimado: <input> tokens in / <output> tokens out (~$<costo> USD)

_(He auditado estirctamente el código relevante. Avísame si quieres un resumen de los hallazgos aquí en el chat)._
```

---

## Formato de Comentario (Obligatorio)

Usa siempre este formato exacto en cada inline comment:

```md
**⚠️ [Práctica]**

**Problema:** <entidad/interfaz> no incluye <campo>, campo usado en <función/flujo>.

**Por qué:** <impacto técnico real y concreto>.

**Sugerencia:**
<cambio mínimo recomendado>
```

Reglas:
- Siempre incluir las 4 secciones con sus títulos en negrita: `**⚠️ [Tipo]**`, `**Problema:**`, `**Por qué:**`, `**Sugerencia:**`.
- Mantener redacción breve y específica al cambio del diff.
- En `Sugerencia`, proponer el cambio mínimo viable. **Si la sugerencia incluye código (llaves, funciones, variables), DEBE ir envuelta en un bloque de código Markdown (\` ```ts \`)**.
- Nunca usar código inline si la sugerencia es más que una simple palabra.
- Si aplica otro tipo de hallazgo (Bug/Security/Performance), mantener el mismo formato y reemplazar solo la etiqueta inicial.

Ejemplo válido:

````md
**⚠️ [Práctica]**

**Problema:** `IVehicleInspectionQuestion` no incluye `assignedtoprovider`, campo usado en `calculateChanges()` y `onRowDataChanged`.

**Por qué:** Acceder a campos fuera de la interfaz rompe el tipado y puede ocultar errores.

**Sugerencia:**
```ts
assignedtoprovider?: boolean; // agregar a la interfaz
```
````

---

## Validaciones Críticas Antes de POST (Evitar gasto de tokens)

| Validación | Fallida | Acción | Por qué |
|-----------|---------|--------|---------|
| Token válido | POST a /2.0/user retorna 401 | ❌ Detener | Token inválido, no reintentar |
| Línea en hunk válido | Línea fuera de rango añadido/modificado | ⏭️ Omitir | Evita 400 por línea inválida |
| Archivo no es test/spec | Path contiene test/spec/dist | ⏭️ Omitir | No comentar archivos de test |
| Comentario en formato obligatorio | Faltan bloques Problema/Por qué/Sugerencia | ⏭️ Omitir | Mantiene calidad consistente |
| Comentario ≤ 120 tokens | Excede límite | ⏭️ Reescribir o Omitir | Evita ruido y sobrecosto |
| Sin duplicados | Ya existe en misma línea/path/texto normalizado | ⏭️ Omitir | Evita spam en re-ejecuciones |
| API disponible | POST retorna 500 | ❌ Detener | Server error, no reintentar |
| Línea existe en servidor | POST retorna 400 | ⏭️ Omitir | Línea cambió desde parse del diff |
| No exposición de secretos | TOKEN o Authorization en logs | ❌ Detener y sanitizar | Previene fuga de credenciales |

**Regla de Oro:** Si no pasó pre-validación → OMITIR sin reintentar (ahorra tokens)

**Criterio de fallo operativo:** si hay hallazgos críticos detectados y `publicados = 0`, reportar fallo de ejecución y pedir revisión manual antes de reintentar.
