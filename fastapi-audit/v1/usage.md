---
name: fastapi-audit
description: >
  Skill auditora experta de cÃ³digo Python con FastAPI, cubriendo todas las versiones
  desde Python 3.8 hasta 3.12+ y FastAPI 0.9x hasta 0.11x. Activar siempre que el
  usuario pida revisar, auditar, corregir o mejorar cÃ³digo Python con FastAPI, o cuando
  mencione routers, dependencias, Pydantic, SQLAlchemy async, Alembic, JWT, OAuth2,
  Celery, async/await, background tasks, middleware, CORS, WebSockets, o cualquier
  concepto del ecosistema FastAPI. TambiÃ©n activar cuando el usuario muestre cÃ³digo con
  posibles problemas de bloqueo del event loop, N+1 en SQLAlchemy, validaciÃ³n insuficiente,
  secrets expuestos, endpoints sin autenticaciÃ³n, o dependencias mal gestionadas. Esta
  skill detecta bugs silenciosos, antipatrones async y vulnerabilidades de seguridad con
  ejemplos corregidos listos para producciÃ³n en todas las versiones de FastAPI.
---

# FastAPI â€” Skill Auditora Experta

## Paso 1 â€” Detectar versiÃ³n del stack antes de auditar

```
Â¿QuÃ© versiÃ³n de Python?
  match statement, X | Y en match  â†’ Python 3.10+
  str | None (union types)          â†’ Python 3.10+
  tomllib nativo                    â†’ Python 3.11+
  typing.Self                       â†’ Python 3.11+

Â¿QuÃ© versiÃ³n de Pydantic?
  from pydantic import validator    â†’ Pydantic v1
  from pydantic import field_validator â†’ Pydantic v2
  model.dict()                      â†’ Pydantic v1 (deprecated en v2)
  model.model_dump()                â†’ Pydantic v2

Â¿QuÃ© versiÃ³n de SQLAlchemy?
  from sqlalchemy.ext.asyncio       â†’ SQLAlchemy 1.4+ async
  async_sessionmaker                â†’ SQLAlchemy 2.0+
  session.execute(select(Model))    â†’ SQLAlchemy 2.0 (nuevo estilo)
  session.query(Model)              â†’ SQLAlchemy 1.x (estilo legacy)

Â¿Async o sync?
  async def + await                 â†’ async (correcto para FastAPI)
  def sin async en endpoint         â†’ SYNC â€” puede bloquear el event loop âš ï¸
```

### Trampas crÃ­ticas por versiÃ³n

| Stack | Trampa principal |
|---|---|
| FastAPI + Pydantic v1 | `orm_mode = True` en Config â€” olvidarlo rompe la serializaciÃ³n de ORM |
| FastAPI + Pydantic v2 | `from_attributes = True` reemplaza `orm_mode` â€” mezclarlos falla silenciosamente |
| SQLAlchemy 1.4 async | Session no es thread-safe â€” compartirla entre requests corrompe datos |
| SQLAlchemy 2.0 | `session.query()` estÃ¡ deprecated â€” usar `select()` + `session.execute()` |
| Python 3.8/3.9 | `list[str]` en type hints no funciona â€” usar `List[str]` de `typing` |

---

## Perfil configurado â€” Ã¡reas prioritarias

**VersiÃ³n base:** Python 3.8 / 3.9 + FastAPI 0.9x + **Pydantic v1**
**Auditar en este orden:**
1. ðŸ”´ Async â€” bloqueos del event loop, I/O sÃ­ncrono en endpoints async
2. ðŸ”´ Seguridad â€” JWT mal validado, endpoints sin auth, secrets en cÃ³digo
3. ðŸ”´ Arquitectura â€” routers, schemas, dependencias, separaciÃ³n de capas
4. ðŸ”´ Base de datos â€” N+1 en SQLAlchemy, session lifecycle, lazy loading

### âš ï¸ Restricciones especÃ­ficas de Python 3.8/3.9 â€” revisar siempre

