# Registro de Versiones (Changelog) - angular-mid-review

## v1
**Funcionalidad principal actual:**
- **Auditoría de Sistemas Angular de Transición (v6 - v12):** Diseñada para la era media de Angular donde existían los NgModules de forma obligatoria.
- **Transición a Operadores Pipeables:** Exige el uso del método `pipe(...)` con los operadores de RxJS en contra del estilo legacy.
- **Implementación de HttpClient:** Asegura que los repositorios hayan migrado a `@angular/common/http` descartando manualidades con `.map(res => res.json())`.
- **Restricciones "Hacia Arriba":** Castiga el uso de nuevas características que causarían bugs como _Standalone Components_, método `inject()`, Control de flujos visual nativo `@if / @for`, dado que dichas directrices aparecen en v14+.