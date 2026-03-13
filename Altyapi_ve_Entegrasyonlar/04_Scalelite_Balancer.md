# Bölüm 4: Scalelite ile Yatay Ölçeklendirme ve Yük Dengeleme

BigBlueButton (BBB) doğası gereği "Monolithic" bir yapıya sahiptir; yani tek bir oda (meeting) tek bir sunucunun kaynaklarını kullanabilir. Ancak binlerce eşzamanlı kullanıcıya hizmet vermek için birden fazla BBB sunucusunu bir havuzda toplamak ve yükü dağıtmak gerekir. Bu noktada **Scalelite** devreye girer.

## 1. Scalelite Nedir?

Scalelite, BigBlueButton cluster (küme) yönetimi için açık kaynaklı bir yük dengeleyicidir. Birden fazla "Worker" (İşçi) BBB sunucusunu tek bir API uç noktası arkasında birleştirir.

- **Akıllı Yönlendirme:** Yeni bir toplantı isteği geldiğinde, en az yükü olan (CPU, kullanıcı sayısı veya toplantı sayısına göre) sunucuyu seçer.
- **Şeffaflık:** Önyüz (Moodle, Greenlight) sadece Scalelite ile konuşur. Arkadaki sunuculardan biri kapansa bile sistem çalışmaya devam eder.
- **Kayıt Yönetimi:** BBB sunucularında tamamlanan kayıtları merkezi bir depolama alanına (NFS/S3) aktarır ve oradan sunar.

## 2. Mimari Bileşenler

Scalelite kurulumu genellikle Docker üzerinden yapılır ve şu bileşenlerden oluşur:

| Bileşen | Görevi |
| :--- | :--- |
| **Scalelite API** | Uygulama mantığı, yük dengeleme hesaplamaları. |
| **Scalelite Poller** | BBB sunucularının sağlık durumunu ve yükünü periyodik kontrol eder. |
| **PostgreSQL** | Toplantı metadata ve durum bilgilerini saklar. |
| **Redis** | Pub/Sub ve caching işlemleri için kullanılır. |
| **Nginx** | SSL sonlandırma ve istek yönlendirme. |

## 3. Scalelite Kurulum Adımları (Docker Compose)

Scalelite'ı ayağa kaldırmak için en pratik yöntem resmi Docker Compose yapısını kullanmaktır.

### 3.1 Gerekli Dosyaların Hazırlanması

```bash
mkdir -p /opt/scalelite && cd /opt/scalelite
# Resmi repository'den dosyaları çekebilirsiniz veya elle oluşturabilirsiniz.
```

### 3.2 Örnek `docker-compose.yml` (Özet)

```yaml
services:
  redis:
    image: redis:7-alpine
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: scalelite
      POSTGRES_PASSWORD: GizliSifre
  api:
    image: blindsidenetworks/scalelite:v1.x-api
    env_file: .env
    depends_on: [redis, db]
  poller:
    image: blindsidenetworks/scalelite:v1.x-poller
    env_file: .env
    depends_on: [redis, db]
  nginx:
    image: nginx:alpine
    ports: ["80:80", "443:443"]
    volumes: ["./nginx.conf:/etc/nginx/nginx.conf"]
```

### 3.3 `.env` Dosyası Ayarları

```env
URL_HOST=scalelite.sirket.com
SECRET_KEY_BASE=raSTgeLe_DeGer_uReTiN
# Moodle'a vereceğiniz ana API anahtarı
SCALELITE_TAG=v1
LOADBALANCER_SECRET=ANA_GUCLU_SECRET
POLL_INTERVAL=30
```

## 4. Worker BBB Sunucularının Hazırlanması

Scalelite'a eklenecek her BBB sunucusunda şu adımlar uygulanmalıdır:

1. **Dummy Interface:** NAT arkasındaki sunucular için dummy interface yapılandırması yapılmalıdır.
2. **Post-Scripts:** Kayıtların Scalelite'a aktarılması için `bbb-verne` veya `rsync` tabanlı scriptler kurulmalıdır.

```bash
# Bir BBB sunucusunu Scalelite'a eklemek için URL ve Secret bilgisi:
bbb-conf --secret
```

## 4. Scalelite API Kullanarak Sunucu Ekleme

Scalelite yüklendikten sonra (genellikle `/usr/local/bigbluebutton/scalelite` dizininde), sunucuları şu komutla havuza ekleyebilirsiniz:

```bash
# Scalelite Docker konteyneri içinde:
bundle exec rake servers:add[https://bbb-01.sirket.com/bigbluebutton/api,SECRET_KEY]
```

## 5. Merkezi Kayıt Yönetimi (NFS)

Ölçeklenebilir bir yapıda kayıtlar yerel diskte tutulamaz. Tüm worker sunucular kayıt bittiğinde dosyayı ortak bir NFS dizinine basar.

- **NFS Server:** `/mnt/scalelite-recordings`
- **Moodle Entegrasyonu:** Moodle, kaydı izlemek istediğinde Scalelite üzerinden NFS yoluna erişen bir Nginx alias'ı kullanır.

## 6. Greenlight v3 ile Alternatif Yük Dengeleme

BBB 3.0.x ve Greenlight v3.4+ sürümleriyle birlikte, Scalelite kurmadan "Basit Yük Dengeleme" yapılabilir:

> [!TIP]
> Eğer sadece 2-3 sunucunuz varsa ve kayıt yönetimi karmaşasına girmek istemiyorsanız, Greenlight v3 admin panelinden "Multiple BBB Servers" özelliğini kullanarak sunucu anahtarlarını girmeniz yeterlidir. Greenlight yükü bu sunuculara dağıtabilir.

---

> [!IMPORTANT]
> **Scalelite ve Büyük Odalar:** Scalelite "Tek bir odayı 10 sunucuya bölmez". Sadece odaları farklı sunuculara dağıtır. 500+ kişilik tek bir yayın için hala harici RTMP/Streaming çözümleri (Youtube/Vimeo) entegrasyonu önerilir.
