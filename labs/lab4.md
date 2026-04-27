# Лабораторна робота №4: Аутентифікація та JWT

## 1. Мета роботи
Опанувати механізми аутентифікації та авторизації користувачів у веб-додатках. Навчитися хешувати паролі, генерувати та валідувати JSON Web Tokens (JWT), створювати захищені маршрути та реалізовувати логіку перевірки прав власності на ресурси.

## 2. Теоретичні відомості
- **Аутентифікація** — підтвердження того, ким є користувач (логін + пароль).
- **Авторизація** — перевірка прав доступу до певного ресурсу або дії.
- **JWT (JSON Web Token)** — відкритий стандарт (RFC 7519) для токенів доступу. Складається з: Header, Payload (userId, role), Signature.
- **Хешування паролів** — однобічне перетворення пароля за алгоритмом `bcrypt`. Додає унікальну "сіль" (salt) для захисту від rainbow table атак.
- **Auth Middleware** — перевіряє наявність та валідність токена в HTTP-заголовку `Authorization: Bearer <token>`.

> Продовжуйте у тому самому стеку, який ви обрали у ЛР №1.

<!-- tabs:start -->

#### **Node.js (Express)**

## 3. Завдання

1. Встановити залежності: `npm install bcryptjs jsonwebtoken`.
2. Створити модель `User` з полями `id`, `email`, `password`, `role`.
3. Оновити `Place` та `Review`: додати `createdBy` / `userId` (FK → User).
4. Реалізувати `POST /api/auth/register` (хешування пароля через `bcryptjs`).
5. Реалізувати `POST /api/auth/login` (порівняння через `bcrypt.compare`, генерація JWT).
6. Написати `authMiddleware`, який декодує токен та кладе `req.user` у запит.
7. Захистити маршрути `POST/PUT/DELETE /api/places` та `POST /api/places/:placeId/reviews`.
8. Додати перевірку власності: при PUT/DELETE перевіряти `place.createdBy === req.user.id`.

## 4. Вимоги до реалізації

### 4.1. Доменні моделі
- **User:** `id`, `email` (unique), `password` (hash), `role` (default: `'user'`)
- **Place:** додати `createdBy` (FK → User)
- **Review:** додати `userId` (FK → User)

### 4.2. Файлова структура (доповнення)
```
/src
  /controllers
      auth.controller.js
  /services
      auth.service.js
  /routes
      auth.routes.js
  /middlewares
      auth.middleware.js
  /models
      user.model.js
```

### 4.3. Ключові фрагменти коду

**`src/middlewares/auth.middleware.js`**
```js
const jwt = require('jsonwebtoken');

module.exports = (req, res, next) => {
  const header = req.headers.authorization;
  if (!header?.startsWith('Bearer ')) {
    return res.status(401).json({ success: false, error: 'No token' });
  }
  try {
    req.user = jwt.verify(header.split(' ')[1], process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ success: false, error: 'Invalid token' });
  }
};
```

**`src/services/auth.service.js`**
```js
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/user.model');

async function register(email, password) {
  const hash = await bcrypt.hash(password, 10);
  return User.create({ email, password: hash });
}

async function login(email, password) {
  const user = await User.findOne({ where: { email } });
  if (!user || !(await bcrypt.compare(password, user.password))) {
    throw Object.assign(new Error('Invalid credentials'), { status: 401 });
  }
  const token = jwt.sign(
    { id: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '24h' }
  );
  return { user: { id: user.id, email: user.email, role: user.role }, token };
}

module.exports = { register, login };
```

### 4.4. API Endpoints
- `POST /api/auth/register` — Body: `{ email, password }`
- `POST /api/auth/login` — Response: `{ success: true, data: { user, token } }`

Захищені (Header `Authorization: Bearer <token>`):
- `POST /api/places`, `PUT /api/places/:id`, `DELETE /api/places/:id`
- `POST /api/places/:placeId/reviews`

## 5. Очікуваний результат
1. `POST /api/auth/register` — пароль зберігається як хеш.
2. Неправильний логін → `401 Unauthorized`.
3. Правильний логін → JWT у відповіді.
4. `POST /api/places` без токена → `401 Unauthorized`.
5. При створенні місця — `createdBy` автоматично заповнюється з токена.
6. PUT/DELETE чужого місця → `403 Forbidden`.

#### **Python (FastAPI)**

## 3. Завдання

1. Встановити залежності: `pip install python-jose[cryptography] passlib[bcrypt]`.
2. Створити SQLAlchemy-модель `User` та Pydantic-схеми.
3. Оновити `Place` та `Review`: додати `owner_id` / `user_id` (FK → User).
4. Реалізувати `POST /api/auth/register` та `POST /api/auth/login`.
5. Реалізувати `get_current_user` dependency, яка декодує JWT.
6. Захистити маршрути через `Depends(get_current_user)`.
7. Перевіряти власність при PUT/DELETE.

## 4. Вимоги до реалізації

### 4.1. Доменні моделі
- **User:** `id`, `email` (unique), `password` (hash), `role` (default: `'user'`)
- **Place:** додати `owner_id` (FK → User)
- **Review:** додати `user_id` (FK → User)

### 4.2. Файлова структура (доповнення)
```
/src
  /models
      user_model.py
  /schemas
      user_schema.py
      auth_schema.py
  /services
      auth_service.py
  /routers
      auth_router.py
  /dependencies
      auth.py          # get_current_user
```

