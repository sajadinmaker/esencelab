# EsenceLab — Product Documentation

> User-serving product guide (technical deep docs already in `docs/`).
> Codebase: `/home/sajad/Projects/portfolio/esencelab` | Stack: Next.js + Express + FastAPI AI + Supabase PG | Deploy: Vercel + Render

## 1. What this product is

Hiring loop: resume in → structured skills out → job match + recruiter shortlist. Three deployables, one meaningful AI workflow (`/ai/parse-resume`, `/ai/match` with deterministic overlap×0.55 + TF-IDF×0.45 first, Groq `llama-3.3-70b` optional with local fallback). Secondary LMS/coach/admin features de-emphasized.

**Who it's for:** students mapping resumes to skill gaps; recruiters screening + shortlisting; admins on one control surface.

## 2. How users use it

- Student: upload PDF (4MB) → structured parse → skill gaps → apply to Jobs → track Applications
- Recruiter: request access → admin approves → temp password → login → Jobs CRUD → Applications + match explanations → shortlist
- Admin: gated recruiter review, user/job oversight
- API: Express (JWT + RBAC + rate limits + request IDs) → Supabase PG; sync `fetch` to AI service (`x-internal-service-token`, 8–12s timeout, no queue yet)

## 3. Serve it

```bash
# Windows: .\run-frontend-3100.ps1, .\run-backend-3101.ps1, .\run-ai-3102.ps1
# Linux:
cd frontend && npm ci && npm run dev      # :3100
cd backend && npm ci && npm run dev       # :3101 (JWT_SECRET, SUPABASE_*, AI_SERVICE_URL, AI_INTERNAL_AUTH_TOKEN)
cd ai-service && pip install -r requirements.txt && uvicorn app.main:app --port 3102
# tests:
cd backend && npm run test:all            # RBAC smoke+stress SUPABASE_MOCK=1
cd ../ai-service && python -m unittest tests.smoke_test
node scripts/verify-supabase-schema.js
```

Deploy: `render.yaml` (backend + AI on Render free, frontend Vercel); per-service `Dockerfile`s in `frontend/`, `backend/`, `ai-service/`.

## 4. Gaps → roadmap (from repo)

Split `backend/src/index.ts` ~6K monolith → routes/services/stores (plan in `ARCHITECTURE.md`); split AI `app/main.py` similarly; queue for AI calls; external APM (now in-memory metrics); harden RLS (now permissive `USING(true)`, auth in Express) or document managed-PG choice; expand tests beyond RBAC/smoke; per-tenant quotas; measured benchmarks (now thresholds only: slow 1200ms, p95 alert 2500ms).
