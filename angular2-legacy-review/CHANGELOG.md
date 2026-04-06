# Registro de Versiones (Changelog) - angular2-legacy-review

## v1
**Funcionalidad principal actual:**
- **Auditoría de Sistemas Angular Clásicos (v2 - v5):** Diseñada para soportar proyectos altamente legados usando TypeScript ~2.0.0.
- **Prohibiciones Absolutas:** Frena el uso en TypeScript de los operadores "modernos" como nullish coalescing `??`, y el encadenamiento opcional `?.`, obligando en su lugar comprobaciones lógicas rústicas del tipo `(user && user.perfil === 0)`.
- **Prohibición del operador takeUntil:** Impulsa un manejo duro para de-suscripciones a arreglos locales de Subscription (`this.subscriptions.push(...)`).
- **Módulos Obsoletos:** Vigila y guía el correcto uso o reemplazo del viejo módulo `@angular/http`.