# 06 - Güvenlik ve Hardening (Sıkılaştırma)

Moodle, öğrenci verilerini barındırdığı için saldırganlar için cazip bir hedeftir. Standart bir kurulum birçok güvenlik açığını kapalı bırakır.

## 6.1 Dosya İzinleri (Permission Hardening)

```bash
# Moodle kod dizini (Kök): root sahipliği, www-data sadece okuma
sudo chown -R root:www-data /var/www/moodle
sudo find /var/www/moodle -type f -exec chmod 640 {} \;
sudo find /var/www/moodle -type d -exec chmod 750 {} \;
# public dizinine Nginx erişimi için izin
sudo chmod -R 755 /var/www/moodle/public

# config.php en kısıtlı izinlerde olmalı (kod kökünde, public dışında)
sudo chmod 640 /var/www/moodle/config.php

# moodledata: sadece www-data yazabilmeli
sudo chown -R www-data:www-data /var/www/moodle/moodledata
sudo chmod -R 750 /var/www/moodle/moodledata
```

> [!WARNING]
> Eklenti yüklemek veya Moodle güncellemek için geçici olarak `www-data`'ya yazma yetkisi vermeniz gerekebilir. İşlem bittikten sonra derhal geri alın.

## 6.2 SSL/TLS ve HSTS Yapılandırması

`config.php` içinde mutlaka `$CFG->wwwroot` adresini `https://` ile başlatın:

```php
$CFG->wwwroot = 'https://moodle.sirket.com';

// Moodle'ın çerezleri sadece HTTPS üzerinden göndermesi
$CFG->cookiesecure = true;

// Çerezin sadece HTTP tarafından okunabilmesi (JavaScript erişimini engeller)
$CFG->cookiehttponly = true;
```

Nginx vhost'una HSTS başlığı ekleyin:

```nginx
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

## 6.3 UFW Güvenlik Duvarı Kuralları

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH (port numaranızı değiştirdiyseniz güncelleyin)
sudo ufw allow 22/tcp

# Web trafiği
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw enable
sudo ufw status verbose
```

> [!NOTE]
> Yönetim (Admin) paneline erişimi belirli IP adresleriyle kısıtlamak için Nginx seviyesinde ek bir koruma katmanı ekleyebilirsiniz:
> ```nginx
> location /admin {
>     allow 192.168.1.0/24;
>     allow 10.0.0.5;
>     deny all;
> }
> ```

## 6.4 config.php Hardening Parametreleri

```php
// Hata mesajlarını web arayüzünde gösterme (canlı sistemde kapalı olmalı)
$CFG->debug = 0;
$CFG->debugdisplay = 0;

// Moodle'ın HTTP(S) protokolünü zorla
$CFG->sslproxy = false;   // Nginx SSL termination yapıyorsa true yapın

// SMTP aracılığıyla e-posta: SSL bağlantısı
$CFG->smtpsecure = 'tls';

// Belirli bir IP'den erişim yoksa oturumu sonlandır
// (Mobil kullanıcılar için dikkatli kullanın)
// $CFG->sessioncookiepath = '/';
```

## 6.5 Moodle Yerleşik Güvenlik Denetimi

```
Site Administration > Reports > Security checks
```

Buradaki **tüm kırmızı uyarıları yeşile çevirmek** bir sistem yöneticisinin asli görevidir.

| Kontrol | Önem | Sık Görülen Neden |
| :--- | :--- | :--- |
| Moodledata web'de erişilebilir | 🔴 Kritik | Dizin yanlış konumda |
| OPcache etkin değil | 🔴 Yüksek | php.ini ayarı eksik |
| Cron son 1 günde çalışmamış | 🔴 Yüksek | crontab veya timer yok |
| Register globals açık | 🟡 Orta | Eski PHP yapılandırması |
| Hata ayıklama ekranda | 🟡 Orta | debug=1 bırakılmış |
| open_basedir kullanılmıyor | ⚪ Düşük | İsteğe bağlı |

---

> [!TIP]
> `config.php` dosyasını asla Git reposuna pushlamayın. İçindeki `dbpass` ve `passwordsaltmain` değerlerini hiç kimseyle paylaşmayın. Şifreyi güncellemek gerekirse `admin/cli/reset_password.php` CLI aracını kullanın.
