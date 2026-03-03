# 11 - Mobil Uygulama ve Push Bildirimleri

Moodle Mobile, öğrencilerin en çok kullandığı arayüzdür. Ancak düzgün yapılandırılmazsa "Siteye bağlanamıyor" hataları ile destek taleplerini kilitler.

## 11.1 Web Services Aktivasyonu

Moodle Mobile uygulamasının çalışması için Web Servisleri ve REST protokolü etkinleştirilmelidir.

```
Site Administration > Advanced Features > Enable web services ✅
Site Administration > Plugins > Web services > Manage protocols > REST protocol ✅
Site Administration > Mobile app > Enable mobile web service ✅
```

Config.php üzerinden de zorlayabilirsiniz:

```php
// Mobil uygulamanın bağlanabilmesi için
$CFG->enablewebservices = 1;
$CFG->enablemobilewebservice = 1;
```

## 11.2 Mobil Bağlantı Testi

```bash
# API endpoint'ini test et (site_info döndürmeli)
curl -X GET "https://moodle.sirket.com/webservice/rest/server.php?wsfunction=core_webservice_get_site_info&wstoken=TOKEN&moodlewsrestformat=json"

# Token üretimi (Admin CLI)
sudo -u www-data php8.3 /var/www/moodle/public/admin/tool/mobile/launch.php \
  --service=moodle_mobile_app \
  --username=kullanici_adi
```

> [!NOTE]
> Mobil uygulama ilk bağlandığında `core_webservice_get_site_info` çağrısını yapar. Bu çağrı başarısız oluyorsa 9 kez üzeri 10 site URL'si veya SSL sorunudur — tarayıcıda siteyi açarak SSL sertifikasının geçerli olduğunu doğrulayın.

## 11.3 Mobil Uygulama Görünümü Özelleştirme

```
Site Administration > Mobile app > Mobile appearance
```

| Ayar | Açıklama | Örnek Değer |
| :--- | :--- | :--- |
| Main colour | Ana renk | #1a5897 |
| Link colour | Bağlantı rengi | #0d47a1 |
| Feature disabled | Gizlenecek özellikler | `NoDelegate_CoreCoursesDashboard` |
| Forced language | Zorunlu dil | tr |

## 11.4 Airnotifier ve Push Bildirimleri

Moodle varsayılan olarak bildirimleri **Moodle HQ'nun Airnotifier sunucuları** üzerinden gönderir. Bu ücretsizdir ancak özelleştirilemez.

```
Site Administration > Messaging > Airnotifier settings
```

Varsayılan değerleri bırakırsanız Moodle HQ altyapısı kullanılır. Kendi uygulamanızı App Store/Play Store'da yayınlamak istiyorsanız kendi Airnotifier sunucunuzu kurmanız ve Firebase (FCM) / Apple (APNs) anahtarlarını girmeniz gerekir.

## 11.5 Offline Çalışma ve Senkronizasyon

Moodle Mobile'ın en önemli özelliği, içerikleri **indirip çevrimdışı** kullanabilmesidir.

> [!WARNING]
> SCORM paketlerinin mobil uygulamada çalışması için SCORM dosyalarınızın **mutlak yol** (absolute paths) içermediğinden emin olun. Bazı hazır SCORM araçları `file:///C:/...` gibi yerel yollar kullanır ve bu mobilde çalışmaz.

## 11.6 Mobil Hatalarda Debug

```bash
# Mobil uygulamanın oluşturduğu tokenleri ve aktivitelerini logla
# config.php içine ekle (sadece test ortamında!)
$CFG->debugwebservices = true;

# PHP-FPM hatalarını izle
sudo tail -f /var/log/php-fpm/moodle-error.log | grep -i "webservice\|mobile\|token"
```

---

> [!TIP]
> Öğrencilere uygulamayı tanıtmadan önce `https://moodle.sirket.com/moodle-app-test.html` adresini kontrol edin — Moodle bu özel test sayfasında sitenizin mobil uygulamayla uyumlu olup olmadığını size gösterir.
