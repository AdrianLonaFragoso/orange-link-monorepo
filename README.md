# Orange Link

Asistente personal de salud y fitness. Este repositorio es un monorepo que contiene dos subproyectos como **git submodules**, cada uno con su propio repositorio independiente.

## Repositorios

| Proyecto | Repo | Descripción |
|---|---|---|
| Monorepo (raíz) | [orange-link-monorepo](https://github.com/AdrianLonaFragoso/orange-link-monorepo) | Orquestación, scripts `dev`/`setup` |
| Frontend | [orange-link-apollo](https://github.com/AdrianLonaFragoso/orange-link-apollo) | PWA React 18 + TypeScript + Vite + Tailwind + shadcn/ui |
| Backend | [orange-link-back](https://github.com/AdrianLonaFragoso/orange-link-back) | API REST Express + TypeScript + Prisma + PostgreSQL |

## Cómo clonar

```bash
# Opción 1 — Todo junto (monorepo + submodules)
git clone https://github.com/AdrianLonaFragoso/orange-link-monorepo.git --recurse-submodules

# Opción 2 — Solo frontend (trabajo independiente)
git clone https://github.com/AdrianLonaFragoso/orange-link-apollo.git

# Opción 3 — Solo backend (trabajo independiente)
git clone https://github.com/AdrianLonaFragoso/orange-link-back.git
```

Si ya clonaste el monorepo sin `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

Cada subproyecto tiene su propio historial, ramas, issues y PRs — se pueden trabajar por separado sin necesidad del monorepo.

## Estructura

```
orange-link-apollo/  → Frontend PWA (React 18 + TypeScript + Vite + Tailwind CSS + shadcn/ui)
orange-link-back/    → Backend API REST (Express 4 + TypeScript + Prisma + PostgreSQL/Neon)
```

## Frontend (`orange-link-apollo`)

PWA mobile-first con navegación state-based, persistencia en localStorage y sincronización optimista con el backend. Desarrollada con React 18, React Router DOM, Recharts y Radix UI.

- **Puerto dev**: 8080
- **Proxy**: `/api` → `http://localhost:3001` (configurado en Vite)
- **Estado global**: `useAppStore` — localStorage + sync fire-and-forget al backend

### Pantallas principales

Dashboard, Nutrición, Ayuno (intermitente), Entrenamiento, Hidratación, Suplementos, Estado corporal, Estadísticas (gráficos), Perfil, Calculadoras (proteína/creatina).

## Backend (`orange-link-back`)

API REST con autenticación JWT (access + refresh token rotativo). Arquitectura Express + controladores, middleware de auth y error handler global.

- **Puerto dev**: 3001
- **Base de datos**: PostgreSQL vía Prisma ORM
- **Serverless**: Entry point para Vercel en `api/index.ts`

### Endpoints (base `/api/v1/`)

- `auth/` — registro, login, refresh, logout, me
- `dashboard` — métricas agregadas
- `nutrition/` — planes y comidas
- `fasting` — configuración de ayuno
- `training` — configuración y misiones
- `hydration` — consumo de agua
- `supplements` — checklist de suplementos
- `status/` — mediciones corporales, targets, tips
- `calculators/` — proteína y creatina
- `user/profile` — perfil de usuario
- `admin/` — panel de administración

## Cómo empezar

```bash
npm run setup     # Instalar dependencias en ambos subproyectos
npm run dev       # Iniciar frontend (8080) y backend (3001) en paralelo
```

O por separado:

```bash
cd orange-link-back && npm run dev
cd orange-link-apollo && npm run dev
```

## Variables de entorno

| Proyecto | Archivo | Variables clave |
|---|---|---|
| Frontend | `orange-link-apollo/.env` | `VITE_API_BASE_URL` |
| Backend | `orange-link-back/.env` | `DATABASE_URL`, `PORT`, `JWT_SECRET`, `ADMIN_PASSWORD` |

## Comunicación App ↔ Back

El frontend usa `src/lib/api.ts` para comunicarse con el backend mediante fetch. Sigue un patrón de optimistic updates: actualiza localStorage primero y envía la petición al backend en segundo plano (fire-and-forget).

## Flujo de trabajo con submodules

El monorepo contiene dos **git submodules** (`orange-link-apollo` y `orange-link-back`).
Cada submodule apunta a un **commit específico** de su propio repo. No se hace push directo desde el monorepo a los submodules.

### Modificaste código en un submodule

```bash
# 1. Ir al submodule, commitear y pushear
cd orange-link-apollo
git add .
git commit -m "feat: descripción del cambio"
git push origin main

# 2. Volver al monorepo, actualizar la referencia y pushear
cd ..
git add orange-link-apollo
git commit -m "chore: actualizar referencia de orange-link-apollo"
git push origin main
```

### Clonaste fresco y quieres trabajar en submodules

```bash
git submodule update --init --recursive
cd orange-link-apollo
git checkout -b main origin/main
```

### Consejos clave

- **Siempre pushear los submodules primero**, luego el monorepo. Si pusheas el monorepo sin pushear los submodules, quien clone tendrá referencias rotas.
- Para evitar detached HEAD, haz `git checkout main` dentro de cada submodule antes de empezar a trabajar.
- `git status` en la raíz mostrará `(modified content)` en los submodules cuando el commit apuntado no coincida con el del repo remoto.
