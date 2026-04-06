---
metadata:
  author: https://github.com/favelasquez
name: csharp-ef-audit
description: >
  Skill auditora experta de cÃ³digo C# con Entity Framework, cubriendo todas las
  versiones del stack: .NET Framework 4.8 + EF 6.x, .NET Core 3.1 + EF Core 3.x,
  .NET 6/7/8 + EF Core 6/7/8. Activar siempre que el usuario pida revisar,
  auditar, corregir o mejorar cÃ³digo C# que use Entity Framework o EF Core,
  o cuando mencione DbContext, DbSet, migraciones, LINQ to Entities, Repository
  pattern, Unit of Work, CQRS, MediatR, Clean Architecture, Web API, o cualquier
  patrÃ³n arquitectÃ³nico en .NET. TambiÃ©n activar cuando el usuario muestre cÃ³digo
  con posibles problemas de performance (N+1, eager/lazy loading), seguridad
  (SQL injection, exposiciÃ³n de datos), o diseÃ±o (acoplamiento, responsabilidades
  mezcladas). Esta skill detecta bugs silenciosos, antipatrones y problemas
  arquitectÃ³nicos con ejemplos corregidos listos para producciÃ³n.
  Prioridades de auditorÃ­a configuradas: Queries N+1 y performance EF,
  Seguridad / SQL Injection, Patrones Repository + Unit of Work y cÃ³digo limpio.
---
metadata:
  author: https://github.com/favelasquez

# C# + Entity Framework â€” Skill Auditora Experta

## Paso 1 â€” Detectar la versiÃ³n del stack antes de auditar

Antes de dar cualquier correcciÃ³n, identificar la versiÃ³n del cÃ³digo recibido:

```
Â¿QuÃ© importaciones tiene?
  using System.Data.Entity             â†’ EF 6 (.NET Framework 4.8)
  using Microsoft.EntityFrameworkCore  â†’ EF Core (3.x / 6 / 7 / 8)

Â¿CÃ³mo registra el DbContext?
  new AppDbContext() / web.config      â†’ EF 6 / .NET 4.8
  services.AddDbContext<>()            â†’ EF Core

Â¿Usa minimal API (app.MapGet)?        â†’ .NET 6+
Â¿Usa ExecuteUpdateAsync/DeleteAsync?  â†’ EF Core 8
Â¿Usa record types y DateOnly?         â†’ .NET 6+
Â¿Las propiedades de nav son virtual?  â†’ EF 6 con lazy loading activo âš ï¸
```

### Trampas especÃ­ficas por versiÃ³n

| Stack | EF | Trampa principal |
|---|---|---|
| .NET Framework 4.8 | EF 6.x | Lazy loading **ON** por defecto â€” N+1 silencioso en toda propiedad `virtual` |
| .NET Core 3.1 | EF Core 3.x | EvaluaciÃ³n en cliente eliminada â€” queries intraducibles lanzan excepciÃ³n en runtime |
| .NET 6 / 7 | EF Core 6 / 7 | Sin bulk nativo â€” loops de SaveChanges son comunes y costosos |
| .NET 8 | EF Core 8 | `ExecuteUpdateAsync`/`ExecuteDeleteAsync` disponibles â€” si no se usan, es un smell |

---
metadata:
  author: https://github.com/favelasquez

## Perfil configurado â€” Ã¡reas prioritarias

**PatrÃ³n:** Repository + Unit of Work
**Ãreas crÃ­ticas por orden de prioridad al auditar:**
1. ðŸ”´ Seguridad â€” SQL Injection, exposiciÃ³n de datos, IDOR
2. ðŸ”´ Performance â€” N+1, queries en bucles, paginaciÃ³n en memoria
3. ðŸŸ  Repository/UoW â€” fugas de abstracciÃ³n, SaveChanges mal ubicado, transacciones
4. ðŸŸ¡ CÃ³digo limpio â€” SRP, nombres, async correcto, tipado fuerte

> Para detalles profundos ver archivos en `references/`:
> - `references/ef-performance.md` â€” N+1, eager/lazy/explicit loading, AsNoTracking, proyecciones
> - `references/seguridad.md` â€” SQL Injection, exposiciÃ³n de datos, IDOR, mass assignment
> - `references/patrones.md` â€” Repository sin fugas, UoW, CQRS, Clean Architecture, DDD
> - `references/ef-migraciones.md` â€” Migraciones, Ã­ndices, soft delete, auditorÃ­a automÃ¡tica

