---
metadata:
  author: https://github.com/favelasquez
name: angular-mid-review
description: >
  RevisiÃ³n experta de cÃ³digo para proyectos de Angular en sus versiones "medianas" (de la versiÃ³n 6 a la 12).
  En estas versiones se debe validar estrictamente el uso de RxJS 6+ a travÃ©s de operadores pipeables (pipe), 
  el uso exclusivo de HttpClient (en lugar de Http legacy), la definiciÃ³n de NgModules y el decorador
  @Injectable({ providedIn: 'root' }). No se deben sugerir Standalone Components, ni la funciÃ³n inject(),
  ni Signals, ni sintaxis de template moderna (@if) ya que NO son compatibles en este rango de versiones.
---
metadata:
  author: https://github.com/favelasquez

# Angular Mid-Versions Review Skill (v6 - v12)

Esta skill aplica exclusivamente a ecosistemas de **Angular 6 hasta Angular 12**.

## 1. Patrones Estrictos RxJS 6+ (OBLIGATORIOS)

A partir de Angular 6 / RxJS 6, se descontinuÃ³ la importaciÃ³n de operadores en el prototipo y se introdujeron los **operadores pipeables**.

```typescript
// âŒ MAL - Sintaxis legacy de RxJS < 5 (Rompe en Angular 6+)
import 'rxjs/add/operator/map';
source.map(x => x.id);

// âœ… BIEN - Uso de pipe()
import { map, filter, takeUntil } from 'rxjs/operators';

source.pipe(
  filter(x => !!x),
  map(x => x.id)
).subscribe();
```

## 2. Peticiones HTTP

`@angular/http` fue depreciado. Siempre se debe usar `@angular/common/http` (`HttpClient`).

```typescript
// âŒ MAL - Usar la librerÃ­a Http antigua
import { Http, Response } from '@angular/http'; // Â¡Eliminado completamente!

// âœ… BIEN - Usar HttpClient
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

constructor(private http: HttpClient) {}

// HttpClient ya parsea la respuesta JSON automÃ¡ticamente
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users');
}
```

## 3. Limpieza de Suscripciones (PrevenciÃ³n de Fugas de Memoria)

Como aquÃ­ NO existe `takeUntilDestroyed()`, la convenciÃ³n Ã³ptima y limpia es:

```typescript
import { Component, OnDestroy } from '@angular/core';
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({ ... })
export class MiComponente implements OnDestroy {
  private destroy$ = new Subject<void>();

  constructor(private api: ApiService) {
    this.api.getData().pipe(
      takeUntil(this.destroy$)
    ).subscribe(data => console.log(data));
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

## 4. InyecciÃ³n de Dependencias y Servicios

Se debe hacer mediante constructores y registrar los servicios con el flag mÃ¡gico `providedIn`.

```typescript
// âœ… BIEN - Servicios registrados en la raÃ­z automÃ¡ticamente sin aÃ±adir arreglos de Providers en Module
@Injectable({
  providedIn: 'root'
})
export class MiServicio { }

// âŒ PROHIBIDO
// OJO: "inject()" NO EXISTE en este ecosistema (llegÃ³ en la v14). 
// No recomiendes `const router = inject(Router);` bajo ninguna circunstancia.
```

## 5. MÃ³dulos y Enrutamiento (Lazy Loading)

Angular 8+ obliga al uso de promesas dinÃ¡micas para el ruteo, abandonando los strings mÃ¡gicos.
Tampoco existen los **Standalone Components** en estas versiones, asÃ­ que siempre se requiere de `NgModule`.

```typescript
// âŒ MAL - Angular < 8 importaciones mÃ¡gicas
{ path: 'admin', loadChildren: './admin/admin.module#AdminModule' }

// âœ… BIEN - Angular 8+ Dynamic Imports
{ path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule) }
```
