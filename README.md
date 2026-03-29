# FinTrack — Personal Finance Manager for Students

FinTrack is a full-stack personal finance application focused on student-friendly money management. It combines transaction tracking, smart category suggestions, monthly budgeting, bill splitting, savings goals, and visual analytics in a single responsive app.

The repository includes **two backend runtime options**:

1. **Vercel serverless API** (`/api` + `/lib`) for deployment.
2. **Standalone Express API** (`/server`) for local/server-based execution.

The React frontend (`/client`) talks to `/api` paths, so it works with either backend mode.

---

## What this repository is about

This project is a practical finance tracker with these core capabilities:

- **Transaction management** (income + expenses) with filters, pagination, and CRUD.
- **Rule-based smart categorization** of expenses from description text.
- **Dashboard metrics**: income, expenses, balance, savings rate, category share, and trend visuals.
- **Monthly budgeting**: total and per-category limits with utilization and over-budget alerts.
- **Bill splitting**: equal/custom splits, participant settlement tracking, and status transitions.
- **Savings goals**: target amount progress tracking, deadlines, and add-funds flow.
- **PWA support**: install prompt, manifest, and service worker for app-like usage.

---

## Architecture overview

### Frontend (`client/`)

- React 18 + Vite.
- Routing with `react-router-dom`.
- Charts with `recharts`.
- Motion/transitions with `framer-motion`.
- Toast notifications with `react-hot-toast`.
- Axios API client defaults to:
  - `VITE_API_URL` if provided.
  - otherwise `/api`.

### Backend option A: Vercel serverless (`api/` + `lib/`)

- API route handlers in `api/**`.
- Shared Mongoose models in `lib/models/**`.
- Mongo connection caching for warm function reuse in `lib/db.js`.
- CORS wrapper in `lib/cors.js`.

### Backend option B: Express API (`server/`)

- Conventional Express app in `server/server.js`.
- Controllers in `server/controllers/**`.
- Routes in `server/routes/**`.
- Separate Mongoose models in `server/models/**`.

> Note: The codebase intentionally duplicates model/controller logic between serverless and Express implementations.

---

## Feature modules

### 1) Transactions

- Create income or expense entries.
- Filter by type/category/date range.
- Pagination support.
- Auto-categorization when category is omitted or `Others`.
- Stats endpoint powers dashboard/analytics (monthly totals, category breakdown, daily trend, monthly trend, generated insights).

### 2) Budgeting

- Monthly budget documents keyed by `YYYY-MM`.
- Track total budget + per-category limits.
- Utilization endpoint compares budget vs actual expense aggregates.
- Emits warning/danger alerts at threshold levels.

### 3) Bill Split

- Supports equal split calculation or custom participant amounts.
- Participant settlement updates bill status:
  - `unsettled` → `partially` → `settled`.

### 4) Savings Goals

- Create goals with target amount and deadline.
- Add funds over time.
- Completion auto-detected in model pre-save hook.

### 5) Analytics

- Category distribution.
- Top-category ranking.
- Spending trend views.
- Budget-vs-actual comparison charts.

### 6) Progressive Web App

- `manifest.json` present.
- `sw.js` included.
- Custom install banner via `beforeinstallprompt` event.

---

## Tech stack

- **Frontend**: React, Vite, Axios, Recharts, Framer Motion.
- **Backend**: Node.js, Express (optional runtime), Vercel Functions.
- **Database**: MongoDB + Mongoose.
- **Deployment target**: Vercel (configured via `vercel.json`).

---

## Project structure

```text
personal-finance-manager/
├── api/                    # Vercel serverless API routes
├── lib/                    # Shared serverless utilities + models
├── client/                 # React + Vite frontend
├── server/                 # Standalone Express backend
├── package.json            # Root metadata (Node engine + mongoose dep)
└── vercel.json             # Vercel build + rewrite config
```

---

## API surface (shared behavior)

### Health
- `GET /api/health`

### Transactions
- `GET /api/transactions`
- `POST /api/transactions`
- `PUT /api/transactions/:id`
- `DELETE /api/transactions/:id`
- `GET /api/transactions/stats`
- `GET /api/transactions/suggest-category?description=...`

### Budget
- `GET /api/budget/:month`
- `PUT /api/budget/:month`
- `GET /api/budget/:month/utilization`

### Bills
- `GET /api/bills`
- `POST /api/bills`
- `DELETE /api/bills/:id`
- `POST /api/bills/:id/settle`

### Goals
- `GET /api/goals`
- `POST /api/goals`
- `DELETE /api/goals/:id`
- `POST /api/goals/:id/add-funds`

---

## Local development

You can run the app in either of two ways.

### Option A — Frontend + Express backend

1. Install dependencies:

```bash
npm install
cd client && npm install
cd ../server && npm install
```

2. Create backend environment file:

`server/.env`

```env
MONGO_URI=mongodb+srv://<user>:<pass>@<cluster>/<db>
PORT=5000
NODE_ENV=development
```

3. Start backend:

```bash
cd server
npm run dev
```

4. Start frontend (new terminal):

```bash
cd client
npm run dev
```

5. Open the Vite URL (usually `http://localhost:5173`).

> If using this mode, ensure the frontend resolves `/api` to your Express server (commonly via Vite proxy or setting `VITE_API_URL=http://localhost:5000/api`).

---

### Option B — Vercel-style serverless locally

Use the Vercel CLI so requests hit `/api/**` handlers directly.

1. Add local env vars:

```env
MONGODB_URI=mongodb+srv://<user>:<pass>@<cluster>/<db>
```

2. Run:

```bash
vercel dev
```

This mode best matches production behavior for this repository.

---

## Deployment

### Vercel

The repo is already configured for Vercel:

- Build command: `cd client && npm install && npm run build`
- Output directory: `client/dist`
- Rewrites:
  - `/api/*` → serverless functions
  - everything else → SPA `index.html`

Required environment variable in Vercel project settings:

- `MONGODB_URI`

---

## Important implementation notes

1. **Two backend env var names are used**
   - Serverless functions expect `MONGODB_URI`.
   - Express server expects `MONGO_URI`.

2. **Currency/locale choices are INR-oriented in UI**
   - Formatting uses `en-IN` and `INR` across pages.

3. **No authentication layer**
   - Data is global for whichever database is connected.
   - `createdBy` in bill splits defaults to `Me`.

4. **Server/client validation mismatch to know**
   - Savings goal model requires `deadline`, but the UI labels deadline as optional.
   - In practice, creating a goal without a deadline can fail validation unless backend is adjusted.

5. **Potential local setup friction**
   - The frontend Axios base defaults to `/api`, so direct Vite + Express setups may require explicit API base configuration or proxy.

---

## Suggested improvements

- Add authentication and per-user data isolation.
- Consolidate duplicate backend code (choose one runtime strategy or share a single core package).
- Add automated tests (unit + API integration + UI smoke).
- Add seed scripts and a sample `.env.example` for both runtime modes.
- Fix savings-goal deadline contract (optional in UI vs required in model).
- Add request schema validation consistently across all endpoints.

---

## Scripts quick reference

### Root

- No application runtime scripts are currently defined; root primarily pins engine/dependency metadata.

### Client (`client/package.json`)

- `npm run dev` — start Vite dev server.
- `npm run build` — production build.
- `npm run preview` — serve built frontend locally.

### Server (`server/package.json`)

- `npm run dev` — run Express API with nodemon.
- `npm start` — run Express API with node.

---

## License

No license file is currently present in this repository. Add one if you plan to distribute or open-source the project.
