# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A **Docsify-based static documentation site** for a Ukrainian-language university course on Node.js backend development (3rd-year students). Content lives entirely in Markdown files; there is no build step, no package.json, and no test suite.

The site is deployed to **GitHub Pages** at the `/web/` path (see `<base href="/web/">` in `index.html`).

## Previewing Locally

Docsify requires an HTTP server (it cannot open `index.html` directly as a file). Run one of:

```bash
npx serve .
# or
npx docsify-cli serve .
# or
python -m http.server
```

> **Note:** The `<base href="/web/">` in `index.html` causes broken navigation when serving from `localhost/`. To preview locally, temporarily change it to `<base href="/">` and revert before committing.

## Content Architecture

```
index.html        — Docsify bootstrap (CDN-loaded, no local deps)
_sidebar.md       — Navigation menu (must be updated when adding pages)
README.md         — Course homepage shown at /
context.md        — Course concept: the "Places API" central project
control_questions.md — Review questions grouped by lab
topics.md         — 40 course-work themes (full-stack systems, grouped by domain)
labs/backend/     — Backend lab assignments (lab1.md … lab6.md)
labs/frontend/    — Frontend lab assignments (lab1.md … lab2.md, placeholders)
```

### The Central Project

All six labs build a single **REST API for a "Places" platform** (TripAdvisor-style). Each lab adds a layer:

| Lab | Layer |
|-----|-------|
| lab1 | In-memory CRUD for `Place` entity |
| lab2 | PostgreSQL + ORM + validation |
| lab3 | `Review` entity, One-to-Many relationships |
| lab4 | Users, JWT authentication, ownership checks |
| lab5 | Filtering, pagination, RBAC |
| lab6 | Logging, Helmet/CORS/rate-limiting, deployment |

### Lab File Structure Convention

Each `labs/backend/labN.md` and `labs/frontend/labN.md` follows this section order:
1. **Мета роботи** — learning objectives
2. **Теоретичні відомості** — background theory
3. **Завдання** — numbered tasks
4. **Вимоги до реалізації** — file structure + implementation requirements
5. **API Endpoints** — example requests/responses
6. **Очікуваний результат** — success criteria

## Language

All content is in **Ukrainian**. Headings, task descriptions, and student-facing text should remain in Ukrainian when editing lab files.

## Multi-Technology Tracks (branch: feat/multi-tech-labs)

The `feat/multi-tech-labs` branch adds **FastAPI (Python)** and **.NET (ASP.NET Core / C#)** tracks to all six labs alongside the existing Node.js content. Students pick one stack and stick with it.

Implementation approach: use the **docsify-tabs** plugin to embed three-way tabs directly inside each `labs/backend/labN.md`. Theory sections stay shared above the tabs; tasks, file structure, code examples, and expected results are inside tabs.

Tab markup pattern:
```markdown
<!-- tabs:start -->

#### **Node.js (Express)**
[existing content]

#### **Python (FastAPI)**
[FastAPI content]

#### **.NET (ASP.NET Core)**
[.NET content]

<!-- tabs:end -->
```

The plugin CDN script and config go in `index.html`. Backend plan: `docs/superpowers/plans/2026-04-27-multi-tech-labs.md`. Frontend structure plan: `docs/superpowers/plans/2026-04-27-frontend-labs-integration.md`.

## Adding a New Lab

1. Create `labs/backend/labN.md` or `labs/frontend/labN.md` following the section convention above.
2. Add an entry to `_sidebar.md` under the appropriate section (**Бекенд** or **Фронтенд**).
3. Add a link in `README.md`.
