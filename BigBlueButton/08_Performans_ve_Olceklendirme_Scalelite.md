# Bölüm 8: Performans ve Ölçeklendirme (Scalelite)

BigBlueButton tekil bir sunucudur. Tasarımı gereği 1 odayı (meeting) 2 farklı sunucudaki kaynaklara eşit bölemez (Monolithic Room). Bir oda en fazla 1 sunucunun gücünü kullanabilir. (Önerilen maksimum limit tek odada 100-150 webcamsız katılımcı veya 40-50 webcamli katılımcıdır).

Okullar ve büyük kurumlar binlerce kişiyi aynı anda BBB üzerinde ağırlamak zorundadır. Peki nasıl ölçeklendirilir? Cevap: **Scalelite** veya **Greenlight/BBB kendi LoadBalancer'ı (v3.4+)**.

## 8.1 Scalelite Nedir?

Scalelite, BigBlueButton'ın yaratıcıları (Blindside Networks) tarafından geliştirilen açık kaynaklı bir Yük Dengeleyici (Load Balancer) mimarisidir.

- Kullanıcı (Öğretmen) Moodle veya Greenlight üzerindeki "Derse Katıl" butonuna basar.
- İstek `scalelite.sirket.com` adresine (Nginx) gelir.
- Scalelite arkasındaki 10 adet farklı BBB sunucusunun CPU/RAM yüküne veya içindeki kişi sayısına bakar. 
- En boş olan (örneğin `bbb04.sirket.com`) sunucusuna istek API üzerinden yönlendirilir.
- Odanın tamamı (Tüm giren öğrenciler dahil) `bbb04`'te yaşamını sürdürür.
- İkinci bir öğretmen başka bir ders açtığında, Scalelite onu `bbb07`'ye atar. 

> [!CAUTION]
> Scalelite veya herhangi bir BBB Load Balancer, "tek bir odadaki 500 kişiyi 5 farklı sunucuya bölerek" taşımaz. Ölçeklendirme işlemi ODA/TOPLANTI (Meeting) bazındadır. Yani 10 odanız varsa her birini 1 sunucuya dağıtır. Tek odada devasa bir yayın (Örn: 2000 kişi) yapmak isterseniz webiner (Youtube Live/Rtmp) entegrasyonu gerekir.

## 8.2 Scalelite Mimarisi ve Gereksinimleri

Scalelite kurulumu için klasik BBB mimarisinden farklı bir topoloji gerekir:

1. **Önyüz / LMS:** Moodle, Canvas, Greenlight vb. (Kullanıcı giriş ekranı)
2. **Scalelite Sunucusu (veya Cluster'ı):** Nginx ve Ruby/Postgres tabanlı Docker ile çalışan LB.
3. **Worker BBB'ler:** Çok sayıda (Örn: bbb01, bbb02, bbb03) arkada duran "İşçi" sunucular. (Bu sunucular dışarıdan IP tabanlı olarak direkt erişime kapalı olmasa da, kullanıcıların FQDN'ini bilmesine gerek yoktur).
4. **Ortak Depolama (NFS Server veya AWS S3):** 
   - Öğretmen bbb04'te ders anlattı ve ders kaydedildi diyelim. Öğretmen ertesi gün bbb01'e denk gelirse dünkü kaydını nasıl görecek? 
   - Bütün worker BBB'ler kayıt bittiğinde MP4 / Presentation dosyalarını NFS (Ortak Ağ Sürücüsü) alanına kopyalar (`rsync` veya `scp` mantığıyla).
   - Kullanıcı Moodle'dan "Dünkü Dersi İzle" dediğinde, Scalelite o kaydı doğrudan NFS'nin üzerinden (Nginx alias ile) oynatır.

## 8.3 Yeni Ölçeklendirme Yaklaşımı (Greenlight v3 Load Balancer)

Eğer LMS (Moodle vb) kullanmıyor, sadece Greenlight kullanıyorsanız, Greenlight v3 (Son sürümü) kendi içinden Load Balancing (Yük Dengeleme) yapabilir. Admin panelinden `bbb01`, `bbb02` sunucularının Secret Key'lerini arayüze girersiniz. Greenlight en boş olana otomatik atar. Scalelite kurma zorunluluğunu ortadan kaldıran pratik bir çözümdür, ancak Merkezi Kayıt Yönetimi (NFS) gibi özellikleri için ek bir script/web sunucu konfigürasyonu istemeye devam eder.

## 8.4 Performans Sıkılaştırması (Kamera Limitleri)

LB dışında, elinizdeki tek sunucunun performansını optimize etmenin en iyi yolu kamera limitlerini kısıtlamaktır.

`bbb-web` üzerinden `bigbluebutton.properties` yapılandırmasında bazı değerler CPU tasarrufu sağlar:

- `maxWebcams` : Sistemde aynı anda sadece 5 kamera açılmasına izin verme. (Örn: `maxWebcams=5`)
- `webcamsOnlyForModerator` : Eğer `true` ise, kimse kimsenin kamerasını göremez, herkes sadece öğretmeni görür.
- `presentationBackgroundOnly`: Çizim tahtası optimizasyonu sağlar.
