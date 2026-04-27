# Контрольні питання до лабораторних робіт

> Питання поділені на **загальні** (для всіх стеків) та **технологічні** (оберіть свій стек у вкладці).

---

## Лабораторна робота №1

**Загальні питання:**
1. Що таке REST API і назвіть основні HTTP-методи. Яка семантика кожного з них?
2. Яка різниця між методами `PUT` та `PATCH`?
3. Чим відрізняється Controller від Service у багатошаровій архітектурі? Навіщо розділяти код?
4. Що таке in-memory сховище і які його переваги та недоліки порівняно з БД?

<!-- tabs:start -->

#### **Node.js (Express)**

5. Що таке Node.js? Чим він відрізняється від виконання JavaScript у браузері?
6. Для чого потрібен `express.json()` і що станеться, якщо його не підключити?
7. Що зберігається в `req.body`, `req.params` та `req.query`? Наведіть приклади.
8. Що таке `package.json`? Яка різниця між `dependencies` та `devDependencies`?

#### **Python (FastAPI)**

5. Що таке FastAPI? Чим він відрізняється від Flask?
6. Що таке Pydantic і яку роль відіграє у FastAPI при валідації запитів?
7. Як FastAPI автоматично генерує Swagger-документацію (`/docs`)? Що таке OpenAPI?
8. Що таке `uvicorn` і навіщо він потрібен для запуску FastAPI?

#### **.NET (ASP.NET Core)**

5. Що таке ASP.NET Core? Яку роль відіграє `Program.cs`?
6. Для чого призначені атрибути `[ApiController]` та `[Route]`?
7. Що таке Dependency Injection (DI)? Як він реалізований через `builder.Services`?
8. Що таке `IActionResult`? Назвіть три конкретні результати (`Ok`, `NotFound`, `CreatedAtAction`) та їх HTTP-коди.

<!-- tabs:end -->

---

## Лабораторна робота №2

**Загальні питання:**
1. Що таке ORM? Назвіть переваги та недоліки порівняно з "чистими" SQL-запитами.
2. Що таке міграції бази даних і чому їх важливо використовувати замість ручного створення таблиць?
3. Чому потрібно валідувати дані на бекенді, навіть якщо є валідація на фронтенді?
4. Що таке SQL-ін'єкція і як використання ORM захищає від неї?

<!-- tabs:start -->

#### **Node.js (Express)**

5. Як працює `async/await` у JavaScript? Що повертає `async`-функція?
6. Що таке `next(error)` у Express і як працює Global Error Handler (middleware з 4 параметрами)?
7. Яка різниця між Sequelize та Prisma як ORM для Node.js?

#### **Python (FastAPI)**

5. Як SQLAlchemy async-сесії (`AsyncSession`) інтегруються з FastAPI через `Depends`?
6. Яка різниця між `422 Unprocessable Entity` (Pydantic) та `400 Bad Request`? Коли FastAPI повертає кожен з них?
7. Що таке Alembic і як команда `alembic revision --autogenerate` формує міграцію?

#### **.NET (ASP.NET Core)**

5. Що таке `DbContext` у Entity Framework Core? Яку роль відіграє `DbSet<T>`?
6. Як Data Annotations (`[Required]`, `[MaxLength]`) впливають на валідацію запиту і на схему БД?
7. Яка різниця між `AddSingleton`, `AddScoped` та `AddTransient` при реєстрації сервісів у DI?

<!-- tabs:end -->

---

## Лабораторна робота №3

**Загальні питання:**
1. Які типи зв'язків між таблицями існують у реляційних БД? Наведіть приклади для кожного.
2. Що таке Foreign Key і яку роль він відіграє при побудові зв'язків?
3. Чому для відгуку краще використовувати маршрут `/api/places/:placeId/reviews` замість `/api/reviews`?
4. Що таке каскадне видалення (`ON DELETE CASCADE`) і як воно працює на рівні БД?

<!-- tabs:start -->

#### **Node.js (Express)**

5. Для чого потрібна опція `mergeParams: true` при ініціалізації `express.Router()`? Що станеться без неї?
6. Як в ORM (Sequelize/Prisma) налаштовується зв'язок `hasMany` / `belongsTo`? Покажіть приклад.

#### **Python (FastAPI)**

5. Як `relationship` та `ForeignKey` у SQLAlchemy встановлюють зв'язок між моделями? Що означає `back_populates`?
6. Як параметр `place_id` з URL маршруту `/api/places/{place_id}/reviews` потрапляє у функцію-обробник FastAPI?

#### **.NET (ASP.NET Core)**

