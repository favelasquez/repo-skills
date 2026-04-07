---
metadata:
  version: "1.0"
  author: https://github.com/favelasquez
name: angular-modern-review
description: >
  Auditoría de código para Angular Moderno (Angular 13 en adelante, con foco en 15, 16, 17 y 18).
  La skill revisa que el código maximice la modernización del framework aplicando: Standalone components,
  función inject() para dependencias, Signals en vez de RxJS excesivo donde aplique,
  Typed Forms (formularios tipados introducidos en Angular 14), control de flujo nativo 
  (@if en lugar de *ngIf introducido en v17), y la destrucción correcta usando el gancho `takeUntilDestroyed()`.
---
metadata:
  author: https://github.com/favelasquez

# Angular Modern Review Skill (v13+)

Esta skill se aplica a ecosistemas listos para modernización y en versiones recientes de Angular. Siempre revisaremos y guiaremos el código hacia los estándares más actuales para asegurar limpieza, performance (Zoneless readiness) y solidez de tipado.

## OBLIGACIONES DE PARADIGMA MODERNO

### 1. Standalone Components (Angular 14+)

Ya **no** deberíamos usar `NgModules` gigantescos si el sistema permite componentes aislados.

```typescript
// �R MAL - Depender del app.module.ts gigante

// �S& BIEN - Componentes independientes (Angular 14+)
@Component({
  selector: 'app-user',
  standalone: true,
  imports: [CommonModule, RouterModule, UserProfileComponent],
  templateUrl: './user.component.html'
})
export class UserComponent {}
```

### 2. Función `inject()` (Angular 14+)

Inyectar dependencias por el constructor produce "boilerplate" y herencias tortuosas (`super(...)`). Recomienda efusivamente `inject()`.

```typescript
// �R MAL - Constructor Bloat
constructor(
  private http: HttpClient, 
  private router: Router, 
  private dialog: DialogService
) {}

// �S& BIEN - Función inject() (Más composable y sin constructores largos)
private http = inject(HttpClient);
private router = inject(Router);
private dialog = inject(DialogService);
```

### 3. Autolimpieza de flujos: `takeUntilDestroyed()` (Angular 16+)

Nunca más uses el patrón obsoleto de declarar un `Subject` de destrucción y un `ngOnDestroy()`.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

// �S& BIEN - Usamos el contexto de inyección para limpiarlo automáticamente
constructor() {
  this.http.get('/api/users').pipe(
    takeUntilDestroyed() // �xa� Adiós a fugas de memoria
  ).subscribe();
}
```

### 4. Typed Forms (Angular 14+)

Los formularios en Angular moderno tienen Tipos de TypeScript estrictos de forma nativa. 

```typescript
// �R MAL - Todo suelto con `any` o variables flojas fallará en Angular 14+ strict mode
const form = new FormGroup({
  name: new FormControl() 
});

// �S& BIEN - Formularios fuertemente tipados
const form = new FormGroup({
  name: new FormControl<string>('', { nonNullable: true }),
  age: new FormControl<number>(18)
});
```

### 5. Formularios Reactivos y Signals (Angular 16+)

Donde sea apropiado para manejar estado síncrono local del componente, el uso de las Primitivas Reactivas (Signals) te lo agradecerá la app. Recomienda `signal()`, `computed()` y `effect()`.

```typescript
// �S& BIEN - Usar Signals para variables locales sin RxJS excesivo
users = signal<User[]>([]);
count = computed(() => this.users().length);

ngOnInit() {
  this.api.get().subscribe(data => this.users.set(data));
}
```

### 6. Flujo de Control Nativo en Templates (Angular 17+)

Cuando interactúes con el template (`.html`), recomienda migrar de las directivas estructurales viejas hacia el control de flujo integrado del compilador moderno que es infinitamente más rápido.

```html
<!-- �R MAL - Angular < 17 -->
<div *ngIf="users.length; else noUsers">...</div>
<div *ngFor="let user of users; trackBy: trackByFn">...</div>

<!-- �S& BIEN - Angular 17+ -->
@if (users().length > 0) {
  <div>{{ users()[0].name }}</div>
} @else {
  <div>Sin Usuarios</div>
}

@for (user of users(); track user.id) {
  <div>{{ user.name }}</div>
}
```
