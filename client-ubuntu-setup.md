# Ubuntu Server 24.04 OpenLDAP İstemci Kurulumu / Ubuntu Server 24.04 OpenLDAP Client Installation

## Yazar / Author

A. Kerem Gök

## Sistem Bilgileri / System Information

- **İşletim Sistemi / Operating System:** Ubuntu Server 24.04 LTS
- **IP Adresi / IP Address:** 192.168.205.5
- **Hostname / Hostname:** ldapclient-ubuntu.hermes.local
- **Domain / Domain:** hermes.local

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
# Temel paketlerin kurulumu / Install basic packages
sudo apt install nano wget -y

# Sistem güncellemelerinin yapılması / Perform system updates
sudo apt update -y
sudo apt upgrade -y

# Gerekirse sistemi yeniden başlatma / Reboot if necessary
[ -f /var/run/reboot-required ] && sudo reboot -f
```

### 2. Hostname Yapılandırması / Hostname Configuration

```bash
# Hostname ayarlama / Set hostname
sudo hostnamectl set-hostname ldapclient-ubuntu.hermes.local

# Hosts dosyasını güncelleme / Update hosts file
sudo echo "192.168.205.3 ldapmaster.hermes.local" >> /etc/hosts
sudo echo "192.168.205.5 ldapclient-ubuntu.hermes.local" >> /etc/hosts

# Hosts dosyasını kontrol etme / Check hosts file
cat /etc/hosts
```

### 3. Gerekli Paketlerin Kurulumu / Required Packages Installation

```bash
# LDAP istemci paketlerinin kurulumu / Install LDAP client packages
sudo apt -y install libnss-ldapd libpam-ldapd ldap-utils libsss-sudo
```

### 4. Authselect Yapılandırması / Authselect Configuration

```bash
sudo nano /etc/nsswitch.conf
```

Aşağıdaki satırları düzenleyin / Edit the following lines:

```bash
passwd:         files systemd ldap sss
group:          files systemd ldap sss
shadow:         files systemd ldap sss
```

PAM yapılandırmasını düzenleme / Edit PAM configuration:

```bash
sudo pam-auth-update --enable mkhomedir
```

### 5. OpenLDAP İstemci Yapılandırması / OpenLDAP Client Configuration

```bash
# LDAP yapılandırma dosyasını düzenleme / Edit LDAP configuration file
sudo nano /etc/nslcd.conf
```

Aşağıdaki içeriği ekleyin / Add the following content:

```bash
URI ldap://ldapmaster.hermes.local
BASE dc=hermes,dc=local
SUDOERS_BASE ou=sudo,dc=hermes,dc=local
```

### 6. SSSD Yapılandırması / SSSD Configuration

```bash
# SSSD yapılandırma dosyasını oluşturma / Create SSSD configuration file
sudo nano /etc/sssd/sssd.conf
```

Aşağıdaki içeriği ekleyin / Add the following content:

```bash
[domain/default]
id_provider = ldap
autofs_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldap://ldapmaster.hermes.local
ldap_search_base = dc=hermes,dc=local
sudoers_base ou=sudo,dc=hermes,dc=local
sudo_provider = ldap
ldap_id_use_start_tls = True
ldap_tls_cacertdir = /etc/openldap/certs
cache_credentials = True
ldap_tls_reqcert = allow

[sssd]
services = nss, pam, autofs, sudo
domains = default

[nss]
homedir_substring = /home

[sudo]
```

```bash
# Dosya izinlerini ayarlama / Set file permissions
sudo chmod 0600 /etc/sssd/sssd.conf

# SSSD servisini yeniden başlatma / Restart SSSD service
sudo systemctl restart sssd

# Servis durumunu kontrol etme / Check service status
systemctl status sssd
```

### 7. Bağlantı Testi / Connection Test

```bash
# LDAP kullanıcısını sorgulama / Query LDAP user
ldapsearch -x -b "uid=kerem,ou=people,dc=hermes,dc=local"

# SSH ile bağlantı testi / Test SSH connection
ssh kerem@192.168.205.5
```

### 8. Sudo Yapılandırması / Sudo Configuration

```bash
# nsswitch.conf dosyasını düzenleme / Edit nsswitch.conf file
sudo nano /etc/nsswitch.conf
```

Aşağıdaki satırı ekleyin / Add the following line:

```bash
sudoers: files sss
```

```bash
# SSSD servisini yeniden başlatma / Restart SSSD service
sudo systemctl restart sssd
```

## Test ve Doğrulama / Testing and Verification

1. LDAP Bağlantı Testi / LDAP Connection Test

   ```bash
   ldapsearch -x cn=kerem -b dc=hermes,dc=local
   ```

2. Kullanıcı Girişi / User Login

   ```bash
   ssh kerem@192.168.205.4
   ```

3. Sudo Yetkileri / Sudo Permissions

   ```bash
   sudo -i
   ```

## Sorun Giderme / Troubleshooting

- Servis durumlarını kontrol edin / Check service status
- Log dosyalarını inceleyin / Examine log files
- Bağlantı ayarlarını doğrulayın / Verify connection settings
- Yetkilendirme ayarlarını kontrol edin / Check authorization settings

## Güvenlik Notları / Security Notes

- Tüm şifreleri güvenli bir şekilde saklayın / Store all passwords securely
- SSL/TLS sertifikalarını düzgün yapılandırın / Configure SSL/TLS certificates properly
- Sudo yetkilerini dikkatli bir şekilde atayın / Assign sudo permissions carefully
- Üretim ortamında daha sıkı güvenlik önlemleri alın / Take stricter security measures in production environment