5. Як `[ForeignKey]` і навігаційні властивості (`Place? Place`) описують зв'язок у EF Core?
6. Як у `OnModelCreating` налаштовується поведінка `OnDelete(DeleteBehavior.Cascade)`?

<!-- tabs:end -->

---

## Лабораторна робота №4

**Загальні питання:**
1. Що таке аутентифікація і чим вона відрізняється від авторизації?
2. З яких трьох частин складається JWT? Чи зашифровані дані у Payload?
3. Чому паролі не можна зберігати у відкритому вигляді? Що таке "сіль" (salt) при хешуванні?
4. Як перевірити в обробнику, чи має право поточний користувач змінити конкретний ресурс?
5. Як встановити термін дії (expiration) JWT і що відбувається на сервері, коли токен протермінувався?

<!-- tabs:start -->

#### **Node.js (Express)**

6. Як Auth Middleware отримує токен із заголовка `Authorization: Bearer <token>` і що робить з ним далі?
7. Яка різниця між `bcrypt.hash()` та `bcrypt.compare()`? Для чого кожна з них використовується?

#### **Python (FastAPI)**

6. Як `HTTPBearer` dependency у FastAPI витягує токен із запиту і передає його для перевірки?
7. Яка різниця між бібліотеками `passlib` і `python-jose` — за що відповідає кожна з них?

#### **.NET (ASP.NET Core)**

6. Як `[Authorize]` атрибут взаємодіє з JWT Bearer middleware? Що відбувається, якщо токен відсутній або недійсний?
7. Як `User.FindFirstValue(ClaimTypes.NameIdentifier)` отримує ID поточного користувача з токена?

<!-- tabs:end -->

---

## Лабораторна робота №5

**Загальні питання:**
1. У чому різниця між `path parameters` та `query parameters`? Коли доцільніше використовувати кожен варіант?
2. Поясніть пагінацію типу `Offset/Limit`. Які проблеми виникають на великих наборах даних?
3. Яка різниця між `Offset/Limit` та `Cursor-based` пагінацією?
4. Що таке RBAC і які альтернативи йому існують (наприклад, ABAC)?

<!-- tabs:start -->

#### **Node.js (Express)**

5. Як реалізується `roleMiddleware(allowedRoles)` і як ланцюжок `authMiddleware → roleMiddleware` передає управління?
6. Як у Sequelize/Prisma реалізувати частковий пошук (`ILIKE`) та пагінацію в одному запиті?

#### **Python (FastAPI)**

5. Як FastAPI автоматично документує query-параметри (`page`, `limit`, `city`) у Swagger UI?
6. Як `func.count()` з SQLAlchemy використовується для підрахунку загальної кількості записів при пагінації?

#### **.NET (ASP.NET Core)**

5. Як `[FromQuery]` атрибут прив'язує query-параметри до параметрів методу контролера?
6. Як Policy-based authorization (`RequireRole`) відрізняється від простого `[Authorize]`?

<!-- tabs:end -->

---

## Лабораторна робота №6

**Загальні питання:**
1. Чим відрізняється production-середовище від development? Які оптимізації та обмеження застосовуються в production?
2. Як працює CORS і чому браузер (але не Postman) блокує запити між різними origins?
3. Що таке Rate Limiting і чому він критично важливий для публічних API?
4. Що таке Docker і яку проблему він вирішує при розгортанні додатків?

<!-- tabs:start -->

#### **Node.js (Express)**

5. Для чого потрібен `helmet`? Назвіть 3 HTTP-заголовки, які він встановлює, та від яких атак вони захищають.
6. Чому `console.log()` недостатньо для production? Які переваги дає `winston` (рівні, транспорти, формат)?
7. Що означає змінна `NODE_ENV` і як вона впливає на поведінку Express та інших бібліотек?

#### **Python (FastAPI)**

5. Як `slowapi` інтегрується з FastAPI і як застосувати ліміт лише до конкретного endpoint-а?
6. Яка різниця між `logging.StreamHandler()` та `logging.FileHandler()`? Як налаштувати різні рівні для консолі та файлу?
7. Як CORS middleware FastAPI (`CORSMiddleware`) дозволяє налаштувати список дозволених origins?

#### **.NET (ASP.NET Core)**

5. Як вбудований Rate Limiter (.NET 7+) відрізняється від сторонніх бібліотек? Що таке `FixedWindowLimiter`?
6. Як Serilog інтегрується в ASP.NET Core через `UseSerilog()` і чим він кращий за вбудований `ILogger` для production?
7. Що таке `[EnableRateLimiting]` атрибут і як застосувати ліміт лише до Auth контролера?

<!-- tabs:end -->
