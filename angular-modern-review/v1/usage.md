---
metadata:
  author: https://github.com/favelasquez
name: angular-modern-review
description: >
  AuditorÃ­a de cÃ³digo para Angular Moderno (Angular 13 en adelante, con foco en 15, 16, 17 y 18).
  La skill revisa que el cÃ³digo maximice la modernizaciÃ³n del framework aplicando: Standalone components,
  funciÃ³n inject() para dependencias, Signals en vez de RxJS excesivo donde aplique,
  Typed Forms (formularios tipados introducidos en Angular 14), control de flujo nativo 
  (@if en lugar de *ngIf introducido en v17), y la destrucciÃ³n correcta usando el gancho `takeUntilDestroyed()`.
---
metadata:
  author: https://github.com/favelasquez

# Angular Modern Review Skill (v13+)

Esta skill se aplica a ecosistemas listos para modernizaciÃ³n y en versiones recientes de Angular. Siempre revisaremos y guiaremos el cÃ³digo hacia los estÃ¡ndares mÃ¡s actuales para asegurar limpieza, performance (Zoneless readiness) y solidez de tipado.

## OBLIGACIONES DE PARADIGMA MODERNO

### 1. Standalone Components (Angular 14+)

Ya **no** deberÃ­amos usar `NgModules` gigantescos si el sistema permite componentes aislados.

```typescript
// âŒ MAL - Depender del app.module.ts gigante

// âœ… BIEN - Componentes independientes (Angular 14+)
@Component({
  selector: 'app-user',
  standalone: true,
  imports: [CommonModule, RouterModule, UserProfileComponent],
  templateUrl: './user.component.html'
})
export class UserComponent {}
```

### 2. FunciÃ³n `inject()` (Angular 14+)

Inyectar dependencias por el constructor produce "boilerplate" y herencias tortuosas (`super(...)`). Recomienda efusivamente `inject()`.

```typescript
// âŒ MAL - Constructor Bloat
constructor(
  private http: HttpClient, 
  private router: Router, 
  private dialog: DialogService
) {}

// âœ… BIEN - FunciÃ³n inject() (MÃ¡s composable y sin constructores largos)
private http = inject(HttpClient);
private router = inject(Router);
private dialog = inject(DialogService);
```

### 3. Autolimpieza de flujos: `takeUntilDestroyed()` (Angular 16+)

Nunca mÃ¡s uses el patrÃ³n obsoleto de declarar un `Subject` de destrucciÃ³n y un `ngOnDestroy()`.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

// âœ… BIEN - Usamos el contexto de inyecciÃ³n para limpiarlo automÃ¡ticamente
constructor() {
  this.http.get('/api/users').pipe(
    takeUntilDestroyed() // ðŸš€ AdiÃ³s a fugas de memoria
  ).subscribe();
}
```

### 4. Typed Forms (Angular 14+)

Los formularios en Angular moderno tienen Tipos de TypeScript estrictos de forma nativa. 

```typescript
// âŒ MAL - Todo suelto con `any` o variables flojas fallarÃ¡ en Angular 14+ strict mode
const form = new FormGroup({
  name: new FormControl() 
});

// âœ… BIEN - Formularios fuertemente tipados
const form = new FormGroup({
  name: new FormControl<string>('', { nonNullable: true }),
  age: new FormControl<number>(18)
});
```

### 5. Formularios Reactivos y Signals (Angular 16+)

Donde sea apropiado para manejar estado sÃ­ncrono local del componente, el uso de las Primitivas Reactivas (Signals) te lo agradecerÃ¡ la app. Recomienda `signal()`, `computed()` y `effect()`.

```typescript
// âœ… BIEN - Usar Signals para variables locales sin RxJS excesivo
users = signal<User[]>([]);
count = computed(() => this.users().length);

ngOnInit() {
  this.api.get().subscribe(data => this.users.set(data));
}
```

### 6. Flujo de Control Nativo en Templates (Angular 17+)

Cuando interactÃºes con el template (`.html`), recomienda migrar de las directivas estructurales viejas hacia el control de flujo integrado del compilador moderno que es infinitamente mÃ¡s rÃ¡pido.

```html
<!-- âŒ MAL - Angular < 17 -->
<div *ngIf="users.length; else noUsers">...</div>
<div *ngFor="let user of users; trackBy: trackByFn">...</div>

<!-- âœ… BIEN - Angular 17+ -->
@if (users().length > 0) {
  <div>{{ users()[0].name }}</div>
} @else {
  <div>Sin Usuarios</div>
}

@for (user of users(); track user.id) {
  <div>{{ user.name }}</div>
}
```
