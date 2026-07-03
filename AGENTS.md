# Orange Link — Contexto del Proyecto

## ⚠️ Regla de Auto-Actualización (OBLIGATORIA)

Cada vez que añadas, modifiques o elimines un feature significativo (página, componente, ruta API, modelo de BD, configuración, dependencia), **debes actualizar automáticamente** las secciones relevantes de este archivo para mantener el contexto sincronizado. No esperes a que el usuario lo pida. Si no estás seguro de qué actualizar, actualiza todo el archivo escaneando ambos proyectos.

## Visión General

Orange Link es un asistente de salud y fitness compuesto por dos proyectos:

- **orange-link-app**: Frontend PWA (React 18 + TypeScript + Vite + Tailwind CSS + shadcn/ui)
- **orange-link-back**: Backend API REST (Express 4 + TypeScript + Prisma + PostgreSQL/Neon)

### Relación App ↔ Back

- La app se comunica con el backend mediante fetch a través de `src/lib/api.ts`
- Variable de entorno: `VITE_API_BASE_URL=http://localhost:3001/api` (dev) o URL de Vercel
- En desarrollo, Vite proxy `/api` → `http://localhost:3001` (configurado en `vite.config.ts`)
- El frontend usa optimistic updates: primero actualiza localStorage, luego envía la petición al backend (fire-and-forget con `.catch(() => {})`)

### Root Monorepo (`package.json` en raíz)

- Ubicación: `dev/package.json`
- Usa `concurrently` para ejecutar frontend y backend con un solo comando
- Script `dev`: corre ambos proyectos en paralelo con prefijos `[app]` y `[back]`
- Script `setup`: instala dependencias del root + ambos subproyectos

---

## orange-link-app (Frontend)

### Stack

| Tecnología | Versión |
|---|---|
| React | ^18.3.1 |
| TypeScript | ^5.8.3 |
| Vite | ^7.0.0 |
| Tailwind CSS | ^3.4.17 |
| React Router DOM | ^6.30.1 |
| Radix UI (shadcn/ui) | ^1.x |
| React Hook Form + Zod | ^7.61.1 / ^3.25.76 |
| Recharts | ^2.15.4 |
| Sonner (toast) | ^1.7.4 |
| date-fns | ^3.6.0 |
| vite-plugin-pwa | ^1.2.0 |
| Lucide React | ^0.462.0 |

### Estructura del Proyecto

```
orange-link-app/
├── src/
│   ├── main.tsx              # Entry point
│   ├── App.tsx               # Componente raíz (Router + TooltipProvider + Toaster)
│   ├── index.css             # Estilos globales + variables CSS + Tailwind directives
│   ├── pages/
│   │   ├── Index.tsx         # Shell principal (splash → screens + bottom nav + header)
│   │   └── NotFound.tsx      # Página 404
│   ├── components/
│   │   ├── Header.tsx        # Header sticky con título + botón perfil + botón settings
│   │   ├── BottomNav.tsx     # Navegación inferior fija (filtrable por visibleModules)
│   │   ├── SplashScreen.tsx  # Splash animado (2s)
│   │   ├── PwaUpdater.tsx    # Notificador de actualización SW
│   │   ├── MissionItem.tsx   # Item reutilizable de misión diaria
│   │   ├── screens/          # Pantallas de la app (ver sección abajo)
│   │   └── ui/               # shadcn/ui primitives (button, dialog, input, tabs, etc.)
│   ├── hooks/
│   │   ├── useAppStore.ts    # Estado global (localStorage + sync API)
│   │   └── use-toast.ts      # Hook de toast (shadcn/ui)
│   ├── lib/
│   │   ├── api.ts            # Cliente API (fetch wrapper con get/post/put/del)
│   │   └── utils.ts          # Utilidades (cn() para merging de clases)
│   ├── data/
│   │   ├── nutritionData.ts  # Alimentos permitidos/prohibidos (estático, español)
│   │   └── recipesData.ts    # Recetas por categoría (desayunos, comidas, cenas, snacks)
│   └── test/
│       ├── setup.ts          # Config Vitest (jsdom, matchMedia mock)
│       └── example.test.ts
├── public/                   # Favicon, PWA icons, robots.txt
├── index.html                # HTML shell (es-MX, meta PWA)
├── vite.config.ts            # Proxy /api → :3001, PWA manifest, alias @/ → src/
├── tailwind.config.ts        # Dark mode class, fuente Nunito, animaciones custom
├── components.json           # shadcn/ui config
└── package.json
```

