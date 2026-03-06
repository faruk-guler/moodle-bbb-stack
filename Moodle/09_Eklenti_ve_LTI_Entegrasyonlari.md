# 09 - Eklenti (Plugin) ve LTI Entegrasyonları

Moodle'ın gücü devasa eklenti kütüphanesinden gelir. Ancak her eklenti potansiyel bir güvenlik açığı ve performans yükü olabilir.

## 9.1 Eklenti Kurulum Stratejisi

Arayüz üzerinden (`Site Administration > Install plugins`) eklenti yüklemek her zaman risklidir: dosya izinleri `www-data`'ya ait olur, geri alma şansı yoktur. Doğru yöntem **Git** ile kendi dizinine çekmektir.

```bash
# Eklenti dizin yapısı
/var/www/moodle/public/
├── blocks/          # Blok eklentileri (dashboard widget'ları)
├── mod/             # Etkinlik modülleri (Quiz, Assignment vb.)
├── theme/           # Temalar
├── local/           # Yerel özelleştirmeler
├── auth/            # Kimlik doğrulama eklentileri (LDAP, SAML)
└── question/type/   # Soru tipi eklentileri
```

```bash
# Git ile eklenti kurma örneği (Format Tiles)
cd /var/www/moodle/public/course/format
sudo git clone https://github.com/gjb2048/moodle-format_tiles.git tiles
sudo chown -R root:www-data /var/www/moodle/public/course/format/tiles
sudo chmod -R 755 /var/www/moodle/public/course/format/tiles
```

Ardından `Site Administration > Notifications` sayfasını ziyaret ederek DB şemasını güncelleyin.

> [!NOTE]
> Eklenti kurduktan sonra `public/admin/cli/upgrade.php` çalıştırabilirsiniz. Web arayüzüne gitmek zorunda değilsiniz.

```bash
sudo -u www-data php8.3 /var/www/moodle/public/admin/cli/upgrade.php --non-interactive
```

## 9.2 Eklenti Güncelleme (Git Yöntemi)

```bash
# Tüm eklentileri tek tek güncellemek yerine toplu güncelleme
cd /var/www/moodle/public/course/format/tiles
sudo git pull origin main

# Veritabanı şemasını güncelle
sudo -u www-data php8.3 /var/www/moodle/public/admin/cli/upgrade.php --non-interactive
sudo -u www-data php8.3 /var/www/moodle/public/admin/cli/purge_caches.php
```

## 9.3 Atıl Eklentilerin Temizlenmesi

`Site Administration > Plugins > Plugins overview`

| Durum | Anlamı | Eylem |
| :--- | :--- | :--- |
| Missing from disk | Klasör silindi ama DB kaydı var | DB'yi temizle |
| Enabled, not used | Aktif ama hiç kullanılmıyor | Deaktif + sil |
| Outdated | Güncel değil | Git pull ile güncelle |
| Incompatible | Bu Moodle sürümüyle uyumsuz | Kaldır |

```bash
# Disk'ten eklenti dizini sildikten sonra kalan DB kayıtlarını temizle
sudo -u www-data php8.3 /var/www/moodle/public/admin/cli/uninstall_plugins.php --plugins=mod_eskieklenti --run
```

## 9.4 LTI (Learning Tools Interoperability)

Moodle'ın dış dünyayla konuşma standartıdır (Örn: bir yayınevinin interaktif içeriğini Moodle içine gömmek).

### LTI 1.3 Advantage Kurulumu

1. `Site Administration > Plugins > Activity Modules > External Tool > Manage external tools`
2. **Add tool** butonuna tıklayın
3. **Tool URL:** Harici içerik sağlayıcının LTI URL'si
4. **Consumer key / Secret:** LTI 1.1 için; LTI 1.3'te bunların yerine **RSA anahtar çifti** kullanılır

```
LTI 1.1: Shared Key + Secret (Daha eski, basit)
LTI 1.3: Platform/Client IDs + RS256 Anahtar Çifti (Daha güvenli)
```

> [!TIP]
> LTI araçlarını bağlarken mutlaka "**Deep Linking**" (Content Selection) desteğini sağlayıcıdan onaylayın. Bu özellik sayesinde eğitmen, harici içerik sağlayıcının kataloğuna doğrudan Moodle içinden göz atabilir.

## 9.5 Güvenlik Tavsiyesi

```bash
# Eklentilerin son güncellenme tarihlerini kontrol edin
# https://moodle.org/plugins/index.php adresinden filtreleyin
# "Last released" kolonunu 1 yıldan eski olan eklentilerden uzak durun

# Eklentinin desteklenen Moodle versiyonunu kontrol et
cat /var/www/moodle/public/mod/eklentime/version.php | grep "requires"
```

---

> [!WARNING]
> Moodle Plugin Directory'de onaylanmamış, 2 yıldan uzun süredir güncellenmemiş eklentileri kurumsal ortamda kullanmayın. Her eklentinin kaynak kodunu incelemeniz veya en azından GitHub Issues sekmesini takip etmeniz önerilir.
