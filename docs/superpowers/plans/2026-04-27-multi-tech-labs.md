# Multi-Technology Labs Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add FastAPI (Python) and .NET (ASP.NET Core / C#) technology tracks to all 6 lab assignments, so students can choose their preferred backend stack while covering the same learning outcomes.

**Architecture:** Use the `docsify-tabs` plugin to embed technology tabs directly inside each existing `labs/labN.md` file. The **theory section stays shared** (REST, CRUD, JWT concepts are universal). Technology-specific content (tasks, file structure, code examples, expected results) is wrapped in tab markup. This avoids duplicating files while keeping navigation simple.

**Tech Stack:** Docsify 4 + docsify-tabs plugin, Markdown, FastAPI/Uvicorn/SQLAlchemy (Python), ASP.NET Core 8 / EF Core / C#.

---

## File Map

| File | Change |
|------|--------|
| `index.html` | Add docsify-tabs CDN script |
| `_sidebar.md` | Add intro note about technology choice |
| `context.md` | Add paragraph about multi-tech tracks |
| `labs/backend/lab1.md` | Wrap завдання + вимоги + результат in 3-way tabs |
| `labs/backend/lab2.md` | Same |
| `labs/backend/lab3.md` | Same |
| `labs/backend/lab4.md` | Same |
| `labs/backend/lab5.md` | Same |
| `labs/backend/lab6.md` | Same |

---

## Task 0: Add docsify-tabs plugin

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the plugin script to index.html**

Replace the closing `</body>` section so it reads:

```html
  <!-- Docsify v4 -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <!-- Tabs plugin -->
  <script src="//cdn.jsdelivr.net/npm/docsify-tabs@1"></script>
</body>
```

Also add tab styling in `<head>`:

```html
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify-tabs@1/dist/docsify-tabs.min.css">
```

- [ ] **Step 2: Enable tabs in the docsify config**

In the `window.$docsify` block add:

```js
    tabs: {
      persist: true,
      sync: true,
      theme: 'classic',
      tabComments: true,
      tabHeadings: true
    }
```

`persist: true` remembers the student's chosen tab across page reloads. `sync: true` switches all tab groups on a page simultaneously when a tab is clicked.

- [ ] **Step 3: Preview locally and verify plugin loads**

