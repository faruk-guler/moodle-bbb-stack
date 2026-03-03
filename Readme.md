# 🎓 Moodle & BigBlueButton — Kurumsal BT Altyapı ve Mimarisi 2026

<p align="center">
  <img src="./bbb-moodle-logo.png" alt="BBB + Moodle" width="40%">
</p>

<p align="center">
  <strong>Moodle 5.1.3</strong> · <strong>BigBlueButton 3.0.22</strong> · <strong>Greenlight 3.5</strong><br>
  Mart 2026 · Türkçe · Açık Kaynak
</p>

```console
# Yazar: faruk-guler 2026
# Page: www.farukguler.com
```

---

## 📌 Bu Proje Nedir?

Bu depo, **Moodle** (LMS) ve **BigBlueButton** (Video Konferans) platformlarını kurumsal seviyede kurmak, yönetmek ve ölçeklendirmek isteyen **sistem yöneticileri**, **DevOps mühendisleri** ve **sunucu yöneticileri** için hazırlanmış eksiksiz bir Türkçe teknik dokümantasyon setidir.

> Geleneksel "next-next" kurulum kılavuzlarından farklı olarak, bu rehber gerçek dünya senaryolarına (sınav haftası darboğazları, NAT arkası sorunları, SSL kriz yönetimi) odaklanır.

---

## 📂 Proje Yapısı

```
Antigravity/
├── Moodle/                    ← 15 bölüm — Moodle 5.1.x SysAdmin Rehberi
│   ├── 01_Moodle_Mimari_ve_Bilesenler.md
│   ├── 02_Sunucu_Kurulumu_ve_Gereksinimler.md
│   ├── 03_Veritabani_ve_Caching_Redis.md
│   ├── 04_Dosya_Yonetimi_ve_Moodledata.md
│   ├── 05_Cron_ve_Ad_hoc_Task_Yonetimi.md
│   ├── 06_Guvenlik_ve_Hardening.md
│   ├── 07_Yedekleme_ve_Surum_Yukseltme.md
│   ├── 08_Tema_ve_Arayuz_Ozellestirme.md
│   ├── 09_Eklenti_ve_LTI_Entegrasyonlari.md
│   ├── 10_Performans_ve_Olceklendirme.md
│   ├── 11_Mobil_Uygulama_ve_Bildirimler.md
│   ├── 12_Analitik_ve_Raporlama.md
│   ├── 13_BigBlueButton_Moodle_Entegrasyonu.md
│   ├── 14_Sik_Karsilasilan_Hatalar_Debug.md
│   ├── 15_Moodle_4_5_ve_Gelecek_Vizyonu.md
│   └── README.md
│
├── BigBlueButton/             ← 16 bölüm — BBB v3.0.x SysAdmin Rehberi
│   ├── 01_BBB_Mimari_ve_Bilesenler.md
│   ├── 02_Sistem_Gereksinimleri_ve_Kurulum.md
│   ├── 03_Ag_Guvenlik_Duvari_ve_TURN_STUN.md
│   ├── 04_Yonetim_ve_CLI_Araclari.md
│   ├── 05_Toplanti_ve_Oda_Ayarlari.md
│   ├── 06_Kayit_Yonetimi_Recording.md
│   ├── 07_Ileri_Duzey_Ozellestirme_ve_Branding.md
│   ├── 08_Performans_ve_Olceklendirme_Scalelite.md
│   ├── 09_Izleme_Loglama_ve_SorunGiderme.md
│   ├── 10_Guncelleme_Yedekleme_ve_Guvenlik.md
│   ├── 11_API_Kullanimi_ve_Dis_Entegrasyonlar.md
│   ├── 12_Greenlight_v3_Ileri_Yonetim.md
│   ├── 13_SIP_Trunk_ve_Dial_In_Entegrasyonu.md
│   ├── 14_BBB_3_0_Yenilikleri_ve_Kullanimi.md
│   ├── 15_Sik_Karsilasilan_Hatalar_ve_Kriz_Yonetimi.md
│   ├── 16_Ogrenme_Analitikleri_ve_Derin_Arayuz_Ayarlari.md
│   └── README.md
│
├── Kimlik_Hizmetleri_ve_SSO.md  ← Active Directory (LDAP), SAML2 (Azure/SSO) entegrasyonu
├── Eposta_Altyapisi_ve_DMARC.md ← Moodle/BBB SMTP ayarları, SPF/DKIM ve Spam koruması
├── Merkezi_Log_Yonetimi.md      ← Loki/Grafana & ELK Stack ile merkezi loglama
└── Readme.md                    ← Bu dosya
```

---

## 🟢 Moodle Rehberi — Öne Çıkanlar

