# Kurumsal Kimlik Yönetimi ve SSO (LDAP / SAML2 / OAuth2)

Moodle ve BigBlueButton'ı (BBB) binlerce kullanıcılı kurumsal bir ortama (Üniversite, Holding, Kamu) entegre ederken manuel kullanıcı açmak operasyonel bir yüktür. Kullanıcıların mevcut e-posta adresleri veya şirket şifreleri ile her iki sisteme giriş yapabilmesi (Single Sign-On - SSO) bir zorunluluktur.

Bu belgede, Moodle ve Greenlight (BBB arayüzü) sistemlerini Microsoft Entra ID (eski adıyla Azure AD), Google Workspace veya şirket içi Active Directory (LDAP) gibi merkezi kimlik sağlayıcılarına nasıl bağlayacağınızı inceleyeceğiz.

## 1. Active Directory (LDAP) Entegrasyonu

Şirketinizin kendi sunucularında çalışan klasik bir Active Directory varsa, en hızlı entegrasyon yöntemi LDAP'tır.

### 1.1 Moodle LDAP Yapılandırması

Moodle yerleşik bir LDAP eklentisi ile gelir.

1. **Eklentiyi Aktifleştirin:** `Site Administration > Plugins > Authentication > Manage authentication` altından `LDAP server`'ı (Göz ikonuna tıklayarak) aktif edin ve ayarlarına girin.
2. **Sunucu Ayarları:**
   * **Host URI:** `ldap://dc01.sirket.local` veya SSL kullanıyorsanız `ldaps://dc01.sirket.local` (Önerilen).
   * **Version:** 3
   * **TLS Tipi:** Eğer `ldap://` kullanıp güvenlik istiyorsanız "Use TLS" = Yes.
3. **Bağlantı Ayarları (Bind):**
   * **Distinguished name:** `CN=MoodleRead,OU=ServiceAccounts,DC=sirket,DC=local` (Sadece okuma yetkisi olan özel bir servis hesabı (Service Account) açın).
   * **Password:** Servis hesabının şifresi.