### Screens (Pantallas)

Todas en `src/components/screens/`. La navegación es state-based: `Index.tsx` usa variable `screen` + `switch` para renderizar la pantalla activa.

| Archivo | Propósito |
|---|---|
| `Dashboard.tsx` | Home: barra de progreso, grid 2x2 de módulos, misión diaria (5 items), misiones mensuales |
| `NutritionScreen.tsx` | Plan de comidas actual, selección del día, alimentos permitidos/prohibidos |
| `MealSelectionScreen.tsx` | Selección de comidas por categoría (desayuno/comida/cena/snack) |
| `FastingScreen.tsx` | Tracker de ayuno intermitente (16:8, 14:10, 12:12), timer, configuración |
| `TrainingScreen.tsx` | Entrenamiento: vista semanal, ejercicios con reps/sets, intensidad, templates |
| `HydrationScreen.tsx` | Tracker de agua: botones + cantidad, barra de progreso, calculadora |
| `SupplementsScreen.tsx` | Checklist de suplementos (creatina, B12, magnesio, enzimas) |
| `StatusScreen.tsx` | Métricas corporales: peso, grasa, músculo, BMI, metas, tips inteligentes |
| `StatsScreen.tsx` | Gráficos históricos (Recharts LineChart): peso, grasa, BMI, etc. |
| `SettingsScreen.tsx` | Configuración: toggle visibilidad de módulos en BottomNav y Dashboard |
| `ProfileScreen.tsx` | Perfil de usuario: nombre, email, member since, edición |
| `ProteinCalculatorScreen.tsx` | Calculadora de proteína diaria |
| `CreatineCalculatorScreen.tsx` | Calculadora de dosis de creatina |
| `LoginScreen.tsx` | Login/registro con email + contraseña (JWT) |

### BottomNav (7 Tabs)

Home, Nutrition, Fasting, Training, Water, Supplements, Status — filtrable por `visibleModules` en Settings.

### Estado Global (useAppStore)

- **Ubicación**: `src/hooks/useAppStore.ts`
- **Alcance**: misiones diarias, entrenamiento, suplementos, hidratación, ayuno, nutrición, estado corporal, comidas, perfil
- **Persistencia**: localStorage bajo clave `orangelink-state` (reseteo diario automático a medianoche CDMX)
- **Patrón**: acciones optimistas (actualiza local → fire-and-forget al backend)
- **Training defaults**: `defaultState` arranca con schedule/templates/restDays/dayLabels vacíos (sin rutinas precargadas)

### API Client (`src/lib/api.ts`)

```typescript
api.auth.login(email, password) | .register(name, email, password) | .logout() | .me()
api.dashboard.get()
api.supplements.list() | .add(name) | .remove(id) | .toggle(id)
api.hydration.get() | .updateGoal(goal) | .addIntake(amount) | .calculate(data)
api.fasting.get() | .update(data)
api.nutrition.plans() | .current() | .updatePlan(name) | .updateMeals(meals)
api.training.get() | .update(data)
api.status.get() | .history() | .create(data) | .updateTargets(targets) | .tips()
api.user.profile.get() | .update(data)
api.calculators.protein(data) | .creatine(data)
```

---

## orange-link-back (Backend)

### Stack

| Tecnología | Versión |
|---|---|
| Express | ^4.21.2 |
| TypeScript | ^5.8.3 |
| Prisma | ^6.6.0 |
| PostgreSQL (Neon) | — |
| cors | ^2.8.5 |
| morgan | ^1.10.0 |
| jsonwebtoken | — |
| bcryptjs | — |
| cookie-parser | — |

### Estructura del Proyecto

