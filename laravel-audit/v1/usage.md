---
name: laravel-audit
description: >
  Skill auditora experta de cÃ³digo Laravel 6 hasta Laravel 11, cubriendo Eloquent ORM,
  seguridad, arquitectura y performance. Activar siempre que el usuario pida revisar,
  auditar, corregir o mejorar cÃ³digo PHP con Laravel, o cuando mencione Eloquent,
  Artisan, migrations, Blade, Controllers, Middleware, Service Providers, Policies,
  Gates, Sanctum, Passport, Jobs, Queues, Events, Livewire, Inertia, PHPUnit, Pest,
  Repository pattern en Laravel, Service Layer, o cualquier concepto del ecosistema
  Laravel. TambiÃ©n activar cuando el usuario muestre cÃ³digo con posibles problemas de
  N+1 en Eloquent, SQL Injection en Query Builder, Mass Assignment sin protecciÃ³n,
  lÃ³gica de negocio en controladores, o falta de validaciÃ³n. Esta skill detecta bugs
  silenciosos, antipatrones y vulnerabilidades con ejemplos corregidos listos para
  producciÃ³n en todas las versiones de Laravel del 6 al 11.
---

# Laravel 6â€“11 â€” Skill Auditora Experta

## Paso 1 â€” Detectar la versiÃ³n antes de auditar

```
Â¿QuÃ© version de Laravel es?
  composer.json: "laravel/framework": "^6.x"  â†’ Laravel 6 (LTS, PHP 7.2+)
  composer.json: "laravel/framework": "^8.x"  â†’ Laravel 8 (modelos en Models/, factories tipadas)
  composer.json: "laravel/framework": "^10.x" â†’ Laravel 10 (PHP 8.1+, readonly props)
  composer.json: "laravel/framework": "^11.x" â†’ Laravel 11 (bootstrap/app.php slim, PHP 8.2+)

SeÃ±ales en el cÃ³digo:
  Route::get(..., 'Controller@method')   â†’ Laravel 6/7 (string-based routes)
  Route::get(..., [Controller::class, 'method']) â†’ Laravel 8+ (class-based)
  $fillable en el modelo                 â†’ cualquier versiÃ³n
  $guarded = []                          â†’ âš ï¸ sin protecciÃ³n de mass assignment
  HasFactory en el modelo                â†’ Laravel 8+
  #[Route(...)] atributos PHP 8          â†’ Laravel 10/11
```

### Cambios crÃ­ticos por versiÃ³n

