---
name: laravel-audit
description: >
  Skill auditora experta de código Laravel 6 hasta Laravel 11, cubriendo Eloquent ORM,
  seguridad, arquitectura y performance. Activar siempre que el usuario pida revisar,
  auditar, corregir o mejorar código PHP con Laravel, o cuando mencione Eloquent,
  Artisan, migrations, Blade, Controllers, Middleware, Service Providers, Policies,
  Gates, Sanctum, Passport, Jobs, Queues, Events, Livewire, Inertia, PHPUnit, Pest,
  Repository pattern en Laravel, Service Layer, o cualquier concepto del ecosistema
  Laravel. También activar cuando el usuario muestre código con posibles problemas de
  N+1 en Eloquent, SQL Injection en Query Builder, Mass Assignment sin protección,
  lógica de negocio en controladores, o falta de validación. Esta skill detecta bugs
  silenciosos, antipatrones y vulnerabilidades con ejemplos corregidos listos para
  producción en todas las versiones de Laravel del 6 al 11.
---

# Laravel 6–11 — Skill Auditora Experta

## Paso 1 — Detectar la versión antes de auditar

```
¿Qué version de Laravel es?
  composer.json: "laravel/framework": "^6.x"  → Laravel 6 (LTS, PHP 7.2+)
  composer.json: "laravel/framework": "^8.x"  → Laravel 8 (modelos en Models/, factories tipadas)
  composer.json: "laravel/framework": "^10.x" → Laravel 10 (PHP 8.1+, readonly props)
  composer.json: "laravel/framework": "^11.x" → Laravel 11 (bootstrap/app.php slim, PHP 8.2+)

Señales en el código:
  Route::get(..., 'Controller@method')   → Laravel 6/7 (string-based routes)
  Route::get(..., [Controller::class, 'method']) → Laravel 8+ (class-based)
  $fillable en el modelo                 → cualquier versión
  $guarded = []                          → ⚠️ sin protección de mass assignment
  HasFactory en el modelo                → Laravel 8+
  #[Route(...)] atributos PHP 8          → Laravel 10/11
```

### Cambios críticos por versión

| Versión | PHP mín | Cambio clave para auditar |
|---|---|---|
| Laravel 6 LTS | 7.2 | `string-based` route actions, sin `Models/` folder |
| Laravel 7 | 7.2.5 | `Route::view()`, `blade` components anónimos |
| Laravel 8 | 7.3 | Modelos en `App\Models\`, factories con tipos, `match()` |
| Laravel 9 | 8.0 | PHP 8 named args, `enum` nativo, `str()` helper |
| Laravel 10 | 8.1 | `readonly` properties, `Process` facade, sin deprecaciones |
| Laravel 11 | 8.2 | `bootstrap/app.php` simplificado, `Dumpable` trait |

---

## Perfil configurado — áreas prioritarias

**Stack:** API REST · Livewire/Inertia · Queues/Jobs/Events · Testing
**Auditar en este orden:**
1. 🔴 Seguridad — Mass Assignment, SQL Injection, Auth/Policies, IDOR
2. 🔴 Eloquent — N+1, eager loading, scopes mal usados
3. 🟠 Arquitectura — lógica en controladores, Service Layer, Repository
4. 🟡 Performance — caché, queries lentas, jobs/queues, índices

> Archivos de referencia detallados:
> - `references/eloquent.md` — N+1, relaciones, scopes, mutators, casting
> - `references/seguridad.md` — Mass Assignment, SQL Injection, Policies, Sanctum
> - `references/arquitectura.md` — Service Layer, Repository, Form Requests, Resources
> - `references/performance.md` — Cache, Query optimization, Jobs, índices en migraciones
> - `references/testing.md` — PHPUnit, Pest, factories, mocking, API testing

---

## Checklist de auditoría — ejecutar en cada revisión

### 🔴 Seguridad

- [ ] ¿El modelo tiene `$guarded = []` sin `$fillable`? (mass assignment total)
- [ ] ¿Hay `DB::select()` o `whereRaw()` con interpolación de variables? (SQL Injection)
- [ ] ¿Los endpoints usan `$this->authorize()` o Policies? (falta de autorización)
- [ ] ¿Los recursos de un usuario se filtran por su ID? (`Auth::id()` en queries — IDOR)
- [ ] ¿Las rutas de API tienen middleware `auth:sanctum` o equivalente?
- [ ] ¿Los datos del request se validan antes de usarse? (Form Request o `validate()`)
- [ ] ¿Se usa `bcrypt()` / `Hash::make()` para contraseñas? (nunca MD5/SHA1)
- [ ] ¿Los archivos subidos se validan en tipo MIME, no solo extensión?

### 🔴 Eloquent — N+1 y relaciones

- [ ] ¿Hay accesos a relaciones dentro de bucles `foreach` sin eager loading?
- [ ] ¿Los `with()` cargan relaciones innecesarias en endpoints que no las usan?
- [ ] ¿Se usa `->get()` donde bastaría `->first()` o `->exists()`?
- [ ] ¿Hay `count()` de colecciones PHP donde debería ser `->count()` en DB?
- [ ] ¿Las relaciones polimórficas tienen índices en `*_type` y `*_id`?
- [ ] ¿Se usan `select()` para limitar columnas en queries de solo lectura?

### 🟠 Arquitectura

- [ ] ¿Los controladores tienen más de 5 métodos de lógica de negocio?
- [ ] ¿La validación está en el controlador en lugar de Form Request?
- [ ] ¿Los Jobs/Events tienen lógica de negocio directa o delegan a servicios?
- [ ] ¿Los API Resources/Transformers controlan qué datos se exponen?
- [ ] ¿Los scopes de Eloquent están en el modelo o repetidos en controladores?
- [ ] ¿Se usan `API Resources` o se devuelven modelos/arrays crudos?

### 🟡 Performance

- [ ] ¿Las queries frecuentes están cacheadas con `Cache::remember()`?
- [ ] ¿Los Jobs pesados están en queues o se ejecutan sincrónicamente en el request?
- [ ] ¿Las migraciones tienen índices en columnas usadas en `WHERE` frecuentes?
- [ ] ¿Se usa `chunk()` o `lazy()` para procesar colecciones grandes?
- [ ] ¿Las relaciones `hasMany` con muchos registros usan `withCount()` en vez de `->count()`?

---

## Bugs críticos más comunes

### 1. N+1 — el clásico de Eloquent

```php
// ❌ N+1 — por cada post se ejecuta una query para cargar el autor
$posts = Post::all(); // SELECT * FROM posts
foreach ($posts as $post) {
    echo $post->author->name; // SELECT * FROM users WHERE id = ? (x N veces)
}

