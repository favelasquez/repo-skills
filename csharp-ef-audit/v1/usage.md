---
metadata:
  author: https://github.com/favelasquez
name: csharp-ef-audit
description: >
  Skill auditora experta de código C# con Entity Framework, cubriendo todas las
  versiones del stack: .NET Framework 4.8 + EF 6.x, .NET Core 3.1 + EF Core 3.x,
  .NET 6/7/8 + EF Core 6/7/8. Activar siempre que el usuario pida revisar,
  auditar, corregir o mejorar código C# que use Entity Framework o EF Core,
  o cuando mencione DbContext, DbSet, migraciones, LINQ to Entities, Repository
  pattern, Unit of Work, CQRS, MediatR, Clean Architecture, Web API, o cualquier
  patrón arquitectónico en .NET. También activar cuando el usuario muestre código
  con posibles problemas de performance (N+1, eager/lazy loading), seguridad
  (SQL injection, exposición de datos), o diseño (acoplamiento, responsabilidades
  mezcladas). Esta skill detecta bugs silenciosos, antipatrones y problemas
  arquitectónicos con ejemplos corregidos listos para producción.
  Prioridades de auditoría configuradas: Queries N+1 y performance EF,
  Seguridad / SQL Injection, Patrones Repository + Unit of Work y código limpio.
---
metadata:
  author: https://github.com/favelasquez

# C# + Entity Framework � Skill Auditora Experta

## Paso 1 � Detectar la versión del stack antes de auditar

Antes de dar cualquier corrección, identificar la versión del código recibido:

```
¿Qué importaciones tiene?
  using System.Data.Entity             �  EF 6 (.NET Framework 4.8)
  using Microsoft.EntityFrameworkCore  �  EF Core (3.x / 6 / 7 / 8)

¿Cómo registra el DbContext?
  new AppDbContext() / web.config      �  EF 6 / .NET 4.8
  services.AddDbContext<>()            �  EF Core

¿Usa minimal API (app.MapGet)?        �  .NET 6+
¿Usa ExecuteUpdateAsync/DeleteAsync?  �  EF Core 8
¿Usa record types y DateOnly?         �  .NET 6+
¿Las propiedades de nav son virtual?  �  EF 6 con lazy loading activo �a�️
```

### Trampas específicas por versión

| Stack | EF | Trampa principal |
|---|---|---|
| .NET Framework 4.8 | EF 6.x | Lazy loading **ON** por defecto � N+1 silencioso en toda propiedad `virtual` |
| .NET Core 3.1 | EF Core 3.x | Evaluación en cliente eliminada � queries intraducibles lanzan excepción en runtime |
| .NET 6 / 7 | EF Core 6 / 7 | Sin bulk nativo � loops de SaveChanges son comunes y costosos |
| .NET 8 | EF Core 8 | `ExecuteUpdateAsync`/`ExecuteDeleteAsync` disponibles � si no se usan, es un smell |

---
metadata:
  author: https://github.com/favelasquez

## Perfil configurado � áreas prioritarias

**Patrón:** Repository + Unit of Work
**Áreas críticas por orden de prioridad al auditar:**
1. �x� Seguridad � SQL Injection, exposición de datos, IDOR
2. �x� Performance � N+1, queries en bucles, paginación en memoria
3. �xx� Repository/UoW � fugas de abstracción, SaveChanges mal ubicado, transacciones
4. �xx� Código limpio � SRP, nombres, async correcto, tipado fuerte

> Para detalles profundos ver archivos en `references/`:
> - `references/ef-performance.md` � N+1, eager/lazy/explicit loading, AsNoTracking, proyecciones
> - `references/seguridad.md` � SQL Injection, exposición de datos, IDOR, mass assignment
> - `references/patrones.md` � Repository sin fugas, UoW, CQRS, Clean Architecture, DDD
> - `references/ef-migraciones.md` � Migraciones, índices, soft delete, auditoría automática

---
metadata:
  author: https://github.com/favelasquez

## Checklist de auditoría � ejecutar en cada revisión

### �x� Crítico � Seguridad

