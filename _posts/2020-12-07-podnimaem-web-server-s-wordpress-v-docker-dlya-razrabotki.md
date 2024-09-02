---
title: "Поднимаем web-сервер с Wordpress в Docker для разработки"
date: "2020-12-07"
categories: 
  - Linux
  - Docker
  - Wordpress
tags: 
  - "docker"
  - "docker-compose"
  - "kickstart"
  - "nginx"
  - "php-fpm"
  - "wordpress"
image:
  path: /commons/156398bb67a3d4ee7ab97075a1137791.jpg
  alt: "Поднимаем web-сервер с Wordpress в Docker"
---

Иногда для тестирования плагинов, тем, доработки функционала требует чистый Wordpress. В данной статье я продемонстрирую, каким оразом я разворачиваю рабочее окружение с готовым web-сервером (Nginx + php-fpm + MariaDB) и установленным финальным релизом Wordpress.

## Требования

Для работы нам понадобится:

- Программа для виртуализации [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Образ [CentOS 8.2](https://mirror.yandex.ru/centos/8.2.2004/isos/x86_64/CentOS-8.2.2004-x86_64-dvd1.iso) (7.2 Gb)
- Хостинг, на который мы загрузим наш kickstart-файл, для быстрой установки предварительно настроенной ОС

## Подготовка виртуальной машины

Запускаем VirtualBox и создаем новую виртуальную машину:

- Имя: wordpress
- Тип: Linux
- Версия: Red Hat (64-bit)
- Объем памяти: 4096 Mb
- Жесткий диск: создать новый
- Тип жесткого диска: VDI
- Формат: Динамический
- Размер: 20 Gb

![](/assets/img/posts/2020/12/07/wpdoc1.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc2.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc3.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc4.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc5.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc7.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc6.png){: w="300" }

Добавляем скаченный образ в библиотеку VirtualBox, что бы загрузиться с него

![](/assets/img/posts/2020/12/07/wpdoc8.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc9.png){: w="300" }

## Установка CentOS 8

Centos 8 я устанавливаю из kickstart-файла, загруженного на сервер заранее. В этом файле у меня уже прописан: пользователь, пароль, сертификат, список предустановленного ПО

```
#
# vmlinuz initrd=initrd.img inst.ks=http://<your-server>/ks-centos8.cfg
#
# version=RHEL8
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
# Use text install
text
# Keyboard layouts
keyboard --vckeymap=ru --xlayouts='us','ru' --switch='grp:alt_shift_toggle'
# System language
lang en_US.UTF-8 --addsupport=ru_RU.UTF-8

# Network information
# dhcp
#network  --bootproto=dhcp --device=link --ipv6=auto --activate
# static NAT
network --bootproto=static --device=link --gateway=10.0.2.2 --ip=10.0.2.15 --nameserver=8.8.8.8,8.8.4.4 --netmask=255.255.255.0 --ipv6=auto --activate
# static brige
#network --bootproto=static --device=link --gateway=192.168.11.1 --ip=192.168.11.200 --nameserver=192.168.11.1 --netmask=255.255.255.0 --ipv6=auto --activate
network --hostname=localhost.localdomain

# Root password
rootpw --iscrypted $6$lZma0ZiWGD0yY4hW$.T/sRzouE5X.fghfghfghfghfgh/ODmBV7U1PvbcWmWM/7h6oZqkp.6eRDdy3x.YICI441BWk5QfVYDav7Z/
# Add user
user --groups=wheel --name=admin --iscrypted --password=$6$R9QSOFvUWKc816UF$cyXMFXtCSat1zPsqa806/dfgdfgdfgdfgTwVj1hV8nlFEXK2HrVg2C3kLTw38xPoGcy5193lhGxS7aJT/

# Add ssh user key
sshkey --username=root "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFCiz11YRaT3/C7QUVOJdfgdfgdfgdfgdfgdffg9k/+e94dYb ed25519-root"
sshkey --username=admin "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGv4Pt+Ocj3WEdfgdfgdfgdfgdfgdfgddffgdgdfgdfgdiEe ed25519-admin"

# Disable the Setup Agent on first boot
firstboot --disable
# Do not configure the X Window System
skipx
# System services
#services --disabled="chronyd"
# System timezone
timezone Europe/Moscow --isUtc --nontp

# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
# Partition clearing information
zerombr
clearpart --all --initlabel --drives=sda
#clearpart --all --initlabel --drives=sda,sdb
# Disk partitioning information
part /boot --fstype="xfs" --ondisk=sda --size=512
part pv.1874 --fstype="lvmpv" --ondisk=sda --size=1 --grow
#part pv.7906 --fstype="lvmpv" --ondisk=sdb --size=1 --grow
volgroup centos --pesize=4096 pv.1874
#volgroup vg_docker --pesize=4096 pv.7906
logvol swap  --fstype="swap" --size=512 --name=swap --vgname=centos
#logvol /var  --fstype="xfs" --size=1024 --grow --name=var --vgname=centos
logvol /  --fstype="xfs" --size=1024 --grow --name=root --vgname=centos
#logvol /var/lib/docker  --fstype="xfs" --size=1024 --grow --name=lv_docker --vgname=vg_docker

%packages
@^minimal-environment
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

%post
echo "admin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/admin
dnf -y update
dnf -y install epel-release
dnf -y install wget tar zip unzip bzip2 traceroute net-tools nano bind-utils telnet htop atop iftop lsof git rsync policycoreutils-python-utils
%end
```

В этом файле надо поменять хэш паролей пользователей root и admin (заранее сгенерировав их), а так же поменять публичную часть сертификатов пользователей

Запускаем созданную виртуальную машину, и в окне выбора загрузочного диска выбирает присоединенный образ

![](/assets/img/posts/2020/12/07/wpdoc10.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc11.png){: w="300" }

В момент загрузки (когда появится меню загрузочного диска) надо нажать ESC и ввести:

```
vmlinuz initrd=initrd.img inst.ks=http://<your-server>/ks-centos8.cfg
```

![](/assets/img/posts/2020/12/07/wpdoc12.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc13.png){: w="300" }

В процессе того, как устанавливается ОС, жмем правой кнопкой мыши на иконку настройки сети и настраиваем её

- Тип подключения: NAT
- Дополнительно: Проброс портов

![](/assets/img/posts/2020/12/07/wpdoc14.png){: w="300" }

![](/assets/img/posts/2020/12/07/wpdoc15.png){: w="300" }

## Установка Docker и Docker-Compose

После установки и загрузки ОС, запускаем putty или kitty, конектимся к localhost:22 и вводим команды по очереди. Команды вводим без знака $

```sh
$ sudo dnf -y install -y yum-utils device-mapper-persistent-data lvm2
$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
$ sudo dnf -y install docker-ce --nobest
$ sudo usermod -aG docker $(whoami)
$ newgrp docker
$ sudo systemctl enable --now docker
```

```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$ sudo firewall-cmd --zone=public --add-masquerade --permanent
$ sudo firewall-cmd --reload
```

## Установка Web-сервера и Wordpress в Docker

Ну и финальный шаг, клонируем репозиторий, устанавливаем docker / docker-compose и запускаем web-сервер

```sh
$ git clone https://github.com/bullvinkl/wordpress-nginx-docker.git
$ cd wordpress-nginx-docker
$ docker-compose up -d
```

Запускаем браузер, переходим по адресу: [locahost](http://localhost)

Видео для наглядности: [Youtube](https://youtu.be/Pz-YLLzkEBQ)