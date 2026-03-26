# 🔐 Certs — Self-Signed SSL Sertifika Referans Klasörü

Bu klasör, bir BBB veya Moodle sunucusuna **Self-Signed (kendi imzalı) SSL sertifikası** oluşturmak için örnek yapılandırma dosyasını (`csr.conf`) ve referans sertifika dosyalarını barındırır.

> [!NOTE]
> Bu klasördeki `.key`, `.crt` ve `.pem` dosyaları **örnek/dummy** içeriklidir — gerçek bir sunucuya veya alan adına ait değildir. Yalnızca dokümantasyon ve test amaçlı referans olarak eklenmiştir.

## Sertifika Oluşturma Adımları (`create-cert.txt` içeriğinden)

```bash
# 1. Self-signed sertifika oluşturun
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout bbb.sirket.com.key \
  -out bbb.sirket.com.crt \
  -config csr.conf -extensions v3_req

# 2. Nginx için DH parametresi (Güvenlik için önerilen)
mkdir -p /etc/nginx/ssl/
openssl dhparam -out /etc/nginx/ssl/dhp-2048.pem 2048

# 3. Sertifikayı doğrulayın
openssl x509 -in bbb.sirket.com.crt -text -noout

# 4. Sertifikayı sunucuya tanıtın
sudo cp bbb.sirket.com.crt /usr/local/share/ca-certificates/bbb.sirket.com.crt
sudo update-ca-certificates
```text
## En İyi Pratik

Production ortamında gerçek özel anahtarları (`*.key`) hiçbir zaman Git reposuna göndermeyin.