| VersiÃ³n | PHP mÃ­n | Cambio clave para auditar |
|---|---|---|
| Laravel 6 LTS | 7.2 | `string-based` route actions, sin `Models/` folder |
| Laravel 7 | 7.2.5 | `Route::view()`, `blade` components anÃ³nimos |
| Laravel 8 | 7.3 | Modelos en `App\Models\`, factories con tipos, `match()` |
| Laravel 9 | 8.0 | PHP 8 named args, `enum` nativo, `str()` helper |
| Laravel 10 | 8.1 | `readonly` properties, `Process` facade, sin deprecaciones |
| Laravel 11 | 8.2 | `bootstrap/app.php` simplificado, `Dumpable` trait |

---

## Perfil configurado â€” Ã¡reas prioritarias

**Stack:** API REST Â· Livewire/Inertia Â· Queues/Jobs/Events Â· Testing
**Auditar en este orden:**
1. ðŸ”´ Seguridad â€” Mass Assignment, SQL Injection, Auth/Policies, IDOR
2. ðŸ”´ Eloquent â€” N+1, eager loading, scopes mal usados
3. ðŸŸ  Arquitectura â€” lÃ³gica en controladores, Service Layer, Repository
4. ðŸŸ¡ Performance â€” cachÃ©, queries lentas, jobs/queues, Ã­ndices

> Archivos de referencia detallados:
> - `references/eloquent.md` â€” N+1, relaciones, scopes, mutators, casting
> - `references/seguridad.md` â€” Mass Assignment, SQL Injection, Policies, Sanctum
> - `references/arquitectura.md` â€” Service Layer, Repository, Form Requests, Resources
> - `references/performance.md` â€” Cache, Query optimization, Jobs, Ã­ndices en migraciones
> - `references/testing.md` â€” PHPUnit, Pest, factories, mocking, API testing

---

## Checklist de auditorÃ­a â€” ejecutar en cada revisiÃ³n

### ðŸ”´ Seguridad

- [ ] Â¿El modelo tiene `$guarded = []` sin `$fillable`? (mass assignment total)
- [ ] Â¿Hay `DB::select()` o `whereRaw()` con interpolaciÃ³n de variables? (SQL Injection)
- [ ] Â¿Los endpoints usan `$this->authorize()` o Policies? (falta de autorizaciÃ³n)
- [ ] Â¿Los recursos de un usuario se filtran por su ID? (`Auth::id()` en queries â€” IDOR)
- [ ] Â¿Las rutas de API tienen middleware `auth:sanctum` o equivalente?
- [ ] Â¿Los datos del request se validan antes de usarse? (Form Request o `validate()`)
- [ ] Â¿Se usa `bcrypt()` / `Hash::make()` para contraseÃ±as? (nunca MD5/SHA1)
- [ ] Â¿Los archivos subidos se validan en tipo MIME, no solo extensiÃ³n?

### ðŸ”´ Eloquent â€” N+1 y relaciones

- [ ] Â¿Hay accesos a relaciones dentro de bucles `foreach` sin eager loading?
- [ ] Â¿Los `with()` cargan relaciones innecesarias en endpoints que no las usan?
- [ ] Â¿Se usa `->get()` donde bastarÃ­a `->first()` o `->exists()`?
- [ ] Â¿Hay `count()` de colecciones PHP donde deberÃ­a ser `->count()` en DB?
- [ ] Â¿Las relaciones polimÃ³rficas tienen Ã­ndices en `*_type` y `*_id`?
- [ ] Â¿Se usan `select()` para limitar columnas en queries de solo lectura?

### ðŸŸ  Arquitectura

- [ ] Â¿Los controladores tienen mÃ¡s de 5 mÃ©todos de lÃ³gica de negocio?
- [ ] Â¿La validaciÃ³n estÃ¡ en el controlador en lugar de Form Request?
- [ ] Â¿Los Jobs/Events tienen lÃ³gica de negocio directa o delegan a servicios?
- [ ] Â¿Los API Resources/Transformers controlan quÃ© datos se exponen?
- [ ] Â¿Los scopes de Eloquent estÃ¡n en el modelo o repetidos en controladores?
- [ ] Â¿Se usan `API Resources` o se devuelven modelos/arrays crudos?

### ðŸŸ¡ Performance

- [ ] Â¿Las queries frecuentes estÃ¡n cacheadas con `Cache::remember()`?
- [ ] Â¿Los Jobs pesados estÃ¡n en queues o se ejecutan sincrÃ³nicamente en el request?
- [ ] Â¿Las migraciones tienen Ã­ndices en columnas usadas en `WHERE` frecuentes?
- [ ] Â¿Se usa `chunk()` o `lazy()` para procesar colecciones grandes?
- [ ] Â¿Las relaciones `hasMany` con muchos registros usan `withCount()` en vez de `->count()`?

---

## Bugs crÃ­ticos mÃ¡s comunes

### 1. N+1 â€” el clÃ¡sico de Eloquent

```php
// âŒ N+1 â€” por cada post se ejecuta una query para cargar el autor
$posts = Post::all(); // SELECT * FROM posts
foreach ($posts as $post) {
    echo $post->author->name; // SELECT * FROM users WHERE id = ? (x N veces)
}

// âœ… Eager loading â€” un solo JOIN
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name; // ya cargado, 0 queries extra
}

// âœ… Lazy eager loading â€” cuando no sabes si necesitarÃ¡s la relaciÃ³n
$posts = Post::all();
if ($needsAuthors) {
    $posts->load('author'); // una sola query adicional para todos
}