---
metadata:
  author: https://github.com/favelasquez

## Checklist de auditorÃ­a â€” ejecutar en cada revisiÃ³n

### ðŸ”´ CrÃ­tico â€” Seguridad

- [ ] Â¿Hay interpolaciÃ³n de strings en `FromSqlRaw` / `ExecuteSqlRaw`? (SQL Injection)
- [ ] Â¿Las entidades se devuelven directamente en endpoints? (exposiciÃ³n de datos)
- [ ] Â¿Los endpoints filtran por `userId` del token ademÃ¡s del ID de recurso? (IDOR)
- [ ] Â¿Se acepta una entidad del cliente para hacer `Update` directo? (mass assignment)
- [ ] Â¿Los connection strings estÃ¡n en cÃ³digo fuente o repositorio? (secrets expuestos)

### ðŸ”´ CrÃ­tico â€” Performance N+1

- [ ] Â¿Hay accesos a propiedades de navegaciÃ³n dentro de bucles `foreach`?
- [ ] Â¿Las propiedades son `virtual` en EF 6 sin lazy loading deshabilitado?
- [ ] Â¿Se llama `SaveChanges`/`SaveChangesAsync` dentro de un bucle?
- [ ] Â¿Las queries de paginaciÃ³n hacen `ToList()` antes del `Skip/Take`?
- [ ] Â¿Hay `Count()` donde bastarÃ­a `Any()`?
- [ ] Â¿El DbContext tiene lifetime Singleton o Transient? (debe ser Scoped)

### ðŸ”´ CrÃ­tico â€” Repository + Unit of Work

- [ ] Â¿El repositorio devuelve `IQueryable<T>`? (fuga de abstracciÃ³n)
- [ ] Â¿El repositorio llama `SaveChanges` internamente? (rompe el UoW)
- [ ] Â¿El DbContext se inyecta directamente en controladores? (saltea el repositorio)
- [ ] Â¿Las operaciones que deben ser atÃ³micas carecen de transacciÃ³n explÃ­cita?
- [ ] Â¿El UoW gestiona el `SaveChanges` en un solo punto de control?

### ðŸŸ¡ Performance â€” OptimizaciÃ³n

- [ ] Â¿Se usa `AsNoTracking()` en todas las queries de solo lectura?
- [ ] Â¿Los `Include` cargan grafos completos donde basta una proyecciÃ³n `Select`?
- [ ] Â¿Las proyecciones `Select` ocurren antes del `ToList()`?
- [ ] Â¿Se usan mÃ©todos `async` en contextos Web API?
- [ ] Â¿Las colecciones mÃºltiples con `Include` usan `AsSplitQuery()`? (EF Core)

### ðŸŸ¡ CÃ³digo limpio

- [ ] Â¿Los mÃ©todos hacen mÃ¡s de una cosa? (SRP)
- [ ] Â¿Hay tipos `object` sin justificaciÃ³n donde se puede tipar fuerte?
- [ ] Â¿Los nombres de repositorios/servicios reflejan el dominio, no la tecnologÃ­a?
- [ ] Â¿La lÃ³gica de negocio vive en controladores en lugar de servicios/dominio?
- [ ] Â¿Los DTOs de entrada y salida estÃ¡n separados?

---
metadata:
  author: https://github.com/favelasquez

## Bugs crÃ­ticos mÃ¡s comunes â€” referencia rÃ¡pida

### 1. N+1 â€” el mÃ¡s comÃºn en proyectos con Repository

```csharp
// âŒ N+1 â€” el repositorio devuelve entidades, el servicio itera propiedades de nav
// ProductoRepository.GetActivos() -> SELECT * FROM Productos (sin Include)
// Luego en el servicio:
foreach (var producto in productos)
{
    var cat = producto.Categoria.Nombre; // query extra por cada producto
}

// âœ… El repositorio proyecta todo lo necesario â€” sin N+1 posible
public async Task<IReadOnlyList<ProductoDto>> GetActivosAsync()
    => await _context.Productos
        .Where(p => p.Activo)
        .Select(p => new ProductoDto(
            p.Id,
            p.Nombre,
            p.Precio,
            p.Categoria.Nombre))   // JOIN en SQL, no query extra
        .AsNoTracking()
        .ToListAsync();
```

