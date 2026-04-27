# Лабораторна робота №6: Production Readiness (Безпека, Логування, Деплой)

## 1. Мета роботи
Опанувати інструменти для підготовки backend-додатку до production-середовища. Навчитись налаштовувати CORS, обмежувати кількість запитів (Rate Limiting), додавати захист HTTP-заголовків, впроваджувати логування та підготувати проєкт до розгортання в Docker/хмарі.

## 2. Теоретичні відомості
- **CORS (Cross-Origin Resource Sharing)** — HTTP-заголовки, що дозволяють або забороняють браузеру звертатись до API з іншого домену.
- **HTTP Security Headers** — заголовки (`X-Content-Type-Options`, `X-Frame-Options`, CSP тощо), що захищають від XSS та інших атак.
- **Rate Limiting** — обмеження кількості запитів з однієї IP-адреси за певний час. Захист від brute-force та DDoS.
- **Логування** — структурований запис подій сервера у консоль та файли. HTTP-запити логуються окремо від системних подій.
- **Docker** — контейнеризація додатку для однотипного розгортання у будь-якому середовищі.

> Продовжуйте у тому самому стеку, який ви обрали у ЛР №1.

<!-- tabs:start -->

#### **Node.js (Express)**

## 3. Завдання

1. Встановити пакети: `npm install cors helmet express-rate-limit morgan winston`.
2. Підключити `helmet` як перший глобальний middleware.
3. Налаштувати `cors` (дозволені origins — у `.env`).
4. Налаштувати `express-rate-limit` для `/api/auth` (макс. 20 запитів за 15 хв).
5. Підключити `morgan` для логування HTTP-запитів.
6. Налаштувати `winston`: помилки → `logs/error.log`, всі → `logs/combined.log`.
7. Інтегрувати `winston` у Global Error Handler замість `console.error`.
8. Написати `Dockerfile` та перевірити запуск через `docker build && docker run`.

## 4. Вимоги до реалізації

### 4.1. Файлова структура (доповнення)
```
/src
  /config
      logger.js
  /middlewares
      rate-limiter.middleware.js
/logs
  .gitkeep
Dockerfile
.dockerignore
```

### 4.2. Ключові фрагменти коду

**`src/config/logger.js`**
```js
const { createLogger, format, transports } = require('winston');

module.exports = createLogger({
  format: format.combine(format.timestamp(), format.json()),
  transports: [
    new transports.Console({ format: format.combine(format.colorize(), format.simple()) }),
    new transports.File({ filename: 'logs/error.log', level: 'error' }),
    new transports.File({ filename: 'logs/combined.log' }),
  ],
});
```

**`src/middlewares/rate-limiter.middleware.js`**
```js
const rateLimit = require('express-rate-limit');

module.exports = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 20,
  message: {
    success: false,
    error: 'Too many requests from this IP, please try again after 15 minutes',
    status: 429,
  },
});
```

**`server.js`** (інтеграція)
```js
const helmet = require('helmet');
const cors = require('cors');
const morgan = require('morgan');
const authLimiter = require('./src/middlewares/rate-limiter.middleware');

app.use(helmet());
app.use(cors({ origin: process.env.ALLOWED_ORIGINS?.split(',') || '*' }));
app.use(morgan(process.env.NODE_ENV === 'production' ? 'combined' : 'dev'));
app.use('/api/auth', authLimiter);
```

**`Dockerfile`**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "src/server.js"]
```

### 4.3. Очікуваний формат помилки rate limiting
```json
{
  "success": false,
  "error": "Too many requests from this IP, please try again after 15 minutes",
  "status": 429
}
```

## 5. Очікуваний результат
1. Відповіді містять security-заголовки від Helmet (видно у Postman → Headers).
2. Після 20 запитів до `/api/auth` за 15 хв → `429 Too Many Requests`.
3. HTTP-запити логуються у консоль через morgan.
4. Помилки записуються у `logs/error.log`.
5. `docker build -t places-api . && docker run -p 3000:3000 places-api` запускає додаток.

#### **Python (FastAPI)**

## 3. Завдання

1. Встановити пакети: `pip install slowapi`.
2. Підключити CORS middleware.
3. Налаштувати Rate Limiting через `slowapi`.
4. Налаштувати структуроване логування через вбудований `logging`.
5. Написати `Dockerfile` та перевірити запуск.

## 4. Вимоги до реалізації

### 4.1. Файлова структура (доповнення)
```
/logs
  .gitkeep
Dockerfile
.dockerignore
requirements.txt  # slowapi додано
```

### 4.2. Ключові фрагменти коду

**`main.py`** (оновлено повністю)
```python
import logging
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
import os

# Логування
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
    handlers=[
        logging.StreamHandler(),
        logging.FileHandler("logs/combined.log"),
    ]
)
error_handler = logging.FileHandler("logs/error.log")
error_handler.setLevel(logging.ERROR)
logging.getLogger().addHandler(error_handler)

