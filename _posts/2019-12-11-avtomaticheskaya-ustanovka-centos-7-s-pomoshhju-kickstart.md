---
title: "Автоматическая установка Centos 7 с помощью kickstart"
date: "2019-12-11"
categories: 
  - Linux
  - Kickstart
tags: 
  - "centos"
  - "kickstart"
image:
  path: /commons/1365176_fbbc.jpg
  alt: "Установка Centos 7 с помощью kickstart"
---

> **kickstart** — метод быстрой установки операционных систем, основанных на Red Hat Linux

Каждый раз, когда вы устанавливаете Centos, в домашней директории пользователя root создается файл,содержащий параметры установки

```sh
$ cat /root
anaconda-ks.cfg
```

## Пример файла kickstart

```
#version=Centos7
# System authorization information
auth --enableshadow --passalgo=sha512

# Install OS instead of upgrade
install

# Reboot after installation
reboot --eject

# License agreement
eula --agreed

# Use CDROM installation media
cdrom
# Use network installation
#url --url="https://mirror.yandex.ru/centos/7/os/x86_64"

# Use graphical install
#graphical
# Use text mode install
text

# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda

# Keyboard layouts
keyboard --vckeymap=us --xlayouts='ru','us' --switch='grp:alt_shift_toggle'

# System language
lang en_US.UTF-8 --addsupport=ru_RU.UTF-8

# Network information
#network  --bootproto=static --device=eth0 --gateway=192.168.77.1 --ip=192.168.77.222 --nameserver=8.8.8.8 --netmask=255.255.255.0 --ipv6=auto --activate
#network  --bootproto=dhcp --device=enp0s3 --ipv6=auto --activate
network  --bootproto=dhcp --ipv6=auto --activate
network  --hostname=localhost.localdomain

# Root password
#rootpw --lock
rootpw --iscrypted $6$0eoSvBXMw0y0mftg$.NfjecMdfgdfgdfgdfgIOa/RFS7wUD5tjS8lh9iuaqkcK6rp/iay72E9yr0L0IJd7kg.zv742n0yklSQ.W7F3Uk9Lh/

# Add user
user --name=admin --groups=wheel --iscrypted --password=$6$R9QSOFvUWKc816UF$cyXMFXtadfger55806/Y4tDAOqsaF8miQdaWTwVj1hV8nlFEXK2HrVg2C3kLTw38xPoGcy5193lhGxS7aJT/

# Add ssh user key
sshkey --username=admin "ssh-ed25519 AAAAC3NzadfgdfgdE5AAAAIGv4Pt+Ocj3WEW3u/p8RMlH6r4TqW7qCiTofqnmKGiEe ed25519-admin"
sshkey --username=root "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AASSDrrfz11YRaT3/C7QUVOJC5klAunWtFRhHJ9k/+e94dYb ed25519-root"

# System services
# убрать лишние сервисы и добавить нужные
#services --disabled=autofs,alsa-state,avahi-daemon,bluetooth,pcscd,cachefilesd,colord,fancontrol,fcoe,firewalld,firstboot-graphical,gdm,httpd,initial-setup,initial-setup-text,initial-setup-graphical,initial-setup-reconfiguration,kdump,libstoragemgmt,ModemManager,tog-pegasus,tmp.mount,tuned \
# --enabled=bacula-fd,chronyd,edac,gpm,numad,rsyslog,sendmail,smartd,sm-client,sssd,zabbix-agent
services --disabled=NetworkManager
services --enabled=chronyd

# System timezone
#timezone Europe/Moscow --isUtc --nontp
timezone Europe/Moscow  --isUtc --ntpservers=192.168.1.2,192.168.1.3

# Firewall rule
#firewall --enabled --port=22822:tcp
#firewall --disabled --service=ssh

# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda

# Partition clearing information
# Указывает, что мы будем повторно инициализировать наш диск. (Рекомендуется при автоматической установке)
zerombr

#clearpart --none --initlabel
#  Указывает, что все разделы будут очищены на диске sda
clearpart --all --initlabel --drives=sda

# Disk partitioning information
#part pv.157 --fstype="lvmpv" --ondisk=sda --size=7167
# grow - Эта команда указывает установщику anaconda создать максимально большой раздел.
# pv.01 - не используется после установки
part pv.01 --fstype="lvmpv" --ondisk=sda --size=1024 --grow
part /boot --fstype="xfs" --ondisk=sda --size=512
volgroup centos --pesize=4096 pv.01
logvol swap --fstype="swap" --size=4096 --name=swap --vgname=centos
#logvol / --fstype="xfs" --maxsize=16384 --size=4096 --name=root --vgname=centos
logvol / --fstype="xfs" --maxsize=16384 --size=16384 --name=root --vgname=centos
logvol /var --fstype="xfs" --size=1024 --grow --name=var --vgname=centos

##### 3. Package installation
#url --url="https://mirror.yandex.ru/centos/7/os/x86_64/"
#repo --install --name="CentOS" --baseurl="http://mirror.centos.org/centos/7/os/x86_64/"
#repo --name="CentOS" --baseurl="https://mirror.yandex.ru/centos/7/os/x86_64/"
#repo --name="EPEL" --baseurl="https://dl.fedoraproject.org/pub/epel/7/x86_64/"
#repo -–name=EPEL -–baseurl=http://download.fedoraproject.org/pub/epel/7/x86_64/

%packages
@^minimal
@core
kexec-tools
chrony
sudo
#policycoreutils-python
# remove from Core:
-aic94xx-firmware
-alsa-firmware
-bfa-firmware
#-dracut-config-rescue
-ivtv-firmware
-iwl1000-firmware
-iwl100-firmware
-iwl105-firmware
-iwl135-firmware
-iwl2000-firmware
-iwl2030-firmware
-iwl3160-firmware
-iwl3945-firmware
-iwl4965-firmware
-iwl5000-firmware
-iwl5150-firmware
-iwl6000-firmware
-iwl6000g2a-firmware
-iwl6000g2b-firmware
-iwl6050-firmware
-iwl7260-firmware
#-kernel-tools
-libertas-sd8686-firmware
-libertas-sd8787-firmware
-libertas-usb8388-firmware
#-microcode_ctl
#-NetworkManager
#-NetworkManager-tui
-ql2100-firmware
-ql2200-firmware
-ql23xx-firmware

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

%post
#yum install -y policycoreutils-python
echo "admin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/admin
# Change ssh port
#/usr/bin/sed -i "s%#Port 22%Port 43389%g" "/etc/ssh/sshd_config"
#/usr/bin/sed -i "s%#PermitRootLogin yes%PermitRootLogin no%g" "/etc/ssh/sshd_config"
#/sbin/semanage port -a -t ssh_port_t -p tcp 22822
#/usr/bin/firewall-cmd --permanent --zone=public --remove-service=ssh
%end
```

