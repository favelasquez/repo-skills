---
name: commits
description: Estándar para escribir mensajes de commit siguiendo Conventional Commits
license: Apache-2.0
metadata:
  author: transportationamerica-setup
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

# Commits v1 — Conventional Commits

Eres un asistente experto en Conventional Commits. Cuando el usuario invoque este skill, **debes ejecutar el flujo git completo** siguiendo estos pasos en orden:

## Flujo a ejecutar

1. Corre `git status` para ver qué archivos han cambiado
2. Corre `git diff` (y `git diff --cached` si hay staged changes) para entender los cambios
3. Genera un mensaje de commit siguiendo el estándar Conventional Commits (ver formato abajo)
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
- Subject máximo **72 caracteres**
- `type` y `description` son **obligatorios**; `scope` es recomendado
- Usar **imperativo presente**: "add feature", no "added feature"
- No terminar el subject con punto
- **El mensaje de commit debe estar siempre en inglés**

### Types

| Type | Cuándo usarlo |
|---|---|
| `feat` | Nueva funcionalidad para el usuario |
| `fix` | Corrección de un bug |
| `docs` | Cambios solo en documentación |
| `style` | Formato, espacios — sin cambio de lógica |
| `refactor` | Refactorización sin nueva feature ni bug fix |
| `test` | Añadir o corregir tests |
| `chore` | Mantenimiento, dependencias, configuración |
| `perf` | Mejoras de rendimiento |
| `ci` | Cambios en configuración de CI/CD |
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
- Un commit = un solo cambio lógico. Si necesitas "and", probablemente son dos commits
- El body explica el **qué y por qué**, no el cómo
- Infórmale al usuario qué mensaje de commit generaste y el resultado de cada comando ejecutado
