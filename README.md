# TaleForge

Full-stack interactive storytelling app:
- Frontend: React + Tailwind (Vite)
- Backend: Django REST Framework + JWT
- Database: PostgreSQL
- Optional: OpenAI-powered dynamic node generation

## Monorepo Layout

```text
backend/   # Django API
frontend/  # React app
```

---

## Local Development

### 1) Backend

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Create local database, then run:

```bash
python manage.py migrate
python manage.py seed_example_stories
python manage.py runserver
```

Backend runs at `http://127.0.0.1:8000`.

### 2) Frontend

```bash
cd frontend
npm install
cp .env.example .env
npm run dev
```

Frontend runs at `http://localhost:5173`.

---

## Railway Deployment (GitHub)

This repo is ready for Railway as **2 services** in one project:
1. `backend` service (Django API)
2. `frontend` service (Vite static preview)

Add PostgreSQL in the same Railway project and link it to backend.

### Backend service (Railway)

- Create service from GitHub repo
- Set **Root Directory**: `backend`
- Build command (auto-detected is fine):
  - `pip install -r requirements.txt`
- Start command:
  - `bash start.sh`

`start.sh` already does:
- `python manage.py migrate --noinput`
- `python manage.py collectstatic --noinput`
- starts Gunicorn on `$PORT`

Set backend env vars:

```env
DEBUG=false
DJANGO_SECRET_KEY=<strong-random-secret>
ALLOWED_HOSTS=.railway.app,.up.railway.app
CORS_ALLOWED_ORIGINS=https://<your-frontend-domain>
CSRF_TRUSTED_ORIGINS=https://<your-frontend-domain>
FRONTEND_URL=https://<your-frontend-domain>
SECURE_SSL_REDIRECT=true
SECURE_HSTS_SECONDS=3600
DB_SSL_REQUIRE=true
```

### Frontend service (Railway)

- Create second service from same GitHub repo
- Set **Root Directory**: `frontend`
- Build command:
  - `npm ci && npm run build`
- Start command:
  - `npm run start`

Set frontend env var:

```env
VITE_API_BASE_URL=https://<your-backend-domain>/api
```

---

## Railway PostgreSQL Notes

If backend is linked to Railway PostgreSQL service, Railway usually injects `DATABASE_URL` automatically.
That is the best setup.

If it is not auto-populated in backend env vars, add this variable reference manually:

```env
DATABASE_URL=${{Postgres.DATABASE_URL}}
```

If you only have **database name + password**, that is not enough by itself to connect externally.
You also need:
- host
- port
- username

Get these from Railway Postgres service -> **Connect** tab.
Then either:
1. set full `DATABASE_URL`, or
2. set `DB_NAME`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`, `DB_USER`.

The backend supports both.

---

## Useful URLs

- API docs: `https://<your-backend-domain>/api/docs/`
- OpenAPI schema: `https://<your-backend-domain>/api/schema/`

---

## Optional OpenAI Setup

Backend env vars:

```env
OPENAI_API_KEY=<your-key>
OPENAI_MODEL=gpt-4.1-mini
```

If missing, AI-choice flows automatically fall back to static next nodes.
