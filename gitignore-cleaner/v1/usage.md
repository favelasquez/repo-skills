---
name: gitignore-cleaner
description: Analiza el proyecto, detecta carpetas innecesarias (build, cache, dependencias, temporales) y las agrega al .gitignore
license: Apache-2.0
metadata:
  author: https://github.com/favelasquez
  version: "1.0"
scope:
  - git
  - gitignore
  - cleanup
  - filesystem
permissions:
  allow:
    - filesystem
    - repositories
  deny:
    - database
---

# Gitignore Cleaner v1 � Detector de carpetas innecesarias

Eres un experto en gestión de repositorios. Cuando se invoque esta skill, **ejecuta el flujo completo** para detectar carpetas innecesarias y actualizar el `.gitignore`.

## Flujo a ejecutar

### 1. Detectar el stack del proyecto

Escanea el directorio raíz para identificar la tecnología:

| Archivo | Tecnología |
|---------|-----------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `angular.json` | Angular |
| `requirements.txt` / `pyproject.toml` | Python |
| `*.csproj` / `*.sln` | .NET |
| `go.mod` | Go |
| `pom.xml` / `build.gradle` | Java |
| `Gemfile` | Ruby |
| `Cargo.toml` | Rust |
| `composer.json` | PHP |

### 2. Listar todas las carpetas del proyecto

Usa la herramienta Bash para listar las carpetas de primer y segundo nivel:

```bash
find . -maxdepth 3 -type d -not -path '*/.git/*' | sort
```

### 3. Leer el .gitignore actual

Si existe un `.gitignore`, léelo para saber qué ya está ignorado. Si no existe, trátalo como vacío.

### 4. Clasificar las carpetas detectadas

Analiza cada carpeta encontrada y clasifícala según estas categorías:

#### Carpetas SIEMPRE innecesarias (añadir sin preguntar)
| Patrón | Descripción |
|--------|-------------|
| `node_modules/` | Dependencias Node.js |
| `.npm/` | Cache de npm |
| `dist/` | Build de producción |
| `build/` | Directorio de compilación |
| `out/` | Output de compilación |
| `.next/` | Build de Next.js |
| `.nuxt/` | Build de Nuxt.js |
| `.cache/` | Cache general |
| `__pycache__/` | Cache de Python |
| `.pytest_cache/` | Cache de pytest |
| `*.egg-info/` | Metadata de paquetes Python |
| `venv/` / `.venv/` / `env/` | Entornos virtuales Python |
| `bin/` / `obj/` | Build de .NET |
| `target/` | Build de Rust/Java |
| `.gradle/` | Cache de Gradle |
| `vendor/` | Dependencias PHP/Go |
| `coverage/` | Reportes de cobertura |
| `.nyc_output/` | Coverage de NYC |
| `logs/` | Logs generados |
| `tmp/` / `temp/` | Archivos temporales |
| `.DS_Store` | Metadata de macOS |
| `Thumbs.db` | Metadata de Windows |
| `.idea/` | Configuración de IntelliJ |
| `.vscode/` | Configuración local de VSCode (si no es intencional) |

#### Carpetas POTENCIALMENTE innecesarias (preguntar al usuario)
| Patrón | Descripción |
|--------|-------------|
| `uploads/` | Archivos subidos por usuarios |
| `static/` | Assets estáticos generados |
| `public/build/` | Build dentro de public |
| `*.local/` | Directorios locales |
| `generated/` | Código autogenerado |
| `fixtures/` | Datos de prueba grandes |
| `snapshots/` | Snapshots de tests |

### 5. Filtrar las ya ignoradas

Excluye del análisis cualquier carpeta que ya esté en el `.gitignore` actual.

### 6. Presentar el reporte al usuario

Muestra una tabla clara con las carpetas detectadas:

```
## Carpetas innecesarias detectadas

### �S& Se agregarán automáticamente al .gitignore:
| Carpeta | Motivo |
|---------|--------|
| node_modules/ | Dependencias de Node.js � no deben versionarse |
| dist/ | Build de producción � se regenera con npm run build |
| ...   | ... |

### � Carpetas para revisar (requieren confirmación):
| Carpeta | Motivo de duda |
|---------|---------------|
| uploads/ | Podría contener archivos que sí necesitas versionar |
| ...     | ... |

### �S& Ya están en .gitignore (sin cambios):
- .env
- ...
```

### 7. Confirmar con el usuario

Usa **AskUserQuestion** para preguntar:

> "He encontrado [N] carpetas innecesarias. ¿Confirmas que las agregue todas al .gitignore? También puedes indicar cuáles excluir."

Opciones a presentar:
- **Agregar todas** � añade todas las detectadas automáticamente + las potenciales
- **Solo las automáticas** � añade solo las de la categoría "siempre innecesarias"
- **Seleccionar manualmente** � el usuario escribe cuáles incluir/excluir

Si el usuario elige **Seleccionar manualmente**, usa **AskUserQuestion** de nuevo para pedirle la lista.

### 8. Actualizar el .gitignore

Basándote en la respuesta del usuario:

1. Lee el `.gitignore` existente (o crea uno nuevo si no existe)
2. Agrega una sección comentada al final con las nuevas entradas:

```gitignore

# === Agregado por gitignore-cleaner ===
# Carpetas de dependencias
node_modules/

# Carpetas de build
dist/
build/

# ... etc, agrupadas por categoría con comentarios
```

3. **No dupliques** entradas que ya existen en el archivo
4. **Preserva** todo el contenido original del `.gitignore`

### 9. Verificar con git

Si el proyecto tiene git inicializado, ejecuta:

```bash
git ls-files --others --directory --exclude-standard | grep "/$"
```

Esto muestra carpetas sin trackear. Si alguna de las carpetas que se van a ignorar ya tiene archivos trackeados en git, advierte al usuario:

> "�a�️ La carpeta `dist/` tiene archivos trackeados en git. Para ignorarla completamente necesitas ejecutar: `git rm -r --cached dist/`"

### 10. Reporte final

Informa al usuario:
- Cuántas entradas se agregaron al `.gitignore`
- Si hubo carpetas con archivos ya trackeados (con los comandos para desindexarlos)
- Si se creó el `.gitignore` desde cero o se actualizó uno existente

## Reglas
- **Nunca elimines** entradas existentes del `.gitignore`
- **Siempre agrupa** las nuevas entradas por categoría con comentarios descriptivos
- **No ignores** carpetas que contengan código fuente del proyecto (ej: `src/`, `app/`, `lib/`)
- Si encuentras un `.gitignore` con patrones conflictivos, menciónalos al usuario
- Usa rutas relativas en el `.gitignore` (sin `/` inicial a menos que sea necesario)



