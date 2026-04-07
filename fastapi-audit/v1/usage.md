---
metadata:
  version: "1.0"
name: fastapi-audit
description: >
  Skill auditora experta de código Python con FastAPI, cubriendo todas las versiones
  desde Python 3.8 hasta 3.12+ y FastAPI 0.9x hasta 0.11x. Activar siempre que el
  usuario pida revisar, auditar, corregir o mejorar código Python con FastAPI, o cuando
  mencione routers, dependencias, Pydantic, SQLAlchemy async, Alembic, JWT, OAuth2,
  Celery, async/await, background tasks, middleware, CORS, WebSockets, o cualquier
  concepto del ecosistema FastAPI. También activar cuando el usuario muestre código con
  posibles problemas de bloqueo del event loop, N+1 en SQLAlchemy, validación insuficiente,
  secrets expuestos, endpoints sin autenticación, o dependencias mal gestionadas. Esta
  skill detecta bugs silenciosos, antipatrones async y vulnerabilidades de seguridad con
  ejemplos corregidos listos para producción en todas las versiones de FastAPI.
---

# FastAPI � Skill Auditora Experta

## Paso 1 � Detectar versión del stack antes de auditar

```
¿Qué versión de Python?
  match statement, X | Y en match  �  Python 3.10+
  str | None (union types)          �  Python 3.10+
  tomllib nativo                    �  Python 3.11+
  typing.Self                       �  Python 3.11+

¿Qué versión de Pydantic?
  from pydantic import validator    �  Pydantic v1
  from pydantic import field_validator �  Pydantic v2
  model.dict()                      �  Pydantic v1 (deprecated en v2)
  model.model_dump()                �  Pydantic v2

¿Qué versión de SQLAlchemy?
  from sqlalchemy.ext.asyncio       �  SQLAlchemy 1.4+ async
  async_sessionmaker                �  SQLAlchemy 2.0+
  session.execute(select(Model))    �  SQLAlchemy 2.0 (nuevo estilo)
  session.query(Model)              �  SQLAlchemy 1.x (estilo legacy)

¿Async o sync?
  async def + await                 �  async (correcto para FastAPI)
  def sin async en endpoint         �  SYNC � puede bloquear el event loop �a�️
```

### Trampas críticas por versión

| Stack | Trampa principal |
|---|---|
| FastAPI + Pydantic v1 | `orm_mode = True` en Config � olvidarlo rompe la serialización de ORM |
| FastAPI + Pydantic v2 | `from_attributes = True` reemplaza `orm_mode` � mezclarlos falla silenciosamente |
| SQLAlchemy 1.4 async | Session no es thread-safe � compartirla entre requests corrompe datos |
| SQLAlchemy 2.0 | `session.query()` está deprecated � usar `select()` + `session.execute()` |
| Python 3.8/3.9 | `list[str]` en type hints no funciona � usar `List[str]` de `typing` |

---

## Perfil configurado � áreas prioritarias

**Versión base:** Python 3.8 / 3.9 + FastAPI 0.9x + **Pydantic v1**
**Auditar en este orden:**
1. �x� Async � bloqueos del event loop, I/O síncrono en endpoints async
2. �x� Seguridad � JWT mal validado, endpoints sin auth, secrets en código
3. �x� Arquitectura � routers, schemas, dependencias, separación de capas
4. �x� Base de datos � N+1 en SQLAlchemy, session lifecycle, lazy loading

### �a�️ Restricciones específicas de Python 3.8/3.9 � revisar siempre

