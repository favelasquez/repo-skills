---
metadata:
  version: "1.0"
  author: https://github.com/favelasquez
name: angular-mid-review
description: >
  Revisión experta de código para proyectos de Angular en sus versiones "medianas" (de la versión 6 a la 12).
  En estas versiones se debe validar estrictamente el uso de RxJS 6+ a través de operadores pipeables (pipe), 
  el uso exclusivo de HttpClient (en lugar de Http legacy), la definición de NgModules y el decorador
  @Injectable({ providedIn: 'root' }). No se deben sugerir Standalone Components, ni la función inject(),
  ni Signals, ni sintaxis de template moderna (@if) ya que NO son compatibles en este rango de versiones.
---
metadata:
  author: https://github.com/favelasquez

# Angular Mid-Versions Review Skill (v6 - v12)

Esta skill aplica exclusivamente a ecosistemas de **Angular 6 hasta Angular 12**.

## 1. Patrones Estrictos RxJS 6+ (OBLIGATORIOS)

A partir de Angular 6 / RxJS 6, se descontinuó la importación de operadores en el prototipo y se introdujeron los **operadores pipeables**.

```typescript
// �R MAL - Sintaxis legacy de RxJS < 5 (Rompe en Angular 6+)
import 'rxjs/add/operator/map';
source.map(x => x.id);

// �S& BIEN - Uso de pipe()
import { map, filter, takeUntil } from 'rxjs/operators';

source.pipe(
  filter(x => !!x),
  map(x => x.id)
).subscribe();
```

## 2. Peticiones HTTP

`@angular/http` fue depreciado. Siempre se debe usar `@angular/common/http` (`HttpClient`).

```typescript
// �R MAL - Usar la librería Http antigua
import { Http, Response } from '@angular/http'; // ¡Eliminado completamente!

// �S& BIEN - Usar HttpClient
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

constructor(private http: HttpClient) {}

// HttpClient ya parsea la respuesta JSON automáticamente
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users');
}
```

## 3. Limpieza de Suscripciones (Prevención de Fugas de Memoria)

Como aquí NO existe `takeUntilDestroyed()`, la convención óptima y limpia es:

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

## 4. Inyección de Dependencias y Servicios

Se debe hacer mediante constructores y registrar los servicios con el flag mágico `providedIn`.

```typescript
// �S& BIEN - Servicios registrados en la raíz automáticamente sin añadir arreglos de Providers en Module
@Injectable({
  providedIn: 'root'
})
export class MiServicio { }

// �R PROHIBIDO
// OJO: "inject()" NO EXISTE en este ecosistema (llegó en la v14). 
// No recomiendes `const router = inject(Router);` bajo ninguna circunstancia.
```

## 5. Módulos y Enrutamiento (Lazy Loading)

Angular 8+ obliga al uso de promesas dinámicas para el ruteo, abandonando los strings mágicos.
Tampoco existen los **Standalone Components** en estas versiones, así que siempre se requiere de `NgModule`.

```typescript
// �R MAL - Angular < 8 importaciones mágicas
{ path: 'admin', loadChildren: './admin/admin.module#AdminModule' }

// �S& BIEN - Angular 8+ Dynamic Imports
{ path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule) }
```
