# Лабораторна робота №5: Просунуті запити (Фільтрація, Пагінація, Ролі)

## 1. Мета роботи
Опанувати обробку параметрів запиту (Query Parameters) для реалізації фільтрації та пагінації даних у REST API. Навчитися реалізовувати рольову модель доступу (RBAC — Role-Based Access Control) для розмежування прав звичайних користувачів та адміністраторів.

## 2. Теоретичні відомості
- **Query Parameters** — частина URL після `?`, що передає налаштування запиту: `/api/places?city=Київ&page=1`.
- **Пагінація** — розділення даних на сторінки. Параметри: `page` (номер) та `limit` (кількість на сторінку).
- **Фільтрація** — відбір записів за критеріями (наприклад, місто). Оператор `ILIKE` в PostgreSQL дозволяє частковий нечутливий до регістру пошук.
- **RBAC (Role-Based Access Control)** — права прив'язані до ролей (`user`, `admin`). Адмін може виконувати дії, недоступні звичайному користувачу.

> Продовжуйте у тому самому стеку, який ви обрали у ЛР №1.

<!-- tabs:start -->

#### **Node.js (Express)**

## 3. Завдання

1. Модифікувати `GET /api/places` для підтримки query-параметрів `page`, `limit`, `city`.
2. Реалізувати пагінацію через `LIMIT` / `OFFSET` в ORM-запиті.
3. Додати фільтрацію за `city` (частковий збіг, `ILIKE`).
4. Написати `roleMiddleware(allowedRoles)` — перевіряє `req.user.role`.
5. Дозволити адміністраторам видаляти/редагувати будь-яке місце чи відгук.
6. Протестувати через Postman.

## 4. Вимоги до реалізації

### 4.1. Файлова структура (доповнення)
```
/src
  /middlewares
      role.middleware.js
```

### 4.2. Ключові фрагменти коду

**`src/middlewares/role.middleware.js`**
```js
module.exports = (allowedRoles) => (req, res, next) => {
  if (!allowedRoles.includes(req.user?.role)) {
    return res.status(403).json({ success: false, error: 'Forbidden' });
  }
  next();
};
```

**`src/services/place.service.js`** (оновити `getAll`)
```js
async function getAll({ page = 1, limit = 10, city } = {}) {
  const where = city ? { city: { [Op.iLike]: `%${city}%` } } : {};
  const offset = (page - 1) * limit;
  const { count, rows } = await Place.findAndCountAll({
    where,
    limit: Number(limit),
    offset,
  });
  return {
    data: rows,
    meta: {
      total: count,
      page: Number(page),
      limit: Number(limit),
      totalPages: Math.ceil(count / limit),
    },
  };
}
```

**Логіка власності + адмін у контролері**
```js
async function deletPlace(req, res, next) {
  try {
    const place = await placeService.getById(req.params.id);
    if (!place) return res.status(404).json({ success: false, error: 'Not found' });

    const isOwner = place.createdBy === req.user.id;
    const isAdmin = req.user.role === 'admin';
    if (!isOwner && !isAdmin) {
      return res.status(403).json({ success: false, error: 'Forbidden' });
    }
    await placeService.delete(req.params.id);
    res.status(204).send();
  } catch (err) {
    next(err);
  }
}
```

### 4.3. API Endpoint
- `GET /api/places?page=1&limit=5&city=Київ`

*Приклад відповіді:*
```json
{
  "success": true,
  "data": [ { "id": "...", "name": "..." } ],
  "meta": { "total": 24, "page": 1, "limit": 5, "totalPages": 5 }
}
```

## 5. Очікуваний результат
1. `GET /api/places` без параметрів → перша сторінка (10 записів).
2. `?city=Львів` → тільки місця з Львова.
3. `user` без власності → `403 Forbidden` при DELETE.
4. `admin` → може видалити будь-яке місце.

#### **Python (FastAPI)**

## 3. Завдання

