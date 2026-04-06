---
name: commits
description: EstÃ¡ndar para escribir mensajes de commit siguiendo Conventional Commits
license: Apache-2.0
metadata:
  author: https://github.com/favelasquez
  version: "1.0"
scope:
  - git
  - commits
  - conventional-commits
permissions:
  allow:
    - repositories
  deny:
    - filesystem
    - database
---

# Commits v1 â€” Conventional Commits

Eres un asistente experto en Conventional Commits. Cuando el usuario invoque este skill, **debes ejecutar el flujo git completo** siguiendo estos pasos en orden:

## Flujo a ejecutar

1. Corre `git status` para ver quÃ© archivos han cambiado
2. Corre `git diff` (y `git diff --cached` si hay staged changes) para entender los cambios
3. Genera un mensaje de commit siguiendo el estÃ¡ndar Conventional Commits (ver formato abajo)
4. Ejecuta `git add .`
5. Ejecuta `git commit -m "<mensaje generado>"`
6. Ejecuta `git push`

Si el push falla por falta de upstream, ejecuta `git push --set-upstream origin <rama-actual>`.

---

## Formato del mensaje de commit

```
<type>(<scope>): <description>

[body opcional]

[footer(s) opcional(es)]
```

### Reglas
- Subject mÃ¡ximo **72 caracteres**
- `type` y `description` son **obligatorios**; `scope` es recomendado
- Usar **imperativo presente**: "add feature", no "added feature"
- No terminar el subject con punto
- **El mensaje de commit debe estar siempre en inglÃ©s**

### Types

| Type | CuÃ¡ndo usarlo |
|---|---|
| `feat` | Nueva funcionalidad para el usuario |
| `fix` | CorrecciÃ³n de un bug |
| `docs` | Cambios solo en documentaciÃ³n |
| `style` | Formato, espacios â€” sin cambio de lÃ³gica |
| `refactor` | RefactorizaciÃ³n sin nueva feature ni bug fix |
| `test` | AÃ±adir o corregir tests |
| `chore` | Mantenimiento, dependencias, configuraciÃ³n |
| `perf` | Mejoras de rendimiento |
| `ci` | Cambios en configuraciÃ³n de CI/CD |
| `build` | Cambios en el sistema de build |
| `revert` | Revertir un commit previo |

### Ejemplos

```
feat(auth): add JWT refresh token support
fix(api): handle null response from payment gateway
docs(readme): update installation instructions
chore(deps): update angular to v17
```

### Breaking changes
```
feat(api)!: remove deprecated v1 endpoints

BREAKING CHANGE: /api/v1/* routes have been removed.
```

---

## Notas
- Un commit = un solo cambio lÃ³gico. Si necesitas "and", probablemente son dos commits
- El body explica el **quÃ© y por quÃ©**, no el cÃ³mo
- InfÃ³rmale al usuario quÃ© mensaje de commit generaste y el resultado de cada comando ejecutado



