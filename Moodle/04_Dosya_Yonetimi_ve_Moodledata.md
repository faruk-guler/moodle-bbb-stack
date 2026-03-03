# 04 - Dosya Yönetimi ve Moodledata Güvenliği

Moodle'da dosyalar veritabanında değil, `moodledata` adını verdiğimiz fiziksel bir klasörde saklanır. Bu klasörün yönetimi hem hız hem de güvenlik için kritiktir.

## 4.1 Klasör Yapısı ve izin Hiyerarşisi

```
/var/www/moodle/moodledata/
├── filedir/        → İçerik hash'leriyle saklanan gerçek dosyalar
│   ├── 00/
│   │   └── 00/ → SHA1 hash'in ilk 2 karakterine göre alt dizin
│   └── ...
├── trashdir/       → Silinen dosyalar (30 gün sonra otomatik temizlenir)
├── cache/          → Moodle uygulama cache'i (temizlenebilir)
├── localcache/     → Sunucu bazlı geçici dosyalar (temizlenebilir)
├── sessions/       → Redis kullanmıyorsanız PHP session dosyaları
├── temp/           → Yükleme sırasında kullanılan geçici alan
└── lang/           → Dil paketleri ve özelleştirmeler
```

## 4.2 İzin Yönetimi

```bash
# moodledata dizinini oluşturun (web root DIŞINDA!)
sudo mkdir -p /var/www/moodle/moodledata
sudo chown -R www-data:www-data /var/www/moodle/moodledata
sudo chmod -R 750 /var/www/moodle/moodledata

# Moodle kod dizini: Web sunucusu okuyabilmeli, yazamamalı
sudo chown -R root:www-data /var/www/moodle/public
sudo chmod -R 755 /var/www/moodle/public
# İstisna: config.php daha kısıtlı olmalı
sudo chmod 640 /var/www/moodle/config.php
```

> [!WARNING]
> `www-data` kullanıcısının `/var/www/moodle/public` dizinine **yazma** yetkisi vermemelisiniz. Aksi takdirde bir eklenti açığı üzerinden tüm Moodle kodu değiştirilebilir.

## 4.3 File Storage Mimarisi (Content Hash Sistemi)

Moodle, dosyaları SHA1 içerik hash'iyle saklar:

```
Kullanıcı "ozet.pdf" yükledi
  ↓
SHA1 hash hesaplandı: a94f3ba1c12...
  ↓
Dosya yolu: /var/www/moodle/moodledata/filedir/a9/4f/a94f3ba1c12...
  ↓
Aynı dosya başka derse yüklenirse → Disk'e yeniden yazılmaz, sadece DB kaydı eklenir
```

Bu sistem disk kullanımını önemli ölçüde azaltır ancak bir dezavantajı vardır:

> [!NOTE]
> Bir dosyayı sildiğinizde `filedir` dizinindeki gerçek dosya hemen silinmez; 30 gün `trashdir`'de bekler. Disk doluysa `sudo -u www-data php /var/www/moodle/public/admin/cli/purge_caches.php` ile zararsız geçici dosyaları temizleyebilirsiniz.

## 4.4 config.php ile File Storage Yönetimi

```php
// Büyük dosyalar için paralel yüklemeyi aktif et
$CFG->allowobjectembed = 0;

// Moodle'ın dosya sunmak için Nginx'i kullanması (X-Accel-Redirect)
// Nginx vhost'unuza şunu ekleyin:
// location /dataroot/ { internal; alias /var/www/moodle/moodledata/; }
$CFG->xsendfile = 'X-Accel-Redirect';
$CFG->xsendfilealiases = ['/dataroot/' => '/var/www/moodle/moodledata/'];
```

## 4.5 Disk Doluluk Takibi

```bash
# moodledata boyutunu öğren
du -sh /var/www/moodle/moodledata/

# Alt dizin bazında analiz
du -sh /var/www/moodle/moodledata/filedir/
du -sh /var/www/moodle/moodledata/cache/
du -sh /var/www/moodle/moodledata/localcache/

# Temizlenebilir alan (cache ve temp dizinleri)
du -sh /var/www/moodle/moodledata/temp/ /var/www/moodle/moodledata/cache/ /var/www/moodle/moodledata/localcache/
```

## 4.6 Yedekleme Stratejisi

Veritabanı yedeği tek başına işe yaramaz. `moodledata` ve veritabanı yedeğinin **aynı anda** alınması gerekir; birisi diğerinden eski olursa "Missing file" hataları ortaya çıkar.

> [!IMPORTANT]
> Yedek alırken site **bakım moduna** (maintenance mode) alınmalıdır. Aksi takdirde yedek esnasında yeni yüklenen dosyalar DB'ye kayıt düşer ama `moodledata`'ya yazılmamış olabilir.

```bash
# 1. Bakım modunu aç
sudo -u www-data php /var/www/moodle/public/admin/cli/maintenance.php --enable

# 2. Eşzamanlı yedek al
pg_dump -U moodleuser moodledb | gzip > /backup/moodle-db-$(date +%F).sql.gz
tar -czf /backup/moodledata-$(date +%F).tar.gz /var/www/moodle/moodledata/

# 3. Bakım modunu kapat
sudo -u www-data php /var/www/moodle/public/admin/cli/maintenance.php --disable
```

---

> [!TIP]
> Nginx `X-Accel-Redirect` ile dosya dağıtımını Nginx'e devredin. Bu sayede PHP-FPM süreçleri büyük dosya indirmelerinde meşgul kalmaz ve diğer isteklere hizmet verebilir.