### 4.3. Ключові фрагменти коду

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

def create_token(user_id: str, role: str) -> str:
    payload = {
        "sub": user_id,
        "role": role,
        "exp": datetime.utcnow() + timedelta(hours=24)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except JWTError:
        raise ValueError("Invalid token")
```

**`src/dependencies/auth.py`**
```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession
from src.db.database import get_db
from src.models.user_model import User
from src.services.auth_service import decode_token

bearer = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(bearer),
    db: AsyncSession = Depends(get_db)
) -> User:
    try:
        payload = decode_token(credentials.credentials)
    except ValueError:
        raise HTTPException(status_code=401, detail="Invalid token")
    user = await db.get(User, payload["sub"])
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    return user
```

**Захист маршруту та перевірка власності**
```python
from src.dependencies.auth import get_current_user

@router.post("/", response_model=PlaceResponse, status_code=201)
async def create_place(
    data: PlaceCreate,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    return await place_service.create(db, data, owner_id=current_user.id)

@router.delete("/{place_id}", status_code=204)
async def delete_place(
    place_id: str,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    place = await place_service.get_by_id(db, place_id)
    if not place:
        raise HTTPException(status_code=404, detail="Place not found")
    if place.owner_id != current_user.id:
        raise HTTPException(status_code=403, detail="Forbidden")
    await place_service.delete(db, place_id)
```

### 4.4. API Endpoints
- `POST /api/auth/register` — Body: `{ email, password }`
- `POST /api/auth/login` — Response: `{ user: {...}, token: "eyJ..." }`

## 5. Очікуваний результат
1. `POST /api/auth/register` — пароль зберігається як bcrypt-хеш.
2. Неправильний логін → `401 Unauthorized`.
3. Правильний логін → JWT.
4. `POST /api/places` без токена → `403 Forbidden` (HTTPBearer вимагає токен).
5. При створенні місця — `owner_id` заповнюється автоматично.
6. PUT/DELETE чужого місця → `403 Forbidden`.

#### **.NET (ASP.NET Core)**

## 3. Завдання

1. Встановити пакети: `dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer` та `dotnet add package BCrypt.Net-Next`.
2. Створити модель `User` та додати `DbSet<User>` до `AppDbContext`.
3. Оновити `Place` та `Review`: додати `OwnerId` / `UserId` (FK → User).
4. Реалізувати `AuthController` з `Register` та `Login` endpoints.
5. Налаштувати JWT Bearer authentication у `Program.cs`.
6. Захистити endpoints атрибутом `[Authorize]`.
7. Перевіряти власність при PUT/DELETE через `User.FindFirstValue`.

## 4. Вимоги до реалізації

### 4.1. Доменні моделі
- **User:** `Id` (Guid), `Email` (unique), `Password` (hash), `Role` (default: `"user"`)
- **Place:** додати `OwnerId` (Guid, FK → User)
- **Review:** додати `UserId` (Guid, FK → User)

### 4.2. Файлова структура (доповнення)
```
/Models
    User.cs
/Controllers
    AuthController.cs
/Services
    AuthService.cs
```

### 4.3. Ключові фрагменти коду

**`Models/User.cs`**
```csharp
using System.ComponentModel.DataAnnotations;

namespace PlacesApi.Models;

public class User
{
    public Guid Id { get; set; } = Guid.NewGuid();

    [Required, EmailAddress]
    public string Email { get; set; } = string.Empty;

    [Required]
    public string Password { get; set; } = string.Empty;

    public string Role { get; set; } = "user";
}
```

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

    public string CreateToken(Guid userId, string role)
    {
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(config["Jwt:Secret"]!));
        var token = new JwtSecurityToken(
            claims: [
                new Claim(ClaimTypes.NameIdentifier, userId.ToString()),
                new Claim(ClaimTypes.Role, role)
            ],
            expires: DateTime.UtcNow.AddHours(24),
            signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256)
        );
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

**JWT налаштування у `Program.cs`**
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
builder.Services.AddScoped<AuthService>();

// після app.Build():
app.UseAuthentication();
app.UseAuthorization();
```

**Захист та перевірка власності у `PlacesController.cs`**
```csharp
[Authorize]
[HttpPost]
public async Task<IActionResult> Create([FromBody] Place place)
{
    var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);
    place.OwnerId = userId;
    // ...
}

[Authorize]
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(Guid id)
{
    var place = await service.GetByIdAsync(id);
    if (place is null) return NotFound();

    var userId = Guid.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!);
    if (place.OwnerId != userId) return Forbid();

    await service.DeleteAsync(id);
    return NoContent();
}
```

**`appsettings.json`** (додати)
```json
"Jwt": {
  "Secret": "your-super-secret-key-at-least-32-chars"
}
```

### 4.4. API Endpoints
- `POST /api/auth/register` — Body: `{ email, password }`
- `POST /api/auth/login` — Response: `{ user: {...}, token: "eyJ..." }`

## 5. Очікуваний результат
1. `POST /api/auth/register` — пароль зберігається як bcrypt-хеш.
2. Неправильний логін → `401 Unauthorized`.
3. Правильний логін → JWT у відповіді.
4. `POST /api/places` без токена → `401 Unauthorized`.
5. При створенні місця — `OwnerId` заповнюється з токена.
6. PUT/DELETE чужого місця → `403 Forbidden`.

<!-- tabs:end -->