### 2. DbContext lifetime incorrecto

```csharp
// âŒ SINGLETON â€” change tracker acumula entidades de TODOS los requests
//    concurrentes â†’ corrupciÃ³n de datos, memory leak, errores de concurrencia
services.AddSingleton<AppDbContext>();

// âŒ TRANSIENT â€” mÃºltiples contextos por request â†’ UoW no puede coordinar SaveChanges
services.AddTransient<AppDbContext>();

// âœ… SCOPED â€” una instancia por request, compartida por todos los repositorios del UoW
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
// AddDbContext es Scoped por defecto
```

### 3. SQL Injection â€” por versiÃ³n

```csharp
// âŒ VULNERABLE â€” en cualquier versiÃ³n de EF
var usuarios = context.Usuarios
    .FromSqlRaw($"SELECT * FROM Usuarios WHERE Email = '{email}'")
    .ToList();

// âœ… EF Core â€” parÃ¡metro posicional
context.Usuarios.FromSqlRaw("SELECT * FROM Usuarios WHERE Email = {0}", email)

// âœ… EF Core â€” string interpolado seguro (crea SqlParameter automÃ¡ticamente)
context.Usuarios.FromSqlInterpolated($"SELECT * FROM Usuarios WHERE Email = {email}")

// âœ… EF 6 (.NET 4.8) â€” parÃ¡metro posicional
context.Usuarios.SqlQuery<Usuario>("SELECT * FROM Usuarios WHERE Email = @p0", email)

// âœ… SIEMPRE PREFERIR â€” LINQ parametriza automÃ¡ticamente, ninguna superficie de ataque
context.Usuarios.Where(u => u.Email == email).AsNoTracking().ToList()
```

### 4. Repositorio llama SaveChanges â€” rompe el Unit of Work

```csharp
// âŒ Cada repositorio guarda por su cuenta â€” imposible hacer operaciones atÃ³micas
public class ProductoRepository
{
    public async Task AddAsync(Producto p)
    {
        _context.Productos.Add(p);
        await _context.SaveChangesAsync(); // UoW pierde el control aquÃ­
    }
}

// âœ… El repositorio solo trackea entidades â€” el UoW decide cuÃ¡ndo persistir
public class ProductoRepository : IProductoRepository
{
    private readonly AppDbContext _context;
    public ProductoRepository(AppDbContext context) => _context = context;

    public void Add(Producto p)    => _context.Productos.Add(p);
    public void Remove(Producto p) => _context.Productos.Remove(p);
    // Sin SaveChanges aquÃ­ â€” nunca
}

public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    public IProductoRepository Productos { get; }
    public IOrdenRepository Ordenes { get; }

    public Task<int> SaveChangesAsync() => _context.SaveChangesAsync(); // Ãºnico punto

    // Uso en servicio â€” ambas operaciones son atÃ³micas
    // await _uow.Productos.Add(producto);
    // await _uow.Inventario.DescontarStock(producto.Id, cantidad);
    // await _uow.SaveChangesAsync(); // una sola transacciÃ³n
}
```

### 5. IQueryable â€” fuga de abstracciÃ³n

```csharp
// âŒ IQueryable expuesto â€” EF se filtra a toda la aplicaciÃ³n
public interface IProductoRepository
{
    IQueryable<Producto> GetAll(); // cualquiera agrega .Include(), .Where(), etc.
}
// Consecuencia: no puedes mockear en tests, no puedes cambiar EF sin tocar toda la app

// âœ… MÃ©todos con semÃ¡ntica de dominio, retorno concreto e inmutable
public interface IProductoRepository
{
    Task<ProductoDto>                GetByIdAsync(int id);
    Task<IReadOnlyList<ProductoDto>> GetActivosByCategoriaAsync(int categoriaId);
    Task<PagedResult<ProductoDto>>   BuscarAsync(ProductoFiltro filtro);
    Task<bool>                       ExisteCodigoAsync(string codigo, int? excludeId = null);
    void Add(Producto producto);
    void Remove(Producto producto);
}
```

### 6. Exponer entidades directamente en la API

