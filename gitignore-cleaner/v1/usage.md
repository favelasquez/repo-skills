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

# Gitignore Cleaner v1 â€” Detector de carpetas innecesarias

Eres un experto en gestiÃ³n de repositorios. Cuando se invoque esta skill, **ejecuta el flujo completo** para detectar carpetas innecesarias y actualizar el `.gitignore`.

## Flujo a ejecutar

### 1. Detectar el stack del proyecto

Escanea el directorio raÃ­z para identificar la tecnologÃ­a:

| Archivo | TecnologÃ­a |
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

Si existe un `.gitignore`, lÃ©elo para saber quÃ© ya estÃ¡ ignorado. Si no existe, trÃ¡talo como vacÃ­o.

### 4. Clasificar las carpetas detectadas

Analiza cada carpeta encontrada y clasifÃ­cala segÃºn estas categorÃ­as:

#### Carpetas SIEMPRE innecesarias (aÃ±adir sin preguntar)
| PatrÃ³n | DescripciÃ³n |
|--------|-------------|
| `node_modules/` | Dependencias Node.js |
| `.npm/` | Cache de npm |
| `dist/` | Build de producciÃ³n |
| `build/` | Directorio de compilaciÃ³n |
| `out/` | Output de compilaciÃ³n |
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
| `.idea/` | ConfiguraciÃ³n de IntelliJ |
| `.vscode/` | ConfiguraciÃ³n local de VSCode (si no es intencional) |

#### Carpetas POTENCIALMENTE innecesarias (preguntar al usuario)
| PatrÃ³n | DescripciÃ³n |
|--------|-------------|
| `uploads/` | Archivos subidos por usuarios |
| `static/` | Assets estÃ¡ticos generados |
| `public/build/` | Build dentro de public |
| `*.local/` | Directorios locales |
| `generated/` | CÃ³digo autogenerado |
| `fixtures/` | Datos de prueba grandes |
| `snapshots/` | Snapshots de tests |

### 5. Filtrar las ya ignoradas

Excluye del anÃ¡lisis cualquier carpeta que ya estÃ© en el `.gitignore` actual.

### 6. Presentar el reporte al usuario

Muestra una tabla clara con las carpetas detectadas:

```
## Carpetas innecesarias detectadas

### âœ… Se agregarÃ¡n automÃ¡ticamente al .gitignore:
| Carpeta | Motivo |
|---------|--------|
| node_modules/ | Dependencias de Node.js â€” no deben versionarse |
| dist/ | Build de producciÃ³n â€” se regenera con npm run build |
| ...   | ... |

### â“ Carpetas para revisar (requieren confirmaciÃ³n):
| Carpeta | Motivo de duda |
|---------|---------------|
| uploads/ | PodrÃ­a contener archivos que sÃ­ necesitas versionar |
| ...     | ... |

### âœ… Ya estÃ¡n en .gitignore (sin cambios):
- .env
- ...
```

### 7. Confirmar con el usuario

Usa **AskUserQuestion** para preguntar:

> "He encontrado [N] carpetas innecesarias. Â¿Confirmas que las agregue todas al .gitignore? TambiÃ©n puedes indicar cuÃ¡les excluir."

Opciones a presentar:
- **Agregar todas** â€” aÃ±ade todas las detectadas automÃ¡ticamente + las potenciales
- **Solo las automÃ¡ticas** â€” aÃ±ade solo las de la categorÃ­a "siempre innecesarias"
- **Seleccionar manualmente** â€” el usuario escribe cuÃ¡les incluir/excluir

Si el usuario elige **Seleccionar manualmente**, usa **AskUserQuestion** de nuevo para pedirle la lista.

### 8. Actualizar el .gitignore

BasÃ¡ndote en la respuesta del usuario:

1. Lee el `.gitignore` existente (o crea uno nuevo si no existe)
2. Agrega una secciÃ³n comentada al final con las nuevas entradas:

```gitignore

# === Agregado por gitignore-cleaner ===
# Carpetas de dependencias
node_modules/

# Carpetas de build
dist/
build/

# ... etc, agrupadas por categorÃ­a con comentarios
```

3. **No dupliques** entradas que ya existen en el archivo
4. **Preserva** todo el contenido original del `.gitignore`

### 9. Verificar con git

Si el proyecto tiene git inicializado, ejecuta:

```bash
git ls-files --others --directory --exclude-standard | grep "/$"
```

Esto muestra carpetas sin trackear. Si alguna de las carpetas que se van a ignorar ya tiene archivos trackeados en git, advierte al usuario:

> "âš ï¸ La carpeta `dist/` tiene archivos trackeados en git. Para ignorarla completamente necesitas ejecutar: `git rm -r --cached dist/`"

### 10. Reporte final

Informa al usuario:
- CuÃ¡ntas entradas se agregaron al `.gitignore`
- Si hubo carpetas con archivos ya trackeados (con los comandos para desindexarlos)
- Si se creÃ³ el `.gitignore` desde cero o se actualizÃ³ uno existente

## Reglas
- **Nunca elimines** entradas existentes del `.gitignore`
- **Siempre agrupa** las nuevas entradas por categorÃ­a con comentarios descriptivos
- **No ignores** carpetas que contengan cÃ³digo fuente del proyecto (ej: `src/`, `app/`, `lib/`)
- Si encuentras un `.gitignore` con patrones conflictivos, menciÃ³nalos al usuario
- Usa rutas relativas en el `.gitignore` (sin `/` inicial a menos que sea necesario)