## Разбор содержимого файла

Указывается источник cdrom, либо можно использовать установку по сети

```
# Use CDROM installation media
cdrom
# Use network installation
#url --url="https://mirror.yandex.ru/centos/7/os/x86_64"
```

Указывается раскладка, языковая поддержка, клавиши переключения раскладки

```
# Keyboard layouts
keyboard --vckeymap=us --xlayouts='ru','us' --switch='grp:alt_shift_toggle'

# System language
lang en_US.UTF-8 --addsupport=ru_RU.UTF-8
```

Далее идет опция сетевого подключения, hostname. Можно выбрать получение сетевых параметров через DHCP, либо Static IP

```
# Network information
#network  --bootproto=static --device=eth0 --gateway=192.168.77.1 --ip=192.168.77.222 --nameserver=8.8.8.8 --netmask=255.255.255.0 --ipv6=auto --activate
#network  --bootproto=dhcp --device=enp0s3 --ipv6=auto --activate
network  --bootproto=dhcp --ipv6=auto --activate
network  --hostname=localhost.localdomain
```

Затем идут параметры пользователей  
rootpw --lock - запрет подключения к серверу root-ом

```
# Root password
#rootpw --lock
rootpw --iscrypted $6$0eoSvBXMw0y0mftg$.NfjecMdfgdfgdfgdfgIOa/RFS7wUD5tjS8lh9iuaqkcK6rp/iay72E9yr0L0IJd7kg.zv742n0yklSQ.W7F3Uk9Lh/

# Add user
user --name=admin --groups=wheel --iscrypted --password=$6$R9QSOFvUWKc816UF$cyXMFXtadfger55806/Y4tDAOqsaF8miQdaWTwVj1hV8nlFEXK2HrVg2C3kLTw38xPoGcy5193lhGxS7aJT/

# Add ssh user key
sshkey --username=admin "ssh-ed25519 AAAAC3NzadfgdfgdE5AAAAIGv4Pt+Ocj3WEW3u/p8RMlH6r4TqW7qCiTofqnmKGiEe ed25519-admin"
sshkey --username=root "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AASSDrrfz11YRaT3/C7QUVOJC5klAunWtFRhHJ9k/+e94dYb ed25519-root"
```