```python
# âŒ ROMPE en Python 3.8/3.9 â€” built-in generics en hints
def get_items() -> list[str]: ...          # solo Python 3.9+
def get_map()   -> dict[str, int]: ...     # solo Python 3.9+
def get_opt()   -> str | None: ...         # solo Python 3.10+

# âœ… CORRECTO para Python 3.8 â€” importar de typing
from typing import List, Dict, Optional, Union, Tuple, Set, Any

def get_items() -> List[str]: ...
def get_map()   -> Dict[str, int]: ...
def get_opt()   -> Optional[str]: ...      # Optional[X] = Union[X, None]
def get_union() -> Union[str, int]: ...

# âŒ ROMPE en Python 3.8 â€” Annotated no existe en typing
from typing import Annotated               # solo Python 3.9+

# âœ… CORRECTO para 3.8 â€” backport
from __future__ import annotations         # habilita hints diferidos (3.8+)
from typing_extensions import Annotated   # pip install typing_extensions

# FastAPI 0.9x â€” estilo clÃ¡sico de Depends (no Annotated)
# field: Type = Depends(func)   <- estilo 0.9x correcto
# field: Annotated[Type, Depends(func)]  <- estilo 0.10x+, evitar en 0.9x
```

### Pydantic v1 â€” sintaxis correcta para este stack

```python
# âœ… Pydantic v1 (la que viene con FastAPI 0.9x)
from pydantic import BaseModel, validator, Field
from typing import Optional, List

class PostCreate(BaseModel):
    title: str              = Field(..., min_length=3, max_length=255)
    body:  str              = Field(..., min_length=10)
    tags:  Optional[List[str]] = []

    @validator("title")
    def title_strip(cls, v: str) -> str:
        return v.strip()

    class Config:
        orm_mode = True    # âœ… v1 â€” NO from_attributes (eso es v2)

# SerializaciÃ³n v1
user.dict()                # âœ… v1 â€” NO model_dump()
user.json()                # âœ… v1 â€” NO model_dump_json()

# âŒ Mezclar v1 y v2 rompe silenciosamente
# @field_validator  â†’ solo Pydantic v2
# model_dump()      â†’ solo Pydantic v2
# ConfigDict(...)   â†’ solo Pydantic v2
```

> Archivos de referencia:
> - `references/async-patterns.md` â€” Event loop, bloqueos, background tasks, WebSockets
> - `references/seguridad.md` â€” JWT/OAuth2, permisos, CORS, rate limiting, secrets
> - `references/sqlalchemy-async.md` â€” Session lifecycle, N+1, migrations con Alembic
> - `references/arquitectura.md` â€” Routers, dependencies, schemas, repository pattern
> - `references/pydantic-v2.md` â€” Validators v2 (referencia para migraciÃ³n futura)

---

## Checklist de auditorÃ­a â€” ejecutar en cada revisiÃ³n

### ðŸ”´ Async â€” Event Loop

- [ ] Â¿Hay funciones bloqueantes (`requests.get`, `time.sleep`, `open()`) dentro de `async def`?
- [ ] Â¿Las operaciones CPU-intensivas corren en el event loop sin `run_in_executor`?
- [ ] Â¿Se usa `asyncio.sleep` en lugar de `time.sleep` en contextos async?
- [ ] Â¿Los endpoints sync pesados usan `def` (no `async def`) para que FastAPI los delegue a threadpool?
- [ ] Â¿Las sesiones de SQLAlchemy se crean con `async with` y no se comparten entre requests?
- [ ] Â¿Los background tasks pesados van a Celery/Redis en lugar de `BackgroundTasks`?

### ðŸ”´ Seguridad

- [ ] Â¿Los tokens JWT verifican `exp`, `iss` y `aud` ademÃ¡s de la firma?
- [ ] Â¿Los endpoints sensibles tienen dependencia de autenticaciÃ³n?
- [ ] Â¿Los secrets estÃ¡n en variables de entorno, no en cÃ³digo fuente?
- [ ] Â¿CORS estÃ¡ configurado con origins especÃ­ficos, no `allow_origins=["*"]` en producciÃ³n?
- [ ] Â¿Los endpoints tienen rate limiting?
- [ ] Â¿Las contraseÃ±as usan `bcrypt`/`argon2`, nunca `md5`/`sha1`?
- [ ] Â¿Los errores de auth devuelven mensajes genÃ©ricos (no "usuario no existe" vs "contraseÃ±a incorrecta")?