// âœ… Con select â€” solo las columnas necesarias
$posts = Post::with(['author: https://github.com/favelasquez
```

### 2. Mass Assignment sin protecciÃ³n

```php
// âŒ $guarded vacÃ­o â€” cualquier campo del request se asigna a la DB
class User extends Model
{
    protected $guarded = []; // sin protecciÃ³n total
}

// En el controlador:
User::create($request->all()); // puede incluir is_admin=1, role=superuser, etc.

// âœ… CORRECTO â€” lista explÃ­cita de campos permitidos
class User extends Model
{
    protected $fillable = ['name', 'email', 'password']; // solo estos
    // 'is_admin', 'role', 'email_verified_at' â€” no se pueden asignar masivamente
}

// âœ… En el controlador â€” validar Y limitar campos
public function store(Request $request)
{
    $validated = $request->validate([
        'name'     => 'required|string|max:255',
        'email'    => 'required|email|unique:users',
        'password' => 'required|min:8|confirmed',
    ]);
    User::create($validated); // solo los campos validados
}
```

### 3. SQL Injection en Query Builder

```php
// âŒ VULNERABLE â€” interpolaciÃ³n directa en whereRaw
$users = DB::table('users')
    ->whereRaw("name = '$request->name'") // SQL Injection
    ->get();

// âŒ VULNERABLE â€” en orderBy tambiÃ©n
$column = $request->sort; // input del usuario
$users = User::orderBy($column)->get(); // puede inyectar SQL

// âœ… SEGURO â€” bindings en whereRaw
$users = DB::table('users')
    ->whereRaw('name = ?', [$request->name])
    ->get();

// âœ… SEGURO â€” whitelist para ordenamiento dinÃ¡mico
$allowedSorts = ['name', 'email', 'created_at'];
$sort = in_array($request->sort, $allowedSorts) ? $request->sort : 'created_at';
$users = User::orderBy($sort)->get();

// âœ… SEGURO â€” Eloquent parametriza automÃ¡ticamente
$users = User::where('name', $request->name)->get();
```

### 4. IDOR â€” acceso a recursos de otros usuarios

```php
// âŒ Cualquier usuario autenticado puede ver/editar Ã³rdenes ajenas
public function show(Order $order)
{
    return new OrderResource($order); // no verifica que sea del usuario actual
}

// âœ… CORRECTO â€” OpciÃ³n A: verificar manualmente
public function show(Order $order)
{
    if ($order->user_id !== Auth::id()) {
        abort(403);
    }
    return new OrderResource($order);
}

// âœ… CORRECTO â€” OpciÃ³n B: Policy (recomendada, reutilizable)
// app/Policies/OrderPolicy.php
public function view(User $user, Order $order): bool
{
    return $user->id === $order->user_id;
}

// En el controlador:
public function show(Order $order)
{
    $this->authorize('view', $order); // lanza 403 automÃ¡ticamente si falla
    return new OrderResource($order);
}

// âœ… CORRECTO â€” OpciÃ³n C: scopear la query al usuario (mÃ¡s segura)
public function show(int $id)
{
    $order = Auth::user()->orders()->findOrFail($id); // 404 si no es suyo
    return new OrderResource($order);
}
```

### 5. LÃ³gica de negocio en controladores

```php
// âŒ Controlador gordo â€” validaciÃ³n, lÃ³gica, envÃ­o de emails, todo mezclado
public function register(Request $request)
{
    $request->validate(['name' => 'required', 'email' => 'required|email']);
    $user = User::create([...]);
    $user->assignRole('customer');
    Cart::create(['user_id' => $user->id]);
    Mail::to($user)->send(new WelcomeMail($user));
    Log::info('User registered: ' . $user->id);
    return response()->json($user, 201);
}

// âœ… Controlador delgado â€” delega a Service + Form Request
// app/Http/Requests/RegisterRequest.php
class RegisterRequest extends FormRequest
{
    public function rules(): array
    {
        return ['name' => 'required|string', 'email' => 'required|email|unique:users'];
    }
}

// app/Services/UserService.php
class UserService
{
    public function register(array $data): User
    {
        $user = User::create($data);
        $user->assignRole('customer');
        Cart::create(['user_id' => $user->id]);
        UserRegistered::dispatch($user); // evento â†’ listener envÃ­a el mail
        return $user;
    }
}

// app/Http/Controllers/AuthController.php
public function register(RegisterRequest $request, UserService $userService)
{
    $user = $userService->register($request->validated());
    return new UserResource($user);
}
```

### 6. Devolver modelos crudos en APIs

```php
// âŒ Expone todos los campos, incluyendo password, tokens, campos internos
public function index()
{
    return User::all(); // JSON con password_hash, remember_token, etc.
}

// âœ… API Resource â€” contrato explÃ­cito de lo que se expone
// app/Http/Resources/UserResource.php
class UserResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id'         => $this->id,
            'name'       => $this->name,
            'email'      => $this->email,
            'created_at' => $this->created_at->toISOString(),
            'orders_count' => $this->when(
                $this->relationLoaded('orders'),
                fn() => $this->orders->count()
            ),
        ];
    }
}