```csharp
// âŒ Expone PasswordHash, tokens internos, relaciones circulares
[HttpGet("{id}")]
public async Task<Usuario> GetUsuario(int id)
    => await _uow.Usuarios.GetByIdAsync(id); // devuelve entidad completa

// âœ… DTO de salida con exactamente lo que el cliente necesita
public record UsuarioDto(int Id, string Nombre, string Email, string Rol);

[HttpGet("{id}")]
public async Task<ActionResult<UsuarioDto>> GetUsuario(int id)
{
    var dto = await _uow.Usuarios.GetByIdAsync(id); // repositorio devuelve DTO
    return dto is null ? NotFound() : Ok(dto);
}
```

### 7. Lazy loading silencioso â€” EF 6 / .NET 4.8

```csharp
// âš ï¸ EF 6 â€” cada propiedad virtual dispara un SELECT adicional al acceder
public class Orden
{
    public virtual Cliente          Cliente { get; set; }  // query al acceder
    public virtual ICollection<Item> Items { get; set; }   // query al iterar
}

// âœ… Deshabilitarlo globalmente en EF 6
public class AppDbContext : DbContext
{
    public AppDbContext()
    {
        Configuration.LazyLoadingEnabled   = false;
        Configuration.ProxyCreationEnabled = false;
    }
}
// Luego usar .Include() explÃ­cito o proyecciones Select en los repositorios
```

---
metadata:
  author: https://github.com/favelasquez

## Diferencias clave entre versiones de EF

| OperaciÃ³n | EF 6 (.NET 4.8) | EF Core 3â€“7 | EF Core 8 |
|---|---|---|---|
| Bulk delete | Z.EntityFramework (ext) | Z.EntityFramework (ext) | `ExecuteDeleteAsync()` nativo |
| Bulk update | Z.EntityFramework (ext) | Z.EntityFramework (ext) | `ExecuteUpdateAsync()` nativo |
| Lazy loading | **ON** por defecto | OFF por defecto | OFF por defecto |
| Raw SQL select | `Database.SqlQuery<T>()` | `FromSqlRaw()` | igual |
| Raw SQL execute | `Database.ExecuteSqlCommand()` | `ExecuteSqlRawAsync()` | igual |
| Transacciones | `Database.BeginTransaction()` | `BeginTransactionAsync()` | igual |
| Split queries | No disponible | `AsSplitQuery()` | igual |
| JSON columns | No disponible | No disponible | `OwnsMany` a columna JSON |
| Compiled queries | `CompiledQuery.Compile()` | `EF.CompileAsyncQuery()` | igual |
| Ver SQL generado | `context.Database.Log =` | `.LogTo()` / `.ToQueryString()` | igual |

| Concurrencia optimista | `[Timestamp]` + `DbUpdateConcurrencyException` (EF6 namespace) | `[Timestamp]` + `DbUpdateConcurrencyException` (EFCore namespace) | igual |

---
metadata:
  author: https://github.com/favelasquez

## Concurrencia optimista â€” Lost Update Problem

El escenario clÃ¡sico: dos usuarios editan el mismo registro. El segundo pisa los cambios del primero sin saberlo. EF resuelve esto con **concurrencia optimista** usando un token de fila.

### Paso 1 â€” Agregar `[Timestamp]` a la entidad

```csharp
// EF 6 (.NET 4.8) â€” atributo del namespace System.ComponentModel.DataAnnotations
public class User
{
    public int    Id       { get; set; }
    public string Name     { get; set; }
    public string Email    { get; set; }
    public bool   IsActive { get; set; }

    // âœ… SQL Server actualiza este valor en cada UPDATE automÃ¡ticamente
    [Timestamp]
    public byte[] RowVersion { get; set; }
}

// âœ… Alternativa con Fluent API en EF 6 (si no quieres atributos en la entidad)
modelBuilder.Entity<User>()
    .Property(u => u.RowVersion)
    .IsRowVersion(); // equivale a [Timestamp]
```

EF 6 aÃ±ade el `RowVersion` al `WHERE` de cada `UPDATE` automÃ¡ticamente:
```sql
-- Sin concurrencia: UPDATE Users SET Name='Pedro' WHERE Id=1
-- Con [Timestamp]:  UPDATE Users SET Name='Pedro' WHERE Id=1 AND RowVersion=0x0000000000000076B
-- Si RowVersion cambiÃ³ (otro usuario ya guardÃ³) â†’ 0 filas afectadas â†’ excepciÃ³n
```

