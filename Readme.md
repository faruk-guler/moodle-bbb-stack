# 🎓 Moodle & BigBlueButton — Kurumsal BT Altyapı ve Mimari 2026

<p align="center">
  <img src="./img/bbb-moodle-logo.png" alt="BBB + Moodle" width="40%">
</p>

<p align="center">
  <strong>Moodle 5.1.3</strong> · <strong>BigBlueButton 3.0.22</strong> · <strong>Greenlight 3.5</strong> · <strong>PILOS 4.x</strong><br>
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

```text
├── Moodle/                    ← 15 bölüm — Moodle 5.1.x SysAdmin Rehberi
│   ├── 01_Moodle_Mimari_ve_Bilesenler.md
│   ├── ...
│   └── README.md
│
├── BigBlueButton/             ← 17 bölüm — BBB v3.0.x SysAdmin Rehberi
│   ├── 01_BBB_Mimari_ve_Bilesenler.md
│   ├── ...
│   └── README.md
│
├── Altyapi_ve_Entegrasyonlar/ ← 4 bölüm — Ortak Entegrasyon Rehberi
│   ├── 01_Kimlik_Hizmetleri_ve_SSO.md
│   ├── 02_Eposta_Altyapisi_ve_DMARC.md
│   ├── 03_Merkezi_Log_Yonetimi.md
│   └── 04_Scalelite_Balancer.md
├── Certs/                       ← Self-Signed SSL Sertifika Referans Örnekleri
└── Readme.md                    ← Bu dosya
```

---

## 🟢 Moodle Rehberi — Öne Çıkanlar

| Konu | İçerik |
| :--- | :--- |
| **Mimari** | Moodle 5.x `/public` dizin yapısı, istek akışı, config.php parametreleri |
| **Kurulum** | PHP 8.3/8.4, PostgreSQL 16, Nginx vhost, PHP-FPM havuz yapılandırması |
| **Caching** | Redis Session/MUC, OPcache, PostgreSQL performance tuning |
| **Güvenlik** | UFW, HSTS, SSL/TLS, config.php hardening, SAML2 (Microsoft Entra ID) SSO |
| **Yedekleme** | Otomatik bash betiği, Git tabanlı sürüm yükseltme, rollback stratejisi |
| **Ölçeklendirme** | Nginx upstream LB, NFS shared storage, PostgreSQL read replicas |
| **Entegrasyon** | BBB API bağlantısı, LTI 1.3 Advantage, Webhook, Mobil uygulama (Push) |

> **Kritik Not:** Moodle 5.0 ile gelen `/public` dizin mimarisi, web sunucusu root'unun değişmesini zorunlu kılmıştır. Tüm bölümlerdeki komutlar bu yeni yapıya göre güncellenmiştir.

---

## 🔵 BigBlueButton Rehberi — Öne Çıkanlar

| Konu | İçerik |
| :--- | :--- |
| **Mimari** | Sadece Mediasoup (Kurento yok), bbb-web, bbb-html5, Redis, FreeSWITCH |
| **Kurulum** | `bbb-install.sh` (v3.0.x-release), Ubuntu 22.04 LTS zorunluluğu, Let's Encrypt SSL |
| **Ağ ve Güvenlik** | Dummy interface (NAT arkası), UFW port yönetimi, Ayrı Coturn TURN/STUN Sunucusu |
| **Arayüz (Frontend)** | Greenlight v3.5 (PostgreSQL & React) veya akademik odaklı **PILOS v4** alternatifleri |
| **Ölçeklendirme** | Scalelite cluster, Greenlight v3 dahili Load Balancer, NFS ortak kayıt paylaşımı |
| **Kimlik Doğrulama** | Keycloak üzerinden OpenID Connect (OIDC) ile LDAP/Active Directory entegrasyonu |
| **Genel Özellikler** | TLDraw beyaz tahta, Plugin Architecture, Sanal arka plan, SIP Trunk Dial-in entegrasyonu |

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
cat /var/www/moodle/version.php | grep "\$release"

# Cache temizle
sudo -u www-data php8.3 /var/www/moodle/admin/cli/purge_caches.php

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
  <sub>⭐ Bu projeyi beğendiyseniz yıldız bırakmayı unutmayın ❤️ faruk-guler</sub>
</p>