| Konu | İçerik |
| :--- | :--- |
| **Mimari** | Moodle 5.x `/public` dizin yapısı, istek akışı, config.php parametreleri |
| **Kurulum** | PHP 8.3/8.4, PostgreSQL 16, Nginx vhost, PHP-FPM havuz yapılandırması |
| **Caching** | Redis Session/MUC, OPcache, PostgreSQL performance tuning |
| **Güvenlik** | UFW, HSTS, SSL/TLS, config.php hardening, Security Checks tablosu |
| **Yedekleme** | Otomatik bash betiği, Git tabanlı sürüm yükseltme, rollback stratejisi |
| **Ölçeklendirme** | Nginx upstream LB, NFS shared storage, PostgreSQL read replicas |
| **Entegrasyon** | BBB API bağlantısı, LTI 1.3, Webhook, mobil uygulama, Learning Analytics |

> **Kritik Not:** Moodle 5.0 ile gelen `/public` dizin mimarisi, web sunucusu root'unun değişmesini zorunlu kılmıştır. Tüm bölümlerdeki komutlar bu yeni yapıya göre güncellenmiştir.

---

## 🔵 BigBlueButton Rehberi — Öne Çıkanlar

| Konu | İçerik |
| :--- | :--- |
| **Mimari** | Nginx, bbb-web, bbb-html5, Redis, MongoDB, FreeSWITCH, Mediasoup |
| **Kurulum** | `bbb-install.sh` (v3.0.x-release), Ubuntu 22.04, Let's Encrypt SSL |
| **Ağ** | NAT arkası dummy interface, UFW port yönetimi, Coturn TURN/STUN |
| **Kayıt** | Recording pipeline (Archive → Sanity → Process → Publish) |
| **Ölçeklendirme** | Scalelite cluster, NFS kayıt paylaşımı, multi-node yapı |
| **Entegrasyon** | REST API checksum, Webhooks, Greenlight v3, SIP Trunk, LMS bağlantıları |
| **3.0 Yenilikleri** | TLDraw beyaz tahta, Plugin Architecture, sanal arka plan, Greenlight 3.5 |

> **Kritik Not:** BBB 3.0.x yalnızca Ubuntu 22.04 LTS üzerinde resmi destek sunar. Ubuntu 24.04 desteği BBB 4.0 ile planlanmaktadır.

---

## 🛠️ Teknoloji Yığını

| Katman | Moodle | BigBlueButton |
| :--- | :--- | :--- |
| **İşletim Sistemi** | Ubuntu 22.04 / 24.04 LTS | Ubuntu 22.04 LTS |
| **Web Sunucusu** | Nginx | Nginx |
| **Uygulama** | PHP 8.3 (FPM) | Java (Grails) + NodeJS |
| **Veritabanı** | PostgreSQL 16 | PostgreSQL + MongoDB |
| **Cache / MQ** | Redis 7.x | Redis (Pub/Sub) |
| **Medya** | — | FreeSWITCH + Mediasoup |
| **SSL/TLS** | Let's Encrypt | Let's Encrypt |

---

## 🎯 Kimler İçin?

1. **Sistem Yöneticileri:** Kurumsal LMS ve video konferans altyapısını sıfırdan kurmak isteyenler
2. **DevOps Mühendisleri:** Mevcut altyapıyı optimize etmek, ölçeklendirmek ve izlemek isteyenler
3. **Eğitim Kurumları:** Üniversite, okul veya eğitim şirketlerinde uzaktan eğitim platformu yönetenler
4. **Sorun Çözücüler:** "Beyaz ekran", WebRTC 1007/1020 hataları, 502 Gateway gibi kriz anlarında çözüm arayanlar

---

## 📋 Hızlı Başlangıç Komutları

### Moodle Durumu
```bash
# Moodle sürümünü kontrol et
cat /var/www/moodle/public/version.php | grep "\$release"

# Cache temizle
sudo -u www-data php8.3 /var/www/moodle/public/admin/cli/purge_caches.php

# Cron durumunu kontrol et
sudo journalctl -u moodle-cron.service --since "1 hour ago"
```

### BigBlueButton Durumu
```bash
# BBB sürüm ve servis durumu
sudo bbb-conf --version
sudo bbb-conf --status
sudo bbb-conf --check

# API bilgileri (Moodle entegrasyonu için)
sudo bbb-conf --secret
```

---

## 📄 Lisans

Bu dokümantasyon seti açık kaynak olarak paylaşılmaktadır. Eğitim amaçlı kullanabilir, dağıtabilir ve katkıda bulunabilirsiniz.

---

<p align="center">
  <sub>⭐ Bu projeyi beğendiyseniz yıldız bırakmayı unutmayın <3 faruk-guler</sub>
</p>
