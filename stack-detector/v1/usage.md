---
metadata:
  author: https://github.com/favelasquez
name: stack-detector
description: >
  Detecta automáticamente el stack tecnológico de un proyecto (Angular, Laravel, Python/FastAPI,
  C#/.NET) inspeccionando archivos de configuración como package.json, composer.json,
  requirements.txt, pyproject.toml, *.csproj, global.json, angular.json, etc.
  Activar SIEMPRE que el usuario pida revisar, auditar o analizar código sin haber especificado
  versiones, o cuando diga ""revisa mi proyecto"", ""audita mi código"", ""tengo este proyecto"",
  ""qué tan bien está escrito"", ""dame feedback de mi código"", ""revisa esto"" y adjunte archivos
  o comparta rutas. También activar cuando el usuario diga frases como ""¿qué tecnologías tengo?"",
  ""¿qué versión uso?"", ""¿cómo está mi stack?"". Esta skill identifica PRIMERO el stack exacto con
  versiones, construye un perfil de tecnología, y luego activa la skill de auditoría correcta
  con el contexto de versión adecuado para dar comentarios precisos y relevantes.
---
metadata:
  author: https://github.com/favelasquez

# Stack Detector

Esta skill tiene DOS responsabilidades:
1. **Detectar** el stack tecnológico con versiones exactas
2. **Activar** la skill de auditoría correcta con ese contexto de versión

---
metadata:
  author: https://github.com/favelasquez

## PASO 1 — Detección del Stack

### Archivos que debes leer primero (en orden)

Cuando el usuario comparte archivos o una ruta de proyecto, busca y lee:

| Archivo | Tecnología |
|---|---|
| "package.json" | Angular, Node.js, dependencias JS |
| "angular.json" | Versión Angular CLI / workspace config |
| "composer.json" | Laravel, PHP y paquetes |
| "artisan" (archivo) | Confirma proyecto Laravel |
| "requirements.txt" | Python y versiones de paquetes |
| "pyproject.toml" | Python moderno (Poetry, PEP 621) |
| "Pipfile" | Python con Pipenv |
| "*.csproj" | C# / .NET — contiene <TargetFramework> |
| "global.json" | Versión del SDK de .NET |
| "Directory.Build.props" | Versión centralizada en .NET |
| ".nvmrc" / ".node-version" | Versión de Node.js |
| "Dockerfile" / "docker-compose.yml" | Versiones de runtime / imagen base |

> Si el usuario comparte código suelto sin archivos de config, pide explícitamente el archivo de dependencias correspondiente. No asumas versiones.

---
metadata:
  author: https://github.com/favelasquez

## PASO 2 — Construir el Perfil de Stack

Una vez leídos los archivos, construye y muestra al usuario un **Perfil de Stack** con este formato:

`
📦 PERFIL DE STACK DETECTADO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔷 [Tecnología Principal]
   Framework:     [nombre] [versión]
   Lenguaje:      [nombre] [versión]
   Runtime:       [nombre] [versión]

📚 Dependencias clave:
   [paquete]: [versión]
   [paquete]: [versión]
   ...

⚠️  Notas de versión:
   [cualquier dato importante: EOL, LTS, breaking changes conocidos]

🔗 Skills de auditoría que aplicarán:
   → [nombre-skill] (por [razón])
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
`

---
metadata:
  author: https://github.com/favelasquez

## PASO 3 — Activar la Skill de Auditoría Correcta

Con el perfil construido, **lee el archivo de referencia de versión correspondiente** y procede a la auditoría. La selección es:

### Angular / TypeScript
→ Lee "references/angular-versions.md"

- Angular 2.x hasta 5.x → activa skill `angular2-legacy-review` (TypeScript antiguo, vieja librería Http, y RxJS 5)
- Angular 6–12 → activa skill `angular-mid-review` (Cambio a RxJS 6 pipe() y HttpClient, prohíbe funciones de > Angular 14)
- Angular 13+ → activa skill `angular-modern-review` (Standalone components, inject(), Signals, @if/@for templating, y takeUntilDestroyed)

### Laravel / PHP
→ Lee "references/laravel-versions.md"

- Laravel 6–8 → usa sección "legacy" (sin Jetstream nativo, sin Vite)
- Laravel 9–10 → usa sección "modern" (Vite, PHP 8.1+)
- Laravel 11+ → usa sección "latest" (nueva estructura de directorios)
- En todos los casos activa también skill "laravel-audit"

### Python / FastAPI / Django
→ Lee "references/python-versions.md"

- FastAPI cualquier versión → activa skill "fastapi-audit"
- Django: usa guías de "references/python-versions.md" sección "django"
- Python < 3.8 → marca como EOL, advierte

### C# / .NET / Entity Framework
→ Lee "references/dotnet-versions.md"

- .NET Framework 4.x + EF 6.x → activa skill "csharp-ef-audit" sección legacy
- .NET Core 3.1 / .NET 5 → activa skill "csharp-ef-audit" sección core
- .NET 6/7/8/9 + EF Core → activa skill "csharp-ef-audit" sección modern

---
metadata:
  author: https://github.com/favelasquez

## PASO 4 — Auditoría con Contexto de Versión

Al hacer la auditoría, **siempre prefija cada observación con su relevancia de versión**:

`
⚠️  [Angular 15+] El uso de "ngModule" en este componente es legacy.
    → En Angular 15+ se prefieren Standalone Components.

✅  [Laravel 10] Este uso de "Route::resource()" es correcto para esta versión.

🔴  [.NET 6+] "DbContext" no debe ser instanciado manualmente aquí.
    → Usa inyección de dependencias con "AddDbContext<>()".

💡  [Python 3.9] Puedes simplificar este type hint: "list[str]" en vez de "List[str]".
`

---
metadata:
  author: https://github.com/favelasquez

## PASO 5 — Stack Mixto / Monorepo

Si el proyecto tiene múltiples tecnologías (ej: frontend Angular + backend Laravel + microservicio Python):

1. Construye un perfil por cada sub-proyecto
2. Aplica la skill correcta a cada parte
3. Marca claramente en la auditoría a qué capa pertenece cada comentario:
   `
   [FRONTEND] [BACKEND] [API] [INFRA]
   `

---
metadata:
  author: https://github.com/favelasquez

## Preguntas de Clarificación

Si faltan archivos de configuración, pregunta directamente:

- "¿Puedes compartir tu "package.json"? Necesito saber qué versión de Angular usas."
- "¿Tienes un "composer.json"? Quiero ver la versión de Laravel antes de auditar."
- "¿Cuál es tu versión de Python? ("python --version") y ¿usas "requirements.txt" o "pyproject.toml"?"
- "¿Tu proyecto de .NET tiene un archivo ".csproj"? Necesito el "<TargetFramework>".

No improvises versiones. Si no sabes la versión exacta, dilo y pide confirmación.

---
metadata:
  author: https://github.com/favelasquez

## Archivos de Referencia

Lee el archivo correspondiente ANTES de auditar cada tecnología:

- "references/angular-versions.md" — Diferencias clave entre versiones de Angular
- "references/laravel-versions.md" — Diferencias clave entre versiones de Laravel  
- "references/python-versions.md" — Diferencias entre versiones Python/FastAPI/Django
- "references/dotnet-versions.md" — Diferencias entre versiones .NET/EF Core

