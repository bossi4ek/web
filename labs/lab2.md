# Лабораторна робота №2: Інтеграція Бази Даних

## 1. Мета роботи
Опанувати роботу з реляційними базами даних (PostgreSQL), використовуючи ORM (Object-Relational Mapping). Навчитися створювати моделі даних, виконувати асинхронні операції з БД, а також реалізувати механізми глобальної обробки помилок та валідації вхідних даних.

## 2. Теоретичні відомості
- **PostgreSQL** — об'єктно-реляційна система управління базами даних (СУБД).
- **ORM (Object-Relational Mapping)** — технологія, яка пов'язує таблиці БД з об'єктами коду. Дозволяє працювати з даними без написання SQL вручну.
- **Міграції** — версіонований механізм зміни схеми БД, що дозволяє відтворити або відкотити структуру таблиць.
- **Асинхронність (Async/Await)** — механізм виконання операцій (запитів до БД) без блокування основного потоку.
- **Глобальна обробка помилок** — централізований обробник, який перехоплює всі помилки та повертає клієнту стандартизований JSON-відповідь.
- **Валідація даних** — перевірка вхідних даних на відповідність правилам перед тим, як вони потраплять до БД.

> Продовжуйте у тому самому стеку, який ви обрали у ЛР №1.

<!-- tabs:start -->

#### **Node.js (Express)**

## 3. Завдання

1. Розгорнути локально або у хмарі базу даних PostgreSQL.
2. Підключити до проєкту ORM (`Sequelize` або `Prisma`) та налаштувати з'єднання з БД (параметри підключення — у файл `.env`).
3. Створити модель `Place` у БД. Використати міграції або синхронізацію ORM.
4. Модифікувати `PlaceService`: замінити in-memory масив на асинхронні виклики ORM.
5. Створити схему валідації (Joi або Zod) та Validation Middleware, який перевіряє `req.body` перед контролером.
6. Реалізувати Error Handling Middleware (4 параметри: `err, req, res, next`). Усі контролери мають передавати помилки через `next(error)`.
7. Протестувати оновлені endpoints через Postman.

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Place` (БД)
- `id` (UUID або Serial Integer — Primary Key)
- `name` (VARCHAR, обов'язкове)
- `description` (Text, необов'язкове)
- `city` (VARCHAR, обов'язкове)
- `address` (VARCHAR, обов'язкове)

### 4.2. Файлова структура (доповнення до ЛР №1)
```
/src
  /config
      db.js              # налаштування підключення ORM
  /middlewares
      error.middleware.js
      validate.middleware.js
  /validations
      place.validation.js
  /models
      place.model.js
.env
```

### 4.3. Формат відповіді при помилці
```json
{
  "success": false,
  "error": "Place with ID 5 not found",
  "status": 404
}
```

### 4.4. Формат відповіді при помилці валідації
```json
{
  "success": false,
  "error": "Validation error: 'city' is required",
  "status": 400
}
```

## 5. Очікуваний результат
1. Додаток успішно підключається до PostgreSQL. Таблиця `places` створюється через міграцію або синхронізацію.
2. Всі CRUD-операції з ЛР №1 зберігають/зчитують дані з реальної БД.
3. Некоректні дані у `POST /api/places` повертають `400 Bad Request` з описом помилки.
4. Запит на неіснуюче місце повертає `404`, внутрішня помилка — `500` через Error Middleware.

#### **Python (FastAPI)**

## 3. Завдання

1. Розгорнути локально або у хмарі базу даних PostgreSQL.
2. Встановити залежності: `pip install sqlalchemy asyncpg alembic python-dotenv psycopg2-binary`.
3. Налаштувати підключення до БД у `.env`: `DATABASE_URL=postgresql+asyncpg://user:pass@localhost/places_db`.
4. Створити SQLAlchemy-модель `Place` та налаштувати Alembic для міграцій.
5. Замінити in-memory список на асинхронні виклики SQLAlchemy у `place_service.py`.
6. Реалізувати глобальний обробник помилок через FastAPI `exception_handler`.
7. Pydantic-валідація вже вбудована — переконатися, що поля мають правильні типи та обмеження (`min_length`, `max_length`).
8. Протестувати оновлені endpoints через Postman або Swagger UI.

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Place` (БД)
- `id` (String, UUID, Primary Key)
- `name` (String 255, NOT NULL)
- `description` (Text, nullable)
- `city` (String 100, NOT NULL)
- `address` (String 255, NOT NULL)

### 4.2. Файлова структура (доповнення до ЛР №1)
```
/src
  /db
      database.py       # engine + SessionLocal + Base
  /models
      place_model.py    # SQLAlchemy ORM модель
