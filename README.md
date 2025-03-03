# OpenLDAP Kurulum Kılavuzu

Bu belge, Alma Linux 8 üzerinde OpenLDAP sunucu ve istemci kurulumunu detaylı bir şekilde anlatmaktadır.

## Yazar
A. Kerem Gök

## Sistem Gereksinimleri

### Sunucu
- OS: Alma Linux 8.6
- IP: 192.168.205.3
- Hostname: ldapmaster.hermes.local
- Domain: hermes.local

### İstemci
- OS: Alma Linux 8.6
- IP: 192.168.205.4
- Hostname: ldapclient.hermes.local
- Domain: hermes.local

## Temel Bilgiler

LDAP Terminolojisi:
- CN – Common Name (Ortak Ad)
- O – Organizational (Organizasyonel)
- OU – Organizational Unit (Organizasyonel Birim)
- SN – Last Name (Soyadı)
- DC – Domain Component (Alan Bileşeni)
- DN – Distinguished Name (Ayırt Edici Ad)

## Sunucu Kurulumu

1. Sistem Güncellemeleri
   - RPM anahtarı güncelleme
   - Temel paketlerin kurulumu
   - Sistem güncellemelerinin yapılması

2. Ağ Yapılandırması
   - Hostname ayarları
   - Hosts dosyası güncellemeleri

3. OpenLDAP Kurulumu
   - Symas OpenLDAP paketlerinin kurulumu
   - SLAPD veritabanı yapılandırması
   - Güvenlik duvarı ayarları

4. LDAP Yapılandırması
   - Admin şifresi oluşturma
   - Şema dosyalarının import edilmesi
   - Domain ayarları
   - OU (Organizational Unit) yapılandırması

5. SSL/TLS Yapılandırması
   - Sertifika oluşturma
   - OpenLDAP SSL ayarları

6. Kullanıcı Yönetimi
   - Test kullanıcılarının oluşturulması (kerem ve abdullah)
   - Sudo yetkilerinin yapılandırılması

## İstemci Kurulumu

1. Sistem Hazırlığı
   - Temel güncellemeler
   - Hostname ve hosts ayarları

2. Gerekli Paketlerin Kurulumu
   - OpenLDAP istemci paketleri
   - SSSD ve ilgili bileşenler

3. LDAP İstemci Yapılandırması
   - OpenLDAP istemci ayarları
   - SSSD yapılandırması
   - Yetkilendirme ayarları

4. Sudo Yapılandırması
   - SSSD sudo entegrasyonu
   - nsswitch.conf düzenlemeleri

## Güvenlik Notları

- Tüm şifreler güvenli bir şekilde saklanmalıdır
- SSL/TLS sertifikaları düzgün yapılandırılmalıdır
- Sudo yetkileri dikkatli bir şekilde atanmalıdır

## Test ve Doğrulama

1. Sunucu Tarafı:
   - LDAP servis kontrolü
   - Kullanıcı listeleme
   - Sudo yetkilerinin kontrolü

2. İstemci Tarafı:
   - Bağlantı testi
   - Kullanıcı girişi
   - Sudo yetkilerinin testi

## Sorun Giderme

- Servis durumlarını kontrol edin
- Log dosyalarını inceleyin
- Güvenlik duvarı ayarlarını kontrol edin
- Sertifika yapılandırmalarını doğrulayın

## Önemli Notlar

- Bu kurulum kılavuzu test ortamı için hazırlanmıştır
- Üretim ortamında daha sıkı güvenlik önlemleri alınmalıdır
- Düzenli yedekleme ve güncelleme planı oluşturulmalıdır 