- [ ] ¿Hay interpolación de strings en `FromSqlRaw` / `ExecuteSqlRaw`? (SQL Injection)
- [ ] ¿Las entidades se devuelven directamente en endpoints? (exposición de datos)
- [ ] ¿Los endpoints filtran por `userId` del token además del ID de recurso? (IDOR)
- [ ] ¿Se acepta una entidad del cliente para hacer `Update` directo? (mass assignment)
- [ ] ¿Los connection strings están en código fuente o repositorio? (secrets expuestos)

### �x� Crítico � Performance N+1

- [ ] ¿Hay accesos a propiedades de navegación dentro de bucles `foreach`?
- [ ] ¿Las propiedades son `virtual` en EF 6 sin lazy loading deshabilitado?
- [ ] ¿Se llama `SaveChanges`/`SaveChangesAsync` dentro de un bucle?
- [ ] ¿Las queries de paginación hacen `ToList()` antes del `Skip/Take`?
- [ ] ¿Hay `Count()` donde bastaría `Any()`?
- [ ] ¿El DbContext tiene lifetime Singleton o Transient? (debe ser Scoped)

### �x� Crítico � Repository + Unit of Work

- [ ] ¿El repositorio devuelve `IQueryable<T>`? (fuga de abstracción)
- [ ] ¿El repositorio llama `SaveChanges` internamente? (rompe el UoW)
- [ ] ¿El DbContext se inyecta directamente en controladores? (saltea el repositorio)
- [ ] ¿Las operaciones que deben ser atómicas carecen de transacción explícita?
- [ ] ¿El UoW gestiona el `SaveChanges` en un solo punto de control?

### �xx� Performance � Optimización

- [ ] ¿Se usa `AsNoTracking()` en todas las queries de solo lectura?
- [ ] ¿Los `Include` cargan grafos completos donde basta una proyección `Select`?
- [ ] ¿Las proyecciones `Select` ocurren antes del `ToList()`?
- [ ] ¿Se usan métodos `async` en contextos Web API?
- [ ] ¿Las colecciones múltiples con `Include` usan `AsSplitQuery()`? (EF Core)

### �xx� Código limpio

- [ ] ¿Los métodos hacen más de una cosa? (SRP)
- [ ] ¿Hay tipos `object` sin justificación donde se puede tipar fuerte?
- [ ] ¿Los nombres de repositorios/servicios reflejan el dominio, no la tecnología?
- [ ] ¿La lógica de negocio vive en controladores en lugar de servicios/dominio?
- [ ] ¿Los DTOs de entrada y salida están separados?

---
metadata:
  author: https://github.com/favelasquez

## Bugs críticos más comunes � referencia rápida

### 1. N+1 � el más común en proyectos con Repository

```csharp
// �R N+1 � el repositorio devuelve entidades, el servicio itera propiedades de nav
// ProductoRepository.GetActivos() -> SELECT * FROM Productos (sin Include)
// Luego en el servicio:
foreach (var producto in productos)
{
    var cat = producto.Categoria.Nombre; // query extra por cada producto
}

// �S& El repositorio proyecta todo lo necesario � sin N+1 posible
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
// �R SINGLETON � change tracker acumula entidades de TODOS los requests
//    concurrentes �  corrupción de datos, memory leak, errores de concurrencia
services.AddSingleton<AppDbContext>();

// �R TRANSIENT � múltiples contextos por request �  UoW no puede coordinar SaveChanges
services.AddTransient<AppDbContext>();

// �S& SCOPED � una instancia por request, compartida por todos los repositorios del UoW
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
// AddDbContext es Scoped por defecto
```

### 3. SQL Injection � por versión

```csharp
// �R VULNERABLE � en cualquier versión de EF
var usuarios = context.Usuarios
    .FromSqlRaw($"SELECT * FROM Usuarios WHERE Email = '{email}'")
    .ToList();

// �S& EF Core � parámetro posicional
context.Usuarios.FromSqlRaw("SELECT * FROM Usuarios WHERE Email = {0}", email)

// �S& EF Core � string interpolado seguro (crea SqlParameter automáticamente)
context.Usuarios.FromSqlInterpolated($"SELECT * FROM Usuarios WHERE Email = {email}")

// �S& EF 6 (.NET 4.8) � parámetro posicional
context.Usuarios.SqlQuery<Usuario>("SELECT * FROM Usuarios WHERE Email = @p0", email)

// �S& SIEMPRE PREFERIR � LINQ parametriza automáticamente, ninguna superficie de ataque
context.Usuarios.Where(u => u.Email == email).AsNoTracking().ToList()
```

