# OpenLDAP Kurulum Kılavuzu
# OpenLDAP Installation Guide

Bu belge, Alma Linux 8 üzerinde OpenLDAP sunucu ve istemci kurulumunu detaylı bir şekilde anlatmaktadır. / This document provides detailed instructions for installing OpenLDAP server and client on Alma Linux 8.

## Yazar / Author
A. Kerem Gök

## Genel Bakış
## Overview

Bu kılavuz iki ana bölümden oluşmaktadır / This guide consists of two main sections:

1. OpenLDAP Sunucu Kurulumu (`server-setup.md`) / OpenLDAP Server Installation (`server-setup.md`)
2. OpenLDAP İstemci Kurulumu (`client-setup.md`) / OpenLDAP Client Installation (`client-setup.md`)

## Sistem Gereksinimleri
## System Requirements

### Sunucu / Server
- **İşletim Sistemi / Operating System:** Alma Linux 8.6
- **IP Adresi / IP Address:** 192.168.205.3
- **Hostname / Hostname:** ldapmaster.hermes.local
- **Domain / Domain:** hermes.local
- **Şifre / Password:** azadazad
- **SSHA şifresi / SSHA password:** {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz

### İstemci / Client
- **İşletim Sistemi / Operating System:** Alma Linux 8.6
- **IP Adresi / IP Address:** 192.168.205.4
- **Hostname / Hostname:** ldapclient.hermes.local
- **Domain / Domain:** hermes.local

## LDAP Terminolojisi
## LDAP Terminology

- CN – Common Name (Ortak Ad)
- O – Organizational (Organizasyonel)
- OU – Organizational Unit (Organizasyonel Birim)
- SN – Last Name (Soyadı)
- DC – Domain Component (Alan Bileşeni)
- DN – Distinguished Name (Ayırt Edici Ad)

## Kurulum Adımları
## Installation Steps

### Sunucu Kurulumu
### Server Installation

1. Sistem Güncellemeleri / System Updates
   - RPM anahtarı güncelleme / RPM key update
   - Temel paketlerin kurulumu / Basic package installation
   - Sistem güncellemelerinin yapılması / System updates installation

2. Ağ Yapılandırması / Network Configuration
   - Hostname ayarları / Hostname settings
   - Hosts dosyası güncellemeleri / Hosts file updates

3. OpenLDAP Kurulumu / OpenLDAP Installation
   - Symas OpenLDAP paketlerinin kurulumu / Symas OpenLDAP packages installation
   - SLAPD veritabanı yapılandırması / SLAPD database configuration
   - Güvenlik duvarı ayarları / Firewall settings

4. LDAP Yapılandırması / LDAP Configuration
   - Admin şifresi oluşturma / Create admin password
   - Şema dosyalarının import edilmesi / Import schema files
   - Domain ayarları / Domain settings
   - OU (Organizational Unit) yapılandırması / OU (Organizational Unit) configuration

5. SSL/TLS Yapılandırması / SSL/TLS Configuration
   - Sertifika oluşturma / Certificate creation
   - OpenLDAP SSL ayarları / OpenLDAP SSL settings

6. Kullanıcı Yönetimi / User Management
   - Test kullanıcılarının oluşturulması (kerem ve abdullah) / Create test users (kerem and abdullah)
   - Sudo yetkilerinin yapılandırılması / Configure sudo permissions

Detaylı kurulum adımları için `server-setup.md` dosyasına bakınız. / For detailed installation steps, please refer to `server-setup.md`.

### İstemci Kurulumu
### Client Installation

1. Sistem Hazırlığı / System Preparation
   - Temel güncellemeler / Basic updates
   - Hostname ve hosts ayarları / Hostname and hosts settings

2. Gerekli Paketlerin Kurulumu / Required Packages Installation
   - OpenLDAP istemci paketleri / OpenLDAP client packages
   - SSSD ve ilgili bileşenler / SSSD and related components

3. LDAP İstemci Yapılandırması / LDAP Client Configuration
   - OpenLDAP istemci ayarları / OpenLDAP client settings
   - SSSD yapılandırması / SSSD configuration
   - Yetkilendirme ayarları / Authorization settings

4. Sudo Yapılandırması / Sudo Configuration
   - SSSD sudo entegrasyonu / SSSD sudo integration
   - nsswitch.conf düzenlemeleri / nsswitch.conf modifications

Detaylı kurulum adımları için `client-setup.md` dosyasına bakınız. / For detailed installation steps, please refer to `client-setup.md`. 

## Test ve Doğrulama
## Testing and Verification

### Sunucu Tarafında
### Server Side
1. LDAP servisi durumunu kontrol edin / Check LDAP service status
2. Kullanıcıları listeleyin / List users
3. Sudo yetkilerini kontrol edin / Check sudo permissions

### İstemci Tarafında
### Client Side
1. LDAP bağlantısını test edin / Test LDAP connection
2. Kullanıcı girişini test edin / Test user login
3. Sudo yetkilerini test edin / Test sudo permissions

## Güvenlik Önlemleri
## Security Measures

1. Şifre Güvenliği / Password Security
   - Güçlü şifreler kullanın / Use strong passwords
   - Şifreleri güvenli bir şekilde saklayın / Store passwords securely
   - Düzenli şifre değişimi politikası uygulayın / Implement regular password change policy

2. SSL/TLS Güvenliği / SSL/TLS Security
   - Sertifikaları düzgün yapılandırın / Configure certificates properly
   - Gerekli güvenlik protokollerini kullanın / Use required security protocols
   - Sertifikaları düzenli olarak yenileyin / Renew certificates regularly

3. Yetkilendirme / Authorization
   - Sudo yetkilerini dikkatli atayın / Assign sudo permissions carefully
   - En az yetki prensibini uygulayın / Apply principle of least privilege
   - Yetkileri düzenli olarak gözden geçirin / Review permissions regularly

4. Ağ Güvenliği / Network Security
   - Güvenlik duvarı kurallarını doğru yapılandırın / Configure firewall rules correctly
   - Gerekli portları açın / Open required ports
   - Düzenli güvenlik denetimi yapın / Perform regular security audits

## Sorun Giderme
## Troubleshooting

1. Servis Sorunları / Service Issues
   - Servis durumlarını kontrol edin / Check service status
   - Log dosyalarını inceleyin / Examine log files
   - Yapılandırma dosyalarını kontrol edin / Check configuration files

2. Bağlantı Sorunları / Connection Issues
   - Ağ bağlantısını test edin / Test network connection
   - DNS çözümlemesini kontrol edin / Check DNS resolution
   - Güvenlik duvarı ayarlarını kontrol edin / Check firewall settings

3. Yetkilendirme Sorunları / Authorization Issues
   - LDAP bağlantısını kontrol edin / Check LDAP connection
   - Kullanıcı yetkilerini doğrulayın / Verify user permissions 
   - SSSD yapılandırmasını kontrol edin / Check SSSD configuration

## Önemli Notlar
## Important Notes

- Bu kurulum kılavuzu test ortamı için hazırlanmıştır / This installation guide is prepared for test environment
- Üretim ortamında daha sıkı güvenlik önlemleri alınmalıdır / Stricter security measures should be taken in production environment
- Düzenli yedekleme ve güncelleme planı oluşturulmalıdır / Regular backup and update plan should be created
- Tüm yapılandırma değişikliklerini belgelendirin / Document all configuration changes
- Güvenlik güncellemelerini düzenli olarak takip edin / Follow security updates regularly 