```
orange-link-back/
├── src/
│   ├── index.ts              # Express app (CORS, morgan, routes, error handler, start)
│   ├── lib/
│   │   └── prisma.ts         # Singleton PrismaClient
│   ├── middleware/
│   │   ├── auth.ts           # Auth (JWT Bearer token)
│   │   └── errorHandler.ts   # Error handler + AppError class
│   ├── routes/               # Definiciones Express Router
│   │   ├── health.ts
│   │   ├── dashboard.ts
│   │   ├── supplements.ts
│   │   ├── hydration.ts
│   │   ├── fasting.ts
│   │   ├── nutrition.ts
│   │   ├── training.ts
│   │   ├── status.ts
│   │   ├── calculators.ts
│   │   ├── user.ts
│   │   ├── auth.ts
│   │   └── admin.ts
│   └── controllers/          # Lógica de negocio
│       ├── dashboard.ts
│       ├── supplements.ts
│       ├── hydration.ts
│       ├── fasting.ts
│       ├── nutrition.ts
│       ├── training.ts
│       ├── status.ts
│       ├── calculators.ts
│       ├── user.ts
│       ├── auth.ts
│       └── admin.ts
├── prisma/
│   ├── schema.prisma         # Esquema de BD (10 modelos)
│   └── seed.ts               # Seed: templates de ejercicios (sin rutinas predefinidas)
├── api/
│   └── index.ts              # Entry point Vercel serverless
├── vercel.json               # Rutas → api/index.ts @vercel/node
└── package.json
```

### API Endpoints

