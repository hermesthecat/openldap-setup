# Detaylı Sudo Yetkisi / Detailed sudo permissions

Belirli Hostlara İzin Verme / Allow access to specific hosts:

```bash
dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: ALL
sudoHost: server1.hermes.local
sudoRunAsUser: ALL
sudoUser: kerem
```

Birden Fazla Host İçin Yetkilendirme / Allow access to multiple hosts:

```bash
dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: ALL
sudoHost: server1.hermes.local
sudoHost: server2.hermes.local
sudoRunAsUser: ALL
sudoUser: kerem
```

Belirli IP Adresleri İçin Yetkilendirme / Allow access to specific IP addresses:

```bash
dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: ALL
sudoHost: 192.168.1.10
sudoHost: 192.168.1.20
sudoRunAsUser: ALL
sudoUser: kerem
```

Belirli Bir IP Aralığı (Subnet) İçin Yetkilendirme / Allow access to a specific IP range (subnet):

```bash
dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: ALL
sudoHost: 192.168.1.0/24
sudoRunAsUser: ALL
sudoUser: kerem

```

sudoHost Wildcard Kullanımı (*) / Wildcard usage (*):

```bash
dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: ALL
sudoHost: *.hermes.local
sudoRunAsUser: ALL
sudoUser: kerem
```

Belirli Komutlara İzin Verme / Allow access to specific commands:

```bash
dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: /sbin/reboot
sudoCommand: /bin/systemctl restart apache2
sudoCommand: /usr/bin/apt update
sudoHost: ALL
sudoRunAsUser: ALL
sudoUser: kerem
```

Belirli bir dizindeki tüm komutlara izin verme / Allow access to all commands in a specific directory:

```bash
dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: /usr/bin/*
sudoHost: ALL
sudoRunAsUser: ALL
sudoUser: kerem
```

Belirli Kullanıcılarla Sınırlama / Restrict access to specific users:

```bash
#sudoRunAsUser: ALL ayarı, kullanıcının herhangi bir kullanıcı olarak (root dahil) komut çalıştırmasına izin verir. Bunu belirli kullanıcılarla sınırlandırabilirsiniz. 

dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: ALL
sudoHost: ALL
sudoRunAsUser: www-data
sudoUser: kerem
```

```bash
#Bu durumda, kerem sadece şu komutu çalıştırabilir:

sudo -u www-data some_command

#Ama sudo -u root some_command çalıştırmasına izin verilmez.

#Sadece www-data ve mysql kullanıcısı olarak çalıştırma izni verme

dn: cn=kerem,ou=sudo,dc=hermes,dc=local
objectClass: sudoRole
objectClass: top
cn: kerem
sudoCommand: ALL
sudoHost: ALL
sudoRunAsUser: www-data
sudoRunAsUser: mysql
sudoUser: kerem
```