// ✅ Eager loading — un solo JOIN
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name; // ya cargado, 0 queries extra
}

// ✅ Lazy eager loading — cuando no sabes si necesitarás la relación
$posts = Post::all();
if ($needsAuthors) {
    $posts->load('author'); // una sola query adicional para todos
}

// ✅ Con select — solo las columnas necesarias
$posts = Post::with(['author:id,name,email'])->select('id', 'title', 'user_id')->get();
```

### 2. Mass Assignment sin protección

```php
// ❌ $guarded vacío — cualquier campo del request se asigna a la DB
class User extends Model
{
    protected $guarded = []; // sin protección total
}

// En el controlador:
User::create($request->all()); // puede incluir is_admin=1, role=superuser, etc.

// ✅ CORRECTO — lista explícita de campos permitidos
class User extends Model
{
    protected $fillable = ['name', 'email', 'password']; // solo estos
    // 'is_admin', 'role', 'email_verified_at' — no se pueden asignar masivamente
}

// ✅ En el controlador — validar Y limitar campos
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
// ❌ VULNERABLE — interpolación directa en whereRaw
$users = DB::table('users')
    ->whereRaw("name = '$request->name'") // SQL Injection
    ->get();

// ❌ VULNERABLE — en orderBy también
$column = $request->sort; // input del usuario
$users = User::orderBy($column)->get(); // puede inyectar SQL

// ✅ SEGURO — bindings en whereRaw
$users = DB::table('users')
    ->whereRaw('name = ?', [$request->name])
    ->get();

// ✅ SEGURO — whitelist para ordenamiento dinámico
$allowedSorts = ['name', 'email', 'created_at'];
$sort = in_array($request->sort, $allowedSorts) ? $request->sort : 'created_at';
$users = User::orderBy($sort)->get();

// ✅ SEGURO — Eloquent parametriza automáticamente
$users = User::where('name', $request->name)->get();
```

### 4. IDOR — acceso a recursos de otros usuarios

```php
// ❌ Cualquier usuario autenticado puede ver/editar órdenes ajenas
public function show(Order $order)
{
    return new OrderResource($order); // no verifica que sea del usuario actual
}

// ✅ CORRECTO — Opción A: verificar manualmente
public function show(Order $order)
{
    if ($order->user_id !== Auth::id()) {
        abort(403);
    }
    return new OrderResource($order);
}