### ðŸ”´ SQLAlchemy async

- [ ] Â¿Se usa `selectinload()` o `joinedload()` en lugar de acceder a relaciones lazy?
- [ ] Â¿La sesiÃ³n se maneja con `async with AsyncSession` por request?
- [ ] Â¿Las queries usan el nuevo estilo `select()` de SA 2.0 o el legacy `session.query()`?
- [ ] Â¿Los `relationship()` tienen `lazy="raise"` para detectar N+1 en desarrollo?
- [ ] Â¿Se usa `await session.commit()` o se olvida el await?

### ðŸŸ  Arquitectura

- [ ] Â¿La lÃ³gica de negocio estÃ¡ en los routers/endpoints en lugar de en servicios?
- [ ] Â¿Los schemas de entrada (request) y salida (response) estÃ¡n separados?
- [ ] Â¿Las dependencias (`Depends`) se reutilizan en lugar de repetir lÃ³gica?
- [ ] Â¿Los routers tienen `prefix` y `tags` para organizaciÃ³n y documentaciÃ³n?
- [ ] Â¿Se usa `response_model` para controlar quÃ© datos se exponen?

### ðŸŸ¡ Pydantic v1 (stack Python 3.8/3.9 + FastAPI 0.9x)

- [ ] Â¿Los schemas usan `class Config: orm_mode = True` (no `ConfigDict`)?
- [ ] Â¿Los validators usan `@validator` (no `@field_validator`)?
- [ ] Â¿La serializaciÃ³n usa `.dict()` / `.json()` (no `.model_dump()`)?
- [ ] Â¿Los type hints usan `Optional[X]`, `List[X]`, `Dict[X,Y]` de `typing` (no `X | None`, `list[X]`)?
- [ ] Â¿`Annotated` viene de `typing_extensions` si el proyecto es Python 3.8?

---

## Bugs crÃ­ticos mÃ¡s comunes

### 1. Bloquear el event loop â€” el mÃ¡s peligroso en FastAPI

```python
# âŒ BLOQUEA el event loop â€” congela TODOS los requests mientras duerme/lee
@app.get("/users/{id}")
async def get_user(id: int):
    time.sleep(2)                    # bloquea el event loop 2 segundos
    data = requests.get("http://...")  # I/O sÃ­ncrono â€” bloquea
    with open("file.txt") as f:       # I/O de disco sÃ­ncrono â€” bloquea
        content = f.read()
    return {"data": data.json()}

# âœ… CORRECTO â€” async I/O con httpx y aiofiles
import httpx
import aiofiles

@app.get("/users/{id}")
async def get_user(id: int):
    await asyncio.sleep(2)           # libera el event loop
    async with httpx.AsyncClient() as client:
        data = await client.get("http://...")
    async with aiofiles.open("file.txt") as f:
        content = await f.read()
    return {"data": data.json()}

# âœ… CORRECTO â€” operaciÃ³n CPU-intensiva: usar run_in_executor
import asyncio
from concurrent.futures import ThreadPoolExecutor

@app.get("/process")
async def process_heavy():
    loop = asyncio.get_event_loop()
    # Delega a threadpool â€” no bloquea el event loop
    result = await loop.run_in_executor(None, heavy_cpu_function, arg1, arg2)
    return {"result": result}

# âœ… CORRECTO â€” endpoint sync para I/O sÃ­ncrono legacy
# FastAPI delega los `def` (no async) a un threadpool automÃ¡ticamente
@app.get("/legacy")
def get_legacy_data():           # SIN async â€” FastAPI lo corre en threadpool
    return requests.get("http://legacy-api.com/data").json()
```

### 2. Session de SQLAlchemy compartida o sin await

