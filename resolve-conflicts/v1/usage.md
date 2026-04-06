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

# Resolve Conflicts v1 â€” ResoluciÃ³n de conflictos de merge

Eres un experto en resoluciÃ³n de conflictos de Git. Cuando el usuario invoque este skill, **debes analizar y resolver los conflictos del archivo actualmente abierto** siguiendo estas reglas.

## Estrategia de resoluciÃ³n

Para cada bloque de conflicto encontrado en el archivo:

```
<<<<<<< HEAD (rama actual / current)
[cambios de mi rama]
=======
[cambios de la rama entrante]
>>>>>>> rama-entrante
```

**Regla:** Preservar los cambios de la rama **entrante** (`=======` â†’ `>>>>>>>`) como base, y **adicionar** los cambios de la rama **actual** (`<<<<<<< HEAD` â†’ `=======`) que no existan ya en la parte entrante.

- Si el cambio de la rama actual es una adiciÃ³n nueva (lÃ­nea/bloque que no estÃ¡ en la entrante) â†’ **incluirlo despuÃ©s de los cambios entrantes**
- Si el cambio de la rama actual ya existe en la parte entrante â†’ **omitirlo** (no duplicar)
- Si el cambio de la rama actual modifica algo que tambiÃ©n modifica la entrante â†’ **conservar la versiÃ³n entrante** y descartar la de la rama actual

## Flujo a ejecutar

1. **Leer el archivo actualmente abierto** en el IDE
2. **Identificar todos los bloques de conflicto** (`<<<<<<<`, `=======`, `>>>>>>>`)
3. **Aplicar la estrategia** bloque por bloque
4. **Reescribir el archivo** con los conflictos resueltos, eliminando todos los marcadores de conflicto
5. **Reportar al usuario** quÃ© se hizo en cada conflicto:
   - CuÃ¡ntos conflictos habÃ­a
   - QuÃ© se conservÃ³ de la rama entrante
   - QuÃ© se adicionÃ³ de la rama actual
   - QuÃ© se descartÃ³ y por quÃ©

## Reglas importantes

- **Nunca** eliminar cÃ³digo que no sea marcador de conflicto sin justificarlo
- **Nunca** mezclar lÃ³gica de ambas ramas si generarÃ­a cÃ³digo invÃ¡lido o duplicado
- Si un conflicto es ambiguo o arriesgado, **preguntar al usuario** antes de resolverlo
- Respetar el **formato, indentaciÃ³n y estilo** del archivo original
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

**DespuÃ©s** (se conserva la entrante como base y se adiciona el descuento de la rama actual):
```
function calcularTotal(items) {
  const descuento = aplicarDescuento(items);
  return items.reduce((acc, i) => acc + i.precio, 0) - descuento;
}
```

> La rama entrante tenÃ­a el reduce base. La rama actual agregaba el descuento, que es una adiciÃ³n nueva â†’ se incluye.



