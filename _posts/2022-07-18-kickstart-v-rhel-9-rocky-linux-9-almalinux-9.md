---
title: "Kickstart в RHEL 9 / Rocky linux 9 / AlmaLinux 9"
date: "2022-07-18"
categories: 
  - Automation
tags: 
  - "almalinux"
  - "centos"
  - "kickstart"
  - "linux"
  - "rhel"
  - "rocky-linux"
image:
  path: /commons/1c84bc2f-5343-4bee-b45c-60299c453816.png
  alt: "Kickstart в RHEL-like 9"
---

> **Kickstart** - это метод быстрой установки операционных систем, основанных на Red Hat Linux. Он позволяет автоматизировать процесс установки ОС, упрощая задачу создания конфигурационного файла ответов. Для облегчения этого процесса существует программный пакет с графическим интерфейсом - system-config-kickstart.
{: .prompt-tip }

С выходом RHEL 9 / AlmaLinux 9 /Rocky linux 9 были внесены изменения

- Удаленная авторизация пользователя root по умолчанию отключена (наконец то)

Что бы вернуть авторизацию пользователя root надо выполнить команды:

```sh
# echo "PermitRootLogin yes" > /etc/ssh/sshd_config.d/01-permitrootlogin.conf
# systemctl restart sshd
```

- Настройки сетевого интерфейса полностью переделаны и теперь расположены:

```
/etc/NetworkManager/system-connections/{NAME}.nmconnection
```

Удален пакет сетевых скриптов. Для настройки сетевого интерфейса все так же можно использовать псевдо-графическую утилиту `nmtui`, либо консольную утилиту `mncli`

- Удалена поддержка настройки «SELINUX=disabled» для отключения SELinux в /etc/selinux/config

Для отключения SELinux надо выполнить команды:

```sh
$ sudo grubby --update-kernel ALL --args selinux=0
$ sudo reboot
```

Так же были внесены изменения в конфиг авто установщика (kickstart)

- убрали параметр "install" (# Install OS instead of upgrade)

- внесли изменения в настройки системного языка (# System language): `lang en_US --addsupport=ru_RU`

- убрали область `%anaconda`

Таким образом, мой конфиг для тестовых ВМ для Virtualbox выглядит следующим образом:

```
# version=RHEL9
# Use text install
text
# License agreement
eula --agreed
# Reboot after installation
reboot --eject
# System language
lang en_US --addsupport=ru_RU
# Keyboard layouts
keyboard --vckeymap=ru --xlayouts='us','ru' --switch='grp:alt_shift_toggle'
# System authorization information
auth --enableshadow --passalgo=sha512
# Use CDROM installation media
cdrom

# Network information
#network  --bootproto=dhcp --device=link --ipv6=auto --activate
network --bootproto=static --device=link --gateway=10.0.2.2 --ip=10.0.2.15 --nameserver=8.8.8.8,8.8.4.4 --netmask=255.255.255.0 --ipv6=auto --activate
network --hostname=localhost

# Root password
rootpw --iscrypted $6$тут-зашифрованный-пароль/
# Add user
user --groups=wheel --name=admin --iscrypted --password=$6$тут-зашифрованный-пароль/

# Add ssh user key
sshkey --username=root "ssh-ed25519 AAAAтут-публичная-часть-сертификата ed25519-root"
sshkey --username=admin "ssh-ed25519 AAAAтут-публичная-часть-сертификата ed25519-admin"

# Disable the Setup Agent on first boot
firstboot --disable
# Do not configure the X Window System
skipx
# System services
#services --disabled="chronyd"
# System timezone
timezone Europe/Moscow --utc

# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
# Partition clearing information
zerombr
clearpart --all --initlabel --disklabel=gpt --drives=sda
#clearpart --all --initlabel --drives=sda,sdb

# Disk partitioning information
part /boot --fstype="xfs" --size=1024 --label="boot" --ondisk=sda
part biosboot --fstype="biosboot" --size=1 --ondisk=sda
#part /boot/efi --fstype="xfs" --size=200 --label="efi" --ondisk=sda
part pv.1874 --fstype="lvmpv" --ondisk=sda --size=1 --grow
#part pv.7906 --fstype="lvmpv" --ondisk=sdb --size=1 --grow
volgroup rl --pesize=4096 pv.1874
#volgroup vg_docker --pesize=4096 pv.7906
logvol swap  --fstype="swap" --size=1024 --name=swap --vgname=rl
#logvol /var  --fstype="xfs" --size=1024 --grow --name=var --vgname=rl
logvol /  --fstype="xfs" --size=1024 --grow --name=root --vgname=rl
#logvol /var/lib/docker  --fstype="xfs" --size=1024 --grow --name=lv_docker --vgname=vg_docker

# Create repositories
repo --name=BaseOS --baseurl=https://download.rockylinux.org/pub/rocky/9/BaseOS/x86_64/os/
repo --name=AppStream --baseurl=https://download.rockylinux.org/pub/rocky/9/AppStream/x86_64/os/
repo --name=Extras --baseurl=https://download.rockylinux.org/pub/rocky/9/extras/x86_64/os/

%packages
@^minimal-environment
wget
curl
traceroute
net-tools
nano
bind-utils
telnet
lsof
git
rsync
policycoreutils-python-utils
tcpdump
mlocate
cloud-utils-growpart

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%post
echo "admin ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/admin
dnf -y update
dnf -y install epel-release
dnf -y install htop atop iftop
%end
```