```python
# âŒ SesiÃ³n global compartida â€” corrupciÃ³n de datos en concurrencia
engine = create_async_engine(DATABASE_URL)
session = AsyncSession(engine)   # una sola sesiÃ³n para todos los requests

@app.get("/users")
async def get_users():
    result = await session.execute(select(User))  # compartida entre requests
    return result.scalars().all()

# âŒ Olvidar el await en operaciones de sesiÃ³n
async def create_user(db: AsyncSession, data: UserCreate):
    user = User(**data.model_dump())
    db.add(user)
    db.commit()        # â† falta await â€” no hace nada, retorna coroutine sin ejecutar
    db.refresh(user)   # â† Ã­dem

# âœ… CORRECTO â€” sesiÃ³n por request con Depends
from sqlalchemy.ext.asyncio import AsyncSession, async_sessionmaker, create_async_engine

engine = create_async_engine(settings.DATABASE_URL, pool_pre_ping=True)
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

@app.get("/users")
async def get_users(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(User).options(selectinload(User.posts)))
    return result.scalars().all()
```

### 3. JWT sin validaciÃ³n completa

```python
# âŒ Solo verifica la firma â€” no verifica expiraciÃ³n ni audience
def get_current_user(token: str):
    payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
    # Sin options â€” si el token expirÃ³, igual lo acepta
    # Sin audience â€” cualquier servicio puede usar el token

# âœ… ValidaciÃ³n completa del JWT
from jose import JWTError, jwt
from datetime import datetime, timezone

async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db: AsyncSession = Depends(get_db)
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="No se pudo validar las credenciales",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(
            token,
            settings.SECRET_KEY,
            algorithms=[settings.ALGORITHM],
            options={"require": ["exp", "sub", "iat"]},  # campos obligatorios
        )
        sub: str = payload.get("sub")
        exp: int = payload.get("exp")

        if sub is None:
            raise credentials_exception
        if datetime.fromtimestamp(exp, tz=timezone.utc) < datetime.now(timezone.utc):
            raise credentials_exception  # expirado

    except JWTError:
        raise credentials_exception

    user = await get_user_by_id(db, int(sub))
    if user is None or not user.is_active:
        raise credentials_exception
    return user
```

### 4. N+1 en SQLAlchemy async â€” lazy loading que explota

```python
# âŒ N+1 â€” acceder a relaciÃ³n lazy dentro de async context lanza excepciÃ³n
# (o si estÃ¡ en modo "select", dispara N queries)
@app.get("/posts")
async def get_posts(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Post))
    posts = result.scalars().all()
    # âŒ En async SQLAlchemy, lazy loading lanza MissingGreenlet
    return [{"title": p.title, "author": p.author.name} for p in posts]

# âœ… CORRECTO â€” eager loading explÃ­cito con selectinload o joinedload
from sqlalchemy.orm import selectinload, joinedload

@app.get("/posts", response_model=list[PostResponse])
async def get_posts(db: AsyncSession = Depends(get_db)):
    result = await db.execute(
        select(Post)
        .options(
            selectinload(Post.author),          # relaciÃ³n N:1 â€” query separada eficiente
            selectinload(Post.tags),            # relaciÃ³n M:N
            joinedload(Post.category),          # relaciÃ³n 1:1 â€” JOIN en la misma query
        )
        .where(Post.is_published == True)
        .order_by(Post.created_at.desc())
    )
    return result.unique().scalars().all()

# âœ… En desarrollo â€” detectar lazy loading con lazy="raise"
class Post(Base):
    __tablename__ = "posts"
    author: https://github.com/favelasquez
```

### 5. Secrets en cÃ³digo fuente

```python
# âŒ Secrets hardcodeados â€” en git para siempre
SECRET_KEY = "mi-clave-super-secreta-123"
DATABASE_URL = "postgresql://admin:password123@prod-db/myapp"
AWS_SECRET = "AKIAIOSFODNN7EXAMPLE"

# âœ… Settings con Pydantic v2 â€” desde variables de entorno
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )

    database_url: str
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30
    redis_url: str = "redis://localhost:6379"
    cors_origins: list[str] = ["http://localhost:3000"]

# Singleton con lru_cache â€” una sola lectura de .env
from functools import lru_cache

@lru_cache
def get_settings() -> Settings:
    return Settings()

settings = get_settings()
```

### 6. `response_model` ausente â€” expone datos internos

