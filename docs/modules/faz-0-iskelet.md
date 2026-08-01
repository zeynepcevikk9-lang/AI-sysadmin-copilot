# Faz 0: Proje İskeleti

## Kapsam

- Monorepo klasör yapısı (`backend/`, `frontend/`, `docs/`)
- Backend: FastAPI iskeleti (`app/main.py`, `app/core/config.py`, `app/core/db.py`, `app/core/celery_app.py`), `/health` endpoint, Alembic migration altyapısı (henüz model yok)
- Frontend: Next.js (App Router, TypeScript, TailwindCSS), Vitest + Testing Library ile smoke test
- `docker-compose.yml`: postgres, redis, minio, backend, celery_worker, frontend
- CI: `.github/workflows/ci.yml` (backend: ruff + pytest, frontend: lint + test + build)
- `.pre-commit-config.yaml`: backend için ruff (lint + format)

## Doğrulama

- Backend: `pytest` → 1 test geçti (`test_health_returns_ok`), `ruff check .` → temiz.
- Frontend: `npm run test` → 1 test geçti (Home sayfası render), `npm run lint` bekleniyor.
- `alembic current` çalıştırıldığında `env.py` doğru şekilde `app.core.config` ayarlarına bağlanıyor (Postgres henüz ayakta olmadığı için beklenen `ConnectionRefusedError` alındı — bu, altyapının doğru kurulduğunun kanıtı).
- `docker compose up` doğrulaması: Docker Desktop bu makinede WSL2 gerektiriyor, kurulum tamamlanınca yapılacak.

## Bilinen Sorunlar

- `npm audit`: `next`'in iç `postcss`/`sharp` bağımlılığında yüksek önemli, upstream kaynaklı açık. Bkz. [architecture.md](../architecture.md#bilinen-riskler--takip-edilenler).
