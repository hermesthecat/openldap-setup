# Alma Linux 8 OpenLDAP Sunucu Kurulumu
# Alma Linux 8 OpenLDAP Server Installation

## Yazar / Author
A. Kerem Gök

## Sistem Bilgileri / System Information

- **İşletim Sistemi / Operating System:** Alma Linux 8.6
- **IP Adresi / IP Address:** 192.168.205.3
- **Hostname / Hostname:** ldapmaster.hermes.local
- **Domain / Domain:** hermes.local
- **Şifre / Password:** azadazad
- **SSHA şifresi / SSHA password:** {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz
- **Test Kullanıcıları / Test Users:** kerem ve abdullah / kerem and abdullah

## LDAP Terminolojisi / LDAP Terminology

- CN – Common Name (Ortak Ad)
- O – Organizational (Organizasyonel)
- OU – Organizational Unit (Organizasyonel Birim)
- SN – Last Name (Soyadı)
- DC – Domain Component (Alan Bileşeni)
- DN – Distinguished Name (Ayırt Edici Ad)

## Kurulum Adımları / Installation Steps

### 1. Sistem Güncellemeleri / System Updates

```bash
# RPM anahtarı güncelleme / Update RPM key
rpm --import https://repo.almalinux.org/almalinux/RPM-GPG-KEY-AlmaLinux

# Temel paketlerin kurulumu / Install basic packages
yum install nano wget -y

# Sistem güncellemelerinin yapılması / Perform system updates
sudo dnf update -y

# Gerekirse sistemi yeniden başlatma / Reboot if necessary
[ -f /var/run/reboot-required ] && sudo reboot -f
```

### 2. Hostname Yapılandırması / Hostname Configuration

```bash
# Hostname ayarlama / Set hostname
sudo hostnamectl set-hostname ldapmaster.hermes.local

# Hosts dosyasını güncelleme / Update hosts file
sudo echo "192.168.205.3 ldapmaster.hermes.local" >> /etc/hosts
sudo echo "192.168.205.4 ldapclient.hermes.local" >> /etc/hosts

# Hosts dosyasını kontrol etme / Check hosts file
cat /etc/hosts
```

### 3. OpenLDAP Paketlerinin Kurulumu / OpenLDAP Packages Installation

```bash
# OpenLDAP repo ekleme / Add OpenLDAP repo
sudo wget -q https://repo.symas.com/configs/SOFL/rhel8/sofl.repo -O /etc/yum.repos.d/sofl.repo

# Paketlerin kurulumu / Install packages
sudo dnf install symas-openldap-clients symas-openldap-servers -y

# Kurulumu kontrol etme / Verify installation
rpm -qa | grep ldap
```

### 4. SLAPD Veritabanı Yapılandırması / SLAPD Database Configuration

```bash
# Örnek veritabanı konfigürasyonunu kopyalama / Copy sample database configuration
sudo cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
sudo chown ldap. /var/lib/ldap/DB_CONFIG 

# SLAPD servisini etkinleştirme ve başlatma / Enable and start SLAPD service
sudo systemctl enable --now slapd

# Servis durumunu kontrol etme / Check service status
systemctl status slapd
```

### 5. Güvenlik Duvarı Ayarları / Firewall Settings

```bash
# LDAP servislerini güvenlik duvarına ekleme / Add LDAP services to firewall
sudo firewall-cmd --add-service={ldap,ldaps} --permanent
sudo firewall-cmd --reload
```

### 6. LDAP Admin Şifresi Yapılandırması / LDAP Admin Password Configuration

```bash
# Admin şifresi oluşturma / Create admin password
slappasswd

# Root şifre güncelleme dosyası oluşturma / Create root password update file
sudo nano /root/changerootpw.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
```
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}2jEeYRSHerfUNztuwVDHnQtDSKrMKrXz
```

```bash
# Şifreyi güncelleme / Update password
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /root/changerootpw.ldif
```

### 7. Şema Dosyalarının İçe Aktarılması / Schema Files Import

```bash
# Temel şemaların import edilmesi / Import basic schemas
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

# Sudo şeması ayarlaması / Setup sudo schema
sudo cp /usr/share/doc/sudo/schema.OpenLDAP /etc/openldap/schema/sudo.schema
```

Sudo şeması için LDIF dosyası oluşturma / Create LDIF file for sudo schema:
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

# Sudo şemasını uygulama / Apply sudo schema
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/sudo.ldif
```