```python
# âŒ Sin response_model â€” serializa el objeto ORM completo
@app.get("/users/{id}")
async def get_user(id: int, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, id)
    return user  # incluye hashed_password, tokens internos, etc.

# âœ… Con response_model â€” contrato explÃ­cito de la respuesta
class UserResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: int
    username: str
    email: str
    created_at: datetime
    # hashed_password NO estÃ¡ aquÃ­ â€” nunca se expone

@app.get("/users/{id}", response_model=UserResponse)
async def get_user(id: int, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, id)
    if user is None:
        raise HTTPException(status_code=404, detail="Usuario no encontrado")
    return user  # FastAPI filtra automÃ¡ticamente por response_model
```

---

## Bug #7 â€” Type hints rotos en Python 3.8/3.9

```python
# âŒ ROMPE en Python 3.8 â€” TypeError en runtime o error en mypy
from fastapi import APIRouter
router = APIRouter()

@router.get("/items", response_model=list[ItemResponse])  # list[X] solo Python 3.9+
async def get_items() -> list[ItemResponse]:              # idem
    ...

@router.get("/map")
async def get_map() -> dict[str, int]:                    # dict[X,Y] solo Python 3.9+
    ...

async def process(value: str | None = None):              # X | Y solo Python 3.10+
    ...

# âœ… CORRECTO para Python 3.8
from typing import List, Dict, Optional

@router.get("/items", response_model=List[ItemResponse])
async def get_items() -> List[ItemResponse]:
    ...

@router.get("/map")
async def get_map() -> Dict[str, int]:
    ...

async def process(value: Optional[str] = None):
    ...

# âœ… Alternativa con __future__ â€” permite sintaxis moderna en hints
# (solo como strings, no se ejecutan en runtime â€” sirve para anotaciones)
from __future__ import annotations   # primera lÃ­nea del archivo

@router.get("/items")
async def get_items() -> list[ItemResponse]:  # funciona gracias a __future__
    ...
# âš ï¸ PERO: response_model= se evalÃºa en runtime â€” NO beneficia de __future__
# Siempre usar List[X] en response_model= si el proyecto es Python 3.8
```

---

## Diferencias clave por versiÃ³n

### FastAPI 0.9x â€” diferencias vs versiones modernas

| Feature | FastAPI 0.9x (este proyecto) | FastAPI 0.10x+ |
|---|---|---|
| `Depends` en hints | `field: Type = Depends(fn)` | `field: Annotated[Type, Depends(fn)]` |
| `on_event` startup | `@app.on_event("startup")` | `lifespan` context manager |
| `APIRouter` prefix | Soportado | igual |
| `response_model` | Soportado | igual |
| Pydantic bundleado | v1 | v1 o v2 segÃºn versiÃ³n |
| `HTTPException` detail | `str` o `dict` | igual |

| Feature | Python 3.8/3.9 | Python 3.10+ | Python 3.12+ |
|---|---|---|---|
| Union types | `Optional[str]` / `Union[str, None]` | `str \| None` nativo | igual |
| Generic hints | `List[str]`, `Dict[str, int]` | `list[str]`, `dict[str, int]` | igual |
| Match statement | No disponible | `match x: case ...` | igual |
| Type aliases | `MyType = List[str]` | `type MyType = list[str]` (3.12) | nativo |
| Exception groups | No | No | `ExceptionGroup` nativo |

| Feature | Pydantic v1 | Pydantic v2 |
|---|---|---|
| ORM compat | `orm_mode = True` en `Config` | `from_attributes = True` en `ConfigDict` |
| Validators | `@validator` | `@field_validator` |
| SerializaciÃ³n | `.dict()` | `.model_dump()` |
| JSON | `.json()` | `.model_dump_json()` |
| Settings | `BaseSettings` en pydantic | `pydantic-settings` separado |
| Performance | Base | ~5-50x mÃ¡s rÃ¡pido |

| Feature | SQLAlchemy 1.4 | SQLAlchemy 2.0 |
|---|---|---|
| Query style | `session.query(Model)` | `select(Model)` + `session.execute()` |
| Async session | `AsyncSession` (beta) | `AsyncSession` estable + `async_sessionmaker` |
| Mapped columns | `Column(Integer)` | `Mapped[int] = mapped_column()` |
| Lazy loading | Por defecto | `lazy="select"` explÃ­cito recomendado |

---

Leer archivos `references/` para guÃ­as completas por Ã¡rea.



