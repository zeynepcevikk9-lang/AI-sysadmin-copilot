# Windows ve Mac Cihazlarda Hosting Kurulumu / Değişimi

Bu doküman, AI SysAdmin Copilot uygulamasının Windows ve macOS cihazlarda (geliştirme veya
küçük ölçekli self-host senaryosu için) nasıl ayağa kaldırılacağını, bir işletim sisteminden
diğerine geçerken nelerin değişmesi gerektiğini ve platforma özgü sorunların nasıl
çözüleceğini anlatır. Genel mimari için [architecture.md](architecture.md) dosyasına bakın.

## 1. Ortak Gereksinimler

Her iki platform için de aşağıdakiler gereklidir:

- **Docker Desktop** (son sürüm)
- **Git**
- **Node.js 22+** (yalnızca Docker dışında yerel geliştirme yapılacaksa)
- **Python 3.12+** (yalnızca Docker dışında yerel geliştirme yapılacaksa)
- Boşta olması gereken portlar: `3000` (frontend), `8000` (backend), `5432` (postgres),
  `6379` (redis), `9000`/`9001` (MinIO)

Uygulamanın tamamı `docker-compose.yml` üzerinden ayağa kalktığı için platformlar arası
geçişte kod veya konfigürasyonda **değişiklik gerekmez**; farklılıklar yalnızca Docker
Desktop'ın alt yapısında (WSL2 vs. Apple Silicon/Intel sanallaştırma) ve dosya
yollarında ortaya çıkar.

## 2. Windows Kurulumu

### 2.1 Docker Desktop + WSL2

1. WSL2'yi etkinleştirin:
   ```powershell
   wsl --install
   ```
   Kurulum sonrası bilgisayarı yeniden başlatın.
2. [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/) kurulumunu
   yapın. Kurulum sırasında **"Use WSL 2 instead of Hyper-V"** seçeneğinin işaretli
   olduğundan emin olun.
3. Docker Desktop → **Settings → Resources → WSL Integration** altından kullandığınız
   dağıtımı (ör. `Ubuntu`) etkinleştirin.
