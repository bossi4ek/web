# Frontend Labs Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reorganize backend labs into `labs/backend/`, add two frontend placeholder labs in `labs/frontend/`, and update all navigation and documentation to reflect the new structure.

**Architecture:** Pure file reorganization + new placeholder content. Git mv preserves full git history for the six backend lab files. Frontend placeholders use the same docsify-tabs pattern (`<!-- tabs:start -->`) already established on the backend, with React and Vue as the two tab options. Theory sections are shared above tabs; task sections are tab-specific.

**Tech Stack:** Docsify 4, docsify-tabs plugin, Markdown, git

---

## File Map

| File | Action |
|------|--------|
| `labs/lab1.md` → `labs/backend/lab1.md` | `git mv` |
| `labs/lab2.md` → `labs/backend/lab2.md` | `git mv` |
| `labs/lab3.md` → `labs/backend/lab3.md` | `git mv` |
| `labs/lab4.md` → `labs/backend/lab4.md` | `git mv` |
| `labs/lab5.md` → `labs/backend/lab5.md` | `git mv` |
| `labs/lab6.md` → `labs/backend/lab6.md` | `git mv` |
| `labs/frontend/lab1.md` | create |
| `labs/frontend/lab2.md` | create |
| `_sidebar.md` | update all 6 paths + add Frontend section |
| `README.md` | update title + all quick links |
| `CLAUDE.md` | update Content Architecture + lab path references |
| `docs/superpowers/plans/2026-04-27-multi-tech-labs.md` | update stale `labs/lab*.md` paths |

---

## Task 0: Move backend labs to `labs/backend/`

**Files:**
- Modify (move): `labs/lab1.md` … `labs/lab6.md` → `labs/backend/`

- [ ] **Step 1: Create directory and move all six files**

```bash
mkdir -p labs/backend
git mv labs/lab1.md labs/backend/lab1.md
git mv labs/lab2.md labs/backend/lab2.md
git mv labs/lab3.md labs/backend/lab3.md
git mv labs/lab4.md labs/backend/lab4.md
git mv labs/lab5.md labs/backend/lab5.md
git mv labs/lab6.md labs/backend/lab6.md
```

- [ ] **Step 2: Verify the moves**

```bash
git status
```

Expected output contains six lines like:
```
renamed:    labs/lab1.md -> labs/backend/lab1.md
renamed:    labs/lab2.md -> labs/backend/lab2.md
...
```

- [ ] **Step 3: Commit**

```bash
git commit -m "refactor: move backend labs to labs/backend/"
```

---

## Task 1: Create `labs/frontend/lab1.md`

**Files:**
- Create: `labs/frontend/lab1.md`

- [ ] **Step 1: Create the file with this exact content**

```markdown
# Лабораторна робота №1: Налаштування проєкту та список Places

## 1. Мета роботи

Налаштувати фронтенд-проєкт, ознайомитись зі структурою компонентного підходу, реалізувати відображення списку місць через запит до Places API (`GET /api/places`).

## 2. Теоретичні відомості

- **SPA (Single Page Application)** — веб-додаток, що завантажується один раз і динамічно оновлює контент без перезавантаження сторінки.
- **Компонентний підхід** — UI розбивається на ізольовані, повторно використовувані блоки (компоненти), кожен з яких відповідає за свою частину інтерфейсу.
- **REST API-запити з браузера** — браузер може звертатись до серверних API через `fetch` або бібліотеки (`axios`). Важливо враховувати CORS-налаштування сервера.

> Оберіть один технологічний стек нижче та дотримуйтесь його протягом усього курсу.

<!-- tabs:start -->

#### **React**

> 🚧 Цей розділ знаходиться в розробці.

#### **Vue**

> 🚧 Цей розділ знаходиться в розробці.

<!-- tabs:end -->
```

- [ ] **Step 2: Commit**

```bash
git add labs/frontend/lab1.md
git commit -m "feat(frontend): add lab1 placeholder — project setup and Places list"
```

---

