# Bölüm 14: BigBlueButton 3.0.x (2026 Standartları) — Yenilikler ve SysAdmin Notları

Mart 2026 itibarıyla **BigBlueButton 3.0.22** kararlı (stable) sürümdür. Greenlight varsayılan ön yüzü ise **3.5** sürümüne ulaşmıştır. 3.x serisi, eğitim sektörünün de-facto standardı haline gelmiş; 2.x sürümleri artık tamamen terk edilmiştir.

> [!IMPORTANT]
> BBB 3.0.x yalnızca **Ubuntu 22.04 LTS** üzerinde resmi olarak desteklenmektedir. Ubuntu 24.04 desteği BBB 4.0 ile planlanmaktadır (2027 hedefi).

## 14.1 3.0 Serisinin Temel Mimari Değişimleri

| Değişiklik | BBB 2.x | BBB 3.x |
| :--- | :--- | :--- |
| Medya Sunucusu | Kurento + Mediasoup | **Sadece Mediasoup** |
| Beyaz Tahta | Eski bitmap tabanlı | **TLDraw** (vektörel) |
| Eklenti Sistemi | Yok | **Plugin Architecture** (3.0) |
| Greenlight | v2 (Rails) | **v3 (React + PostgreSQL)** |
| MongoDB Rolü | Gerekli | Hâlâ gerekli (aktif toplantı durumu) |

## 14.2 Yeni TLDraw Beyaz Tahta

BBB 3.0 ile birlikte modern **TLDraw** altyapısına geçilmiştir:

- **Vektörel Çizim:** Nesneler seçilebilir, taşınabilir, gruplanabilir
- **Gelişmiş Araçlar:** Yapışkan notlar, akıllı oklar, pürüzsüz serbest çizim
- **Daha Az Bant Genişliği:** Olaylar NodeJS ve Redis üzerinden optimize edilmiş paketlerle iletilir

## 14.3 Plugin Architecture (BBB 3.0 Yeniliği)

BBB 3.0'ın öne çıkan özelliği: **sunucu tarafında değişiklik yapmadan** yeni özellikler eklenebilen bir eklenti mimarisi.

```bash
# Mevcut yüklü eklentileri görüntüle
ls /etc/bigbluebutton/bbb-html5-with-plugin-support/

# Resmi plugin deposu:
# https://github.com/bigbluebutton/bigbluebutton/tree/v3.0.x-release/bbb-plugin-sdk
```

## 14.4 Greenlight 3.5 Yenilikleri

```bash
# Greenlight Docker image sürümünü kontrol et
cd ~/greenlight-v3

# Yöntem 1: Docker image tag'inden sürümü oku
docker compose images greenlight

# Yöntem 2: Konteyner içinden Rails ortam bilgisini oku
docker compose exec greenlight bundle exec rake greenlight:version 2>/dev/null || \
  docker inspect greenlight-v3-greenlight-1 --format '{{ index .Config.Labels "org.opencontainers.image.version" }}'

# Greenlight 3.5'e güncellemek için
docker compose pull
docker compose up -d
```

Greenlight 3.5 ile gelen yenilikler:

- Basitleştirilmiş arayüz (daha az adımda oda oluşturma)
- Gelişmiş oda ayarları (katılımcı limiti, bekleme odası zorunluluğu)
- OIDC (OpenID Connect) desteği geliştirildi

## 14.5 Kamera ve Sanal Arka Plan

- **Sanal Arka Plan:** Kamera flulaştırma ve özel arka plan ekleme **istemci tarafında** (Client-Side) işlenir — sunucuya ekstra CPU yükü binmez
- **SysAdmin Notu:** Kurumsal arka plan görsellerini `/var/www/bigbluebutton-default/backgrounds/` dizinine ekleyerek kullanıcı panelinde otomatik sunabilirsiniz

## 14.6 Moderatör Araçları (3.0 Geliştirmeleri)

- **Dikkat Takibi:** BBB sekmesinden uzaklaşan kullanıcılar moderatöre görünür
- **Toplu Yönetim:** Breakout odalarından ana odaya tek komutla geri çekme
- **Hard-Mute:** Tüm mikrofonları anında kapatma
- **API Tepki Süreleri:** Milisaniye seviyesine indirilmiştir

## 14.7 Ubuntu 22.04 Standartları

```bash
# BBB 3.0 için sistem kontrolü
lsb_release -a              # Ubuntu 22.04 (Jammy) olmalı
bbb-conf --version          # 3.0.22 veya üstü
bbb-conf --status           # Tüm servisler active/running olmalı
```

---

> [!TIP]
> **2026 Yol Haritası:** BBB 4.0 Kubernetes (K8S) desteği, Ubuntu 24.04 uyumluluğu ve App Gallery (eklenti mağazası) ile planlanan sonraki büyük sürümdür. Production sistemleri için 3.0.x stabilite sürümünde kalmaya devam etmeniz önerilir.
