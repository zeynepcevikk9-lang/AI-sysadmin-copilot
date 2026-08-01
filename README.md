# AI SysAdmin Copilot

Log dosyalarını (ZIP dahil) yükleyip AI destekli analiz, incident report üretimi, PDF/Markdown export ve arama sunan SaaS uygulaması.

Mimari detayları için [docs/architecture.md](docs/architecture.md) dosyasına bakın.

## Gereksinimler

- Node.js 22+
- Python 3.12+
- Docker Desktop (Windows'ta WSL2 backend gerektirir)

## Geliştirme Ortamı

1. `.env.example` dosyasını `.env` olarak kopyalayın ve gerekirse değerleri düzenleyin (özellikle `ANTHROPIC_API_KEY`).
2. Tüm servisleri ayağa kaldırın:

   ```bash
   docker compose up -d
   ```

3. Backend: http://localhost:8000/health — Frontend: http://localhost:3000 — MinIO Console: http://localhost:9001

## Testler

**Backend:**

```bash
cd backend
python -m venv .venv
.venv\Scripts\pip install -r requirements-dev.txt
.venv\Scripts\pytest
.venv\Scripts\ruff check .
```

**Frontend:**

```bash
cd frontend
npm install
npm run test
npm run lint
```

## Proje Durumu

Faz planı ve ilerleme için [docs/architecture.md](docs/architecture.md#faz-planı) dosyasına bakın.
