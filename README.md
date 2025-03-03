# OpenLDAP Kurulum Kılavuzu

Bu belge, Alma Linux 8 üzerinde OpenLDAP sunucu ve istemci kurulumunu detaylı bir şekilde anlatmaktadır.

## Yazar
A. Kerem Gök

## Genel Bakış

Bu kılavuz iki ana bölümden oluşmaktadır:
1. OpenLDAP Sunucu Kurulumu (`server-setup.md`)
2. OpenLDAP İstemci Kurulumu (`client-setup.md`)

## Sistem Gereksinimleri

### Sunucu
- **İşletim Sistemi:** Alma Linux 8.6
- **IP Adresi:** 192.168.205.3
- **Hostname:** ldapmaster.hermes.local
- **Domain:** hermes.local
- **Şifre:** azadazad
- **SSHA şifresi:** {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz

### İstemci
- **İşletim Sistemi:** Alma Linux 8.6
- **IP Adresi:** 192.168.205.4
- **Hostname:** ldapclient.hermes.local
- **Domain:** hermes.local

## LDAP Terminolojisi

- CN – Common Name (Ortak Ad)
- O – Organizational (Organizasyonel)
- OU – Organizational Unit (Organizasyonel Birim)
- SN – Last Name (Soyadı)
- DC – Domain Component (Alan Bileşeni)
- DN – Distinguished Name (Ayırt Edici Ad)

## Kurulum Adımları

### Sunucu Kurulumu

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

Detaylı kurulum adımları için `server-setup.md` dosyasına bakınız.

### İstemci Kurulumu

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

Detaylı kurulum adımları için `client-setup.md` dosyasına bakınız.

## Test ve Doğrulama

### Sunucu Tarafında
1. LDAP servisi durumunu kontrol edin
2. Kullanıcıları listeleyin
3. Sudo yetkilerini kontrol edin

### İstemci Tarafında
1. LDAP bağlantısını test edin
2. Kullanıcı girişini test edin
3. Sudo yetkilerini test edin

## Güvenlik Önlemleri

1. Şifre Güvenliği
   - Güçlü şifreler kullanın
   - Şifreleri güvenli bir şekilde saklayın
   - Düzenli şifre değişimi politikası uygulayın

2. SSL/TLS Güvenliği
   - Sertifikaları düzgün yapılandırın
   - Gerekli güvenlik protokollerini kullanın
   - Sertifikaları düzenli olarak yenileyin

3. Yetkilendirme
   - Sudo yetkilerini dikkatli atayın
   - En az yetki prensibini uygulayın
   - Yetkileri düzenli olarak gözden geçirin

4. Ağ Güvenliği
   - Güvenlik duvarı kurallarını doğru yapılandırın
   - Gerekli portları açın
   - Düzenli güvenlik denetimi yapın

## Sorun Giderme

1. Servis Sorunları
   - Servis durumlarını kontrol edin
   - Log dosyalarını inceleyin
   - Yapılandırma dosyalarını kontrol edin

2. Bağlantı Sorunları
   - Ağ bağlantısını test edin
   - DNS çözümlemesini kontrol edin
   - Güvenlik duvarı ayarlarını kontrol edin

3. Yetkilendirme Sorunları
   - LDAP bağlantısını kontrol edin
   - Kullanıcı yetkilerini doğrulayın
   - SSSD yapılandırmasını kontrol edin

## Önemli Notlar

- Bu kurulum kılavuzu test ortamı için hazırlanmıştır
- Üretim ortamında daha sıkı güvenlik önlemleri alınmalıdır
- Düzenli yedekleme ve güncelleme planı oluşturulmalıdır
- Tüm yapılandırma değişikliklerini belgelendirin
- Güvenlik güncellemelerini düzenli olarak takip edin 