```python
# �R ROMPE en Python 3.8/3.9 � built-in generics en hints
def get_items() -> list[str]: ...          # solo Python 3.9+
def get_map()   -> dict[str, int]: ...     # solo Python 3.9+
def get_opt()   -> str | None: ...         # solo Python 3.10+

# �S& CORRECTO para Python 3.8 � importar de typing
from typing import List, Dict, Optional, Union, Tuple, Set, Any

def get_items() -> List[str]: ...
def get_map()   -> Dict[str, int]: ...
def get_opt()   -> Optional[str]: ...      # Optional[X] = Union[X, None]
def get_union() -> Union[str, int]: ...

# �R ROMPE en Python 3.8 � Annotated no existe en typing
from typing import Annotated               # solo Python 3.9+

# �S& CORRECTO para 3.8 � backport
from __future__ import annotations         # habilita hints diferidos (3.8+)
from typing_extensions import Annotated   # pip install typing_extensions

# FastAPI 0.9x � estilo clásico de Depends (no Annotated)
# field: Type = Depends(func)   <- estilo 0.9x correcto
# field: Annotated[Type, Depends(func)]  <- estilo 0.10x+, evitar en 0.9x
```

### Pydantic v1 � sintaxis correcta para este stack

```python
# �S& Pydantic v1 (la que viene con FastAPI 0.9x)
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
        orm_mode = True    # �S& v1 � NO from_attributes (eso es v2)

# Serialización v1
user.dict()                # �S& v1 � NO model_dump()
user.json()                # �S& v1 � NO model_dump_json()

# �R Mezclar v1 y v2 rompe silenciosamente
# @field_validator  �  solo Pydantic v2
# model_dump()      �  solo Pydantic v2
# ConfigDict(...)   �  solo Pydantic v2
```

> Archivos de referencia:
> - `references/async-patterns.md` � Event loop, bloqueos, background tasks, WebSockets
> - `references/seguridad.md` � JWT/OAuth2, permisos, CORS, rate limiting, secrets
> - `references/sqlalchemy-async.md` � Session lifecycle, N+1, migrations con Alembic
> - `references/arquitectura.md` � Routers, dependencies, schemas, repository pattern
> - `references/pydantic-v2.md` � Validators v2 (referencia para migración futura)

---

## Checklist de auditoría � ejecutar en cada revisión

### �x� Async � Event Loop

- [ ] ¿Hay funciones bloqueantes (`requests.get`, `time.sleep`, `open()`) dentro de `async def`?
- [ ] ¿Las operaciones CPU-intensivas corren en el event loop sin `run_in_executor`?
- [ ] ¿Se usa `asyncio.sleep` en lugar de `time.sleep` en contextos async?
- [ ] ¿Los endpoints sync pesados usan `def` (no `async def`) para que FastAPI los delegue a threadpool?
- [ ] ¿Las sesiones de SQLAlchemy se crean con `async with` y no se comparten entre requests?
- [ ] ¿Los background tasks pesados van a Celery/Redis en lugar de `BackgroundTasks`?

### �x� Seguridad

- [ ] ¿Los tokens JWT verifican `exp`, `iss` y `aud` además de la firma?
- [ ] ¿Los endpoints sensibles tienen dependencia de autenticación?
- [ ] ¿Los secrets están en variables de entorno, no en código fuente?
- [ ] ¿CORS está configurado con origins específicos, no `allow_origins=["*"]` en producción?
- [ ] ¿Los endpoints tienen rate limiting?
- [ ] ¿Las contraseñas usan `bcrypt`/`argon2`, nunca `md5`/`sha1`?
- [ ] ¿Los errores de auth devuelven mensajes genéricos (no "usuario no existe" vs "contraseña incorrecta")?

### �x� SQLAlchemy async

- [ ] ¿Se usa `selectinload()` o `joinedload()` en lugar de acceder a relaciones lazy?
- [ ] ¿La sesión se maneja con `async with AsyncSession` por request?
- [ ] ¿Las queries usan el nuevo estilo `select()` de SA 2.0 o el legacy `session.query()`?
- [ ] ¿Los `relationship()` tienen `lazy="raise"` para detectar N+1 en desarrollo?
- [ ] ¿Se usa `await session.commit()` o se olvida el await?

### �xx� Arquitectura

