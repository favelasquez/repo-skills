---
metadata:
  author: https://github.com/favelasquez
name: angular2-legacy-review
description: >
  RevisiÃ³n experta de cÃ³digo Angular en versiÃ³n 2.4.x (legacy) con TypeScript ~2.0.0,
  ag-grid 18.x, RxJS 5.x, Bootstrap 3.x y el stack completo de trip2cli. Usar esta
  skill siempre que el usuario pida revisar, auditar, corregir o mejorar cÃ³digo Angular
  de versiones antiguas (Angular 2, Angular 4), o cuando mencione paquetes como
  ag-grid 18, rxjs 5, ng-bootstrap alpha, ngx-toastr 5, angular-cli 1.0 beta,
  mydatepicker, ng2-bs3-modal, sweetalert2 7.x o cualquier librerÃ­a del stack legacy.
  TambiÃ©n activar cuando el usuario pida ejemplos de componentes, servicios,
  suscripciones, interfaces, pipes o directivas en Angular 2. Esta skill provee
  ejemplos funcionales reales, patrones correctos para este stack especÃ­fico y
  antipatrones a evitar.
---
metadata:
  author: https://github.com/favelasquez

# Angular 2.4.x Legacy Review Skill

## Stack de referencia (trip2cli)

| TecnologÃ­a | VersiÃ³n |
|---|---|
| Angular | ^2.4.5 |
| Angular CLI | 1.0.0-beta.32 |
| TypeScript | ~2.0.0 |
| RxJS | ^5.1.0 |
| ag-grid | ^18.1.2 |
| ag-grid-angular | ^18.1.0 |
| ag-grid-enterprise | ^18.1.0 |
| @ng-bootstrap/ng-bootstrap | 1.0.0-alpha.14 |
| Bootstrap | ^3.3.7 |
| zone.js | ^0.7.6 |
| ng2-bs3-modal | ^0.15.0 |
| ngx-toastr | ^5.2.3 |
| sweetalert2 | ^7.33.1 |
| moment | ^2.30.0 |
| socket.io-client | ^2.2.0 |
| firebase | ^8.1.2 |
| jquery | ^3.5.0 |
| Node.js recomendado | 10.19.0 (nvm) |

> Para detalles profundos de cada tema, ver los archivos en `references/`:
> - `references/componentes-servicios.md` â€” Componentes, servicios, DI
> - `references/rxjs5-suscripciones.md` â€” RxJS 5, Subscription, Subject
> - `references/ag-grid-18.md` â€” ag-grid 18 con Angular 2
> - `references/interfaces-tipos.md` â€” Interfaces, tipos, enums TypeScript 2.0
> - `references/routing-guards.md` â€” Router Angular 2, Guards, Resolvers
> - `references/librerias-extras.md` â€” ng-bootstrap, toastr, sweetalert2, moment, firebase

---
metadata:
  author: https://github.com/favelasquez

## Reglas de oro para este stack

### 1. Decoradores obligatorios en componentes
```typescript
// âœ… CORRECTO â€” Angular 2.4.x
@Component({
  selector: 'app-mi-componente',
  templateUrl: './mi-componente.component.html',
  styleUrls: ['./mi-componente.component.css'],
  providers: []  // Solo si el servicio es local al componente
})
export class MiComponente implements OnInit, OnDestroy {
  // ...
}
```

### 2. Ciclo de vida correcto
```typescript
// âœ… CORRECTO â€” Siempre implementar la interfaz
export class MiComponente implements OnInit, OnDestroy {
  ngOnInit(): void { }
  ngOnDestroy(): void { }
}

// âŒ MAL â€” Sin interfaz implementada
export class MiComponente {
  ngOnInit() { }   // Funciona pero no es type-safe
}
```

### 3. InyecciÃ³n de dependencias
```typescript
// âœ… Angular 2 â€” Inyectar en constructor
constructor(
  private miServicio: MiServicio,
  private router: Router,
  private route: ActivatedRoute
) {}

// âŒ MAL en Angular 2 â€” inject() no existe aÃºn
// const servicio = inject(MiServicio);  // Solo Angular 14+
```

### 4. Ganchos y caracterÃ­sticas modernas de TS PROHIBIDAS
En TypeScript ~2.0.0 (usado en Angular 2.4.x) **NO EXISTEN** muchas caracterÃ­sticas y ganchos que damos por sentado hoy en dÃ­a.
```typescript
// âŒ MAL â€” Optional Chaining (?.)
const nombre = user?.perfil?.nombre; // RomperÃ¡ la transpilaciÃ³n.

// âŒ MAL â€” Nullish Coalescing (??)
const cantidad = total ?? 0; // RomperÃ¡ la transpilaciÃ³n.

// âœ… BIEN â€” Validaciones al estilo Legacy
const nombre = (user && user.perfil) ? user.perfil.nombre : ''; // o un valor por defecto seguro
const cantidad = (total || total === 0) ? total : 0; // Ojo en legacy: `||` trata `0` como falsy, por eso se requiere el `=== 0`. Evitar usar solo `!total` si `0` es un valor numÃ©rico vÃ¡lido.
```

