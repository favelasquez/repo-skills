# Registro de Versiones (Changelog) - stack-detector

## v1
**Funcionalidad principal actual:**
- **Inspección de Archivos Clave:** Lee automáticamente metadatos de configuración (p. ej. `package.json`, `composer.json`, `*.csproj`, `requirements.txt`) para obtener versiones duras de las tecnologías en el repositorio objetivo.
- **Clasificador (Routing):** Enruta de forma condicional hacia las skills correctas (`laravel-audit`, `csharp-ef-audit`, `fastapi-audit`).
- **Lógica Trifásica de Angular:** Implementa reglas específicas para enrutar ecosistemas Angular basado en el `package.json` hacia sus eras concretas:
  - Entre Angular v2.x a v5.x ➜ `angular2-legacy-review`.
  - Entre Angular v6.x a v12.x ➜ `angular-mid-review`.
  - Desde Angular v13.x+ ➜ `angular-modern-review`.
- **Salida Estandarizada:** Genera el informe "Perfil de Stack" y entrega directrices al orquestador antes de que la auditoría siquiera empiece.