4. **Kullanıcı Arama Ayarları:**
   * **Contexts:** `OU=Ogrenciler,DC=sirket,DC=local; OU=Akademisyenler,DC=sirket,DC=local` (Moodle sadece bu OU'lar içinde kullanıcı arar).
   * **Search subcontexts:** Yes
   * **User attribute:** `sAMAccountName` (Active Directory için standart kullanıcı adı niteliği).
5. **Data Mapping (Veri Eşleme):**
   * AD'deki kullanıcının adı, soyadı, e-posta gibi bilgilerini Moodle alanlarına eşleyin.
   * `First name` → `givenName`
   * `Surname` → `sn`
   * `Email address` → `mail`
   * **Update local:** On Every Login (Her girişte AD'den bilgiyi çekip Moodle'ı günceller).

### 1.2 Greenlight (BBB) LDAP Yapılandırması

Greenlight (v3) ortam dosyasında (`.env`) LDAP parametreleri doldurulmalıdır.

```bash
cd ~/greenlight-v3
sudo nano .env
```

```env
# Varsayılan kimlik doğrulama yöntemini değiştirin
AUTH_METHOD=ldap

# LDAP Ayarları
LDAP_SERVER=dc01.sirket.local
LDAP_PORT=389  # Veya SSL için 636
LDAP_METHOD=tls # Veya ssl/plain
LDAP_UID=sAMAccountName
LDAP_BASE=DC=sirket,DC=local
LDAP_BIND_DN=CN=MoodleRead,OU=ServiceAccounts,DC=sirket,DC=local
LDAP_PASSWORD=servis_sifresi
```

Değişiklikleri uygulayın: `docker compose down && docker compose up -d`. Artık kullanıcılarınız şirket şifreleriyle toplantı odası açabilir.

---

## 2. SAML2 Entegrasyonu (Modern Kurumsal Çözüm)

LDAP eski, şifreyi ağ üzerinden (şifrelenmiş olsa bile) gönderen bir protokoldür. Günümüzde **Microsoft Entra ID (Azure AD)** veya **Keycloak** kullanılıyorsa, endüstri standardı SAML2 (Security Assertion Markup Language) olmalıdır.

SAML2 mimarisinde kullanıcı şifresini asla Moodle'a girmez; Moodle, kullanıcıyı Microsoft/Google giriş sayfasına yönlendirir, doğrulama orada yapılır ve Moodle'a "Bu kullanıcı doğrulandı, e-postası budur, adı şudur" diyen kriptolu bir token (Assertion) döner.

### 2.1 Moodle SAML2 Eklentisi Kurulumu

Moodle çekirdeğinde SAML2 yoktur, popüler olan `auth_saml2` (Catalyst IT) eklentisini Git ile kurmalıyız:

```bash
cd /var/www/moodle/public/auth
sudo git clone https://github.com/catalyst/moodle-auth_saml2.git saml2
sudo chown -R root:www-data saml2
sudo -u www-data php8.3 /var/www/moodle/public/admin/cli/upgrade.php --non-interactive
```

### 2.2 Entra ID (Azure AD) Tarafı — Enterprise Application

1. Microsoft Entra admin paneline girin: `Enterprise applications > New application > Create your own application` (Non-gallery). Adını "Moodle LMS" yapın.
2. **Single sign-on** seçeneğini **SAML** olarak belirleyin.
3. Microsoft size üç bilgi verecektir: bunlara `IdP (Identity Provider) Metadata` denir. Özellikle `App Federation Metadata Url` linki sizin için en önemlisidir.

### 2.3 Moodle Tarafında SAML Yapılandırması

1. `Site Administration > Plugins > Authentication > SAML2` sayfasına gidin.
2. **IdP metadata XML OR URL:** Entra ID'den kopyaladığınız Metadata URL'sini (<kbd>https://login.microsoftonline.com/.../federationmetadata.xml</kbd>) buraya yapıştırıp kaydedin.
3. Moodle'ın sizin Entra ID uygulamanıza vereceği bir yanıt URL'si olacaktır. `Site Administration > Plugins > Authentication > SAML2 > Settings` sayfasının en altında bulunan kendi `SP Metadata XML` bağlantınızı kopyalayın.
4. Bu bilgiyi (SP Metadata) Entra ID tarafına geri dönüp `Upload metadata file` sekmesinden yükleyin. Moodle ve Microsoft artık şifreli bir güven tazeledi.

### 2.4 SAML Veri Eşleştirme (Data Mapping)

Entra ID, Moodle'a varsayılan olarak `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname` gibi çok uzun isimli nitelikler gönderir.

Moodle SAML2 ayarlarında **Data mapping** sekmesinde şu eşleştirmeleri yapın:
- **First name:** `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`
- **Surname:** `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`
- **Email:** `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` (VEYA `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`)

Artık Moodle giriş sayfasında kullanıcı adı/şifre yerine "Microsoft ile Giriş Yap" şeklinde kurumsal bir buton olacaktır.

---

## 3. Moodle Üzerinden BBB'ye Otomatik SSO

Peki kullanıcılarınız Moodle'a Microsoft Single Sign-On ile giriş yaptıktan sonra, dersteki BBB etkinliği için tekrar Greenlight'a girmek zorunda mı?

**HAYIR.**

Moodle ile BBB arasındaki LTI tabanlı `mod_bigbluebuttonbn` eklentisi arka planda kendi güven tünelini (`Secret Key` ve `Checksum`) kullanır.
Öğrenci Moodle'da "Toplantıya Katıl" butonuna bastığında, Moodle API aracılığıyla BBB sunucusuna şu komutu gönderir:

> *"Benim sistemimde Ahmet Yılmaz (ahmet@sirket.com) adında, Viewer (Öğrenci) rolüne sahip ve doğrulanmış bir kullanıcım var. Bu kullanıcı senin internalID'si XYZ-123 olan odana girmek istiyor. Bu kullanıcıyı kabul et."*

Bu sayede, kullanıcılar Moodle'a başarılı şekilde kimlik doğruladıklarında (ister LDAP, ister SAML, ister yerel hesap olsun), BBB'ye bir daha şifre girmeden, otomatik olarak isimleri ve rolleriyle (Öğretmenler moderatör olarak) bağlanırlar.

> [!TIP]
> **Tavsiye Edilen Mimari:** Eğer ana eğitim platformunuz Moodle ise, öğrencilere Greenlight (BBB arayüzü) adresini hiç vermeyin. `mod_bigbluebuttonbn` köprüsünü kurun. Tüm öğrenciler sadece `moodle.sirket.com` adresine Microsoft (SAML) hesaplarıyla giriş yapsın ve oradan direkt odalara bağlansın. Bu, kimlik karmaşasını sıfıra indirir.

---

## 4. Kullanıcı Temizliği (Life-Cycle Management)

SSO ile içeri binlerce kişi aktardınız, peki kişi şirketten/üniversiteden ayrıldığında Moodle'dan nasıl düşecek?

**Moodle Auth Plugin Ayarları:**
`Auth plugins` altında bulunan `Remove external users` seçeneğini **Suspend internal** olarak ayarlayın ve Sistem Cron'unun bu eşitlemeyi gece 03:00'te yapmasını (`Sync users` taskı) sağlayın. Böylece öğrenci AD'den silindiği gece Moodle'dan da yetkileri alınmış olur (`suspended`), ancak Moodle'daki sınav notları ve forum yazıları raporlama amacıyla korunur. Asla "Delete" seçeneğini kullanmayın.
