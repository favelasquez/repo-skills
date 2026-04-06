---
name: ag-grid
description: High performance data grid for web applications
license: Apache-2.0
metadata:
  author: https://github.com/favelasquez
  version: "18.0"
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

# AG Grid v18 â€” Usage

## Overview
AG Grid v18 is a feature-rich data grid supporting Angular, React, and Vue. It uses a
legacy dual-API model with separate `gridApi` and `columnApi` objects, virtual row
rendering for performance, and classic theme classes.

## Setup

```typescript
import { AgGridModule } from 'ag-grid-angular';
import { ColDef, GridOptions } from 'ag-grid-community';
```

Register in AppModule:

```typescript
@NgModule({
  imports: [AgGridModule.withComponents([])]
})
export class AppModule {}
```

## Basic usage

```typescript
columnDefs: ColDef[] = [
  { headerName: 'Name', field: 'name', sortable: true, filter: true },
  { headerName: 'Price', field: 'price', valueFormatter: params => '$' + params.value },
  { headerName: 'In Stock', field: 'inStock', cellRenderer: params => params.value ? 'Yes' : 'No' }
];

rowData = [
  { name: 'iPhone X', price: 799, inStock: true },
  { name: 'Samsung Galaxy S9', price: 649, inStock: false }
];

onGridReady(params: any): void {
  this.gridApi = params.api;
  this.columnApi = params.columnApi;
}
```

```html
<ag-grid-angular
  style="width: 100%; height: 400px;"
  class="ag-theme-balham"
  [rowData]="rowData"
  [columnDefs]="columnDefs"
  (gridReady)="onGridReady($event)">
</ag-grid-angular>
```

## Notes
- Uses legacy dual-API: `gridApi` and `columnApi` are separate objects
- Available themes: `ag-theme-balham`, `ag-theme-material`, `ag-theme-blue`, `ag-theme-fresh`
- Row selection configured via `rowSelection: 'single' | 'multiple'` in gridOptions
- Pagination: set `[pagination]="true"` and `[paginationPageSize]="20"` on the component
- Sorting and filtering are opt-in per column via `sortable: true` and `filter: true`



