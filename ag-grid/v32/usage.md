---
name: ag-grid
description: High performance data grid for web applications
license: Apache-2.0
metadata:
  author: https://github.com/favelasquez
  version: "32.0"
scope:
  - ag-grid
  - angular
  - react
permissions:
  allow:
    - http
    - repositories
  deny:
    - filesystem
    - database
---

# AG Grid v32 â€” Usage

## Overview
AG Grid v32 introduces a unified `api` object (Column API merged into Grid API), the new
Theme Params system replacing CSS class-based themes, and improved TypeScript generics
for strongly-typed row data.

## Setup

```typescript
import { AgGridAngular } from 'ag-grid-angular';
import { ColDef, GridReadyEvent, themeQuartz } from 'ag-grid-community';
```

No module registration needed â€” standalone component model:

```typescript
@Component({
  imports: [AgGridAngular],
  ...
})
export class MyComponent {}
```

## Basic usage

```typescript
theme = themeQuartz;

columnDefs: ColDef<Product>[] = [
  { field: 'name', sortable: true, filter: true },
  { field: 'price', valueFormatter: p => '$' + p.value },
  { field: 'inStock', headerName: 'In Stock' }
];

rowData: Product[] = [
  { name: 'iPhone 15', price: 999, inStock: true },
  { name: 'Samsung Galaxy S24', price: 849, inStock: true }
];

onGridReady(event: GridReadyEvent<Product>): void {
  // Single unified API â€” no separate columnApi
  event.api.sizeColumnsToFit();
}
```

```html
<ag-grid-angular
  style="width: 100%; height: 400px;"
  [theme]="theme"
  [rowData]="rowData"
  [columnDefs]="columnDefs"
  (gridReady)="onGridReady($event)">
</ag-grid-angular>
```

## Notes
- Requires Angular 17+
- Legacy `columnApi` is removed â€” all methods are on `gridApi`
- Themes via `themeQuartz`, `themeAlpine`, `themeMaterial` from `ag-grid-community`
- Use `ColDef<T>` with a generic type for full type safety on `field` and value getters
- `autoSizeStrategy` replaces the old `sizeColumnsToFit` pattern for initial sizing



