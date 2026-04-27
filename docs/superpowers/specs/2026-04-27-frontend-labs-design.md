# Frontend Labs Integration — Design Spec

**Date:** 2026-04-27
**Branch:** feat/multi-tech-labs

---

## Goal

Add two frontend lab placeholders to the existing Docsify documentation site for the "Web Programming" course. The frontend labs build a UI for the same Places API project that backend labs produce. Students choose one frontend stack (React or Vue) and use it throughout all frontend labs.

---

## File Structure

Current `labs/lab*.md` files move into a `backend/` subfolder. Frontend labs live in a parallel `frontend/` subfolder.

```
labs/
  backend/
    lab1.md   ← renamed from labs/lab1.md (content unchanged)
    lab2.md
    lab3.md
    lab4.md
    lab5.md
    lab6.md
  frontend/
    lab1.md   ← new placeholder
    lab2.md   ← new placeholder
```

No content inside backend lab files changes — only their path.

---

## Navigation (`_sidebar.md`)

Four sections instead of three:

```markdown
- **Вступ**
  - [Головна сторінка](README.md)
  - [Концепція курсу та проєкт](context.md)

- **Бекенд (Node.js / FastAPI / .NET)**
  - [ЛР №1: Основи backend + Places API](labs/backend/lab1.md)
  - [ЛР №2: Інтеграція Бази Даних](labs/backend/lab2.md)
  - [ЛР №3: Робота з пов'язаними даними](labs/backend/lab3.md)
  - [ЛР №4: Аутентифікація та JWT](labs/backend/lab4.md)
  - [ЛР №5: Просунуті запити](labs/backend/lab5.md)
  - [ЛР №6: Production Readiness](labs/backend/lab6.md)

- **Фронтенд (React / Vue)**
  - [ЛР №1: Налаштування проєкту та список Places](labs/frontend/lab1.md)
  - [ЛР №2: Форми, CRUD та навігація](labs/frontend/lab2.md)

- **Додатково**
  - [Контрольні питання](control_questions.md)
  - [Теми курсових робіт](topics.md)
```

---

## Frontend Lab Structure

Each frontend lab follows the same section convention as backend labs:

1. **Мета роботи** — learning objectives
2. **Теоретичні відомості** — background theory (shared, above tabs)
3. Technology tabs (`<!-- tabs:start -->` / `<!-- tabs:end -->`)
   - `#### **React**`
   - `#### **Vue**`
   - Each tab contains: Завдання, Вимоги до реалізації, Очікуваний результат
4. Placeholder marker inside tabs: `> 🚧 Цей розділ знаходиться в розробці.`

---

## Frontend Lab Topics

### ЛР №1 — Налаштування проєкту та список Places

**Мета:** налаштувати фронтенд-проєкт, створити першу компоненту, відобразити список місць через `GET /api/places`.

- React tab: `create-react-app` або Vite + React, компонент `PlaceList`, `fetch`/`axios`
- Vue tab: `create-vue` (Vite), компонент `PlaceList.vue`, `fetch`/`axios`

### ЛР №2 — Форми, CRUD та навігація

**Мета:** реалізувати форми створення/редагування місць, підключити `POST`/`PUT`/`DELETE`, додати навігацію між сторінками.

- React tab: `react-router-dom`, компоненти `PlaceForm`, `PlaceDetail`
- Vue tab: `vue-router`, компоненти `PlaceForm.vue`, `PlaceDetail.vue`

---

## Files Changed

| File | Action |
|------|--------|
| `labs/lab1.md` → `labs/backend/lab1.md` | git mv |
| `labs/lab2.md` → `labs/backend/lab2.md` | git mv |
| `labs/lab3.md` → `labs/backend/lab3.md` | git mv |
| `labs/lab4.md` → `labs/backend/lab4.md` | git mv |
| `labs/lab5.md` → `labs/backend/lab5.md` | git mv |
| `labs/lab6.md` → `labs/backend/lab6.md` | git mv |
| `labs/frontend/lab1.md` | create |
| `labs/frontend/lab2.md` | create |
| `_sidebar.md` | update paths + new frontend section |
| `README.md` | update quick links |
| `control_questions.md` | update any internal links |
| `context.md` | update any internal links |
| `CLAUDE.md` | update Content Architecture section |

---

## Out of Scope

- Actual frontend lab content (fill-in is a future task)
- `topics.md` changes
- Backend lab content changes
- `docsify-tabs` plugin (already planned in multi-tech-labs branch)
