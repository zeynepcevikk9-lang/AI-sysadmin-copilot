# Windows ve Mac Cihazlarda Bilgisayar Adı (Host İsmi) Değiştirme

Bu doküman, Windows ve macOS cihazlarda bilgisayar/host adının nasıl değiştirileceğini
adım adım anlatır.

## 1. Windows

### 1.1 Ayarlar Üzerinden (Windows 10/11)

1. **Ayarlar** (Settings) → **Sistem** (System) → **Hakkında** (About) yolunu izleyin.
2. **Cihaz adı** (Device name) bölümünde **Yeniden adlandır** (Rename this PC)
   butonuna tıklayın.
3. Yeni adı girin ve **İleri**'ye (Next) tıklayın.
4. Değişikliğin geçerli olması için bilgisayarı **yeniden başlatın**.

### 1.2 Komut Satırı Üzerinden (PowerShell — Yönetici olarak)

```powershell
Rename-Computer -NewName "YeniBilgisayarAdi" -Restart
```

`-Restart` parametresi bilgisayarı otomatik olarak yeniden başlatır; kaldırırsanız
değişikliğin etkili olması için manuel yeniden başlatma gerekir.

### 1.3 Denetim Masası Üzerinden (Alternatif Yöntem)

1. `sysdm.cpl` komutunu **Çalıştır** (Win + R) ile açın.
2. **Bilgisayar Adı** sekmesinde **Değiştir...** (Change...) butonuna tıklayın.
3. Yeni adı girip **Tamam**'a basın, ardından yeniden başlatın.

### 1.4 Kurallar ve Kısıtlamalar

- Ad en fazla 15 karakter olabilir (NetBIOS uyumluluğu için).
- Yalnızca harf, rakam ve `-` (tire) karakteri kullanılabilir; boşluk ve özel
  karakterler (`_ . , ; : ! @ # $ % ^ & * ( ) = + [ ] { } | \ / ? < > "`)
  desteklenmez.
- Ad yalnızca rakamlardan oluşamaz.
- Bir **Active Directory domain'ine bağlı** makinelerde ad değişikliği için domain
  yönetici yetkisi ve genellikle bir sonraki yeniden başlatmada domain
  controller ile senkronizasyon gerekir.

## 2. macOS

macOS'ta aslında birbirinden farklı **üç** ad vardır; hepsini aynı anda değiştirmek
gerekir:

| Ad türü | Nerede kullanılır |
|---|---|
| **Computer Name** | Finder paylaşımlarında, AirDrop'ta görünen ad |
| **Local Hostname (Bonjour)** | `.local` alan adı (ör. `MacBook-Pro.local`) |
| **HostName** | Terminal `hostname` komutunun döndürdüğü, SSH/ağ araçlarının kullandığı ad |

### 2.1 Sistem Ayarları Üzerinden (Computer Name)

1. **Sistem Ayarları** (System Settings) → **Genel** (General) → **Paylaşım**
   (Sharing) yolunu izleyin.
2. En üstteki **Bilgisayar Adı** (Computer Name) alanına yeni adı yazın.
3. Bu değişiklik Local Hostname'i de otomatik günceller (boşlukları `-` ile
   değiştirerek).

### 2.2 Terminal Üzerinden (Her Üç Adı da Tam Kontrol Etmek İçin)

```bash
sudo scutil --set ComputerName "YeniBilgisayarAdi"
sudo scutil --set LocalHostName "YeniBilgisayarAdi"
sudo scutil --set HostName "YeniBilgisayarAdi"
```

`LocalHostName` yalnızca harf, rakam ve `-` içerebilir (boşluk ve Türkçe karakter
kullanmayın). Değişiklikten sonra doğrulamak için:

```bash
scutil --get ComputerName
scutil --get LocalHostName
scutil --get HostName
hostname
```

### 2.3 mDNSResponder'ı Yeniden Başlatma (Ağda Hemen Görünmesi İçin)

Ad değişikliği bazen ağdaki diğer cihazlara (Bonjour/AirDrop) hemen yansımaz;
DNS önbelleğini/servisini yeniden başlatın:

```bash
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

## 3. Doğrulama Kontrol Listesi

- [ ] Windows: `hostname` komutu (CMD/PowerShell) yeni adı gösteriyor
- [ ] Windows: Ayarlar → Sistem → Hakkında'da yeni ad görünüyor
- [ ] Mac: `scutil --get ComputerName`, `LocalHostName`, `HostName` üçü de tutarlı
- [ ] Mac: Terminal `hostname` komutu yeni adı gösteriyor
- [ ] Ağdaki diğer cihazlardan (AirDrop, dosya paylaşımı, SSH) yeni ad ile erişim
      test edildi
- [ ] Yeniden başlatma sonrası ad kalıcı olarak korunuyor

## 4. Sık Karşılaşılan Sorunlar

| Belirti | Neden | Çözüm |
|---|---|---|
| Windows'ta ad değişikliği kabul edilmiyor | Domain politikası ad değişikliğini kısıtlıyor | Domain yöneticisinden yetki isteyin |
| Mac'te eski ad hâlâ ağda görünüyor | mDNSResponder önbelleği | Bölüm 2.3'teki komutları çalıştırın |
| Mac'te `LocalHostName` ayarlanamıyor | Adda boşluk/Türkçe karakter var | Yalnızca harf, rakam, `-` kullanın |
| Windows'ta 15 karakter sınırı aşılıyor | NetBIOS kısıtlaması | Adı 15 karakter veya altına indirin |
