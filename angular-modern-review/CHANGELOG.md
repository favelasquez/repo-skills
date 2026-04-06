# Registro de Versiones (Changelog) - angular-modern-review

## v1
**Funcionalidad principal actual:**
- **Auditoría de Alta Vanguardia para Angular Moderno (v13+ / v18):** Enfatiza los "nuevos dogmas" que recomiendan el equipo de Angular.
- **Componentes Aislados y Funcionales:** Presiona las sugerencias hacia _Standalone Components_ para matar el boiler plate de los NgModule. Recomienda el uso del patrón `inject(Service)` en vez de inyectar dependencias pesadas en el constructor.
- **Reactive Primitives:** Observa el código y sugiere usar estado con `Signal` (`signal`, `computed`, `effect`) en lugar del uso abultado tradicional de RxJS `BehaviorSubject`.
- **Evolución del Templating:** Sugiere refactorizaciones continuas que eliminen los engorrosos `*ngIf/*ngFor` en favor de su renderizador nativo (`@if / @for`).
- **Antifugas RxJS-Interop:** Pide y sugiere el uso constante del hook `takeUntilDestroyed()`.