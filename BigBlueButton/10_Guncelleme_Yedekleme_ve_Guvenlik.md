# Bölüm 10: Güncelleme, Yedekleme ve Güvenlik (Sıkılaştırma)

BigBlueButton, aktif bir proje olduğundan sürekli güvenlik yamaları ve performans iyileştirmelerini barındıran minör veya majör güncellemeler alır.

## 10.1 BigBlueButton'ı Güncellemek (Upgrading)

BigBlueButton'ı güncellemenin iki farklı rotası vardır:
1. **Minör Güncelleme (Örn: 3.0.1'den 3.0.4'e):** Kurulum yaparken kullandığınız bbb-install komutunu birebir aynasını tekrar çalıştırırsınız. Veya sadece `sudo apt update && sudo apt dist-upgrade` yazarsınız, ancak paket yolları değiştiyse `bbb-install` komutu daha garantilidir.
2. **Majör Güncelleme (Örn: 2.7'den 3.0'a):** Büyük sürüm atlarken eski ayarlar (`/etc/bigbluebutton/...`) uyumsuzluk yaratabilir. Bu sebeple temiz Ubuntu 22.04 kurup BBB 3.0 kurmak ve eski kayıtları "aktarmak" (Migrate) önerilen resmi yöntemdir. Eski sunucu formatlanmaz, yeni sunucu kurulur, DNS (bbb.sirket.com) yeni sunucuya kaydırılır.

> [!CAUTION]
> Herhangi bir güncelleme yapmadan önce (apt base güncelleme dahil), ortamda toplantı olmadığından emin olun ve sunucunun Snapshot'ını (Anlık Görüntü) alın! 

## 10.2 BigBlueButton Yedeklemesi (Backup Stratejileri)

BBB sunucusunda yedeklenmesi gereken 3 ana yer vardır:
1. **Sistem Ayarları:** `/etc/bigbluebutton/`, `/etc/nginx/`, `/var/www/bigbluebutton-default/` (Kendi logonuz, ayarlarınız).
2. **Greenlight Veritabanı:** Eğer Greenlight kullanılıyorsa PostgreSQL veritabanı dump alınmalıdır veya `~/greenlight-v3/` klasörü docker down yapıldıktan sonra (.env ile birlikte) yedeklenmelidir.
3. **Kayıtlar (Recordings):** `/var/bigbluebutton/published/` (işlenmiş mp4/sunumlar) ve `/var/bigbluebutton/recording/raw/` (henüz işlenmemiş ham videolar). Bu kısımlar TB'larca veri tutabilir. Bu yüzden haftalık olarak S3 Bucket veya NAS ortamına `rsync` ile basılması önerilir.

```bash
# Örnek Rsync Yedekleme Komutu (Kayıtlar için)
rsync -aP /var/bigbluebutton/published/ root@yedekleme_sunucusu:/backup/bbb/
```

## 10.3 Güvenlik Sıkılaştırması (Linux Hardening & WAF)

BigBlueButton 80, 443 ve geniş bir UDP aralığı kullanır. Klasik bir web sunucusundan daha agresif güvenlik önlemleri alırken (WAF vb) "Websocket / RTP paket düşmesin" diye ayar yapmalısınız.

### 1. Fail2Ban Kurulumu
Kötü niyetli kişilerin SSH veya Greenlight giriş paneline Brute Force (kaba kuvvet) şifre denemelerini önlemek için Fail2Ban hayat kurtarır.
```bash
sudo apt install fail2ban
```
Ardından `/etc/fail2ban/jail.local` dosyası içine özel nginx veya greenlight kuralları eklenebilir.

### 2. WAF (Web Application Firewall - Cloudflare) Uyarıları
**Cloudflare** veya benzeri WAF'ların arkasına BBB konulduğunda (Proxy mode "Orange cloud" yapıldığında):
- WebRTC (UDP paketleri) ve 7443 (WebSocket üzerinden SIP) bağlantıları genelde takılır, kameralar donar, bağlanılamaz!
- BBB'nin **SADECE DNS** ("Grey Cloud" modunda) modunda çalışması gereklidir. Veya Cloudflare Spectrum (pahalı tier) ile UDP/TCP proxy yapılması gerekir.

### 3. Kullanılmayan Portların Kapatılması ve UFW
Kurulum sırasında `-w` parametresi kullanıldığında zaten UFW aktif edilir. SSH portunuzu standart `22` den farklı bir porta taşıyıp sadece kurum ofisine ait IP'lere açabilirsiniz.

### 4. Turn Sunucusunun Kötüye Kullanılmasını (Relay) Engellemek
Coturn (TURN) sunucu kurduğunuzda, başkalarının sizin sunucunuzdan faydalanıp internette gizli tünelleme yapamaması için sadece BBB sunucunuzun IP'sine (veya kimlik doğrulamasına) izin verilmelidir (`turnserver.conf` içerisinde `allowed-peer-ip` yapılandırılmalıdır).

---
**İpucu:** Güvenlik önlemlerini uyguladıktan sonra bir sonraki bölüm olan **API Kullanımı ve Dış Entegrasyonlar** ile devam edin.