### 8. Domain Yapılandırması / Domain Configuration

```bash
# Domain güncelleme dosyası oluşturma / Create domain update file
sudo nano /root/setdomainname.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
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
# Domain güncellemesini uygulama / Apply domain update
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/setdomainname.ldif
```

### 9. Organizasyon Birimlerinin (OU) Oluşturulması / Creating Organizational Units (OU)

```bash
# OU yapılandırma dosyası oluşturma / Create OU configuration file
sudo nano /root/adddomain.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
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
# OU'ları oluşturma / Create OUs
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/adddomain.ldif
```

### 10. Test Kullanıcılarının Oluşturulması / Creating Test Users

Kerem kullanıcısı için / For Kerem user:
```bash
sudo nano /root/kerem.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
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

Abdullah kullanıcısı için / For Abdullah user:
```bash
sudo nano /root/abdullah.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
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
# Kullanıcıları oluşturma / Create users
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/kerem.ldif
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/abdullah.ldif

# Kullanıcıları listeleme / List users
ldapsearch -x cn=kerem -b dc=hermes,dc=local
ldapsearch -x cn=abdullah -b dc=hermes,dc=local
```

### 11. SSL/TLS Yapılandırması / SSL/TLS Configuration

```bash
# Sertifika oluşturma / Create certificate
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/pki/tls/ldapserver.key -out /etc/pki/tls/ldapserver.crt

# Sahiplik ayarları / Set ownership
sudo chown ldap:ldap /etc/pki/tls/{ldapserver.crt,ldapserver.key}
```

TLS yapılandırma dosyası oluşturma / Create TLS configuration file:
```bash
sudo nano /root/add-tls.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
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
# TLS yapılandırmasını uygulama / Apply TLS configuration
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /root/add-tls.ldif

# LDAP yapılandırma dosyasını güncelleme / Update LDAP configuration file
sudo nano /etc/openldap/ldap.conf
```

Aşağıdaki satırı ekleyin / Add the following line:
```
TLS_CACERT     /etc/pki/tls/ldapserver.crt
```

```bash
# Servisi yeniden başlatma / Restart service
sudo systemctl restart slapd
```

### 12. Sudo Yapılandırması / Sudo Configuration

```bash
# Sudo OU oluşturma / Create Sudo OU
sudo nano /root/sudoers.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
```
dn: ou=sudo,dc=hermes,dc=local
objectClass: organizationalUnit
objectClass: top
ou: sudo
description: my-demo LDAP SUDO Entry
```

```bash
# Sudo OU'yu ekleme / Add Sudo OU
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/sudoers.ldif
```

Sudo varsayılan ayarları / Sudo default settings:
```bash
sudo nano /root/sudo-defaults.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
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
# Sudo varsayılanlarını ekleme / Add sudo defaults
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/sudo-defaults.ldif
```

Kerem kullanıcısına sudo yetkisi verme / Grant sudo permissions to Kerem user:
```bash
sudo nano /root/sudo-kerem.ldif
```

Aşağıdaki içeriği ekleyin / Add the following content:
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
# Sudo yetkisini verme / Grant sudo permissions
sudo ldapadd -x -D cn=Manager,dc=hermes,dc=local -W -f /root/sudo-kerem.ldif
```

## Test ve Doğrulama / Testing and Verification

1. LDAP servisi durumunu kontrol edin / Check LDAP service status:
   ```bash
   systemctl status slapd
   ```

2. Kullanıcıları sorgulayın / Query users:
   ```bash
   ldapsearch -x cn=kerem -b dc=hermes,dc=local
   ldapsearch -x cn=abdullah -b dc=hermes,dc=local
   ```

## Sorun Giderme / Troubleshooting

- Servis durumlarını kontrol edin / Check service status
- Log dosyalarını inceleyin / Examine log files
- Güvenlik duvarı ayarlarını kontrol edin / Check firewall settings
- Sertifika yapılandırmalarını doğrulayın / Verify certificate configurations

## Güvenlik Notları / Security Notes

- Tüm şifreleri güvenli bir şekilde saklayın / Store all passwords securely
- SSL/TLS sertifikalarını düzgün yapılandırın / Configure SSL/TLS certificates properly
- Sudo yetkilerini dikkatli bir şekilde atayın / Assign sudo permissions carefully
- Üretim ortamında daha sıkı güvenlik önlemleri alın / Take stricter security measures in production environment 