logger = logging.getLogger(__name__)

# Rate Limiter
limiter = Limiter(key_func=get_remote_address)

app = FastAPI(title="Places API")
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=os.getenv("ALLOWED_ORIGINS", "*").split(","),
    allow_methods=["*"],
    allow_headers=["*"],
)

# HTTP request logging middleware
@app.middleware("http")
async def log_requests(request: Request, call_next):
    logger.info(f"{request.method} {request.url.path}")
    response = await call_next(request)
    if response.status_code >= 400:
        logger.error(f"{request.method} {request.url.path} → {response.status_code}")
    return response
```

**Застосування rate limit на роутері**
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@auth_router.post("/login")
@limiter.limit("20/15minutes")
async def login(request: Request, data: LoginSchema, db: AsyncSession = Depends(get_db)):
    ...
```

**Security headers middleware**
```python
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["Strict-Transport-Security"] = "max-age=31536000"
    return response
```

**`Dockerfile`**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN mkdir -p logs
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 5. Очікуваний результат
1. Відповіді містять security-заголовки (видно у Postman → Headers).
2. Після 20 запитів до `/api/auth/login` за 15 хв → `429 Too Many Requests`.
3. HTTP-запити логуються у консоль та `logs/combined.log`.
4. Помилки записуються у `logs/error.log`.
5. `docker build -t places-api . && docker run -p 8000:8000 places-api` запускає додаток.

#### **.NET (ASP.NET Core)**

## 3. Завдання

1. Налаштувати CORS у `Program.cs`.
2. Налаштувати вбудований Rate Limiting (доступний з .NET 7+).
3. Додати Security Headers через custom middleware.
4. Налаштувати структуроване логування через вбудований `ILogger` + файловий sink (Serilog).
5. Встановити Serilog: `dotnet add package Serilog.AspNetCore` та `dotnet add package Serilog.Sinks.File`.
6. Написати `Dockerfile` та перевірити запуск.

## 4. Вимоги до реалізації

### 4.1. Файлова структура (доповнення)
```
/logs
  .gitkeep
Dockerfile
.dockerignore
```

### 4.2. Ключові фрагменти коду

**`Program.cs`** (оновлено)
```csharp
using Serilog;
using Microsoft.AspNetCore.RateLimiting;
using System.Threading.RateLimiting;

// Serilog
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.File("logs/combined.log", rollingInterval: RollingInterval.Day)
    .WriteTo.File("logs/error.log", restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Error)
    .CreateLogger();

builder.Host.UseSerilog();

// CORS
builder.Services.AddCors(opts =>
    opts.AddPolicy("default", p =>
        p.WithOrigins(builder.Configuration["AllowedOrigins"]?.Split(',') ?? ["*"])
         .AllowAnyMethod()
         .AllowAnyHeader()));

// Rate Limiting
builder.Services.AddRateLimiter(opts =>
{
    opts.AddFixedWindowLimiter("auth", o =>
    {
        o.PermitLimit = 20;
        o.Window = TimeSpan.FromMinutes(15);
        o.QueueLimit = 0;
    });
    opts.RejectionStatusCode = 429;
});

var app = builder.Build();

app.UseSerilogRequestLogging();
app.UseCors("default");
app.UseRateLimiter();
app.UseMiddleware<ExceptionMiddleware>();
app.UseMiddleware<SecurityHeadersMiddleware>();
```

**`Middleware/SecurityHeadersMiddleware.cs`**
```csharp
namespace PlacesApi.Middleware;

public class SecurityHeadersMiddleware(RequestDelegate next)
{
    public async Task InvokeAsync(HttpContext context)
    {
        context.Response.Headers.Append("X-Content-Type-Options", "nosniff");
        context.Response.Headers.Append("X-Frame-Options", "DENY");
        context.Response.Headers.Append("Strict-Transport-Security", "max-age=31536000");
        await next(context);
    }
}
```

**Застосування rate limit на Auth контролері**
```csharp
[EnableRateLimiting("auth")]
[HttpPost("login")]
public async Task<IActionResult> Login([FromBody] LoginDto dto) { ... }
```

**`Dockerfile`**
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
RUN mkdir -p logs

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "PlacesApi.dll"]
```

## 5. Очікуваний результат
1. Відповіді містять security-заголовки (видно у Postman → Headers).
2. Після 20 запитів до `/api/auth/login` за 15 хв → `429 Too Many Requests`.
3. HTTP-запити логуються через Serilog у консоль та `logs/combined.log`.
4. Помилки записуються у `logs/error.log`.
5. `docker build -t places-api . && docker run -p 5000:5000 places-api` запускає додаток.

<!-- tabs:end -->
