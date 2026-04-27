# Лабораторна робота №3: Робота з пов'язаними даними (Reviews)

## 1. Мета роботи
Опанувати роботу з пов'язаними даними в базах даних за допомогою ORM. Навчитися налаштовувати зв'язки типу "один-до-багатьох" (One-to-Many). Зрозуміти принципи побудови та обробки вкладених маршрутів (nested routes) в REST API для роботи з залежними сутностями.

## 2. Теоретичні відомості
- **Зв'язки в БД** — зв'язки між таблицями. Основні типи: "один-до-одного" (1:1), "один-до-багатьох" (1:N), "багато-до-багатьох" (M:N).
- **Зовнішній ключ (Foreign Key)** — колонка, що вказує на Primary Key іншої таблиці, забезпечуючи зв'язок між ними.
- **Вкладені маршрути (Nested Routes)** — підхід до REST API, де URL відображає ієрархію ресурсів: `/api/places/:placeId/reviews`.
- **Каскадне видалення (ON DELETE CASCADE)** — автоматичне видалення пов'язаних записів при видаленні батьківського запису.

> Продовжуйте у тому самому стеку, який ви обрали у ЛР №1.

<!-- tabs:start -->

#### **Node.js (Express)**

## 3. Завдання

1. Створити модель `Review` у ORM згідно з доменною моделлю.
2. Налаштувати зв'язок 1:N між `Place` та `Review` (одне місце — багато відгуків).
3. Додати валідацію для `Review` (рейтинг 1–5, коментар — текст).
4. Налаштувати вкладений роутер з `mergeParams: true` для прив'язки відгуків до місця.
5. Реалізувати контролери та сервіси для `GET` та `POST` відгуків.
6. Протестувати через Postman. Перевірити каскадне видалення відгуків при видаленні місця.

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Review`
- `id` (UUID або Serial Integer — Primary Key)
- `rating` (Integer, обов'язкове, від 1 до 5)
- `comment` (Text, обов'язкове)
- `placeId` (Foreign Key → `places`, обов'язкове, ON DELETE CASCADE)

### 4.2. Файлова структура (доповнення)
```
/src
  /controllers
      review.controller.js
  /services
      review.service.js
  /routes
      review.routes.js
  /validations
      review.validation.js
  /models
      review.model.js
```

### 4.3. API Endpoints
- `GET /api/places/:placeId/reviews` — всі відгуки для місця.
- `POST /api/places/:placeId/reviews` — створити відгук для місця.
- `DELETE /api/reviews/:id` — видалити відгук (опціонально).

*Приклад відповіді на POST запит:*
```json
{
  "success": true,
  "data": {
    "id": "1",
    "rating": 5,
    "comment": "Чудове місце для прогулянок!",
    "placeId": "123e4567-e89b-12d3-a456-426614174000"
  },
  "status": 201
}
```

### 4.4. Підключення вкладеного роутера

```js
// review.routes.js
const router = express.Router({ mergeParams: true });

// server.js / app.js
app.use('/api/places/:placeId/reviews', reviewRouter);
```

## 5. Очікуваний результат
1. Таблиця `reviews` створена з FK на `places`.
2. `POST /api/places/:placeId/reviews` додає відгук до конкретного місця.
3. `GET /api/places/:placeId/reviews` повертає лише відгуки цього місця.
4. Некоректний рейтинг → `400 Bad Request`.
5. Неіснуючий `placeId` → `404 Not Found`.

#### **Python (FastAPI)**

## 3. Завдання

1. Створити SQLAlchemy-модель `Review` та Pydantic-схему.
2. Налаштувати зв'язок 1:N між `Place` та `Review` через `relationship` та `ForeignKey`.
3. Додати валідацію рейтингу (1–5) через Pydantic `Field(ge=1, le=5)`.
4. Реалізувати вкладений роутер для відгуків.
5. Реалізувати сервісні функції та endpoint-и для `GET` і `POST` відгуків.
6. Протестувати через Postman або Swagger. Перевірити каскадне видалення.

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Review`
- `id` (String UUID, Primary Key)
- `rating` (Integer, 1–5, NOT NULL)
- `comment` (Text, NOT NULL)
- `place_id` (String FK → `places.id`, ON DELETE CASCADE)

### 4.2. Файлова структура (доповнення)
```
/src
  /models
      review_model.py
  /schemas
      review_schema.py
  /services
      review_service.py
  /routers
      review_router.py
```

### 4.3. Ключові фрагменти коду

**`src/models/review_model.py`**
```python
from sqlalchemy import String, Text, Integer, ForeignKey
from sqlalchemy.orm import Mapped, mapped_column, relationship
from src.db.database import Base
import uuid

class Review(Base):
    __tablename__ = "reviews"

    id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    rating: Mapped[int] = mapped_column(Integer, nullable=False)
    comment: Mapped[str] = mapped_column(Text, nullable=False)
    place_id: Mapped[str] = mapped_column(
        String, ForeignKey("places.id", ondelete="CASCADE"), nullable=False
    )
    place: Mapped["Place"] = relationship(back_populates="reviews")
```