4. Projeyi WSL2 dosya sistemi içine klonlamanız önerilir (ör. `\\wsl$\Ubuntu\home\<kullanici>\ai-sysadmin-copilot`)
   veya doğrudan bir WSL terminalinden çalışın; Windows tarafındaki `C:\` sürücüsü
   üzerinde çalışmak dosya izleme (hot-reload) performansını ciddi biçimde düşürür.

### 2.2 Projeyi Ayağa Kaldırma (WSL2 terminalinde)

```bash
git clone <repo-url> ai-sysadmin-copilot
cd ai-sysadmin-copilot
cp .env.example .env
# gerekirse .env içindeki ANTHROPIC_API_KEY vb. değerleri düzenleyin
docker compose up -d
```

Servisler:

- Backend: http://localhost:8000/health
- Frontend: http://localhost:3000
- MinIO Console: http://localhost:9001

### 2.3 Windows'a Özgü Notlar

- `docker compose` komutlarını PowerShell/CMD üzerinden değil, WSL2 terminalinden
  çalıştırmak dosya izleme ve I/O performansı açısından önerilir.
- Satır sonu (CRLF/LF) sorunlarını önlemek için `git config --global core.autocrlf input`
  ayarını yapın.
- Windows Defender / kurumsal antivirüs yazılımları Docker'ın kullandığı klasörleri
  (`%APPDATA%\Docker`, WSL disk dosyası) taramaya alıyorsa performans düşer; bu
  klasörleri istisna listesine ekleyin.
- Backend testlerini Docker dışında Windows'ta çalıştırmak isterseniz README'deki
  komutlar zaten Windows sözdizimine göredir (`.venv\Scripts\...`); Mac/Linux'ta
  bunun yerine `.venv/bin/...` kullanılır (bkz. bölüm 4).

## 3. macOS Kurulumu

### 3.1 Docker Desktop

1. [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/) indirin —
   Apple Silicon (M1/M2/M3/M4) için **Apple Chip**, Intel Mac için **Intel Chip**
   sürümünü seçin.
2. Kurulumdan sonra Docker Desktop → **Settings → Resources** altından en az 4 GB RAM
   ve 2 CPU ayırın (varsayılan sistemler için genelde yeterlidir, log analizi
   yoğun kullanımda artırılabilir).
3. `docker --version` ve `docker compose version` ile kurulumu doğrulayın.

### 3.2 Projeyi Ayağa Kaldırma

```bash
git clone <repo-url> ai-sysadmin-copilot
cd ai-sysadmin-copilot
cp .env.example .env
docker compose up -d
```

Servisler Windows'takiyle aynı adreslerden erişilebilir (bkz. 2.2).

### 3.3 macOS'a Özgü Notlar

- **Apple Silicon:** `postgres:17-alpine`, `redis:7-alpine` ve `minio/minio:latest`
  imajları `arm64` desteklidir; ek bir işlem gerekmez. Backend/frontend imajları
  bu repodaki `Dockerfile`'lardan derlendiği için platform sorunu yaşanmaz.
- Dosya paylaşımı performansı için Docker Desktop'ta **VirtioFS** (Settings →
  General → "Choose file sharing implementation") seçili olmalı; bu, `./backend`
  ve `./frontend` bind mount'larının hot-reload hızını doğrudan etkiler.
- Port `5000`/`7000` macOS'ta AirPlay Receiver tarafından kullanılabildiği için
  çakışma yaşarsanız System Settings → General → AirDrop & Handoff → AirPlay
  Receiver'ı kapatın (bu projede varsayılan portlar 3000/8000/5432/6379/9000/9001
  olduğundan genelde etkilenmez, ama özel port değişikliklerinde dikkat edin).

## 4. Docker Olmadan Yerel Geliştirme (Her İki Platform)

Docker Desktop kullanmak istemiyorsanız backend ve frontend'i doğrudan çalıştırabilirsiniz;
bu durumda Postgres/Redis/MinIO'yu da yerel olarak kurmanız (veya sadece bu üç servisi
`docker compose up -d postgres redis minio` ile ayağa kaldırmanız) gerekir.

**Backend:**

```bash
cd backend
python -m venv .venv
```

| Adım | Windows | macOS |
|---|---|---|
| Sanal ortamı etkinleştir | `.venv\Scripts\activate` | `source .venv/bin/activate` |
| Bağımlılıkları kur | `.venv\Scripts\pip install -r requirements-dev.txt` | `.venv/bin/pip install -r requirements-dev.txt` |
| Testleri çalıştır | `.venv\Scripts\pytest` | `.venv/bin/pytest` |
| Lint | `.venv\Scripts\ruff check .` | `.venv/bin/ruff check .` |

**Frontend (her iki platformda aynı):**

```bash
cd frontend
npm install
npm run test
npm run lint
```

## 5. Bir Platformdan Diğerine Geçiş (ör. Windows → Mac veya tersi)

1. `.env` dosyasını **taşımayın** — içinde geliştirme sırlarınız (JWT secret, API key)
   varsa yeni makinede `.env.example`'dan yeniden oluşturup değerleri elle girin.
2. Docker volume'ları (`postgres_data`, `minio_data`) platforma bağlı değildir ama
   varsayılan olarak Docker Desktop'ın kendi VM'i içinde tutulur; verileri taşımak
   isterseniz:
   ```bash
   docker compose down
   docker run --rm -v ai-sysadmin-copilot_postgres_data:/data -v $(pwd):/backup alpine \
     tar czf /backup/postgres_data.tar.gz -C /data .
   ```
   ve yeni makinede aynı volume adıyla `tar xzf` ile geri yükleyin.
3. Kod tarafında platforma özel hiçbir ayar yoktur; `docker compose up -d` yeni
   makinede sıfırdan da çalışır (geliştirme ortamı için genelde veri taşımaya
   gerek yoktur, `docker compose up -d` yeterlidir).

## 6. Sorun Giderme

| Belirti | Olası Neden | Çözüm |
|---|---|---|
| `docker compose up` "Cannot connect to the Docker daemon" hatası verir | Docker Desktop çalışmıyor | Docker Desktop'ı başlatın, sistem tepsisinden durumu kontrol edin |
| Windows'ta çok yavaş hot-reload | Proje `C:\` üzerinde, WSL2 dışında çalıştırılıyor | Projeyi WSL2 dosya sistemine taşıyın (bkz. 2.1) |
| `port is already allocated` hatası | Portu başka bir servis kullanıyor | `docker compose down`, çakışan servisi durdurun veya `docker-compose.yml`'de portu değiştirin |
| Backend Postgres'e bağlanamıyor | `postgres` servisi henüz `healthy` değil | `docker compose ps` ile durumu kontrol edin, birkaç saniye bekleyin |
| Apple Silicon'da imaj çekme hatası | Nadir durumda `arm64` imajı yok | `docker compose build --pull` ile yerelden derleyin |
| Mac'te "AirPlay" nedeniyle port çakışması | AirPlay Receiver aynı portu dinliyor | AirPlay Receiver'ı kapatın veya portu değiştirin |

## 7. Doğrulama Kontrol Listesi

- [ ] `docker compose ps` → tüm servisler `healthy`/`running`
- [ ] http://localhost:8000/health → `200 OK`
- [ ] http://localhost:3000 → frontend açılıyor
- [ ] http://localhost:9001 → MinIO Console giriş ekranı açılıyor
- [ ] `docker compose logs backend --tail 50` içinde hata yok