// Controlador:
public function index()
{
    return UserResource::collection(User::paginate(20));
}
```

---

## Diferencias clave entre versiones de Laravel

| Feature | L6/7 | L8/9 | L10/11 |
|---|---|---|---|
| UbicaciÃ³n modelos | `App\User` | `App\Models\User` | `App\Models\User` |
| Route actions | `'Controller@method'` | `[Controller::class, 'method']` | igual |
| Factories | `define()` callback | Class-based con `definition()` | igual |
| Enums en cast | No | Parcial (L9) | `casts` con enum nativo |
| `str()` helper | No | L9+ | igual |
| `Http::fake()` | BÃ¡sico | `Http::preventStrayRequests()` | igual |
| Bootstrap | `app/Http/Kernel.php` | igual | `bootstrap/app.php` slim (L11) |
| PHP mÃ­nimo | 7.2 | 8.0 (L9) | 8.1 (L10) / 8.2 (L11) |

---

## Race Condition / Double Spend â€” Database Locking

El escenario clÃ¡sico en sistemas de crÃ©ditos, stock o cualquier recurso compartido:
dos requests concurrentes leen el mismo saldo, ambos pasan la validaciÃ³n, ambos descuentan.
Resultado: saldo negativo o recursos duplicados.

### `lockForUpdate()` â€” la soluciÃ³n correcta

```php
// âŒ SIN BLOQUEO â€” Race Condition
DB::transaction(function () use ($userId, $productId) {
    $user = User::find($userId);      // Proceso A lee: credits = 10
                                      // Proceso B tambiÃ©n lee: credits = 10
    if ($user->credits < $product->price) throw new InsufficientCreditsException();
    $user->decrement('credits', 10);  // A: credits = 0 | B: credits = -10 ðŸ’¸
});

// âœ… CON lockForUpdate() â€” Bloqueo pesimista
// SQL generado: SELECT * FROM users WHERE id = ? FOR UPDATE
DB::transaction(function () use ($userId, $productId) {
    // El segundo proceso queda BLOQUEADO aquÃ­ hasta que el primero
    // haga COMMIT â€” entonces lee el saldo ya actualizado (0, no 10)
    $user    = User::where('id', $userId)->lockForUpdate()->firstOrFail();
    $product = Product::where('id', $productId)->lockForUpdate()->firstOrFail();

    if ($user->credits < $product->price) {
        throw new InsufficientCreditsException("Saldo insuficiente.");
    }
    if ($product->stock <= 0) {
        throw new OutOfStockException("Sin stock.");
    }

    $user->decrement('credits', $product->price);
    $product->decrement('stock', 1);

    return Order::create([
        'user_id'    => $user->id,
        'product_id' => $product->id,
        'amount'     => $product->price,
        'status'     => 'completed',
    ]);
    // COMMIT â†’ libera el lock â†’ el proceso B lee credits = 0 â†’ excepciÃ³n âœ…
});
```

### Los dos tipos de lock en Laravel/Eloquent

```php
// lockForUpdate() â€” FOR UPDATE
// Bloquea lectura Y escritura de otros procesos sobre esa fila
// Usar cuando: VAS A MODIFICAR el registro
$user = User::where('id', $id)->lockForUpdate()->firstOrFail();

// sharedLock() â€” LOCK IN SHARE MODE
// Otros pueden leer, nadie puede escribir mientras tÃº lees
// Usar cuando: SOLO LEES pero necesitas consistencia (reportes en tiempo real)
$balance = User::where('id', $id)->sharedLock()->value('credits');
```

### Red de seguridad â€” constraint en la DB

```php
// Defensa final: aunque falle la capa de aplicaciÃ³n, la DB rechaza saldos negativos
// En una migration:
DB::statement('ALTER TABLE users ADD CONSTRAINT credits_non_negative CHECK (credits >= 0)');
// MySQL 8+ y PostgreSQL â€” si credits llegarÃ­a a negativo, lanza excepciÃ³n antes del commit
```

### Checklist de concurrencia â€” agregar a recursos compartidos

- [ ] Â¿Las operaciones de dÃ©bito/descuento usan `lockForUpdate()` dentro de `DB::transaction()`?
- [ ] Â¿Las validaciones de negocio (saldo suficiente, stock > 0) ocurren DESPUÃ‰S del lock?
- [ ] Â¿Hay un constraint `CHECK` en la DB para columnas que no deben ser negativas?
- [ ] Â¿Los Jobs que modifican recursos compartidos usan `withoutOverlapping()` si son Ãºnicos?
- [ ] Â¿Los endpoints de compra/dÃ©bito tienen rate limiting para frenar scripts de doble-click?

---

Leer archivos `references/` para guÃ­as completas por Ã¡rea.