## Task 2: Create `labs/frontend/lab2.md`

**Files:**
- Create: `labs/frontend/lab2.md`

- [ ] **Step 1: Create the file with this exact content**

```markdown
# Лабораторна робота №2: Форми, CRUD та навігація

## 1. Мета роботи

Реалізувати форми для створення та редагування місць, підключити операції `POST`, `PUT`, `DELETE` до Places API, налаштувати навігацію між сторінками додатку.

## 2. Теоретичні відомості

- **Контрольовані форми** — підхід, при якому стан полів форми зберігається в компоненті, а не в DOM. Забезпечує повний контроль над введеними даними та їхньою валідацією.
- **Клієнтська маршрутизація** — навігація між "сторінками" SPA без перезавантаження браузера. Реалізується через router-бібліотеки, що керують URL та відображають відповідні компоненти.
- **Оптимістичне оновлення UI** — техніка, коли інтерфейс оновлюється одразу після дії користувача, не чекаючи підтвердження від сервера.

> Оберіть один технологічний стек нижче та дотримуйтесь його протягом усього курсу.

<!-- tabs:start -->

#### **React**

> 🚧 Цей розділ знаходиться в розробці.

#### **Vue**

> 🚧 Цей розділ знаходиться в розробці.

<!-- tabs:end -->
```

- [ ] **Step 2: Commit**

```bash
git add labs/frontend/lab2.md
git commit -m "feat(frontend): add lab2 placeholder — forms, CRUD and routing"
```

---

## Task 3: Update `_sidebar.md`

**Files:**
- Modify: `_sidebar.md`

- [ ] **Step 1: Replace the entire file content**

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

- [ ] **Step 2: Commit**

```bash
git add _sidebar.md
git commit -m "feat(sidebar): reorganize into Backend and Frontend sections"
```

---

## Task 4: Update `README.md`

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Replace the H1 title**

Old:
```markdown
# 🎓 Web-програмування: Backend розробка (Node.js + Express)
```

New:
```markdown
# 🎓 Web-програмування: Backend + Frontend розробка
```

- [ ] **Step 2: Replace the description sentence**

Old:
```markdown
Тут зібрані всі завдання, теоретичні відомості та вимоги для успішного виконання наскрізного проєкту — розробки REST API для платформи Location/Places.
```

New:
```markdown
Тут зібрані всі завдання, теоретичні відомості та вимоги для успішного виконання наскрізного проєкту — розробки платформи Places (REST API + веб-інтерфейс).
```

- [ ] **Step 3: Replace the quick links section**

Old:
```markdown
### Швидкі посилання:
- [Ознайомитись із загальною концепцією курсу](context.md)
- [Перейти до Лабораторної роботи №1](labs/lab1.md)
- [Перейти до Лабораторної роботи №2](labs/lab2.md)
- [Перейти до Лабораторної роботи №3](labs/lab3.md)
- [Перейти до Лабораторної роботи №4](labs/lab4.md)
- [Перейти до Лабораторної роботи №5](labs/lab5.md)
- [Перейти до Лабораторної роботи №6](labs/lab6.md)
- [Переглянути всі контрольні питання](control_questions.md)
```

