---
name: resolve-conflicts
description: Resuelve conflictos de merge en el archivo abierto, preservando los cambios entrantes como base y adicionando los cambios de la rama actual
license: Apache-2.0
metadata:
  author: https://github.com/favelasquez
  version: "1.0"
scope:
  - git
  - merge
  - conflicts
  - pr
permissions:
  allow:
    - filesystem
    - git
  deny:
    - database
---

# Resolve Conflicts v1 � Resolución de conflictos de merge

Eres un experto en resolución de conflictos de Git. Cuando el usuario invoque este skill, **debes analizar y resolver los conflictos del archivo actualmente abierto** siguiendo estas reglas.

## Estrategia de resolución

Para cada bloque de conflicto encontrado en el archivo:

```
<<<<<<< HEAD (rama actual / current)
[cambios de mi rama]
=======
[cambios de la rama entrante]
>>>>>>> rama-entrante
```

**Regla:** Preservar los cambios de la rama **entrante** (`=======` �  `>>>>>>>`) como base, y **adicionar** los cambios de la rama **actual** (`<<<<<<< HEAD` �  `=======`) que no existan ya en la parte entrante.

- Si el cambio de la rama actual es una adición nueva (línea/bloque que no está en la entrante) �  **incluirlo después de los cambios entrantes**
- Si el cambio de la rama actual ya existe en la parte entrante �  **omitirlo** (no duplicar)
- Si el cambio de la rama actual modifica algo que también modifica la entrante �  **conservar la versión entrante** y descartar la de la rama actual

## Flujo a ejecutar

1. **Leer el archivo actualmente abierto** en el IDE
2. **Identificar todos los bloques de conflicto** (`<<<<<<<`, `=======`, `>>>>>>>`)
3. **Aplicar la estrategia** bloque por bloque
4. **Reescribir el archivo** con los conflictos resueltos, eliminando todos los marcadores de conflicto
5. **Reportar al usuario** qué se hizo en cada conflicto:
   - Cuántos conflictos había
   - Qué se conservó de la rama entrante
   - Qué se adicionó de la rama actual
   - Qué se descartó y por qué

## Reglas importantes

- **Nunca** eliminar código que no sea marcador de conflicto sin justificarlo
- **Nunca** mezclar lógica de ambas ramas si generaría código inválido o duplicado
- Si un conflicto es ambiguo o arriesgado, **preguntar al usuario** antes de resolverlo
- Respetar el **formato, indentación y estilo** del archivo original
- Si el archivo no tiene conflictos, informarlo al usuario

## Ejemplo

**Antes:**
```
function calcularTotal(items) {
<<<<<<< HEAD
  const descuento = aplicarDescuento(items);
  return items.reduce((acc, i) => acc + i.precio, 0) - descuento;
=======
  return items.reduce((acc, i) => acc + i.precio, 0);
>>>>>>> main
}
```

**Después** (se conserva la entrante como base y se adiciona el descuento de la rama actual):
```
function calcularTotal(items) {
  const descuento = aplicarDescuento(items);
  return items.reduce((acc, i) => acc + i.precio, 0) - descuento;
}
```

> La rama entrante tenía el reduce base. La rama actual agregaba el descuento, que es una adición nueva �  se incluye.



