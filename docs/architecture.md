# Mimari

## Genel Bakış

AI SysAdmin Copilot, kullanıcıların log dosyalarını (ZIP dahil) yükleyip AI destekli analiz, incident report üretimi ve export alabildiği bir SaaS uygulamasıdır.

```
Next.js (frontend) --> FastAPI (backend, JWT auth) --> PostgreSQL
                              |
                    Redis (broker/cache) --- MinIO (dosya depolama)
                              |
                     Celery workers --> Anthropic Claude API
```

Log analizi ve ZIP extraction gibi uzun süren işler Celery arkaplan görevlerine alınır; senkron API içinde yapılmaz.

## Monorepo Yapısı

- `backend/` — FastAPI uygulaması. `app/core/` altyapı kodunu (config, db, celery), `app/modules/<isim>/` her domain modülünü (`router.py`, `schemas.py`, `service.py`, `models.py`, kendi `tests/`) barındırır.
- `frontend/` — Next.js (App Router, TypeScript, TailwindCSS).
- `docs/` — mimari doküman, ADR'ler, modül dokümantasyonu.
- `docker-compose.yml` — geliştirme ortamı: postgres, redis, minio, backend, celery_worker, frontend.

## Veri Modeli (multi-tenant baştan)

Bkz. [ADR 0001](adr/0001-multi-tenant-from-day-one.md). Özet: `organizations`, `users`, `memberships` çekirdek tablolar; diğer tüm domain tabloları `organization_id` taşır. Her yeni kullanıcı kayıt olduğunda otomatik bir "default organization" oluşturulur.

## Faz Planı

Uygulama 11 fazda (Faz 0–10) inşa edilir; her faz bağımsız bir milestone'dur ve kendi testleriyle teslim edilir. Güncel durum için proje panosuna (task list) bakın. Gelecek modüller (Docker/K8s/Windows Event Logs/AD/VMware/Security Audit/AI Chat/Knowledge Base) MVP tamamlandıktan sonra ayrı fazlar olarak planlanır; `analysis` modülündeki `BaseAnalyzer` arayüzü bu genişlemeye baştan izin verecek şekilde tasarlanmıştır (bkz. Faz 4 dokümantasyonu).

## Bilinen Riskler / Takip Edilenler

- **npm audit (yüksek önem):** `next@16.2.12`'nin kendi iç derleme bağımlılığı olan `postcss`/`sharp`, upstream'de bilinen bir güvenlik açığı taşıyor. Next zaten en güncel sürümde olduğundan (`npm audit fix --force` önerisi `next@9.3.3`'e düşürmeyi öneriyor, bu kabul edilemez bir regresyon), düzeltme Next.js ekibinden bir patch yayınlanmasını bekliyor. Her fazda `npm audit` tekrar kontrol edilmeli.