1. Додати query-параметри `page`, `limit`, `city` до `GET /api/places`.
2. Реалізувати пагінацію через `.offset()` та `.limit()` у SQLAlchemy.
3. Додати фільтрацію за `city` через `ilike`.
4. Написати dependency `require_role(role)` для перевірки ролі.
5. Дозволити адміністраторам видаляти/редагувати будь-яке місце.
6. Протестувати через Postman або Swagger.

## 4. Вимоги до реалізації

### 4.1. Ключові фрагменти коду

**`src/routers/place_router.py`** (оновлений GET)
```python
from sqlalchemy import select, func

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
        "meta": {
            "total": total,
            "page": page,
            "limit": limit,
            "totalPages": -(-total // limit),  # ceiling division
        }
    }
```

**`src/dependencies/auth.py`** (додати `require_role`)
```python
def require_role(role: str):
    def dependency(current_user: User = Depends(get_current_user)) -> User:
        if current_user.role != role:
            raise HTTPException(status_code=403, detail="Forbidden")
        return current_user
    return dependency
```

**Логіка власності + адмін у роутері**
```python
@router.delete("/{place_id}", status_code=204)
async def delete_place(
    place_id: str,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    place = await place_service.get_by_id(db, place_id)
    if not place:
        raise HTTPException(status_code=404, detail="Not found")

    is_owner = place.owner_id == current_user.id
    is_admin = current_user.role == "admin"
    if not is_owner and not is_admin:
        raise HTTPException(status_code=403, detail="Forbidden")

    await place_service.delete(db, place_id)
```

### 4.2. API Endpoint
- `GET /api/places?page=1&limit=5&city=Київ`

*Відповідь аналогічна Node.js варіанту.*

## 5. Очікуваний результат
1. `GET /api/places` без параметрів → перша сторінка (10 записів).
2. `?city=Львів` → тільки місця з Львова.
3. `user` без власності → `403 Forbidden` при DELETE.
4. `admin` → може видалити будь-яке місце.

#### **.NET (ASP.NET Core)**

## 3. Завдання

1. Додати query-параметри `page`, `limit`, `city` до `GET /api/places`.
2. Реалізувати пагінацію через `.Skip()` / `.Take()`.
3. Додати фільтрацію за `city` через `.Contains()`.
4. Налаштувати Policy-based authorization для ролі `admin`.
5. Дозволити адміністраторам видаляти будь-яке місце.
6. Протестувати через Postman або Swagger.

## 4. Вимоги до реалізації

### 4.1. Ключові фрагменти коду

**`Controllers/PlacesController.cs`** (оновлений GetAll)
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
    var data = await query
        .Skip((page - 1) * limit)
        .Take(limit)
        .ToListAsync();

    return Ok(new
    {
        data,
        meta = new { total, page, limit, totalPages = (int)Math.Ceiling((double)total / limit) }
    });
}
```

**`Program.cs`** (додати Policy)
```csharp
builder.Services.AddAuthorization(opts =>
{
    opts.AddPolicy("AdminOnly", p => p.RequireRole("admin"));
});
```

**Логіка власності + адмін**
```csharp
[Authorize]
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(Guid id)
{
    var place = await service.GetByIdAsync(id);
    if (place is null) return NotFound();

    var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);
    var isAdmin = User.IsInRole("admin");

    if (place.OwnerId != userId && !isAdmin)
        return Forbid();

    await service.DeleteAsync(id);
    return NoContent();
}
```

> **Примітка:** щоб `User.IsInRole` працювало, роль має бути у claims токена як `ClaimTypes.Role`. Переконайтеся, що у `AuthService.CreateToken` ви додаєте `new Claim(ClaimTypes.Role, role)`.

### 4.2. API Endpoint
- `GET /api/places?page=1&limit=5&city=Київ`

## 5. Очікуваний результат
1. `GET /api/places` без параметрів → перша сторінка (10 записів).
2. `?city=Львів` → тільки місця з Львова.
3. `user` без власності → `403 Forbidden` при DELETE.
4. `admin` → може видалити будь-яке місце.

<!-- tabs:end -->