Додати до `Place` моделі:
```python
reviews: Mapped[list["Review"]] = relationship(
    back_populates="place", cascade="all, delete-orphan"
)
```

**`src/schemas/review_schema.py`**
```python
from pydantic import BaseModel, Field

class ReviewCreate(BaseModel):
    rating: int = Field(ge=1, le=5)
    comment: str

class ReviewResponse(ReviewCreate):
    id: str
    place_id: str
```

**`src/routers/review_router.py`**
```python
from fastapi import APIRouter, HTTPException, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from src.db.database import get_db
from src.models.review_model import Review
from src.models.place_model import Place
from src.schemas.review_schema import ReviewCreate, ReviewResponse

router = APIRouter(prefix="/api/places/{place_id}/reviews", tags=["reviews"])

@router.get("/", response_model=list[ReviewResponse])
async def get_reviews(place_id: str, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Review).where(Review.place_id == place_id))
    return result.scalars().all()

@router.post("/", response_model=ReviewResponse, status_code=201)
async def create_review(place_id: str, data: ReviewCreate, db: AsyncSession = Depends(get_db)):
    place = await db.get(Place, place_id)
    if not place:
        raise HTTPException(status_code=404, detail="Place not found")
    review = Review(place_id=place_id, **data.model_dump())
    db.add(review)
    await db.commit()
    await db.refresh(review)
    return review
```

Підключити роутер у `main.py`:
```python
from src.routers.review_router import router as review_router
app.include_router(review_router)
```

### 4.4. Міграція

```bash
alembic revision --autogenerate -m "create reviews table"
alembic upgrade head
```

## 5. Очікуваний результат
1. Таблиця `reviews` створена з FK на `places` (CASCADE).
2. `POST /api/places/{place_id}/reviews` додає відгук.
3. `GET /api/places/{place_id}/reviews` повертає лише відгуки цього місця.
4. `rating` поза діапазоном 1–5 → `422 Unprocessable Entity`.
5. Неіснуючий `place_id` → `404 Not Found`.

#### **.NET (ASP.NET Core)**

## 3. Завдання

1. Створити модель `Review` та додати `DbSet<Review>` до `AppDbContext`.
2. Налаштувати зв'язок 1:N між `Place` та `Review` через `ForeignKey` та навігаційні властивості.
3. Додати Data Annotations для валідації (`Range`, `Required`).
4. Реалізувати `ReviewsController` з вкладеними маршрутами.
5. Виконати міграцію та протестувати через Postman або Swagger.

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Review`
- `Id` (Guid, Primary Key)
- `Rating` (int, 1–5, NOT NULL)
- `Comment` (string, NOT NULL)
- `PlaceId` (Guid, FK → `Places`, ON DELETE CASCADE)

### 4.2. Файлова структура (доповнення)
```
/Models
    Review.cs
/Controllers
    ReviewsController.cs
```

### 4.3. Ключові фрагменти коду

**`Models/Review.cs`**
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace PlacesApi.Models;

public class Review
{
    public Guid Id { get; set; } = Guid.NewGuid();

    [Range(1, 5)]
    public int Rating { get; set; }

    [Required]
    public string Comment { get; set; } = string.Empty;

    public Guid PlaceId { get; set; }

    [ForeignKey(nameof(PlaceId))]
    public Place? Place { get; set; }
}
```

Додати до `Models/Place.cs`:
```csharp
public ICollection<Review> Reviews { get; set; } = [];
```

**`Data/AppDbContext.cs`** (оновити)
```csharp
public DbSet<Review> Reviews => Set<Review>();

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Review>()
        .HasOne(r => r.Place)
        .WithMany(p => p.Reviews)
        .HasForeignKey(r => r.PlaceId)
        .OnDelete(DeleteBehavior.Cascade);
}
```

**`Controllers/ReviewsController.cs`**
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using PlacesApi.Data;
using PlacesApi.Models;

namespace PlacesApi.Controllers;

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

    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(Guid id)
    {
        var review = await db.Reviews.FindAsync(id);
        if (review is null) return NotFound();
        db.Reviews.Remove(review);
        await db.SaveChangesAsync();
        return NoContent();
    }
}
```

### 4.4. Міграція

```bash
dotnet ef migrations add AddReviews
dotnet ef database update
```

## 5. Очікуваний результат
1. Таблиця `Reviews` створена з FK на `Places` (Cascade).
2. `POST /api/places/{placeId}/reviews` додає відгук.
3. `GET /api/places/{placeId}/reviews` повертає лише відгуки цього місця.
4. `Rating` поза 1–5 → `400 Bad Request`.
5. Неіснуючий `placeId` → `404 Not Found`.

<!-- tabs:end -->