Пароль пользователя root и админ можно сгенерировать заранее

```sh
$ python -c "import crypt,random,string; print crypt.crypt(\"my_password\", '\$6\$' + ''.join([random.choice(string.ascii_letters + string.digits) for _ in range(16)]))"
```

Сертификаты так же генерируются заранее

Дальше идет разбивка диска:

```
# Disk partitioning information
#part pv.157 --fstype="lvmpv" --ondisk=sda --size=7167
# grow - Эта команда указывает установщику anaconda создать максимально большой раздел.
# pv.01 - не используется после установки
part pv.01 --fstype="lvmpv" --ondisk=sda --size=1024 --grow
part /boot --fstype="xfs" --ondisk=sda --size=512
volgroup centos --pesize=4096 pv.01
logvol swap --fstype="swap" --size=4096 --name=swap --vgname=centos
#logvol / --fstype="xfs" --maxsize=16384 --size=4096 --name=root --vgname=centos
logvol / --fstype="xfs" --maxsize=16384 --size=16384 --name=root --vgname=centos
logvol /var --fstype="xfs" --size=1024 --grow --name=var --vgname=centos
```

- Использовать диск sda полностью (grow)
- Метод распределения пространства жёсткого диска - LVM
- /boor - 512 Mb, xfs
- Volume Group - centos
- SWAP - 4 Gb
- / - 16 Gb, xfs
- /var - все остальное, xfs

Следующий блок - установленные / удаленные (-) пакеты

```
%packages
@^minimal
@core
kexec-tools
chrony
sudo
#policycoreutils-python
# remove from Core:
-aic94xx-firmware
-alsa-firmware
-bfa-firmware
...
```

И последний блок - bash скрипт

```
%post
#yum install -y policycoreutils-python
echo "admin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/admin
# Change ssh port
#/usr/bin/sed -i "s%#Port 22%Port 43389%g" "/etc/ssh/sshd_config"
#/usr/bin/sed -i "s%#PermitRootLogin yes%PermitRootLogin no%g" "/etc/ssh/sshd_config"
#/sbin/semanage port -a -t ssh_port_t -p tcp 22822
#/usr/bin/firewall-cmd --permanent --zone=public --remove-service=ssh
%end
```

- отключаем ввод пароля sudo для пользователя admin
- закомментированный блок для смены стандартного ssh-порта
- закомментированна опция, которая запрещаем коннект root-ом

## Как пользоваться данным файлом?

Можно выложить файл на http/ftp сервер и при установки Centos с диска жмем ESC и прописываем:

```sh
$ vmlinuz initrd=initrd.img inst.ks=http://192.168.1.10/ks.cfg
```

А можно создать свой образ диска, где будет прописана данная опция. Для этого

Создаем точку монтирования, монтируем образ диска

```sh
$ mkdir /mnt/iso
$ mount /home/CentOS-7-x86_64-Minimal-1908.iso /mnt/iso/
```

Создаем еще один каталог, копируем в него содержимое `/mnt/iso`

```sh
$ mkdir /home/centos
$ cp -rp /mnt/iso/* /home/centos/
```

Добавляем наш `kickstart` в образ

Копируем наш kickstart файл в `/home/centos/`, редактируем isolinux/isolinux.cfg

Пункт меню для автоустановки можно вставить например после секции `label linux`

```
label linux
menu label ^Install CentOS Linux 7
kernel vmlinuz
append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet
```

```
label auto
menu label ^Auto install CentOS Linux 7
kernel vmlinuz
append initrd=initrd.img inst.ks=cdrom:/dev/cdrom:/ks.cfg
```

В последней строчке указано расположение `kickstart файла` в образе диска.  
Если планируется дальнейшее редактирование `ks.cfg`, можно указать расположение на `http/ftp` - сервере

```
label auto
menu label ^Auto install CentOS Linux 7
kernel vmlinuz
append initrd=initrd.img inst.ks=http://192.168.1.10/ks.cfg
```

Создаем сам образ:

```sh
$ cd /home/centos/
$ mkisofs -o /home/centos-cust.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -V 'CentOS 7 x86_64' -boot-load-size 4 -boot-info-table -R -J -v -T .
```