// ✅ CORRECTO — Opción B: Policy (recomendada, reutilizable)
// app/Policies/OrderPolicy.php
public function view(User $user, Order $order): bool
{
    return $user->id === $order->user_id;
}

// En el controlador:
public function show(Order $order)
{
    $this->authorize('view', $order); // lanza 403 automáticamente si falla
    return new OrderResource($order);
}

// ✅ CORRECTO — Opción C: scopear la query al usuario (más segura)
public function show(int $id)
{
    $order = Auth::user()->orders()->findOrFail($id); // 404 si no es suyo
    return new OrderResource($order);
}
```

### 5. Lógica de negocio en controladores

```php
// ❌ Controlador gordo — validación, lógica, envío de emails, todo mezclado
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

// ✅ Controlador delgado — delega a Service + Form Request
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
        UserRegistered::dispatch($user); // evento → listener envía el mail
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
// ❌ Expone todos los campos, incluyendo password, tokens, campos internos
public function index()
{
    return User::all(); // JSON con password_hash, remember_token, etc.
}

// ✅ API Resource — contrato explícito de lo que se expone
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
| Ubicación modelos | `App\User` | `App\Models\User` | `App\Models\User` |
| Route actions | `'Controller@method'` | `[Controller::class, 'method']` | igual |
| Factories | `define()` callback | Class-based con `definition()` | igual |
| Enums en cast | No | Parcial (L9) | `casts` con enum nativo |
| `str()` helper | No | L9+ | igual |
| `Http::fake()` | Básico | `Http::preventStrayRequests()` | igual |
| Bootstrap | `app/Http/Kernel.php` | igual | `bootstrap/app.php` slim (L11) |
| PHP mínimo | 7.2 | 8.0 (L9) | 8.1 (L10) / 8.2 (L11) |

---

## Race Condition / Double Spend — Database Locking

El escenario clásico en sistemas de créditos, stock o cualquier recurso compartido:
dos requests concurrentes leen el mismo saldo, ambos pasan la validación, ambos descuentan.
Resultado: saldo negativo o recursos duplicados.

### `lockForUpdate()` — la solución correcta

```php
// ❌ SIN BLOQUEO — Race Condition
DB::transaction(function () use ($userId, $productId) {
    $user = User::find($userId);      // Proceso A lee: credits = 10
                                      // Proceso B también lee: credits = 10
    if ($user->credits < $product->price) throw new InsufficientCreditsException();
    $user->decrement('credits', 10);  // A: credits = 0 | B: credits = -10 💸
});

// ✅ CON lockForUpdate() — Bloqueo pesimista
// SQL generado: SELECT * FROM users WHERE id = ? FOR UPDATE
DB::transaction(function () use ($userId, $productId) {
    // El segundo proceso queda BLOQUEADO aquí hasta que el primero
    // haga COMMIT — entonces lee el saldo ya actualizado (0, no 10)
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
    // COMMIT → libera el lock → el proceso B lee credits = 0 → excepción ✅
});
```

### Los dos tipos de lock en Laravel/Eloquent

```php
// lockForUpdate() — FOR UPDATE
// Bloquea lectura Y escritura de otros procesos sobre esa fila
// Usar cuando: VAS A MODIFICAR el registro
$user = User::where('id', $id)->lockForUpdate()->firstOrFail();

// sharedLock() — LOCK IN SHARE MODE
// Otros pueden leer, nadie puede escribir mientras tú lees
// Usar cuando: SOLO LEES pero necesitas consistencia (reportes en tiempo real)
$balance = User::where('id', $id)->sharedLock()->value('credits');
```

### Red de seguridad — constraint en la DB

```php
// Defensa final: aunque falle la capa de aplicación, la DB rechaza saldos negativos
// En una migration:
DB::statement('ALTER TABLE users ADD CONSTRAINT credits_non_negative CHECK (credits >= 0)');
// MySQL 8+ y PostgreSQL — si credits llegaría a negativo, lanza excepción antes del commit
```

### Checklist de concurrencia — agregar a recursos compartidos

- [ ] ¿Las operaciones de débito/descuento usan `lockForUpdate()` dentro de `DB::transaction()`?
- [ ] ¿Las validaciones de negocio (saldo suficiente, stock > 0) ocurren DESPUÉS del lock?
- [ ] ¿Hay un constraint `CHECK` en la DB para columnas que no deben ser negativas?
- [ ] ¿Los Jobs que modifican recursos compartidos usan `withoutOverlapping()` si son únicos?
- [ ] ¿Los endpoints de compra/débito tienen rate limiting para frenar scripts de doble-click?

---

Leer archivos `references/` para guías completas por área.
