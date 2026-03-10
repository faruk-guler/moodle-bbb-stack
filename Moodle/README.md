# Moodle v5.1.x — Kurumsal BT Altyapı ve Mimarisi 2026

<img src="./img/moodle_main.png" alt="Moodle Preview" width="50%">

```console
## Yazar: faruk-guler 2026
```

## Moodle Nedir?

Moodle, dünyanın en popüler, açık kaynaklı öğrenme yönetim sistemidir (LMS). PHP tabanlı yapısı, esnek eklenti desteği ve devasa komünitesi ile her seviyeden eğitim kurumuna hitap eder. Sadece içerik sunmakla kalmaz; sınavlar, ödevler, işbirlikçi çalışmalar ve analitik raporlamalar için kapsamlı bir ekosistem sağlar.

Bu proje, **Moodle** için sistem yöneticilerine, devops mühendislerine ve sunucu yöneticilerine yönelik hazırlanan, baştan uca ve eksiksiz bir Türkçe dokümantasyon setidir. Dokümantasyon, Moodle'nın **5.1.3** (Mart 2026 itibarıyla yayımlanmış) kararlı (stable) sürümü odaklı hazırlanmış, geriye dönük uyumluluk notlarıyla 4.5 LTS'e de değinmektedir.

> Moodle 5.0 ile kritik bir mimari değişiklik geldi: Web sunucusu artık `/public` alt dizinini root olarak kullanmaktadır.

## Dokümantasyon Hakkında 📘

Bu rehber, Moodle'ın sadece kurulumunu değil, gerçek dünya senaryolarında nasıl hayatta tutulacağını anlatır. Yüksek trafikli dönemlerde (sınav haftaları gibi) sunucunun nasıl optimize edileceğini, veritabanı darboğazlarının nasıl aşılacağını ve güvenliğin nasıl en üst seviyeye çıkarılacağını ele alır.

### Bölümler

* **01. Mimari ve Bileşenler:** Nginx, PHP-FPM ve PostgreSQL üçgeninde Moodle'ın kalbi.
* **02. Sistem Kurulumu:** Modern Ubuntu/Debian sistemlerde en iyi pratiklerle kurulum.
* **03. Veritabanı ve Caching:** Redis'in Moodle'a kattığı hız ve PostgreSQL ince ayarları.
* **04. Dosya Yönetimi:** Moodledata klasörünün izolasyonu, SHA1 hash sistemi ve X-Accel-Redirect optimizasyonu.
* **05. Cron ve Görev Yönetimi:** Moodle'ın "beyni" olan zamanlanmış görevlerin (Ad-hoc) kontrolü.
* **06. Güvenlik ve Hardening:** SSL ötesi; WAF, HSTS ve Moodle Security Checkup.
* **07. Yedekleme ve Güncelleme:** Git ile hatasız sürüm yükseltme ve SQL yedekleme.
* **08. Tema ve Özelleştirme:** CSS/SCSS ve dil paketleri üzerinde sistemci dokunuşları.
* **09. Eklenti ve LTI:** Harici araçların güvenli entegrasyonu.
* **10. Performans ve Ölçeklendirme:** Load Balancing ve Sticky Sessions mimarisi.
* **11. Mobil Uygulama:** Push bildirimleri ve Firebase ayarları.
* **12. Analitik ve Raporlama:** Veriden anlam çıkarma; Learning Analytics.
* **13. BigBlueButton Entegrasyonu:** Moodle ve BBB'nin mükemmel uyumu.
* **14. Sorun Giderme (Troubleshooting):** Beyaz ekranlar, debug modları ve log analizi.
* **15. Gelecek Vizyonu:** Moodle 5.x, AI entegrasyonu, Docker ve 2027 yol haritası.

---