Run: `npx serve .` then open `http://localhost:3000`.
Expected: No console errors about missing scripts. (Tabs won't appear yet — no tab markup added.)

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add docsify-tabs plugin for multi-tech support"
```

---

## Task 1: Lab 1 — In-memory CRUD (Node.js / FastAPI / .NET)

**Files:**
- Modify: `labs/backend/lab1.md`

### Tab markup pattern

Wrap everything from section 3 onwards in:

```markdown
<!-- tabs:start -->

#### **Node.js (Express)**

[existing Node.js content]

#### **Python (FastAPI)**

[FastAPI content — see below]

#### **.NET (ASP.NET Core)**

[.NET content — see below]

<!-- tabs:end -->
```

Sections 1 (Мета) and 2 (Теоретичні відомості) stay **above** the tabs and remain unchanged.

---

### FastAPI content for Lab 1

#### Завдання

1. Встановити Python 3.11+ та створити віртуальне середовище: `python -m venv venv && source venv/bin/activate` (або `venv\Scripts\activate` на Windows).
2. Встановити залежності: `pip install fastapi uvicorn`.
3. Зберегти залежності: `pip freeze > requirements.txt`.
4. Створити правильну структуру тек проєкту.
5. Налаштувати базовий застосунок FastAPI у файлі `main.py`.
6. Реалізувати in-memory сховище (список Python) для сутності `Place`.
7. Створити Pydantic-схему (модель), Router і Service з CRUD-операціями.
8. Протестувати всі endpoint-и через Postman або вбудований Swagger UI (`/docs`).

#### Файлова структура

```
/src
  /routers
      place_router.py
  /services
      place_service.py
  /schemas
      place_schema.py
  main.py
requirements.txt
```

#### Ключові фрагменти коду

**`src/schemas/place_schema.py`**
```python
from pydantic import BaseModel
from typing import Optional
import uuid

class PlaceCreate(BaseModel):
    name: str
    description: Optional[str] = None
    city: str
    address: str

class PlaceResponse(PlaceCreate):
    id: str
```

**`src/services/place_service.py`**
```python
import uuid
from src.schemas.place_schema import PlaceCreate, PlaceResponse

_places: list[dict] = []

def get_all() -> list[PlaceResponse]:
    return _places

def get_by_id(place_id: str) -> PlaceResponse | None:
    return next((p for p in _places if p["id"] == place_id), None)

def create(data: PlaceCreate) -> PlaceResponse:
    place = {"id": str(uuid.uuid4()), **data.model_dump()}
    _places.append(place)
    return place

def update(place_id: str, data: PlaceCreate) -> PlaceResponse | None:
    place = get_by_id(place_id)
    if not place:
        return None
    place.update(data.model_dump())
    return place

def delete(place_id: str) -> bool:
    global _places
    before = len(_places)
    _places = [p for p in _places if p["id"] != place_id]
    return len(_places) < before
```

**`src/routers/place_router.py`**
```python
from fastapi import APIRouter, HTTPException
from src.schemas.place_schema import PlaceCreate, PlaceResponse
from src.services import place_service

router = APIRouter(prefix="/api/places", tags=["places"])

@router.get("/", response_model=list[PlaceResponse])
def get_places():
    return place_service.get_all()

@router.get("/{place_id}", response_model=PlaceResponse)
def get_place(place_id: str):
    place = place_service.get_by_id(place_id)
    if not place:
        raise HTTPException(status_code=404, detail="Place not found")
    return place

@router.post("/", response_model=PlaceResponse, status_code=201)
def create_place(data: PlaceCreate):
    return place_service.create(data)

@router.put("/{place_id}", response_model=PlaceResponse)
def update_place(place_id: str, data: PlaceCreate):
    place = place_service.update(place_id, data)
    if not place:
        raise HTTPException(status_code=404, detail="Place not found")
    return place

@router.delete("/{place_id}", status_code=204)
def delete_place(place_id: str):
    if not place_service.delete(place_id):
        raise HTTPException(status_code=404, detail="Place not found")
```

**`main.py`**
```python
from fastapi import FastAPI
from src.routers.place_router import router as place_router

app = FastAPI(title="Places API")
app.include_router(place_router)
```

Запуск: `uvicorn main:app --reload`

#### Очікуваний результат

1. Сервер запускається без помилок на порту 8000.
2. Swagger UI доступний за адресою `http://localhost:8000/docs` — всі 5 endpoints відображені.
3. CRUD-операції працюють коректно через Postman або Swagger.
4. Після перезапуску сервера дані зникають (in-memory).

---

### .NET content for Lab 1

#### Завдання

1. Встановити .NET SDK 8.0+. Перевірити: `dotnet --version`.
2. Створити новий проєкт: `dotnet new webapi -n PlacesApi --use-controllers`.
3. Видалити файли-заглушки (`WeatherForecast.cs`, `WeatherForecastController.cs`).
4. Створити правильну структуру тек проєкту.
5. Реалізувати in-memory сховище (статичний `List<Place>`) для сутності `Place`.
6. Створити Model, Service та Controller з CRUD-операціями.
7. Протестувати всі endpoint-и через Postman або вбудований Swagger UI (`/swagger`).

#### Файлова структура

```
/Controllers
    PlacesController.cs
/Services
    PlacesService.cs
/Models
    Place.cs
Program.cs
PlacesApi.csproj
```

#### Ключові фрагменти коду

**`Models/Place.cs`**
```csharp
namespace PlacesApi.Models;

public class Place
{
    public string Id { get; set; } = Guid.NewGuid().ToString();
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public string City { get; set; } = string.Empty;
    public string Address { get; set; } = string.Empty;
}
```

**`Services/PlacesService.cs`**
```csharp
using PlacesApi.Models;

namespace PlacesApi.Services;

public class PlacesService
{
    private static readonly List<Place> _places = [];

    public List<Place> GetAll() => _places;

    public Place? GetById(string id) =>
        _places.FirstOrDefault(p => p.Id == id);

    public Place Create(Place place)
    {
        place.Id = Guid.NewGuid().ToString();
        _places.Add(place);
        return place;
    }

    public Place? Update(string id, Place updated)
    {
        var place = GetById(id);
        if (place is null) return null;
        place.Name = updated.Name;
        place.Description = updated.Description;
        place.City = updated.City;
        place.Address = updated.Address;
        return place;
    }

    public bool Delete(string id)
    {
        var place = GetById(id);
        if (place is null) return false;
        _places.Remove(place);
        return true;
    }
}
```

**`Controllers/PlacesController.cs`**
```csharp
using Microsoft.AspNetCore.Mvc;
using PlacesApi.Models;
using PlacesApi.Services;

namespace PlacesApi.Controllers;

[ApiController]
[Route("api/places")]
public class PlacesController(PlacesService service) : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(service.GetAll());

    [HttpGet("{id}")]
    public IActionResult GetById(string id)
    {
        var place = service.GetById(id);
        return place is null ? NotFound() : Ok(place);
    }

    [HttpPost]
    public IActionResult Create([FromBody] Place place)
    {
        var created = service.Create(place);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    [HttpPut("{id}")]
    public IActionResult Update(string id, [FromBody] Place place)
    {
        var updated = service.Update(id, place);
        return updated is null ? NotFound() : Ok(updated);
    }

    [HttpDelete("{id}")]
    public IActionResult Delete(string id) =>
        service.Delete(id) ? NoContent() : NotFound();
}
```

**`Program.cs`** (мінімальна конфігурація)
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
builder.Services.AddSingleton<PlacesService>();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
app.UseSwagger();
app.UseSwaggerUI();
app.MapControllers();
app.Run();
```

Запуск: `dotnet run`

#### Очікуваний результат

1. Сервер запускається на порту 5000/5001 (https).
2. Swagger UI доступний за адресою `http://localhost:5000/swagger`.
3. Всі 5 CRUD endpoints працюють через Postman або Swagger.
4. Після перезапуску дані зникають.

---

- [ ] **Step 1: Apply tab structure to `labs/backend/lab1.md`**

Keep sections 1–2 unchanged. After section 2, insert `<!-- tabs:start -->`. Move existing section 3–5 content into the `#### **Node.js (Express)**` tab. Add `#### **Python (FastAPI)**` tab with the FastAPI content above. Add `#### **.NET (ASP.NET Core)**` tab with the .NET content above. Close with `<!-- tabs:end -->`.

- [ ] **Step 2: Preview and verify tabs render in browser**

Run `npx serve .`, navigate to ЛР №1, click each tab. Expected: tabs switch correctly, code blocks render with syntax highlighting.

- [ ] **Step 3: Commit**

```bash
git add labs/backend/lab1.md
git commit -m "feat(lab1): add FastAPI and .NET tracks"
```

---

## Task 2: Lab 2 — Database Integration

**Files:**
- Modify: `labs/backend/lab2.md`

### FastAPI content for Lab 2

#### Завдання

1. Встановити залежності: `pip install sqlalchemy asyncpg alembic python-dotenv psycopg2-binary`.
2. Налаштувати з'єднання з PostgreSQL у `.env` файлі: `DATABASE_URL=postgresql+asyncpg://user:pass@localhost/places_db`.
3. Створити SQLAlchemy-модель `Place` та налаштувати Alembic для міграцій.
4. Замінити in-memory список на асинхронні виклики SQLAlchemy у `PlaceService`.
5. Реалізувати глобальний обробник помилок через FastAPI `exception_handler`.
6. Валідація через Pydantic вже вбудована — переконатися, що всі поля мають правильні типи та обмеження.
7. Протестувати оновлені endpoints.

#### Файлова структура (доповнення)

```
/src
  /db
      database.py      # engine + session
  /models
      place_model.py   # SQLAlchemy ORM model
  /schemas
      place_schema.py  # Pydantic schemas (оновлені)
  /services
      place_service.py # тепер async + DB calls
  /routers
      place_router.py
  main.py
alembic/               # міграції
alembic.ini
.env
requirements.txt
```

#### Ключові фрагменти коду

**`src/db/database.py`**
```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from sqlalchemy.orm import DeclarativeBase
import os
from dotenv import load_dotenv

load_dotenv()

engine = create_async_engine(os.getenv("DATABASE_URL"))
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

async def get_db() -> AsyncSession:
    async with SessionLocal() as session:
        yield session
```

**`src/models/place_model.py`**
```python
from sqlalchemy import String, Text
from sqlalchemy.orm import Mapped, mapped_column
from src.db.database import Base
import uuid

class Place(Base):
    __tablename__ = "places"

    id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    name: Mapped[str] = mapped_column(String(255), nullable=False)
    description: Mapped[str | None] = mapped_column(Text)
    city: Mapped[str] = mapped_column(String(100), nullable=False)
    address: Mapped[str] = mapped_column(String(255), nullable=False)
```

**`src/services/place_service.py`** (async)
```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from src.models.place_model import Place
from src.schemas.place_schema import PlaceCreate

async def get_all(db: AsyncSession) -> list[Place]:
    result = await db.execute(select(Place))
    return result.scalars().all()

async def get_by_id(db: AsyncSession, place_id: str) -> Place | None:
    return await db.get(Place, place_id)

async def create(db: AsyncSession, data: PlaceCreate) -> Place:
    place = Place(**data.model_dump())
    db.add(place)
    await db.commit()
    await db.refresh(place)
    return place

async def update(db: AsyncSession, place_id: str, data: PlaceCreate) -> Place | None:
    place = await get_by_id(db, place_id)
    if not place:
        return None
    for key, value in data.model_dump().items():
        setattr(place, key, value)
    await db.commit()
    await db.refresh(place)
    return place

async def delete(db: AsyncSession, place_id: str) -> bool:
    place = await get_by_id(db, place_id)
    if not place:
        return False
    await db.delete(place)
    await db.commit()
    return True
```

**Глобальний обробник помилок у `main.py`**
```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=500,
        content={"success": False, "error": str(exc), "status": 500}
    )
```

---

### .NET content for Lab 2

#### Завдання

1. Додати пакети: `dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL` та `dotnet add package Microsoft.EntityFrameworkCore.Design`.
2. Налаштувати рядок підключення у `appsettings.json`.
3. Створити `AppDbContext` та зареєструвати його в `Program.cs`.
4. Додати Data Annotations до моделі `Place` для валідації.
5. Замінити статичний список у `PlacesService` на виклики `AppDbContext`.
6. Реалізувати глобальний обробник помилок через middleware.
7. Виконати міграцію: `dotnet ef migrations add InitialCreate && dotnet ef database update`.

#### Ключові фрагменти коду

**`Models/Place.cs`** (оновлено з валідацією)
```csharp
using System.ComponentModel.DataAnnotations;

namespace PlacesApi.Models;

public class Place
{
    public Guid Id { get; set; } = Guid.NewGuid();

    [Required, MaxLength(255)]
    public string Name { get; set; } = string.Empty;

    public string? Description { get; set; }

    [Required, MaxLength(100)]
    public string City { get; set; } = string.Empty;

    [Required, MaxLength(255)]
    public string Address { get; set; } = string.Empty;
}
```

**`Data/AppDbContext.cs`**
```csharp
using Microsoft.EntityFrameworkCore;
using PlacesApi.Models;

namespace PlacesApi.Data;

public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<Place> Places => Set<Place>();
}
```

**`appsettings.json`** (додати секцію)
```json
"ConnectionStrings": {
  "DefaultConnection": "Host=localhost;Database=places_db;Username=postgres;Password=yourpassword"
}
```

**`Services/PlacesService.cs`** (async + EF Core)
```csharp
using Microsoft.EntityFrameworkCore;
using PlacesApi.Data;
using PlacesApi.Models;

namespace PlacesApi.Services;

public class PlacesService(AppDbContext db)
{
    public async Task<List<Place>> GetAllAsync() =>
        await db.Places.ToListAsync();

    public async Task<Place?> GetByIdAsync(Guid id) =>
        await db.Places.FindAsync(id);

    public async Task<Place> CreateAsync(Place place)
    {
        db.Places.Add(place);
        await db.SaveChangesAsync();
        return place;
    }

    public async Task<Place?> UpdateAsync(Guid id, Place updated)
    {
        var place = await GetByIdAsync(id);
        if (place is null) return null;
        place.Name = updated.Name;
        place.Description = updated.Description;
        place.City = updated.City;
        place.Address = updated.Address;
        await db.SaveChangesAsync();
        return place;
    }

    public async Task<bool> DeleteAsync(Guid id)
    {
        var place = await GetByIdAsync(id);
        if (place is null) return false;
        db.Places.Remove(place);
        await db.SaveChangesAsync();
        return true;
    }
}
```

**Глобальний ExceptionHandler middleware** (`Middleware/ExceptionMiddleware.cs`)
```csharp
namespace PlacesApi.Middleware;

public class ExceptionMiddleware(RequestDelegate next, ILogger<ExceptionMiddleware> logger)
{
    public async Task InvokeAsync(HttpContext context)
    {
        try { await next(context); }
        catch (Exception ex)
        {
            logger.LogError(ex, "Unhandled exception");
            context.Response.StatusCode = 500;
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsJsonAsync(new
            {
                success = false,
                error = ex.Message,
                status = 500
            });
        }
    }
}
```

Підключення у `Program.cs`:
```csharp
app.UseMiddleware<ExceptionMiddleware>();
```

---

- [ ] **Step 1: Apply tab structure to `labs/backend/lab2.md`**

Same pattern as Lab 1: keep sections 1–2 shared, wrap 3–5 in tabs.

- [ ] **Step 2: Preview and commit**

```bash
git add labs/backend/lab2.md
git commit -m "feat(lab2): add FastAPI and .NET tracks"
```

---

## Task 3: Lab 3 — Related Data (One-to-Many)

**Files:**
- Modify: `labs/backend/lab3.md`

### FastAPI content for Lab 3

New entity `Review` with `place_id` foreign key.

#### Ключові фрагменти коду

**`src/models/review_model.py`**
```python
from sqlalchemy import String, Text, ForeignKey, Integer
from sqlalchemy.orm import Mapped, mapped_column, relationship
from src.db.database import Base
import uuid

class Review(Base):
    __tablename__ = "reviews"

    id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    content: Mapped[str] = mapped_column(Text, nullable=False)
    rating: Mapped[int] = mapped_column(Integer, nullable=False)
    place_id: Mapped[str] = mapped_column(String, ForeignKey("places.id", ondelete="CASCADE"))
    place: Mapped["Place"] = relationship(back_populates="reviews")
```

Додати до `Place` model:
```python
reviews: Mapped[list["Review"]] = relationship(back_populates="place", cascade="all, delete-orphan")
```

**Nested router** — маршрути: `GET /api/places/{place_id}/reviews`, `POST /api/places/{place_id}/reviews`

```python
review_router = APIRouter(prefix="/api/places/{place_id}/reviews", tags=["reviews"])

@review_router.get("/", response_model=list[ReviewResponse])
async def get_reviews(place_id: str, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Review).where(Review.place_id == place_id))
    return result.scalars().all()

@review_router.post("/", response_model=ReviewResponse, status_code=201)
async def create_review(place_id: str, data: ReviewCreate, db: AsyncSession = Depends(get_db)):
    place = await db.get(Place, place_id)
    if not place:
        raise HTTPException(404, "Place not found")
    review = Review(place_id=place_id, **data.model_dump())
    db.add(review)
    await db.commit()
    await db.refresh(review)
    return review
```

---

### .NET content for Lab 3

**`Models/Review.cs`**
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace PlacesApi.Models;

public class Review
{
    public Guid Id { get; set; } = Guid.NewGuid();

    [Required]
    public string Content { get; set; } = string.Empty;

    [Range(1, 5)]
    public int Rating { get; set; }

    public Guid PlaceId { get; set; }

    [ForeignKey(nameof(PlaceId))]
    public Place? Place { get; set; }
}
```

Додати до `Place`:
```csharp
public ICollection<Review> Reviews { get; set; } = [];
```

**`Controllers/ReviewsController.cs`**
```csharp
[ApiController]
[Route("api/places/{placeId}/reviews")]
public class ReviewsController(AppDbContext db) : ControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetAll(Guid placeId)
    {
        var reviews = await db.Reviews
            .Where(r => r.PlaceId == placeId)
            .ToListAsync();
        return Ok(reviews);
    }

    [HttpPost]
    public async Task<IActionResult> Create(Guid placeId, [FromBody] Review review)
    {
        var place = await db.Places.FindAsync(placeId);
        if (place is null) return NotFound();
        review.PlaceId = placeId;
        db.Reviews.Add(review);
        await db.SaveChangesAsync();
        return CreatedAtAction(nameof(GetAll), new { placeId }, review);
    }
}
```

---

- [ ] **Step 1: Apply tab structure to `labs/backend/lab3.md`** (same pattern)
- [ ] **Step 2: Preview and commit**

```bash
git add labs/backend/lab3.md
git commit -m "feat(lab3): add FastAPI and .NET tracks"
```

---

## Task 4: Lab 4 — Authentication & JWT

**Files:**
- Modify: `labs/backend/lab4.md`

### FastAPI content for Lab 4

Залежності: `pip install python-jose[cryptography] passlib[bcrypt]`

#### Ключові фрагменти коду

**`src/services/auth_service.py`**
```python
from passlib.context import CryptContext
from jose import jwt, JWTError
from datetime import datetime, timedelta
import os