### 4. Repositorio llama SaveChanges � rompe el Unit of Work

```csharp
// �R Cada repositorio guarda por su cuenta � imposible hacer operaciones atómicas
public class ProductoRepository
{
    public async Task AddAsync(Producto p)
    {
        _context.Productos.Add(p);
        await _context.SaveChangesAsync(); // UoW pierde el control aquí
    }
}

// �S& El repositorio solo trackea entidades � el UoW decide cuándo persistir
public class ProductoRepository : IProductoRepository
{
    private readonly AppDbContext _context;
    public ProductoRepository(AppDbContext context) => _context = context;

    public void Add(Producto p)    => _context.Productos.Add(p);
    public void Remove(Producto p) => _context.Productos.Remove(p);
    // Sin SaveChanges aquí � nunca
}

public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    public IProductoRepository Productos { get; }
    public IOrdenRepository Ordenes { get; }

    public Task<int> SaveChangesAsync() => _context.SaveChangesAsync(); // único punto

    // Uso en servicio � ambas operaciones son atómicas
    // await _uow.Productos.Add(producto);
    // await _uow.Inventario.DescontarStock(producto.Id, cantidad);
    // await _uow.SaveChangesAsync(); // una sola transacción
}
```

### 5. IQueryable � fuga de abstracción

```csharp
// �R IQueryable expuesto � EF se filtra a toda la aplicación
public interface IProductoRepository
{
    IQueryable<Producto> GetAll(); // cualquiera agrega .Include(), .Where(), etc.
}
// Consecuencia: no puedes mockear en tests, no puedes cambiar EF sin tocar toda la app

// �S& Métodos con semántica de dominio, retorno concreto e inmutable
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
// �R Expone PasswordHash, tokens internos, relaciones circulares
[HttpGet("{id}")]
public async Task<Usuario> GetUsuario(int id)
    => await _uow.Usuarios.GetByIdAsync(id); // devuelve entidad completa

// �S& DTO de salida con exactamente lo que el cliente necesita
public record UsuarioDto(int Id, string Nombre, string Email, string Rol);

[HttpGet("{id}")]
public async Task<ActionResult<UsuarioDto>> GetUsuario(int id)
{
    var dto = await _uow.Usuarios.GetByIdAsync(id); // repositorio devuelve DTO
    return dto is null ? NotFound() : Ok(dto);
}
```

### 7. Lazy loading silencioso � EF 6 / .NET 4.8

```csharp
// �a�️ EF 6 � cada propiedad virtual dispara un SELECT adicional al acceder
public class Orden
{
    public virtual Cliente          Cliente { get; set; }  // query al acceder
    public virtual ICollection<Item> Items { get; set; }   // query al iterar
}

// �S& Deshabilitarlo globalmente en EF 6
public class AppDbContext : DbContext
{
    public AppDbContext()
    {
        Configuration.LazyLoadingEnabled   = false;
        Configuration.ProxyCreationEnabled = false;
    }
}
// Luego usar .Include() explícito o proyecciones Select en los repositorios
```

---
metadata:
  author: https://github.com/favelasquez

## Diferencias clave entre versiones de EF

| Operación | EF 6 (.NET 4.8) | EF Core 3�7 | EF Core 8 |
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

## Concurrencia optimista � Lost Update Problem

El escenario clásico: dos usuarios editan el mismo registro. El segundo pisa los cambios del primero sin saberlo. EF resuelve esto con **concurrencia optimista** usando un token de fila.

### Paso 1 � Agregar `[Timestamp]` a la entidad

```csharp
// EF 6 (.NET 4.8) � atributo del namespace System.ComponentModel.DataAnnotations
public class User
{
    public int    Id       { get; set; }
    public string Name     { get; set; }
    public string Email    { get; set; }
    public bool   IsActive { get; set; }

    // �S& SQL Server actualiza este valor en cada UPDATE automáticamente
    [Timestamp]
    public byte[] RowVersion { get; set; }
}

// �S& Alternativa con Fluent API en EF 6 (si no quieres atributos en la entidad)
modelBuilder.Entity<User>()
    .Property(u => u.RowVersion)
    .IsRowVersion(); // equivale a [Timestamp]
```

