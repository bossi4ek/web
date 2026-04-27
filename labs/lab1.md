# Лабораторна робота №1: Основи backend + Places API (In-memory)

## 1. Мета роботи
Ознайомлення з базовими принципами розробки серверних додатків. Отримання практичних навичок створення RESTful API, маршрутизації (routing) та розуміння принципів багатошарової архітектури (Layered Architecture: Controllers, Services). Реалізація базового CRUD-функціоналу для збереження даних у пам'яті (In-memory).

## 2. Теоретичні відомості
- **REST API (Representational State Transfer)** — архітектурний стиль взаємодії компонентів розподіленого додатка в мережі. Використовує стандартні HTTP-методи (GET, POST, PUT, DELETE).
- **CRUD** — акронім, що відображає чотири базові функції роботи з даними (Create, Read, Update, Delete).
- **Багатошарова архітектура (Layered Architecture)** — підхід до організації коду, де логіка розділена на шари (Routes/Routers, Controllers, Services). Це дозволяє зробити код масштабованим, легким для тестування та читання.
    - *Routes/Routers:* Визначають шляхи URL і HTTP методи.
    - *Controllers:* Обробляють вхідні HTTP-запити, отримують параметри та повертають HTTP-відповіді.
    - *Services:* Містять бізнес-логіку додатку (на цьому етапі — маніпуляції з масивом/списком даних).

> Оберіть один технологічний стек нижче та дотримуйтесь його протягом усього курсу.

<!-- tabs:start -->

#### **Node.js (Express)**

## 3. Завдання

1. Встановити середовище Node.js та ініціалізувати проєкт (`npm init`).
2. Встановити необхідні залежності: `express`, а також `nodemon` для зручності розробки (як dev-залежність).
3. Налаштувати скрипти у `package.json` для запуску проєкту (наприклад, `npm run dev`).
4. Створити правильну структуру тек проєкту (роути, контролери, сервіси).
5. Налаштувати базовий сервер Express у файлі `server.js` або `index.js` (підключити парсер JSON: `express.json()`).
6. Реалізувати in-memory сховище (масив) для сутності `Place`.
7. Створити Controller і Service, які реалізують CRUD-операції для сутності `Place`.
8. Налаштувати маршрутизацію (Routes) та підключити їх до основного сервера.
9. Протестувати всі створені endpoint-и за допомогою Postman.

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Place` (об'єкт у масиві)
- `id` (string) — унікальний ідентифікатор (можна генерувати за допомогою пакета `uuid`).
- `name` (string) — назва місця (наприклад, "Центральний парк").
- `description` (string) — опис місця.
- `city` (string) — місто.
- `address` (string) — адреса.

### 4.2. Файлова структура проєкту
```
/src
  /controllers
      place.controller.js
  /services
      place.service.js
  /routes
      place.routes.js
  server.js (або index.js)
package.json
```

### 4.3. API Endpoints
- `GET /api/places` — отримати список усіх місць.
- `GET /api/places/:id` — отримати місце за його `id`.
- `POST /api/places` — створити нове місце. (Тіло запиту: `name`, `description`, `city`, `address`).
- `PUT /api/places/:id` — повністю оновити місце за `id`.
- `DELETE /api/places/:id` — видалити місце за `id`.

*Приклад відповіді на успішний GET запит:*
```json
{
  "success": true,
  "data": [
    {
      "id": "1",
      "name": "Центральний парк",
      "description": "Великий парк у центрі міста",
      "city": "Київ",
      "address": "вул. Хрещатик"
    }
  ]
}
```

## 5. Очікуваний результат
1. Сервер запускається без помилок на обраному порту (наприклад, 3000).
2. Код не знаходиться в одному файлі. Роутинг, контролери та сервіси рознесені по відповідних папках.
3. Всі 5 API endpoint-ів коректно працюють через Postman.
4. Після перезавантаження сервера дані зникають (in-memory).
5. Робота оформлена у вигляді репозиторію на GitHub / GitLab з першим комітом.

#### **Python (FastAPI)**

## 3. Завдання

1. Встановити Python 3.11+ та створити віртуальне середовище: `python -m venv venv && source venv/bin/activate` (або `venv\Scripts\activate` на Windows).
2. Встановити залежності: `pip install fastapi uvicorn`.
3. Зберегти залежності: `pip freeze > requirements.txt`.
4. Створити правильну структуру тек проєкту.
5. Налаштувати базовий застосунок FastAPI у файлі `main.py`.
6. Реалізувати in-memory сховище (список Python) для сутності `Place`.
7. Створити Pydantic-схему, Router і Service з CRUD-операціями.
8. Протестувати всі endpoint-и через Postman або вбудований Swagger UI (`http://localhost:8000/docs`).

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Place`
- `id` (str) — унікальний ідентифікатор (генерувати через `uuid.uuid4()`).
- `name` (str) — назва місця.
- `description` (str, необов'язкове) — опис місця.
- `city` (str) — місто.
- `address` (str) — адреса.

### 4.2. Файлова структура проєкту
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

### 4.3. Ключові фрагменти коду

**`src/schemas/place_schema.py`**
```python
from pydantic import BaseModel
from typing import Optional

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
from src.schemas.place_schema import PlaceCreate

