# Bölüm 3: Ağ, Güvenlik Duvarı ve TURN/STUN Yapılandırması

BigBlueButton'ın en hassas kısmı network (ağ) yapılandırmasıdır. Kullanıcıların ses veya kamerasının bağlanamaması (meşhur 1007 veya 1020 WebRTC hataları) genellikle ağ veya NAT arkasındaki yapılandırma eksikliklerinden kaynaklanır.

## 3.1 Güvenlik Duvarı (Firewall) Port İzinleri

Bir ortamdaki güvenlik duvarı (ister UFW, ister harici bir donanım firewall - FortiGate, pfSense vb.) BBB sunucusuna aşağıdaki portlara erişim izni vermelidir:

| Port | Protokol | Durum (In/Out) | Görev |
| --- | --- | --- | --- |
| 80/443 | TCP | In/Out | HTTPS (Web) arayüzü ve API çağrıları |
| 7443 | TCP | In/Out | Güvenli WebSocket / FreeSWITCH |
| 16384 - 32768 | UDP | In/Out | Mediasoup / WebRTC (Kamera, Ses RTP / RTCP paketleri) |

*(Eğer Coturn sunucunuz farklı bir makinedeyse, BBB sunucunuzun ayrıca dışarıdaki Coturn sunucusuna UDP 3478 dahilinde erişebilmesi gerekir).*

## 3.2 İç Ağ ve NAT Arkasına Kurulum (Dummy NIC Çözümü)

Kurumsal firmalarda veya ev ağlarında sunucular bazen doğrudan dışarıya bakan statik bir "Public IP" (Dış İP) almazlar. İçeride örneğin `192.168.1.100` adresine sahiptirler ve modem/firewall üzerinden Public IP (örn: `203.0.113.1`) NAT'lanır (Port Forwarding).

BBB kurulumu normal şartlarda sisteminde "Public IP" bulamazsa FreeSWITCH ve WebRTC doğru IP'yi istemcilere anons edemez, bu da "Dışarıdaki insanlar benim sesimi duyamıyor" sorununa (1020 WebRTC hatası) yol açar.

Bu sorunu çözmenin yöntemleri:

### Yöntem 1: Dummy Interface (Sahte Ağ Arabirimi)
Sunucu içine dış IP'mizmiş gibi davranacak sahte bir ağ kartı (Interface) eklemek.

1. `/etc/netplan/` içindeki yapılandırma dosyanıza (ör. `00-installer-config.yaml` veya `50-cloud-init.yaml`) şunu ekleyin:

```yaml
network:
  version: 2
  ethernets:
    ens160: # Orijinal ağ kartınız (örnek ismidir)
      dhcp4: true
    dummy0: # Yeni Eklenen
      match:
        name: dummy0
      addresses:
        - 203.0.113.1/32 # Sizin gerçek Public (Dış) IP'niz
```

2. Sisteme "dummy" modülünü yükleyin:
```bash
sudo modprobe dummy
echo "dummy" | sudo tee -a /etc/modules
```

3. Netplan'ı uygulayın:
```bash
sudo netplan apply
```
*Artık `ifconfig` yazdığınızda sistemin `dummy0` arayüzünde `203.0.113.1` (dış IP) gördüğünü anlayacak ve FreeSWITCH bu adresi anons edecektir.*

### Yöntem 2: FreeSWITCH SIP profiline Extern-IP Eklemek
(Tavsiye edilen dummy'dir, ama zorunlu kalırsanız) `sip_profiles/external.xml` tarzı dosyalardaki `$${local_ip_v4}` değişkenlerini `$${external_rtp_ip}` formatında zorla belirtmek gerekir. (Bu konu `bbb-conf --setip` ile bazen ezildiği için risklidir).

## 3.3 TURN ve STUN (Coturn) Nedir? Neden Gerekir?

### STUN 
Katılımcıya "Senin internette görünen Public IP adresin bu" diyen bir ayna servisidir. 
- Eğer istemci (öğrenci/katılımcı) evinde simetrik NAT/Firewall arkasındaysa sadece STUN işe yaramaz.

### TURN (Traversal Using Relays around NAT)
Katılımcının (veya sunucunun) ağ kısıtlamaları (firewall) veya asimetrik NAT yüzünden **doğrudan** UDP (16384-32768) iletişimini kuramadığı durumlarda aracı bir Tünel gibi davranır.
- İşyeri ağından, katı kısıtlamalı otel ve okul ağlarından bağlanan kullanıcılar TURN sunucusu üzerinden medyayı (443 portundan şifreli olarak) tüneller. 
- Bu sayede 1020 (Medya Bağlantısı Kurulamadı) hataları engellenir.

### Coturn Kurulumu (BBB'den Ayrı Sunucu Önerilir!)

> [!TIP]
> BBB'yi kurduğunuz sunucuya aynı zamanda TURN kurabilirsiniz, fakat yoğunluk arttığında ağ darboğazı olabilir. Tavsiye edilen, `turn.sirket.com` gibi ucuz bir sanal makine (Örn. 2 CPU, 4 GB RAM ubuntu) yaratıp sadece Coturn kurmaktır.

**Ayrı Sunucuda Ubuntu 22.04'e Coturn Kurulumu:**

1. Sunucuyu hazırlayın ve portlarını açın (TCP 443, UDP/TCP 3478, vs). FQDN tanımlayın: `turn.sirket.com`.
2. BBB reposundaki install scriptini kullanarak kurun:
```bash
wget -qO- https://raw.githubusercontent.com/bigbluebutton/bbb-install/refs/heads/v3.0.x-release/bbb-install.sh | bash -s -- -c turn.sirket.com -e admin@sirket.com
```

**BBB Sunucunuzu TURN'e Bağlamak:**

1. `turn.sirket.com` daki *Shared Secret* keyi öğrenin (veya `/etc/turnserver.conf` dosyasından bakın).
2. BBB sunucunuzda şu komutu çalıştırarak TURN adresinizi kaydedin:

```bash
sudo bbb-conf --setturn turn.sirket.com turn_sunucusundaki_gizli_anahtar
```

3. `sudo bbb-conf --check` yazarak `turn.sirket.com` ile düzgün iletişim kurulduğunu kontrol edin.

## 3.4 Çıplak IP Konusu ve IPv6
BigBlueButton kurulumlarında her zaman domain (FQDN) kullanılmalıdır. IP adresi ile SSL sertifikasını geçersiz kılacak şekilde (self-signed) kurabilirsiniz ancak tarayıcılar (Chrome/Firefox vb) "Kameralara ve Mikrofona erişim" iznini güvenli olmayan (HTTP veya geçersiz HTTPS) sitelerde vermeyecektir.
Bazı operatörlerde IPv6 ile `bbb-web` arasında çökmeler olabilir. Sorun yaşarsanız IPv6'yı sunucu bazında devre dışı bırakmayı deneyebilirsiniz:

```bash
sysctl -w net.ipv6.conf.all.disable_ipv6=1
```