Base: `/api/v1/` — las rutas marcadas con 🔒 requieren middleware auth.

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/health` | Dashboard HTML de salud |
| GET | `/api/health/json` | Health check JSON |
| **Dashboard** | | |
| GET | `/api/v1/dashboard` | Dashboard agregado (misiones, módulos, métricas) |
| **User** | | |
| GET | `/api/v1/user/profile` | Perfil (name, email, registrado desde) |
| PUT | `/api/v1/user/profile` | Actualizar perfil |
| **Status** | | |
| GET | `/api/v1/status` | Última medición + targets |
| GET | `/api/v1/status/history` | Historial (últimas 50 mediciones) |
| POST | `/api/v1/status` | Crear medición corporal (weight + height required) |
| PUT | `/api/v1/status/targets` | Actualizar targets (upsert) |
| GET | `/api/v1/status/tips` | Tips inteligentes según progreso |
| **Fasting** | | |
| GET | `/api/v1/fasting` | Config de ayuno (auto-crea default 16:8) |
| PUT | `/api/v1/fasting` | Actualizar plan, ventana, fecha fin |
| **Training** | | |
| GET | `/api/v1/training` | Config de entrenamiento (auto-crea defaults) |
| PUT | `/api/v1/training` | Actualizar intensidad, schedule, templates, etc. |
| PUT | `/api/v1/training/missions/:key` | Toggle misión de entrenamiento |
| **Hydration** | | |
| GET | `/api/v1/hydration` | Consumo de agua del día + meta |
| PUT | `/api/v1/hydration/goal` | Actualizar meta diaria |
| POST | `/api/v1/hydration/intake` | Agregar consumo (incrementa) |
| POST | `/api/v1/hydration/calculate` | Calcular recomendación (weight, activity, sex) |
| **Nutrition** | | |
| GET | `/api/v1/nutrition/plans` | Listar planes disponibles |
| GET | `/api/v1/nutrition/current` | Plan actual + comidas del día |
| PUT | `/api/v1/nutrition/plan` | Seleccionar/cambiar plan |
| PUT | `/api/v1/nutrition/meals` | Actualizar comidas del día |
| **Supplements** | | |
| GET | `/api/v1/supplements` | Suplementos + estado del día |
| POST | `/api/v1/supplements` | Agregar suplemento |
| DELETE | `/api/v1/supplements/:id` | Eliminar suplemento |
| PUT | `/api/v1/supplements/:id/toggle` | Toggle tomado/no tomado |
| **Calculators** | | |
| POST | `/api/v1/calculators/protein` | Calcular proteína (weight, activityLevel) |
| POST | `/api/v1/calculators/creatine` | Calcular creatina (weight, phase) |
| **Auth** | | |
| POST | `/api/v1/auth/register` | ❌ Público — Registrar nuevo usuario (name, email, password) |
| POST | `/api/v1/auth/login` | ❌ Público — Iniciar sesión (email, password) → access + refresh tokens |
| POST | `/api/v1/auth/refresh` | ❌ Público — Refrescar access token (body: refreshToken) |
| POST | `/api/v1/auth/logout` | 🔒 Cerrar sesión (invalida refresh token) |
| GET | `/api/v1/auth/me` | 🔒 Datos del usuario autenticado |
| **Admin** | | |
| GET | `/admin` | Panel HTML (login si no hay sesión) |
| POST | `/admin/login` | Login con ADMIN_PASSWORD → cookie |
| POST | `/admin/logout` | Cerrar sesión admin |
| POST | `/admin/approve/:id` | Aprobar usuario |
| POST | `/admin/reject/:id` | Rechazar usuario |

### Base de Datos (Prisma)

Modelos definidos en `prisma/schema.prisma`:

| Modelo | Tabla | PK | Campos clave |
|---|---|---|---|
| User | users | id | email, name, password, refreshToken, status, avatarUrl, googleId |
| DailyLog | daily_logs | userId+date (unique) | waterIntake, waterGoal, dailyMissions, trainingMissions, supplementsTaken, selectedMeals |
| BodyMeasurement | body_measurements | id | userId, measuredAt, weight, height, bodyFat, visceralFat, skeletalMuscle, bmi, restingMetabolism |
| BodyStatusTargets | body_status_targets | userId | weight, bodyFat, visceralFat, skeletalMuscle, bmi |
| Supplement | supplements | id | userId, name (unique por user+name) |
| TrainingConfig | training_configs | userId | intensity, endDate, schedule (JSON), templates (JSON), trainingCompleted, trainingExerciseCompleted, restDays, dayLabels |
| FastingConfig | fasting_configs | userId | planType, windowStart, windowEnd, endDate |
| NutritionPlan | nutrition_plans | id | name, description |
| UserNutritionPlan | user_nutrition_plans | userId+nutritionPlanId | selectedAt |
| MonthlyMission | monthly_missions | id | userId, month, bodyFat |

### Auth

- JWT con access token (15 min) + refresh token rotativo (7 días)
- bcrypt para hashear contraseñas (10 salt rounds)
- Access token en memoria (variable en api.ts), refresh token en localStorage
- Auto-refresh silencioso en 401: si el access token expira, se refresca automáticamente
- Cierre de sesión invalida el refresh token en DB

### Middleware

- **auth.ts**: Verifica JWT del header `Authorization: Bearer <token>`, extrae `{ userId, email }` del payload. Retorna 401 si el token es inválido o expirado.
- **errorHandler.ts**: Global handler que captura AppError (statusCode + message) o devuelve 500.

---

## Variables de Entorno

### Frontend (`orange-link-app/.env`)

```
VITE_API_BASE_URL=http://localhost:3001/api
```

### Backend (`orange-link-back/.env`)

```
DATABASE_URL=postgresql://...
PORT=3001
JWT_SECRET=<secret-key>
ACCESS_TOKEN_EXPIRY=15m
REFRESH_TOKEN_EXPIRY=7d
ADMIN_PASSWORD=<admin-password>
```

---

## Desarrollo Local

```bash
# Opción 1 (recomendada): Ambos proyectos en una terminal
npm run dev

# Opción 2: Terminales separadas
# Terminal 1: Backend (puerto 3001)
cd orange-link-back && npm run dev

# Terminal 2: Frontend (puerto 8080, proxy /api → 3001)
cd orange-link-app && npm run dev
```

### Comandos Útiles

| Proyecto | Comando | Descripción |
|---|---|---|
| Frontend | `npm run dev` | Dev server :8080 |
| Frontend | `npm run build` | Build producción |
| Frontend | `npm run test` | Tests (Vitest) |
| Frontend | `npm run lint` | ESLint |
| Backend | `npm run dev` | Dev server :3001 (hot-reload con tsx) |
| Backend | `npm run build` | Compilar TS a dist/ |
| Backend | `npm run prisma:migrate` | Migraciones Prisma |
| Backend | `npm run prisma:seed` | Seed data |
| Backend | `npm run prisma:push` | Push schema a DB |
| Root | `npm run dev` | Frontend + Backend en paralelo (concurrently) |
| Root | `npm run setup` | Instalar deps del root + ambos subproyectos |