---
metadata:
  author: https://github.com/favelasquez

## Patrones de Suscripciones (crÃ­tico para evitar memory leaks)

### PatrÃ³n 1: Array de suscripciones + ngOnDestroy
```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs/Subscription';

@Component({ selector: 'app-ejemplo', template: '' })
export class EjemploComponent implements OnInit, OnDestroy {

  // âœ… Array tipado de suscripciones
  private subscriptions: Subscription[] = [];

  ngOnInit(): void {
    // âœ… Agregar cada suscripciÃ³n al array
    this.subscriptions.push(
      this.miServicio.getData().subscribe(data => {
        this.items = data;
      })
    );

    this.subscriptions.push(
      this.route.params.subscribe(params => {
        this.id = params['id'];
      })
    );

    this.subscriptions.push(
      this.otroServicio.eventos$.subscribe(event => {
        this.procesarEvento(event);
      })
    );
  }

  ngOnDestroy(): void {
    // âœ… Matar TODAS las suscripciones en un bucle
    this.subscriptions.forEach(sub => sub.unsubscribe());
    // Opcional: limpiar el array
    this.subscriptions = [];
  }
}
```

### PatrÃ³n 2: Subscription Ãºnica con .add()
```typescript
import { Subscription } from 'rxjs/Subscription';

private masterSub: Subscription = new Subscription();

ngOnInit(): void {
  this.masterSub.add(
    this.servicio.stream$.subscribe(v => this.valor = v)
  );
  this.masterSub.add(
    this.otro$.subscribe(x => this.x = x)
  );
}

ngOnDestroy(): void {
  this.masterSub.unsubscribe(); // Cancela todas las hijas
}
```

### PatrÃ³n 3: PROHIBIDO USAR takeUntil o ganchos modernos
```typescript
// âŒ MAL â€” EL PATRÃ“N `takeUntil()` o `takeUntilDestroyed()` NO ESTÃ DISPONIBLE EN ESTE STACK
// Tampoco estÃ¡n disponibles los ganchos modernos (hooks) en TS.
// âŒ NO HAGAS ESTO:
import { takeUntil } from 'rxjs/operators'; // No funciona / No existe en esta versiÃ³n
import { takeUntilDestroyed } from '@angular/core/rxjs-interop'; // NO DISPONIBLE

// ðŸŸ¢ SOLUCIÃ“N: Usar SIEMPRE el PatrÃ³n 1 (Array de suscripciones) o el PatrÃ³n 2 (Subscription.add()),
// y limpiar las suscripciones manualmente en el mÃ©todo `ngOnDestroy()`.
```

---
metadata:
  author: https://github.com/favelasquez

## Interfaces y Tipos

```typescript
// âœ… CORRECTO â€” Interfaces en archivos separados (models/)
// src/app/models/usuario.model.ts
export interface IUsuario {
  id: number;
  nombre: string;
  email: string;
  rol: RolUsuario;
  activo: boolean;
  fechaCreacion?: Date;        // Opcional con ?
  metadata?: { [key: string]: any };
}

export enum RolUsuario {
  ADMIN = 'ADMIN',
  OPERADOR = 'OPERADOR',
  VISOR = 'VISOR'
}

export interface IApiResponse<T> {
  data: T;
  total: number;
  pagina: number;
  error?: string;
}

// âœ… Uso en componente
export class UsuarioComponent {
  usuario: IUsuario;
  usuarios: IUsuario[] = [];
  response: IApiResponse<IUsuario[]>;
}
```

---
metadata:
  author: https://github.com/favelasquez

## Servicios HTTP (Angular 2 â€” Http, no HttpClient)

```typescript
// âœ… Angular 2.4.x usa Http, NO HttpClient (ese es Angular 4.3+)
import { Injectable } from '@angular/core';
import { Http, Headers, RequestOptions, Response } from '@angular/http';
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/catch';
import 'rxjs/add/observable/throw';

@Injectable()
export class UsuarioService {

  private apiUrl = 'https://api.ejemplo.com';

  constructor(private http: Http) {}

  getUsuarios(): Observable<IUsuario[]> {
    const headers = new Headers({ 'Content-Type': 'application/json' });
    const options = new RequestOptions({ headers });

    return this.http.get(`${this.apiUrl}/usuarios`, options)
      .map((res: Response) => res.json() as IUsuario[])
      .catch((error: Response) => Observable.throw(error.json()));
  }

  crearUsuario(usuario: IUsuario): Observable<IUsuario> {
    const headers = new Headers({ 'Content-Type': 'application/json' });
    const options = new RequestOptions({ headers });
    const body = JSON.stringify(usuario);

    return this.http.post(`${this.apiUrl}/usuarios`, body, options)
      .map((res: Response) => res.json() as IUsuario)
      .catch((error: Response) => Observable.throw(error.json()));
  }
}
```

