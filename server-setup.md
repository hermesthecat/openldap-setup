# Alma Linux 8 OpenLDAP Sunucu Kurulumu

## Yazar
A. Kerem Gök

## Sistem Bilgileri

- **İşletim Sistemi:** Alma Linux 8.6
- **IP Adresi:** 192.168.205.3
- **Hostname:** ldapmaster.hermes.local
- **Domain:** hermes.local
- **Şifre:** azadazad
- **SSHA şifresi:** {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz
- **Test Kullanıcıları:** kerem ve abdullah

## LDAP Terminolojisi

- CN – Common Name (Ortak Ad)
- O – Organizational (Organizasyonel)
- OU – Organizational Unit (Organizasyonel Birim)
- SN – Last Name (Soyadı)
- DC – Domain Component (Alan Bileşeni)
- DN – Distinguished Name (Ayırt Edici Ad)

## Kurulum Adımları

### 1. Sistem Güncellemeleri

```bash
# RPM anahtarı güncelleme
rpm --import https://repo.almalinux.org/almalinux/RPM-GPG-KEY-AlmaLinux

# Temel paketlerin kurulumu
yum install nano wget -y

# Sistem güncellemelerinin yapılması
sudo dnf update -y

# Gerekirse sistemi yeniden başlatma
[ -f /var/run/reboot-required ] && sudo reboot -f
```

### 2. Hostname Yapılandırması

```bash
# Hostname ayarlama
sudo hostnamectl set-hostname ldapmaster.hermes.local

# Hosts dosyasını güncelleme
sudo echo "192.168.205.3 ldapmaster.hermes.local" >> /etc/hosts
sudo echo "192.168.205.4 ldapclient.hermes.local" >> /etc/hosts

# Hosts dosyasını kontrol etme
cat /etc/hosts
```

### 3. OpenLDAP Paketlerinin Kurulumu

```bash
# OpenLDAP repo ekleme
sudo wget -q https://repo.symas.com/configs/SOFL/rhel8/sofl.repo -O /etc/yum.repos.d/sofl.repo

# Paketlerin kurulumu
sudo dnf install symas-openldap-clients symas-openldap-servers -y

# Kurulumu kontrol etme
rpm -qa | grep ldap
```

### 4. SLAPD Veritabanı Yapılandırması

```bash
# Örnek veritabanı konfigürasyonunu kopyalama
sudo cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
sudo chown ldap. /var/lib/ldap/DB_CONFIG 

# SLAPD servisini etkinleştirme ve başlatma
sudo systemctl enable --now slapd

# Servis durumunu kontrol etme
systemctl status slapd
```

### 5. Güvenlik Duvarı Ayarları

```bash
# LDAP servislerini güvenlik duvarına ekleme
sudo firewall-cmd --add-service={ldap,ldaps} --permanent
sudo firewall-cmd --reload
```

### 6. LDAP Admin Şifresi Yapılandırması

```bash
# Admin şifresi oluşturma
slappasswd

# Root şifre güncelleme dosyası oluşturma
sudo nano /root/changerootpw.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz
```

```bash
# Şifreyi güncelleme
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /root/changerootpw.ldif
```

### 7. Şema Dosyalarının İçe Aktarılması

```bash
# Temel şemaların import edilmesi
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

# Sudo şeması ayarlaması
sudo cp /usr/share/doc/sudo/schema.OpenLDAP /etc/openldap/schema/sudo.schema
```

Sudo şeması için LDIF dosyası oluşturma:
```bash
sudo tee /etc/openldap/schema/sudo.ldif<<EOF
dn: cn=sudo,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: sudo
olcAttributeTypes: ( 1.3.6.1.4.1.15953.9.1.1 NAME 'sudoUser' DESC 'User(s) who may  run sudo' EQUALITY caseExactIA5Match SUBSTR caseExactIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )
olcAttributeTypes: ( 1.3.6.1.4.1.15953.9.1.2 NAME 'sudoHost' DESC 'Host(s) who may run sudo' EQUALITY caseExactIA5Match SUBSTR caseExactIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )
olcAttributeTypes: ( 1.3.6.1.4.1.15953.9.1.3 NAME 'sudoCommand' DESC 'Command(s) to be executed by sudo' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )
olcAttributeTypes: ( 1.3.6.1.4.1.15953.9.1.4 NAME 'sudoRunAs' DESC 'User(s) impersonated by sudo (deprecated)' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )
olcAttributeTypes: ( 1.3.6.1.4.1.15953.9.1.5 NAME 'sudoOption' DESC 'Options(s) followed by sudo' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )
olcAttributeTypes: ( 1.3.6.1.4.1.15953.9.1.6 NAME 'sudoRunAsUser' DESC 'User(s) impersonated by sudo' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )
olcAttributeTypes: ( 1.3.6.1.4.1.15953.9.1.7 NAME 'sudoRunAsGroup' DESC 'Group(s) impersonated by sudo' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )
olcObjectClasses: ( 1.3.6.1.4.1.15953.9.2.1 NAME 'sudoRole' SUP top STRUCTURAL DESC 'Sudoer Entries' MUST ( cn ) MAY ( sudoUser $ sudoHost $ sudoCommand $ sudoRunAs $ sudoRunAsUser $ sudoRunAsGroup $ sudoOption $ description ) )
EOF

# Sudo şemasını uygulama
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/sudo.ldif
```

### 8. Domain Yapılandırması

```bash
# Domain güncelleme dosyası oluşturma
sudo nano /root/setdomainname.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=hermes,dc=local

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=hermes,dc=local

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=Manager,dc=hermes,dc=local" read by * none
```

```bash
# Domain güncellemesini uygulama
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/setdomainname.ldif
```

### 9. Organizasyon Birimlerinin (OU) Oluşturulması

```bash
# OU yapılandırma dosyası oluşturma
sudo nano /root/adddomain.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: dc=hermes,dc=local
objectClass: top
objectClass: dcObject
objectclass: organization
o: My example Organisation
dc: hermes

dn: cn=Manager,dc=hermes,dc=local
objectClass: organizationalRole
cn: Manager
description: OpenLDAP Manager

dn: ou=People,dc=hermes,dc=local
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=hermes,dc=local
objectClass: organizationalUnit
ou: Group
```

```bash
# OU'ları oluşturma
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/adddomain.ldif
```

### 10. Test Kullanıcılarının Oluşturulması

Kerem kullanıcısı için:
```bash
sudo nano /root/kerem.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: uid=kerem,ou=People,dc=hermes,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: kerem
sn: temp
userPassword: {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz
loginShell: /bin/bash
uidNumber: 2000
gidNumber: 2000
homeDirectory: /home/kerem
shadowLastChange: 0
shadowMax: 0
shadowWarning: 0

dn: cn=kerem,ou=Group,dc=hermes,dc=local
objectClass: posixGroup
cn: kerem
gidNumber: 2000
memberUid: kerem
```

Abdullah kullanıcısı için:
```bash
sudo nano /root/abdullah.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: uid=abdullah,ou=People,dc=hermes,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: abdullah
sn: temp
userPassword: {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz
loginShell: /bin/bash
uidNumber: 2001
gidNumber: 2001
homeDirectory: /home/abdullah
shadowLastChange: 0
shadowMax: 0
shadowWarning: 0

dn: cn=abdullah,ou=Group,dc=hermes,dc=local
objectClass: posixGroup
cn: abdullah
gidNumber: 2001
memberUid: abdullah
```

```bash
# Kullanıcıları oluşturma
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/kerem.ldif
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/abdullah.ldif

# Kullanıcıları listeleme
ldapsearch -x cn=kerem -b dc=hermes,dc=local
ldapsearch -x cn=abdullah -b dc=hermes,dc=local
```

### 11. SSL/TLS Yapılandırması

```bash
# Sertifika oluşturma
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/pki/tls/ldapserver.key -out /etc/pki/tls/ldapserver.crt

# Sahiplik ayarları
sudo chown ldap:ldap /etc/pki/tls/{ldapserver.crt,ldapserver.key}
```

TLS yapılandırma dosyası oluşturma:
```bash
sudo nano /root/add-tls.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/pki/tls/ldapserver.crt
-
add: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/pki/tls/ldapserver.key
-
add: olcTLSCertificateFile
olcTLSCertificateFile: /etc/pki/tls/ldapserver.crt
```

```bash
# TLS yapılandırmasını uygulama
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /root/add-tls.ldif

# LDAP yapılandırma dosyasını güncelleme
sudo nano /etc/openldap/ldap.conf
```

Aşağıdaki satırı ekleyin:
```
TLS_CACERT     /etc/pki/tls/ldapserver.crt
```

```bash
# Servisi yeniden başlatma
sudo systemctl restart slapd
```

### 12. Sudo Yapılandırması

```bash
# Sudo OU oluşturma
sudo nano /root/sudoers.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: ou=sudo,dc=hermes,dc=local
objectClass: organizationalUnit
objectClass: top
ou: sudo
description: my-demo LDAP SUDO Entry
```

```bash
# Sudo OU'yu ekleme
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/sudoers.ldif
```

Sudo varsayılan ayarları:
```bash
sudo nano /root/sudo-defaults.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: cn=defaults,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: defaults
sudoOption: env_reset
sudoOption: mail_badpass
sudoOption: secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
```

```bash
# Sudo varsayılanlarını ekleme
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/sudo-defaults.ldif
```

Kerem kullanıcısına sudo yetkisi verme:
```bash
sudo nano /root/sudo-kerem.ldif
```

Aşağıdaki içeriği ekleyin:
```
dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: ALL
sudoHost: ALL
sudoRunAsUser: ALL
sudoUser: kerem
```

```bash
# Sudo yetkisini verme
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/sudo-kerem.ldif
```

## Test ve Doğrulama

1. LDAP servisi durumunu kontrol edin:
   ```bash
   systemctl status slapd
   ```

2. Kullanıcıları sorgulayın:
   ```bash
   ldapsearch -x cn=kerem -b dc=hermes,dc=local
   ldapsearch -x cn=abdullah -b dc=hermes,dc=local
   ```

## Sorun Giderme

- Servis durumlarını kontrol edin
- Log dosyalarını inceleyin
- Güvenlik duvarı ayarlarını kontrol edin
- Sertifika yapılandırmalarını doğrulayın

## Güvenlik Notları

- Tüm şifreleri güvenli bir şekilde saklayın
- SSL/TLS sertifikalarını düzgün yapılandırın
- Sudo yetkilerini dikkatli bir şekilde atayın
- Üretim ortamında daha sıkı güvenlik önlemleri alın 