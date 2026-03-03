# Bölüm 15: Sık Karşılaşılan Hatalar ve Kriz Yönetimi (Troubleshooting Gelişmiş)

BigBlueButton (BBB) birbirinden bağımsız birçok servisin (Node.js, Ruby, Java, FreeSWITCH, Mediasoup, Nginx) birleşimidir. Kritik hataların büyük bir kısmı, bu servislerin birbirleriyle iletişiminin (mesaj kuyruklarının veya izinlerin) kopmasından meydana gelir.

Bu bölüm, klasik `bbb-conf --restart` komutunun işe yaramadığı, "Sistem tamamen durdu" krizlerinde hızlıca bakılacak bir SysAdmin acil müdahale rehberidir.

## 15.1 "502 Bad Gateway" Yazan Beyaz Ekran Hatası

Sunucu durup dururken Nginx'in meşhur beyaz `502 Bad Gateway` sayfasını vermeye başladı ise, olay Nginx'ten ziyade arkasındaki API'nin çökmesinden kaynaklanır.

**Kriz Tespiti:** 
1. `sudo bbb-conf --status` yazın. 
2. Eğer listede `bbb-web` durumunda `failed` veya hiç çalışmıyor ibaresi varsa, sistem toplantıları düzenleyen Java uygulamasını ayağa kaldıramamıştır.

**Çözüm Yolları:**
- Bazen API'nin arka planda RAM şişmesi yaşayıp swap yapamaması çökmeye neden olur. `sudo systemctl restart bbb-web` deneyin. Eğer 2 dakika beklemeye rağmen kalkmazsa:
- **Gizli Hata (Var Log):** `/var/log/bigbluebutton/bbb-web.log` dosyasına bakın. "Max Memory" (OOM Killer) hatası görüyorsanız, sunucunuzun RAM'ini artırmanız gerekir.

## 15.2 Kırık Sunumlar (Presentation/PDF Dönüşümünün Takılması)

En kronik problemlerden biridir. Eğitmen PowerPoint (.pptx) veya bir .pdf yüklediğinde ekranda "Conversion is running (Dönüştürülüyor)" yazar fakat 5-10 dakika geçer ve hata verir.

**Mantık:** BBB, ofis dokümanlarını PNG'ye çevirmek için LibreOffice tetikler. Ancak bozuk bir ofis dosyası yüklendiğinde LibreOffice işlemi asılı (zombi process) kalır ve sonraki gelen bütün yüklemelerin kuyruğa takılmasına sebep olur.

**Çözüm (Kuyruğu Temizleme ve Yeniden Başlatma):**
```bash
# Takılı kalan LibreOffice (soffice) işlemlerini anında öldürün
sudo killall -9 soffice.bin
sudo killall -9 bbb-web

# Ardından temp dosyalarındaki kırık/kalıntı yüklemeleri temizleyin
sudo bbb-conf --clean
```
Geri döndüğünde öğretmen sunumu saniyeler içinde tekrar yükleyebilecektir.

## 15.3 Mediasoup Limits (Yüz Bölünmesi veya Kırmızı Ekran)

Kalabalık bir toplantıda (örn: 30 kameranın açık olduğu) öğretmenlerin yüzleri pikselleniyor (Mozaikleniyor), sesler robotikleşiyor veya "Kamera başlatılamadı" hatası geliyorsa:

1. Sunucunuzun Bant Genişliği (1 Gbit/s vb) yetiyor olabilir ancak CPU'nuz, Kırmızı (Red) frame'leri işleyecek güce sahip olmayabilir.
2. Bu durumda varsayılan Video Codec'ini (H264'ten VP8'e) zorlamak veya kameraların Bitrate oranlarını ciddi şekilde kısmak gerekir.

**Müdahale (Bitrate Kısmak):**
- Dosya: `/etc/bigbluebutton/bbb-webrtc-sfu/production.yml`
- Dosya içinde `cameraProfiles` bölümünü bulun ve `bitrate` değerlerini (örneğin High: 800'den 400'e, Low: 100'den 50'ye) düşürün. İşlemden sonra bbb-webrtc-sfu sistemini `systemctl` ile yeniden başlatın.

## 15.4 Sunucu Taşıma (Greenlight / Postgres Veritabanını Kurtarma)

Çöken veya IP'si değişen, donanımı yetmediği için taşınacak olan sunucuda, en değerli veriler Greenlight'ın içindedir (Kim hangi id ile kayıt olmuş, odasının adı ne, dosyalar vb).

**Eski Sunucuda Veritabanı Yedeği Alma (Greenlight v3):**
```bash
cd ~/greenlight-v3
# V3 docker container'ı içindeki postgresql servisine bağlanarak dump alıyoruz.
docker exec greenlight-db pg_dump -U postgres greenlight_production > greenlight_db_yedek.sql
```
*Yukarıdaki `greenlight_db_yedek.sql` dosyasını WinSCP ile bilgisayarınıza indirin.*

**Yeni Sunucuya Yükleme (Restore):**
Önce tamamen boş ve temiz bir Ubuntu 22.04'e BBB ve Greenlight'ı `bbb-install` ile (`-g` parametresiyle) sıfırdan kurun.
```bash
# Eski SQL dosyanızı /root/ dizinine (WinSCP ile) geri atın
cd ~/greenlight-v3

# Eski tabloların çakışmaması için önce yeni tabloları temizleyelim (DİKKAT!)
docker exec greenlight-db psql -U postgres -d greenlight_production -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"

# Ve eski veritabanını içeri batalım
cat /root/greenlight_db_yedek.sql | docker exec -i greenlight-db psql -U postgres -d greenlight_production

# Değişikliklerin oturması için Greenlight'i vites düşürüp kaldıralım
docker compose down && docker compose up -d
```
Artık kullanıcı hesaplarınız, admin paneliniz ve odalarınız saniyeler içinde yeni makinede!

## 15.5 TLS (Sertifika) Süresi Dolduğunda Neler Olur?

Eğer `bbb-install.sh` otomatik yenilemeyi başaramazsa (Let's encrypt botu düşerse veya 80 portunuz kapandıysa), SSL süreniz bitiği an tarayıcılar (Chrome/Safari) sisteminizi bloklar. Mikrofon izin panelleri kaybolur.

**Manuel Acil Yenileme Komutu:**
```bash
sudo certbot --nginx renew
# Yenileme başarılı olduktan sonra Nginx'e reload atmayı unutmayın:
sudo systemctl reload nginx
sudo bbb-conf --restart
```
*(Eğer SSL yenilendiği halde kameralar ve ses WebRTC hatası veriyorsa, bbb-webrtc-sfu eski sertifikanın cache'ini tutuyor demektir; mutlaka sistemi `bbb-conf --restart` ile tazelemeniz gerekir).*
