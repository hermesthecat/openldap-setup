# Alma Linux 8 OpenLDAP İstemci Kurulumu

## Yazar
A. Kerem Gök

## Sistem Bilgileri

- **İşletim Sistemi:** Alma Linux 8.6
- **IP Adresi:** 192.168.205.4
- **Hostname:** ldapclient.hermes.local
- **Domain:** hermes.local

## Ön Gereksinimler

### LDAP Terminolojisi
- CN – Common Name
- O – Organizational
- OU – Organizational Unit
- SN – Last Name
- DC – Domain Component
- DN – Distinguished Name

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
sudo hostnamectl set-hostname ldapclient.hermes.local

# Hosts dosyasını güncelleme
sudo echo "192.168.205.3 ldapmaster.hermes.local" >> /etc/hosts
sudo echo "192.168.205.4 ldapclient.hermes.local" >> /etc/hosts

# Hosts dosyasını kontrol etme
cat /etc/hosts
```

### 3. Gerekli Paketlerin Kurulumu

```bash
# LDAP istemci paketlerinin kurulumu
sudo dnf install openldap-clients sssd sssd-ldap oddjob-mkhomedir libsss_sudo -y

# Kurulumu kontrol etme
rpm -qa | grep ldap
```

### 4. Authselect Yapılandırması

```bash
# Mevcut profilleri listeleme
authselect list

# SSSD profilini seçme
sudo authselect select sssd with-mkhomedir --force

# Oddjobd servisini başlatma
sudo systemctl enable --now oddjobd.service

# Servis durumunu kontrol etme
systemctl status oddjobd.service
```

### 5. OpenLDAP İstemci Yapılandırması

```bash
# /etc/openldap/ldap.conf dosyasını düzenleme
sudo nano /etc/openldap/ldap.conf
```

Aşağıdaki içeriği ekleyin:
```
URI ldap://ldapmaster.hermes.local
BASE dc=hermes,dc=local
SUDOERS_BASE ou=sudo,dc=hermes,dc=local
```

### 6. SSSD Yapılandırması

```bash
# /etc/sssd/sssd.conf dosyasını oluşturma
sudo nano /etc/sssd/sssd.conf
```

Aşağıdaki içeriği ekleyin:
```
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
# Dosya izinlerini ayarlama
sudo chmod 0600 /etc/sssd/sssd.conf

# SSSD servisini yeniden başlatma
sudo systemctl restart sssd

# Servis durumunu kontrol etme
systemctl status sssd
```

### 7. Bağlantı Testi

```bash
# LDAP kullanıcısını sorgulama
ldapsearch -x -b "uid=kerem,ou=people,dc=hermes,dc=local"

# SSH ile bağlantı testi
ssh kerem@192.168.205.4
```

### 8. Sudo Yapılandırması

```bash
# nsswitch.conf dosyasını düzenleme
sudo nano /etc/nsswitch.conf
```

Aşağıdaki satırı ekleyin:
```
sudoers: files sss
```

```bash
# SSSD servisini yeniden başlatma
sudo systemctl restart sssd
```

## Test ve Doğrulama

1. Kullanıcı girişi yapın:
   ```bash
   ssh kerem@192.168.205.4
   ```

2. Sudo yetkilerini test edin:
   ```bash
   sudo -i
   ```

## Sorun Giderme

- Servis durumlarını kontrol edin
- Log dosyalarını inceleyin
- Bağlantı ayarlarını doğrulayın
- Yetkilendirme ayarlarını kontrol edin

## Güvenlik Notları

- Tüm şifreleri güvenli bir şekilde saklayın
- SSL/TLS sertifikalarını düzgün yapılandırın
- Sudo yetkilerini dikkatli bir şekilde atayın 