alembic/
alembic.ini
.env
```

### 4.3. Ключові фрагменти коду

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

**`src/services/place_service.py`** (async, замінює попередній)
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
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import JSONResponse

app = FastAPI(title="Places API")

@app.exception_handler(HTTPException)
async def http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={"success": False, "error": exc.detail, "status": exc.status_code}
    )

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=500,
        content={"success": False, "error": str(exc), "status": 500}
    )
```

**Оновлений роутер з `Depends(get_db)`**
```python
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession
from src.db.database import get_db

@router.get("/", response_model=list[PlaceResponse])
async def get_places(db: AsyncSession = Depends(get_db)):
    return await place_service.get_all(db)

@router.post("/", response_model=PlaceResponse, status_code=201)
async def create_place(data: PlaceCreate, db: AsyncSession = Depends(get_db)):
    return await place_service.create(db, data)
```

### 4.4. Міграція

```bash
alembic init alembic
# У alembic/env.py підключити Base та DATABASE_URL
alembic revision --autogenerate -m "create places table"
alembic upgrade head
```

## 5. Очікуваний результат
1. Додаток підключається до PostgreSQL. Таблиця `places` створюється через Alembic.
2. Всі CRUD-операції зберігають/зчитують дані з реальної БД.
3. Некоректні дані у `POST /api/places` повертають `422 Unprocessable Entity` від Pydantic.
4. Запит на неіснуюче місце повертає `{"success": false, "error": "Place not found", "status": 404}`.

#### **.NET (ASP.NET Core)**

## 3. Завдання

1. Розгорнути локально або у хмарі базу даних PostgreSQL.
2. Додати пакети: `dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL` та `dotnet add package Microsoft.EntityFrameworkCore.Design`.
3. Налаштувати рядок підключення у `appsettings.json`.
4. Створити `AppDbContext` та зареєструвати його у `Program.cs`.
5. Додати Data Annotations до моделі `Place` для валідації.
6. Замінити статичний список у `PlacesService` на виклики `AppDbContext` (async).
7. Реалізувати глобальний ExceptionHandler middleware.
8. Виконати міграцію: `dotnet ef migrations add InitialCreate && dotnet ef database update`.
9. Протестувати оновлені endpoints через Postman або Swagger.

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Place` (БД)
- `Id` (Guid, Primary Key)
- `Name` (VARCHAR 255, NOT NULL)
- `Description` (Text, nullable)
- `City` (VARCHAR 100, NOT NULL)
- `Address` (VARCHAR 255, NOT NULL)

### 4.2. Файлова структура (доповнення до ЛР №1)
```
/Data
    AppDbContext.cs
/Middleware
    ExceptionMiddleware.cs
appsettings.json   # рядок підключення
```

### 4.3. Ключові фрагменти коду

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

**`Middleware/ExceptionMiddleware.cs`**
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

**`Program.cs`** (оновлено)
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
builder.Services.AddDbContext<AppDbContext>(opts =>
    opts.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddScoped<PlacesService>();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
app.UseMiddleware<ExceptionMiddleware>();
app.UseSwagger();
app.UseSwaggerUI();
app.MapControllers();
app.Run();
```

### 4.4. Міграція

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

## 5. Очікуваний результат
1. Додаток підключається до PostgreSQL. Таблиця `Places` створюється через EF Core міграцію.
2. Всі CRUD-операції зберігають/зчитують дані з реальної БД.
3. Некоректні дані у `POST /api/places` (порожній `Name`) повертають `400 Bad Request` від Data Annotations.
4. Запит на неіснуюче місце повертає `404 Not Found`. Внутрішня помилка — `500` через `ExceptionMiddleware`.

<!-- tabs:end -->