### Paso 2 â€” Capturar la excepciÃ³n correcta

```csharp
// âœ… EF 6 â€” namespace distinto a EF Core
using System.Data.Entity.Infrastructure; // DbUpdateConcurrencyException vive aquÃ­ en EF 6

public void Save()
{
    try
    {
        _context.SaveChanges();
    }
    catch (DbUpdateConcurrencyException ex)
    {
        var entry = ex.Entries.Single(); // entidad en conflicto
        ResolverConflicto(entry);
    }
}
// âš ï¸ EF Core usa mismo nombre pero distinto namespace:
// using Microsoft.EntityFrameworkCore; â†’ DbUpdateConcurrencyException
```

### Paso 3 â€” Las 3 estrategias de resoluciÃ³n

```csharp
private void ResolverConflicto(DbEntityEntry entry) // EF 6: DbEntityEntry
{
    var dbValues     = entry.GetDatabaseValues();  // lo que hay en DB ahora
    var clientValues = entry.CurrentValues;         // lo que el usuario querÃ­a guardar
    var origValues   = entry.OriginalValues;        // lo que el usuario vio al cargar

    // â”€â”€â”€ Estrategia A: Notificar al usuario (recomendada para formularios) â”€â”€
    throw new Exception(
        "Otro usuario modificÃ³ este registro. Recarga y vuelve a intentarlo.");

    // â”€â”€â”€ Estrategia B: Client Wins â€” el Ãºltimo siempre gana â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
    // Reemplaza el RowVersion original â†’ EF deja de detectar el conflicto
    entry.OriginalValues.SetValues(dbValues);
    _context.SaveChanges(); // pisa los cambios del Usuario A

    // â”€â”€â”€ Estrategia C: Merge por campo â€” la mÃ¡s correcta â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
    // Solo aplica cambios del cliente donde Ã©l realmente modificÃ³ algo
    foreach (var prop in origValues.PropertyNames)
    {
        var dbVal     = dbValues[prop];
        var clientVal = clientValues[prop];
        var origVal   = origValues[prop];

        // El cliente modificÃ³ este campo â†’ mantener su valor
        // El cliente NO lo tocÃ³ â†’ usar el valor actual de DB (preserva cambios ajenos)
        clientValues[prop] = !Equals(clientVal, origVal) ? clientVal : dbVal;
    }
    entry.OriginalValues.SetValues(dbValues); // actualizar token
    _context.SaveChanges();
    // Resultado: Usuario A conserva su Name, Usuario B aplica solo su Email âœ…
}
```

### QuÃ© ocurre con y sin la protecciÃ³n

```
âŒ Sin [Timestamp]:
  DB inicial: Name="Juan", Email="viejo@mail.com"
  Usuario A guarda: Name="Pedro"                    â†’ DB: Name="Pedro"
  Usuario B guarda: Email="nuevo@mail.com"
    UPDATE SET Name="Juan", Email="nuevo@mail.com"  â†’ Â¡Name revertido silenciosamente!

âœ… Con [Timestamp]:
  DB inicial: Name="Juan", Email="viejo@mail.com", RowVersion=001
  Usuario A guarda: WHERE RowVersion=001 â†’ OK       â†’ DB: Name="Pedro", RV=002
  Usuario B intenta: WHERE RowVersion=001 â†’ 0 filas â†’ DbUpdateConcurrencyException
    â†’ Usuario B recarga â†’ ve Name="Pedro" â†’ solo cambia Email â†’ OK âœ…
```

### Checklist de concurrencia â€” agregar a toda entidad editable

- [ ] Â¿Las entidades que se editan en formularios tienen `[Timestamp]` / `IsRowVersion()`?
- [ ] Â¿El `RowVersion` se incluye en el DTO de ediciÃ³n (para devolverlo en el save)?
- [ ] Â¿El `UnitOfWork.Save()` captura `DbUpdateConcurrencyException`?
- [ ] Â¿La estrategia de resoluciÃ³n es consciente del dominio? (no siempre "client wins")
- [ ] Â¿Se informa al usuario con un mensaje claro, no con una excepciÃ³n genÃ©rica 500?

---
metadata:
  author: https://github.com/favelasquez

Leer archivos `references/` para guÃ­as completas por Ã¡rea.