---
metadata:
  author: https://github.com/favelasquez

## ag-grid 18 â€” Uso correcto

```typescript
// âœ… Importar en NgModule
import { AgGridModule } from 'ag-grid-angular';
// En imports: AgGridModule.withComponents([])

// âœ… Componente con ag-grid 18
import { GridOptions, ColDef, GridApi, ColumnApi } from 'ag-grid';

@Component({
  selector: 'app-tabla',
  template: `
    <ag-grid-angular
      class="ag-theme-balham"
      [rowData]="rowData"
      [columnDefs]="columnDefs"
      [gridOptions]="gridOptions"
      (gridReady)="onGridReady($event)"
      style="width: 100%; height: 400px;">
    </ag-grid-angular>
  `
})
export class TablaComponent implements OnInit, OnDestroy {

  columnDefs: ColDef[] = [
    { headerName: 'Nombre', field: 'nombre', sortable: true, filter: true },
    { headerName: 'Email', field: 'email', width: 200 },
    { headerName: 'Activo', field: 'activo', cellRenderer: (p) => p.value ? 'SÃ­' : 'No' }
  ];

  rowData: IUsuario[] = [];
  gridOptions: GridOptions = {
    rowSelection: 'multiple',
    suppressRowClickSelection: true,
    enableColResize: true,    // ag-grid 18 â€” NO resizable en colDef
    enableSorting: true,      // ag-grid 18 â€” NO sortable global
    enableFilter: true        // ag-grid 18 â€” NO filter global
  };

  private gridApi: GridApi;
  private columnApiGrid: ColumnApi;
  private subscriptions: Subscription[] = [];

  onGridReady(params: any): void {
    this.gridApi = params.api;
    this.columnApiGrid = params.columnApi;
    this.gridApi.sizeColumnsToFit();
  }

  ngOnDestroy(): void {
    this.subscriptions.forEach(s => s.unsubscribe());
  }
}
```

> âš ï¸ **ag-grid 18 vs versiones modernas**: En v18, `sortable`, `resizable` y `filter` se configuran en `gridOptions`, NO en cada `ColDef`. Esto cambiÃ³ en v20+.

---
metadata:
  author: https://github.com/favelasquez

## RxJS 5 â€” Importaciones correctas

```typescript
// âœ… RxJS 5 â€” importar operadores individualmente (NO importar todo)
import { Observable } from 'rxjs/Observable';
import { Subject } from 'rxjs/Subject';
import { BehaviorSubject } from 'rxjs/BehaviorSubject';
import { Subscription } from 'rxjs/Subscription';

// Operadores â€” aÃ±adir al prototipo
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/filter';
import 'rxjs/add/operator/switchMap';
import 'rxjs/add/operator/debounceTime';
import 'rxjs/add/operator/distinctUntilChanged';
import 'rxjs/add/operator/takeUntil';
import 'rxjs/add/operator/catch';
import 'rxjs/add/observable/of';
import 'rxjs/add/observable/from';
import 'rxjs/add/observable/throw';
import 'rxjs/add/observable/combineLatest';

// âŒ MAL â€” importaciÃ³n moderna (RxJS 6+, no existe en RxJS 5)
// import { map, filter } from 'rxjs/operators';
// import { Observable } from 'rxjs';
```

---
metadata:
  author: https://github.com/favelasquez

## Checklist de revisiÃ³n de cÃ³digo

Al revisar cualquier componente Angular 2.4.x verificar:

- [ ] Â¿Implementa `OnDestroy` si tiene suscripciones?
- [ ] Â¿Todas las suscripciones se cancelan en `ngOnDestroy`?
- [ ] Â¿Usa `Http` (no `HttpClient`) para peticiones HTTP?
- [ ] Â¿Importa RxJS operadores individualmente (`rxjs/add/operator/...`)?
- [ ] Â¿Los servicios estÃ¡n declarados en `providers` del mÃ³dulo o componente?
- [ ] Â¿Las interfaces usan prefijo `I` o convenciÃ³n clara?
- [ ] Â¿Ag-grid usa `GridOptions` con `enableSorting/enableFilter` (no en ColDef)?
- [ ] Â¿Evita `any` excesivo con TypeScript 2.0?
- [ ] Â¿Los mÃ³dulos importan solo lo necesario?
- [ ] Â¿Las rutas usan `RouterModule.forRoot()` en AppModule y `forChild()` en mÃ³dulos hijos?

---
metadata:
  author: https://github.com/favelasquez

Leer los archivos `references/` para ejemplos mÃ¡s detallados por tema.

