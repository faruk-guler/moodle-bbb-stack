# 07 - Yedekleme ve Sürüm Yükseltme (Moodle 5.1.x — Git Yöntemi)

Moodle'ı arayüzden "Güncelle" butonuna basarak yükseltmek, büyük veritabanlarında felaketle sonuçlanabilir. Sistem yöneticileri her zaman **CLI** ve **Git** kullanır.

> [!NOTE]
> **Moodle 5.x Yükseltme Yolu:** 4.2.3 veya sonraki bir 4.x sürümünden doğrudan 5.x'e yükseltme desteklenmektedir. 4.1 veya daha eskisindeyseniz önce 4.5 LTS'ye geçin, ardından 5.x'e atlayın.

## 7.1 Tam Yedekleme — The Golden Trio

Bir Moodle yedeği 3 parçadan oluşur. Birisi eksikse yedek işe yaramaz.

| Parça | Komut | Hedef |
| :--- | :--- | :--- |
| Veritabanı | `pg_dump` | `/backup/db/` |
| moodledata | `tar` | `/backup/files/` |
| config.php | `cp` | `/backup/config/` |

## 7.2 Otomatik Yedekleme Betiği

```bash
sudo nano /usr/local/bin/moodle-backup.sh
```

```bash
#!/bin/bash
# Moodle Otomatik Yedekleme Betiği — faruk-guler 2026

set -e

BACKUP_DIR="/backup/moodle"
DATE=$(date +%Y-%m-%d_%H-%M)
DB_NAME="moodledb"
DB_USER="moodleuser"
MOODLE_ROOT="/var/www/moodle/public"   # Moodle 5.x: /public alt dizini
DATA_ROOT="/var/www/moodle/moodledata"  # /public dışında
KEEP_DAYS=7

# Dizin oluştur
mkdir -p "$BACKUP_DIR/$DATE"

echo "[$(date)] Yedekleme başladı..."

# 1. Bakım modunu aç
sudo -u www-data php8.3 "$MOODLE_ROOT/admin/cli/maintenance.php" --enable

# 2. Veritabanı yedeği
PGPASSWORD="cok_guclu_bir_sifre" pg_dump -U "$DB_USER" "$DB_NAME" | \
  gzip > "$BACKUP_DIR/$DATE/moodle-db.sql.gz"
echo "[$(date)] Veritabanı yedeği tamamlandı."

# 3. Moodledata yedeği
tar -czf "$BACKUP_DIR/$DATE/moodledata.tar.gz" "$DATA_ROOT"
echo "[$(date)] Moodledata yedeği tamamlandı."

# 4. config.php yedeği (5.x: config.php, /public dışında üst dizindedir)
cp "/var/www/moodle/config.php" "$BACKUP_DIR/$DATE/config.php"

# 5. Bakım modunu kapat
sudo -u www-data php8.3 "$MOODLE_ROOT/admin/cli/maintenance.php" --disable

# 6. Eski yedekleri temizle
find "$BACKUP_DIR" -maxdepth 1 -type d -mtime +"$KEEP_DAYS" -exec rm -rf {} \;

echo "[$(date)] Yedekleme başarıyla tamamlandı: $BACKUP_DIR/$DATE"
```

```bash
sudo chmod +x /usr/local/bin/moodle-backup.sh
# Crontab'a ekle (Her gece 02:00'de çalışsın)
echo "0 2 * * * root /usr/local/bin/moodle-backup.sh >> /var/log/moodle-backup.log 2>&1" | sudo tee /etc/cron.d/moodle-backup
```

## 7.3 Git ile Sürüm Yükseltme (Upgrade)

> [!WARNING]
> Yükseltme öncesi mutlaka yedek alın. Yükseltme sonrası geri dönmek için `git checkout` ve veritabanı restore gerekecektir.

```bash
# 1. Mevcut Moodle sürümünü kontrol et
cat /var/www/moodle/public/version.php | grep "\$version" | head -1

# 2. Bakım modunu aç
sudo -u www-data php8.3 /var/www/moodle/public/admin/cli/maintenance.php --enable

# 3. Git ile yeni sürüme geç
cd /var/www/moodle
sudo git fetch origin
sudo git checkout MOODLE_51_STABLE    # Güncel 5.1 LTS dalı

# 4. Cache'i temizle
sudo -u www-data php8.3 public/admin/cli/purge_caches.php

# 5. Veritabanı şemasını güncelle
sudo -u www-data php8.3 public/admin/cli/upgrade.php --non-interactive

# 6. Bakım modunu kapat
sudo -u www-data php8.3 public/admin/cli/maintenance.php --disable
```

## 7.4 Yükseltme Sonrası Doğrulama

```bash
# Tüm eklentilerin güncel olduğunu kontrol et
sudo -u www-data php8.3 public/admin/cli/upgrade.php --non-interactive

# Yeni sürüm numarasını doğrula
cat /var/www/moodle/public/version.php | grep "\$release"

# Site yönetim panelinde bildirim varsa
# Site Administration > Notifications sayfasını ziyaret edin
```

## 7.5 Rollback (Geri Dönüş) Stratejisi

```bash
# 1. Git ile eski commit'e geri dön
cd /var/www/moodle
sudo git log --oneline -5                     # Eski commit hash'ini bul
sudo git checkout <eski_commit_hash>

# 2. Veritabanını yedekten geri yükle
sudo -u www-data php8.3 public/admin/cli/maintenance.php --enable
gunzip -c /backup/moodle/2026-03-01_02-00/moodle-db.sql.gz | PGPASSWORD="sifre" psql -U moodleuser moodledb

# 3. moodledata'yı yedekten geri yükle (gerekirse)
sudo -u www-data php8.3 public/admin/cli/maintenance.php --disable
```

---

> [!TIP]
> Yedek betiklerinin başarıyla çalıştığını doğrulamak için `/var/log/moodle-backup.log` dosyasını düzenli kontrol edin. Bir monitoring aracıyla (Zabbix, Nagios) log dosyasındaki hata kelimelerini izlemek daha güvenlidir.