_places: list[dict] = []

def get_all() -> list[dict]:
    return _places

def get_by_id(place_id: str) -> dict | None:
    return next((p for p in _places if p["id"] == place_id), None)

def create(data: PlaceCreate) -> dict:
    place = {"id": str(uuid.uuid4()), **data.model_dump()}
    _places.append(place)
    return place

def update(place_id: str, data: PlaceCreate) -> dict | None:
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

### 4.4. API Endpoints
- `GET /api/places` — отримати список усіх місць.
- `GET /api/places/{place_id}` — отримати місце за `id`.
- `POST /api/places` — створити нове місце.
- `PUT /api/places/{place_id}` — оновити місце.
- `DELETE /api/places/{place_id}` — видалити місце.

## 5. Очікуваний результат
1. Сервер запускається без помилок на порту 8000.
2. Swagger UI доступний за адресою `http://localhost:8000/docs` — всі 5 endpoints відображені.
3. CRUD-операції працюють коректно через Postman або Swagger.
4. Після перезапуску сервера дані зникають (in-memory).
5. Робота оформлена у вигляді репозиторію на GitHub / GitLab з першим комітом.

#### **.NET (ASP.NET Core)**

## 3. Завдання

1. Встановити .NET SDK 8.0+. Перевірити: `dotnet --version`.
2. Створити новий проєкт: `dotnet new webapi -n PlacesApi --use-controllers`.
3. Видалити файли-заглушки (`WeatherForecast.cs`, `WeatherForecastController.cs`).
4. Створити правильну структуру тек проєкту.
5. Реалізувати in-memory сховище (статичний `List<Place>`) для сутності `Place`.
6. Створити Model, Service та Controller з CRUD-операціями.
7. Зареєструвати `PlacesService` у `Program.cs` через DI.
8. Протестувати всі endpoint-и через Postman або вбудований Swagger UI (`/swagger`).

## 4. Вимоги до реалізації

### 4.1. Доменна модель `Place`
- `Id` (string) — унікальний ідентифікатор (`Guid.NewGuid().ToString()`).
- `Name` (string) — назва місця.
- `Description` (string?) — опис місця (необов'язкове).
- `City` (string) — місто.
- `Address` (string) — адреса.

### 4.2. Файлова структура проєкту
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

### 4.3. Ключові фрагменти коду

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

**`Program.cs`**
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

### 4.4. API Endpoints
- `GET /api/places` — отримати список усіх місць.
- `GET /api/places/{id}` — отримати місце за `id`.
- `POST /api/places` — створити нове місце.
- `PUT /api/places/{id}` — оновити місце.
- `DELETE /api/places/{id}` — видалити місце.

## 5. Очікуваний результат
1. Сервер запускається на порту 5000/5001.
2. Swagger UI доступний за адресою `http://localhost:5000/swagger`.
3. Всі 5 CRUD endpoints працюють через Postman або Swagger.
4. Після перезапуску сервера дані зникають (in-memory).
5. Робота оформлена у вигляді репозиторію на GitHub / GitLab з першим комітом.

<!-- tabs:end -->