- [ ] ¿La lógica de negocio está en los routers/endpoints en lugar de en servicios?
- [ ] ¿Los schemas de entrada (request) y salida (response) están separados?
- [ ] ¿Las dependencias (`Depends`) se reutilizan en lugar de repetir lógica?
- [ ] ¿Los routers tienen `prefix` y `tags` para organización y documentación?
- [ ] ¿Se usa `response_model` para controlar qué datos se exponen?

### �xx� Pydantic v1 (stack Python 3.8/3.9 + FastAPI 0.9x)

- [ ] ¿Los schemas usan `class Config: orm_mode = True` (no `ConfigDict`)?
- [ ] ¿Los validators usan `@validator` (no `@field_validator`)?
- [ ] ¿La serialización usa `.dict()` / `.json()` (no `.model_dump()`)?
- [ ] ¿Los type hints usan `Optional[X]`, `List[X]`, `Dict[X,Y]` de `typing` (no `X | None`, `list[X]`)?
- [ ] ¿`Annotated` viene de `typing_extensions` si el proyecto es Python 3.8?

---

## Bugs críticos más comunes

### 1. Bloquear el event loop � el más peligroso en FastAPI

```python
# �R BLOQUEA el event loop � congela TODOS los requests mientras duerme/lee
@app.get("/users/{id}")
async def get_user(id: int):
    time.sleep(2)                    # bloquea el event loop 2 segundos
    data = requests.get("http://...")  # I/O síncrono � bloquea
    with open("file.txt") as f:       # I/O de disco síncrono � bloquea
        content = f.read()
    return {"data": data.json()}

# �S& CORRECTO � async I/O con httpx y aiofiles
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

# �S& CORRECTO � operación CPU-intensiva: usar run_in_executor
import asyncio
from concurrent.futures import ThreadPoolExecutor

@app.get("/process")
async def process_heavy():
    loop = asyncio.get_event_loop()
    # Delega a threadpool � no bloquea el event loop
    result = await loop.run_in_executor(None, heavy_cpu_function, arg1, arg2)
    return {"result": result}

# �S& CORRECTO � endpoint sync para I/O síncrono legacy
# FastAPI delega los `def` (no async) a un threadpool automáticamente
@app.get("/legacy")
def get_legacy_data():           # SIN async � FastAPI lo corre en threadpool
    return requests.get("http://legacy-api.com/data").json()
```

### 2. Session de SQLAlchemy compartida o sin await

```python
# �R Sesión global compartida � corrupción de datos en concurrencia
engine = create_async_engine(DATABASE_URL)
session = AsyncSession(engine)   # una sola sesión para todos los requests

@app.get("/users")
async def get_users():
    result = await session.execute(select(User))  # compartida entre requests
    return result.scalars().all()

# �R Olvidar el await en operaciones de sesión
async def create_user(db: AsyncSession, data: UserCreate):
    user = User(**data.model_dump())
    db.add(user)
    db.commit()        # � � falta await � no hace nada, retorna coroutine sin ejecutar
    db.refresh(user)   # � � ídem

# �S& CORRECTO � sesión por request con Depends
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

### 3. JWT sin validación completa

```python
# �R Solo verifica la firma � no verifica expiración ni audience
def get_current_user(token: str):
    payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
    # Sin options � si el token expiró, igual lo acepta
    # Sin audience � cualquier servicio puede usar el token

# �S& Validación completa del JWT
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

### 4. N+1 en SQLAlchemy async � lazy loading que explota

```python
# �R N+1 � acceder a relación lazy dentro de async context lanza excepción
# (o si está en modo "select", dispara N queries)
@app.get("/posts")
async def get_posts(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Post))
    posts = result.scalars().all()
    # �R En async SQLAlchemy, lazy loading lanza MissingGreenlet
    return [{"title": p.title, "author": p.author.name} for p in posts]

# �S& CORRECTO � eager loading explícito con selectinload o joinedload
from sqlalchemy.orm import selectinload, joinedload

@app.get("/posts", response_model=list[PostResponse])
async def get_posts(db: AsyncSession = Depends(get_db)):
    result = await db.execute(
        select(Post)
        .options(
            selectinload(Post.author),          # relación N:1 � query separada eficiente
            selectinload(Post.tags),            # relación M:N
            joinedload(Post.category),          # relación 1:1 � JOIN en la misma query
        )
        .where(Post.is_published == True)
        .order_by(Post.created_at.desc())
    )
    return result.unique().scalars().all()

# �S& En desarrollo � detectar lazy loading con lazy="raise"
class Post(Base):
    __tablename__ = "posts"
    author: https://github.com/favelasquez
```