EF 6 añade el `RowVersion` al `WHERE` de cada `UPDATE` automáticamente:
```sql
-- Sin concurrencia: UPDATE Users SET Name='Pedro' WHERE Id=1
-- Con [Timestamp]:  UPDATE Users SET Name='Pedro' WHERE Id=1 AND RowVersion=0x0000000000000076B
-- Si RowVersion cambió (otro usuario ya guardó) �  0 filas afectadas �  excepción
```

### Paso 2 � Capturar la excepción correcta

```csharp
// �S& EF 6 � namespace distinto a EF Core
using System.Data.Entity.Infrastructure; // DbUpdateConcurrencyException vive aquí en EF 6

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
// �a�️ EF Core usa mismo nombre pero distinto namespace:
// using Microsoft.EntityFrameworkCore; �  DbUpdateConcurrencyException
```

### Paso 3 � Las 3 estrategias de resolución

```csharp
private void ResolverConflicto(DbEntityEntry entry) // EF 6: DbEntityEntry
{
    var dbValues     = entry.GetDatabaseValues();  // lo que hay en DB ahora
    var clientValues = entry.CurrentValues;         // lo que el usuario quería guardar
    var origValues   = entry.OriginalValues;        // lo que el usuario vio al cargar

    // ������ Estrategia A: Notificar al usuario (recomendada para formularios) ����
    throw new Exception(
        "Otro usuario modificó este registro. Recarga y vuelve a intentarlo.");

    // ������ Estrategia B: Client Wins � el último siempre gana ����������������������������������
    // Reemplaza el RowVersion original �  EF deja de detectar el conflicto
    entry.OriginalValues.SetValues(dbValues);
    _context.SaveChanges(); // pisa los cambios del Usuario A

    // ������ Estrategia C: Merge por campo � la más correcta ����������������������������������������
    // Solo aplica cambios del cliente donde él realmente modificó algo
    foreach (var prop in origValues.PropertyNames)
    {
        var dbVal     = dbValues[prop];
        var clientVal = clientValues[prop];
        var origVal   = origValues[prop];

        // El cliente modificó este campo �  mantener su valor
        // El cliente NO lo tocó �  usar el valor actual de DB (preserva cambios ajenos)
        clientValues[prop] = !Equals(clientVal, origVal) ? clientVal : dbVal;
    }
    entry.OriginalValues.SetValues(dbValues); // actualizar token
    _context.SaveChanges();
    // Resultado: Usuario A conserva su Name, Usuario B aplica solo su Email �S&
}
```

### Qué ocurre con y sin la protección

```
�R Sin [Timestamp]:
  DB inicial: Name="Juan", Email="viejo@mail.com"
  Usuario A guarda: Name="Pedro"                    �  DB: Name="Pedro"
  Usuario B guarda: Email="nuevo@mail.com"
    UPDATE SET Name="Juan", Email="nuevo@mail.com"  �  ¡Name revertido silenciosamente!

�S& Con [Timestamp]:
  DB inicial: Name="Juan", Email="viejo@mail.com", RowVersion=001
  Usuario A guarda: WHERE RowVersion=001 �  OK       �  DB: Name="Pedro", RV=002
  Usuario B intenta: WHERE RowVersion=001 �  0 filas �  DbUpdateConcurrencyException
    �  Usuario B recarga �  ve Name="Pedro" �  solo cambia Email �  OK �S&
```

### Checklist de concurrencia � agregar a toda entidad editable

- [ ] ¿Las entidades que se editan en formularios tienen `[Timestamp]` / `IsRowVersion()`?
- [ ] ¿El `RowVersion` se incluye en el DTO de edición (para devolverlo en el save)?
- [ ] ¿El `UnitOfWork.Save()` captura `DbUpdateConcurrencyException`?
- [ ] ¿La estrategia de resolución es consciente del dominio? (no siempre "client wins")
- [ ] ¿Se informa al usuario con un mensaje claro, no con una excepción genérica 500?

---
metadata:
  author: https://github.com/favelasquez

Leer archivos `references/` para guías completas por área.