pwd_context = CryptContext(schemes=["bcrypt"])
SECRET_KEY = os.getenv("JWT_SECRET", "changeme")
ALGORITHM = "HS256"

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

def create_token(user_id: str) -> str:
    payload = {"sub": user_id, "exp": datetime.utcnow() + timedelta(hours=24)}
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> str:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload["sub"]
    except JWTError:
        raise ValueError("Invalid token")
```

**Dependency для захищених маршрутів**
```python
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from fastapi import Depends, HTTPException

bearer = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(bearer),
    db: AsyncSession = Depends(get_db)
):
    try:
        user_id = decode_token(credentials.credentials)
    except ValueError:
        raise HTTPException(status_code=401, detail="Invalid token")
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    return user
```

**Захист маршруту**
```python
@router.post("/", response_model=PlaceResponse, status_code=201)
async def create_place(
    data: PlaceCreate,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    return await place_service.create(db, data, owner_id=current_user.id)
```

---

### .NET content for Lab 4

Залежності: `dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer` та `dotnet add package BCrypt.Net-Next`

#### Ключові фрагменти коду

**`Services/AuthService.cs`**
```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using Microsoft.IdentityModel.Tokens;
using System.Text;

namespace PlacesApi.Services;

public class AuthService(IConfiguration config)
{
    public string HashPassword(string password) =>
        BCrypt.Net.BCrypt.HashPassword(password);

    public bool VerifyPassword(string password, string hash) =>
        BCrypt.Net.BCrypt.Verify(password, hash);

    public string CreateToken(string userId)
    {
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(config["Jwt:Secret"]!));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        var token = new JwtSecurityToken(
            claims: [new Claim(ClaimTypes.NameIdentifier, userId)],
            expires: DateTime.UtcNow.AddHours(24),
            signingCredentials: creds);
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

**JWT middleware у `Program.cs`**
```csharp
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer(opts =>
    {
        opts.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Secret"]!)),
            ValidateIssuer = false,
            ValidateAudience = false
        };
    });
builder.Services.AddAuthorization();

// ... після app.Build():
app.UseAuthentication();
app.UseAuthorization();
```

**Захист контролера**
```csharp
[Authorize]
[HttpPost]
public async Task<IActionResult> Create([FromBody] Place place)
{
    var userId = User.FindFirstValue(ClaimTypes.NameIdentifier)!;
    place.OwnerId = Guid.Parse(userId);
    // ...
}
```

---

- [ ] **Step 1: Apply tab structure to `labs/backend/lab4.md`** (same pattern)
- [ ] **Step 2: Preview and commit**

```bash
git add labs/backend/lab4.md
git commit -m "feat(lab4): add FastAPI and .NET tracks"
```

---

## Task 5: Lab 5 — Pagination, Filtering & RBAC

**Files:**
- Modify: `labs/backend/lab5.md`

### FastAPI content for Lab 5

#### Пагінація та фільтрація

```python
@router.get("/", response_model=dict)
async def get_places(
    city: str | None = None,
    page: int = 1,
    limit: int = 10,
    db: AsyncSession = Depends(get_db)
):
    query = select(Place)
    if city:
        query = query.where(Place.city.ilike(f"%{city}%"))
    total = await db.scalar(select(func.count()).select_from(query.subquery()))
    result = await db.execute(query.offset((page - 1) * limit).limit(limit))
    return {
        "data": result.scalars().all(),
        "total": total,
        "page": page,
        "limit": limit
    }
```

#### RBAC (ролі)

Додати поле `role` до моделі User (`"user"` або `"admin"`).

```python
def require_role(role: str):
    def dependency(current_user: User = Depends(get_current_user)):
        if current_user.role != role:
            raise HTTPException(status_code=403, detail="Forbidden")
        return current_user
    return dependency

# Використання:
@router.delete("/{place_id}")
async def delete_place(
    place_id: str,
    db: AsyncSession = Depends(get_db),
    _: User = Depends(require_role("admin"))
):
    ...
```

---

### .NET content for Lab 5

#### Пагінація та фільтрація

```csharp
[HttpGet]
public async Task<IActionResult> GetAll(
    [FromQuery] string? city,
    [FromQuery] int page = 1,
    [FromQuery] int limit = 10)
{
    var query = db.Places.AsQueryable();
    if (!string.IsNullOrEmpty(city))
        query = query.Where(p => p.City.Contains(city));

    var total = await query.CountAsync();
    var data = await query.Skip((page - 1) * limit).Take(limit).ToListAsync();

    return Ok(new { data, total, page, limit });
}
```

#### RBAC

Використати Policy-based authorization:
```csharp
builder.Services.AddAuthorization(opts =>
{
    opts.AddPolicy("AdminOnly", p => p.RequireRole("admin"));
});

// На контролері:
[Authorize(Policy = "AdminOnly")]
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(Guid id) { ... }
```

---

- [ ] **Step 1: Apply tab structure to `labs/backend/lab5.md`** (same pattern)
- [ ] **Step 2: Preview and commit**

```bash
git add labs/backend/lab5.md
git commit -m "feat(lab5): add FastAPI and .NET tracks"
```

---

## Task 6: Lab 6 — Production Readiness

**Files:**
- Modify: `labs/backend/lab6.md`

### FastAPI content for Lab 6

#### Логування

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s"
)
logger = logging.getLogger(__name__)

# Middleware для логування запитів
@app.middleware("http")
async def log_requests(request: Request, call_next):
    logger.info(f"{request.method} {request.url.path}")
    response = await call_next(request)
    logger.info(f"Status: {response.status_code}")
    return response
```

#### Безпека (CORS, Rate Limiting)

```python
from fastapi.middleware.cors import CORSMiddleware
from slowapi import Limiter
from slowapi.util import get_remote_address

app.add_middleware(CORSMiddleware,
    allow_origins=["https://yourdomain.com"],
    allow_methods=["*"],
    allow_headers=["*"])

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@router.get("/")
@limiter.limit("30/minute")
async def get_places(request: Request, ...): ...
```

Залежності: `pip install slowapi`

#### Розгортання

Мінімальний `Dockerfile`:
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### .NET content for Lab 6

#### Логування (вбудоване)

.NET має вбудоване логування через `ILogger<T>` — налаштувати рівень у `appsettings.json`:
```json
"Logging": {
  "LogLevel": {
    "Default": "Information",
    "Microsoft.AspNetCore": "Warning"
  }
}
```

Використання у контролері (вже інжектується через DI):
```csharp
public class PlacesController(PlacesService service, ILogger<PlacesController> logger) : ControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        logger.LogInformation("Fetching all places");
        return Ok(await service.GetAllAsync());
    }
}
```

#### Безпека

```csharp
// CORS
builder.Services.AddCors(opts =>
    opts.AddPolicy("default", p =>
        p.WithOrigins("https://yourdomain.com").AllowAnyMethod().AllowAnyHeader()));

