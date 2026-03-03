# Bölüm 12: Greenlight v3 İleri Düzey Yönetim ve Kullanıcı Entegrasyonları (2026 Standartları)

Eğer LMS (Moodle, Canvas) platformlarına sahip değilseniz, 2026 yılı standartlarında işletmeniz veya okulunuz için en ideal arayüz BigBlueButton'ın resmi ön yüzü olan **Greenlight v3**'tür.

Greenlight v3, önceki sürümlerden farklı olarak daha modern (React frontend, Ruby backend, PostgreSQL veritabanı) ve daha yetenekli bir mimariye sahiptir.

## 12.1 Yönetici Paneli ve Rol Yapılandırması (RBAC)

Greenlight v3, Rol Tabanlı Erişim Kontrolü (Role-Based Access Control) sunar. 

- **Super Administrator:** Sitenin ana rengini (branding), kayıt olma politikalarını ve tüm diğer kullanıcıları yönetir.
- **Administrator:** Oda oluşturabilir, diğerlerinin kayıtlarını (recordings) ve odalarını görebilir.
- **User:** Sadece kendi odasını yaratır, kaydını alır, toplantıya davet linki üretir. Kullanıcı diğer odaları göremez.
- **Guest (Misafir):** Hesabı olmayan ancak link ile ("Katıl") butonuna basarak gelen kişilerdir. Greenlight bu kişilere oturum açtırmaz, sadece toplantıya sokar.

`Site Settings` altından "Roller" kısmına gelip yeni roller yaratabilir (Örneğin: "Öğretmen" rolü, "Müdür" rolü) ve bunlara farklı oda açma limitleri koyabilirsiniz.

## 12.2 Kurumsal Kimlik Doğrulama: LDAP ve Active Directory

Eğer 500 personelli bir şirketiniz varsa ve hepsi ayrı bir şifre ezberlemek istemiyorsa (Windows Domain veya OpenLDAP altyapısı mevcuttur), personelin Greenlight'a mevcut şirket e-posta ve şifreleriyle girmesini sağlayabilirsiniz.

**LDAP Entegrasyon Adımları (`~/greenlight-v3/.env`):**

Greenlight'ın kök `.env` dosyasını `nano` ile açıp şu parametreleri aktif etmelisiniz (başındaki `#` kaldırılır):

```env
# LDAP Yapılandırması
LDAP_SERVER=192.168.1.50      # Active Directory sunucu IP'niz.
LDAP_PORT=389                 # Güvenli değilse 389, LDAPS ise 636
LDAP_METHOD=none              # Eğer SSL yoksa none, varsa ssl/tls
LDAP_UID=sAMAccountName       # Active Directory için kullanıcı adı özelliği
LDAP_BASE=DC=sirket,DC=local  # Şirket OU'nuz (Search Base)
LDAP_BIND_DN=CN=ReadUser,OU=Admin,DC=sirket,DC=local
LDAP_PASSWORD=AdminSifreniz
```
Değişiklikten sonra `docker compose down && docker compose up -d` yapın. Artık Gelişmiş Ayarlar altında kullanıcılar kendi AD bilgileriyle sisteme sızasız (Seamless) login olabilecektir.

## 12.3 OAuth2 (Google / Office 365) ile E-posta Üzerinden Giriş

Modern okullar "Google Workspace" (GSuite) veya "Microsoft 365" üzerinden mail kullanırlar. Öğrencilerin "Google ile Giriş Yap" (SSO/OAuth) butonuna tıklaması için:

**Office 365 (Azure AD) Parametreleri:**
```env
# Office 365 Entegrasyonu
OFFICE365_KEY=AzureAD_Application_ID_Buraya
OFFICE365_SECRET=AzureAD_Client_Secret_Buraya
OFFICE365_TENANT=Tenant_ID_Buraya 
```
Microsoft Azure Portal üzerinden bir `App Registration` yapıp Callback (Yönlendirme) URL'sine `https://bbb.sirket.com/auth/office365/callback` adresini kaydetmeniz yeterlidir.

## 12.4 Yük Dengeleme (Load Balancing) Greenlight v3

Bölüm 8'de işlediğimiz Scalelite konseptine alternatif olarak:
Sadece frontend olarak Greenlight kullanıyorsanız, v3'ün kendi iç yük dengeleyicisi vardır.

Yönetici (Administrator) paneline giriş yapın -> **"Server Configurations" (Sunucu Ayarları)** -> **"Add Provider"**.
Buraya `bbb01`, `bbb02` ve `bbb03`'ün URL ve Secret anahtarlarını sırayla girin.
Greenlight, yeni açılan her toplantıyı otomatik olarak yükü en hafif olan (ping veya concurrent user sayısı en az olan) sunucuya atamaya başlayacaktır. (Ancak ortak NFS depolaması hala sizin sorumluluğunuzdadır).
