---
name: orange-link-context
description: >-
  Use when working on the Orange Link project (orange-link-app frontend + orange-link-back backend).
  Activates for any task involving the React PWA, Express API, Prisma/PostgreSQL database,
  API endpoints, components, screens, state management, or controllers.
  Keywords: orange-link, orange-link-app, orange-link-back, frontend, backend, API,
  componente, pantalla, screen, ruta, controlador, Prisma, BD, base de datos,
  useAppStore, Dashboard, Fasting, Training, Hydration, Status, Nutrition, Supplements.
---

# Orange Link — Skill de Contexto

## ⚠️ Regla de Auto-Actualización
Tras cualquier cambio significativo, actualiza también `AGENTS.md` (en la raíz del workspace).

## ¿Qué es Orange Link?

Asistente de salud y fitness compuesto por:
- **orange-link-app** → Frontend PWA (React 18 + TS + Vite + Tailwind + shadcn/ui)
- **orange-link-back** → Backend API REST (Express + TS + Prisma + PostgreSQL/Neon)

## Conexión App ↔ Back

- `VITE_API_BASE_URL` apunta a `http://localhost:3001/api` (dev) o URL Vercel
- Vite proxy: `/api` → `http://localhost:3001` (en `vite.config.ts`)
- API client: `src/lib/api.ts` (fetch wrapper con get/post/put/del)
- Optimistic updates: localStorage primero, luego fetch fire-and-forget

## Frontend (App)

- **Entry**: `src/main.tsx` → `App.tsx` (Router + TooltipProvider + Toaster)
- **Shell**: `pages/Index.tsx` — splash → Header + screen (switch) + BottomNav
- **12 screens** en `src/components/screens/`: Dashboard, Nutrition, MealSelection, Fasting, Training, Hydration, Supplements, Status, Stats, Profile, ProteinCalculator, CreatineCalculator
- **Estado global**: `hooks/useAppStore.ts` — localStorage + sync API (reseteo diario CDMX)
- **BottomNav**: 7 tabs (Home, Nutrition, Fasting, Training, Water, Supplements, Status)
- **Datos estáticos**: `data/nutritionData.ts` (alimentos) y `data/recipesData.ts` (recetas)

## Backend (API)

- **Entry**: `src/index.ts` → Express + CORS + morgan + routes + error handler
- **Estructura**: routes/ (definiciones) + controllers/ (lógica) + middleware/ (auth, errorHandler)
- **Auth**: `DEFAULT_USER_ID` env var o primer usuario en BD (sin JWT/OAuth)
- **Base**: PostgreSQL via Prisma (10 modelos: User, DailyLog, BodyMeasurement, BodyStatusTargets, Supplement, TrainingConfig, FastingConfig, NutritionPlan, UserNutritionPlan, MonthlyMission)
- **Endpoints**: todos bajo `/api/v1/` (dashboard, user, status, fasting, training, hydration, nutrition, supplements, calculators)

## Desarrollo

```bash
# Terminal 1
cd orange-link-back && npm run dev    # :3001

# Terminal 2
cd orange-link-app && npm run dev     # :8080
```