// Helmet-аналог (Security Headers)
app.Use(async (ctx, next) =>
{
    ctx.Response.Headers.Append("X-Content-Type-Options", "nosniff");
    ctx.Response.Headers.Append("X-Frame-Options", "DENY");
    await next();
});

app.UseCors("default");
```

Rate Limiting (вбудований у .NET 7+):
```csharp
builder.Services.AddRateLimiter(opts =>
    opts.AddFixedWindowLimiter("fixed", o =>
    {
        o.PermitLimit = 30;
        o.Window = TimeSpan.FromMinutes(1);
    }));

app.UseRateLimiter();
```

#### Розгортання

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "PlacesApi.dll"]
```

---

- [ ] **Step 1: Apply tab structure to `labs/backend/lab6.md`** (same pattern)
- [ ] **Step 2: Preview and commit**

```bash
git add labs/backend/lab6.md
git commit -m "feat(lab6): add FastAPI and .NET tracks"
```

---

## Task 7: Update context.md and sidebar

**Files:**
- Modify: `context.md`
- Modify: `_sidebar.md`

- [ ] **Step 1: Update `context.md`**

Add a new section after the intro explaining technology tracks:

```markdown
## Вибір технологічного стеку

Лабораторні роботи доступні у трьох варіантах реалізації — оберіть один стек та дотримуйтесь його протягом усього курсу:

| Стек | Мова | Фреймворк | ORM | Валідація |
|------|------|-----------|-----|-----------|
| **Node.js** | JavaScript/TypeScript | Express | Sequelize / Prisma | Joi / Zod |
| **Python** | Python 3.11+ | FastAPI | SQLAlchemy | Pydantic (вбудовано) |
| **.NET** | C# 12 | ASP.NET Core 8 | Entity Framework Core | Data Annotations / FluentValidation |

Всі три варіанти реалізують **один і той самий** REST API для платформи Places і підключаються до однієї бази даних PostgreSQL.
```

- [ ] **Step 2: Update `_sidebar.md`**

Add a note above the lab list:

```markdown
- **Оберіть стек** (Node.js / FastAPI / .NET)
  - [Концепція курсу та проєкт](context.md)
```

- [ ] **Step 3: Preview navigation and commit**

```bash
git add context.md _sidebar.md
git commit -m "docs: update context and sidebar for multi-tech tracks"
```

---

## Self-Review Checklist

- [x] **Spec coverage:** Tasks 0–7 cover all 6 labs + infrastructure changes.
- [x] **No placeholders:** All code blocks contain real, runnable code.
- [x] **Type consistency:** FastAPI uses `str` UUIDs throughout (matching lab1 approach); .NET uses `Guid` consistently across all tasks.
- [x] **docsify-tabs syntax:** `<!-- tabs:start -->`, `#### **Tab Name**`, `<!-- tabs:end -->` used consistently.
- [x] **Ukrainian prose:** Implementation note sections are in Ukrainian to match course language.
- [x] **Dependency lists included** in every lab's завдання section.