New:
```markdown
### Бекенд (Node.js / FastAPI / .NET):
- [Концепція курсу та проєкт](context.md)
- [ЛР №1: Основи backend + Places API](labs/backend/lab1.md)
- [ЛР №2: Інтеграція Бази Даних](labs/backend/lab2.md)
- [ЛР №3: Робота з пов'язаними даними](labs/backend/lab3.md)
- [ЛР №4: Аутентифікація та JWT](labs/backend/lab4.md)
- [ЛР №5: Просунуті запити](labs/backend/lab5.md)
- [ЛР №6: Production Readiness](labs/backend/lab6.md)

### Фронтенд (React / Vue):
- [ЛР №1: Налаштування проєкту та список Places](labs/frontend/lab1.md)
- [ЛР №2: Форми, CRUD та навігація](labs/frontend/lab2.md)

### Додатково:
- [Контрольні питання](control_questions.md)
- [Теми курсових робіт](topics.md)
```

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs(readme): update title and links for full-stack course structure"
```

---

## Task 5: Update `CLAUDE.md`

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Update the file tree in Content Architecture**

Old line:
```
labs/             — One file per lab assignment (lab1.md … lab6.md)
```

New lines (replace the one line with two):
```
labs/backend/     — Backend lab assignments (lab1.md … lab6.md)
labs/frontend/    — Frontend lab assignments (lab1.md … lab2.md, placeholders)
```

- [ ] **Step 2: Update the Lab File Structure Convention note**

Old:
```
Each `labs/labN.md` follows this section order:
```

New:
```
Each `labs/backend/labN.md` and `labs/frontend/labN.md` follows this section order:
```

- [ ] **Step 3: Update the Multi-Technology Tracks section**

In the paragraph starting "Implementation approach:", change:
```
embed three-way tabs directly inside each `labs/labN.md`
```
to:
```
embed three-way tabs directly inside each `labs/backend/labN.md`
```

In the sentence "The full implementation plan is at `docs/superpowers/plans/...`", add a reference to this plan too:
```
Backend plan: `docs/superpowers/plans/2026-04-27-multi-tech-labs.md`. Frontend structure plan: `docs/superpowers/plans/2026-04-27-frontend-labs-integration.md`.
```

- [ ] **Step 4: Update the Adding a New Lab section**

Old:
```
1. Create `labs/labN.md` following the section convention above.
2. Add an entry to `_sidebar.md` under **Лабораторні роботи**.
```

New:
```
1. Create `labs/backend/labN.md` or `labs/frontend/labN.md` following the section convention above.
2. Add an entry to `_sidebar.md` under the appropriate section (**Бекенд** or **Фронтенд**).
```

- [ ] **Step 5: Commit**

```bash
git add CLAUDE.md
git commit -m "docs(claude): update content architecture for backend/frontend split"
```

---

## Task 6: Update stale paths in the multi-tech-labs plan

**Files:**
- Modify: `docs/superpowers/plans/2026-04-27-multi-tech-labs.md`

- [ ] **Step 1: Replace all occurrences of `labs/lab` with `labs/backend/lab`**

Use the Edit tool with `replace_all: true` or run:

```bash
sed -i 's|labs/lab\([0-9]\)|labs/backend/lab\1|g' docs/superpowers/plans/2026-04-27-multi-tech-labs.md
```

Verify: `grep "labs/lab[0-9]" docs/superpowers/plans/2026-04-27-multi-tech-labs.md` should return no matches. `grep "labs/backend/lab" docs/superpowers/plans/2026-04-27-multi-tech-labs.md` should return ~30 matches.

- [ ] **Step 2: Also update the File Map table header row**

Old line in the File Map table:
```
| `labs/lab1.md` | Wrap завдання + вимоги + результат in 3-way tabs |
```

These are already handled by the sed in Step 1. Double-check the table looks correct after the replacement.

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/plans/2026-04-27-multi-tech-labs.md
git commit -m "docs(plans): update backend lab paths to labs/backend/ after refactor"
```

---

## Self-Review

**Spec coverage:**
- ✅ `git mv` all 6 backend labs → Task 0
- ✅ `labs/frontend/lab1.md` placeholder → Task 1
- ✅ `labs/frontend/lab2.md` placeholder → Task 2
- ✅ `_sidebar.md` — new paths + Frontend section → Task 3
- ✅ `README.md` — updated title and links → Task 4
- ✅ `CLAUDE.md` — updated Content Architecture → Task 5
- ✅ `control_questions.md` — grep confirms no lab links, no task needed
- ✅ `context.md` — grep confirms no lab links, no task needed
- ✅ Internal plan doc with stale paths → Task 6

**Placeholder scan:** No TBD/TODO in plan steps. All code blocks contain exact content.

**Type consistency:** No functions or types — pure Markdown file operations. Path `labs/backend/labN.md` used consistently across all tasks.