### 5. Secrets en código fuente

```python
# �R Secrets hardcodeados � en git para siempre
SECRET_KEY = "mi-clave-super-secreta-123"
DATABASE_URL = "postgresql://admin:password123@prod-db/myapp"
AWS_SECRET = "AKIAIOSFODNN7EXAMPLE"

# �S& Settings con Pydantic v2 � desde variables de entorno
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

# Singleton con lru_cache � una sola lectura de .env
from functools import lru_cache

@lru_cache
def get_settings() -> Settings:
    return Settings()

settings = get_settings()
```

### 6. `response_model` ausente � expone datos internos

```python
# �R Sin response_model � serializa el objeto ORM completo
@app.get("/users/{id}")
async def get_user(id: int, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, id)
    return user  # incluye hashed_password, tokens internos, etc.

# �S& Con response_model � contrato explícito de la respuesta
class UserResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: int
    username: str
    email: str
    created_at: datetime
    # hashed_password NO está aquí � nunca se expone

@app.get("/users/{id}", response_model=UserResponse)
async def get_user(id: int, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, id)
    if user is None:
        raise HTTPException(status_code=404, detail="Usuario no encontrado")
    return user  # FastAPI filtra automáticamente por response_model
```

---

## Bug #7 � Type hints rotos en Python 3.8/3.9

```python
# �R ROMPE en Python 3.8 � TypeError en runtime o error en mypy
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

# �S& CORRECTO para Python 3.8
from typing import List, Dict, Optional

@router.get("/items", response_model=List[ItemResponse])
async def get_items() -> List[ItemResponse]:
    ...

@router.get("/map")
async def get_map() -> Dict[str, int]:
    ...

async def process(value: Optional[str] = None):
    ...

# �S& Alternativa con __future__ � permite sintaxis moderna en hints
# (solo como strings, no se ejecutan en runtime � sirve para anotaciones)
from __future__ import annotations   # primera línea del archivo

@router.get("/items")
async def get_items() -> list[ItemResponse]:  # funciona gracias a __future__
    ...
# �a�️ PERO: response_model= se evalúa en runtime � NO beneficia de __future__
# Siempre usar List[X] en response_model= si el proyecto es Python 3.8
```

---

## Diferencias clave por versión

### FastAPI 0.9x � diferencias vs versiones modernas

| Feature | FastAPI 0.9x (este proyecto) | FastAPI 0.10x+ |
|---|---|---|
| `Depends` en hints | `field: Type = Depends(fn)` | `field: Annotated[Type, Depends(fn)]` |
| `on_event` startup | `@app.on_event("startup")` | `lifespan` context manager |
| `APIRouter` prefix | Soportado | igual |
| `response_model` | Soportado | igual |
| Pydantic bundleado | v1 | v1 o v2 según versión |
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
| Serialización | `.dict()` | `.model_dump()` |
| JSON | `.json()` | `.model_dump_json()` |
| Settings | `BaseSettings` en pydantic | `pydantic-settings` separado |
| Performance | Base | ~5-50x más rápido |

| Feature | SQLAlchemy 1.4 | SQLAlchemy 2.0 |
|---|---|---|
| Query style | `session.query(Model)` | `select(Model)` + `session.execute()` |
| Async session | `AsyncSession` (beta) | `AsyncSession` estable + `async_sessionmaker` |
| Mapped columns | `Column(Integer)` | `Mapped[int] = mapped_column()` |
| Lazy loading | Por defecto | `lazy="select"` explícito recomendado |

---

Leer archivos `references/` para guías completas por área.



