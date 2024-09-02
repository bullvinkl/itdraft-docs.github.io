---
title: "Автоматическая установка Debian при помощи preseed"
date: "2019-12-17"
categories: 
  - Debian
  - Preseed
tags: 
  - "debian"
  - "preseed"
image:
  path: /commons/629418_a5b0-1.jpg
  alt: "Автоматическая установка Debian"
---

> **Preseeding** - метод автоматизации установки операционной системы Debian и ее производных.

## Пример файла preseed.cfg

```
### Localization
d-i debian-installer/language string ru
d-i debian-installer/locale string ru_RU.UTF-8
d-i debian-installer/country string RU
d-i localechooser/supported-locales multiselect en_US.UTF-8, ru_RU.UTF-8

### Keyboard
#d-i keyboard-configuration/xkb-keymap select ru
d-i keymap select ru
d-i console-setup/ask_detect boolean false
d-i console-setup/layoutcode string ru
d-i console-setup/variant   select  Россия
d-i console-setup/toggle    select  Alt+Shift

### Network configuration
d-i netcfg/choose_interface select auto
d-i netcfg/dhcp_timeout string 2
#d-i netcfg/get_hostname string unassigned-hostname
#d-i netcfg/get_domain string unassigned-domain
d-i netcfg/get_hostname string localhost
d-i netcfg/get_domain string localdomain
d-i netcfg/confirm_static boolean false
d-i netcfg/disable_dhcp boolean false

# Static
#d-i netcfg/get_nameservers  string 192.168.1.5 192.168.1.6
#d-i netcfg/get_ipaddress    string 192.168.1.10
#d-i netcfg/get_netmask      string 255.255.255.0
#d-i netcfg/get_gateway      string 192.168.1.1
#d-i netcfg/confirm_static   boolean true
#d-i netcfg/get_hostname string tempnode
#d-i netcfg/get_domain string localdomain

### Repo
#d-i mirror/country string manual
#d-i mirror/http/hostname string mirror.yandex.ru
#d-i mirror/http/directory string /debian
#d-i mirror/http/proxy string

### Timezone
d-i clock-setup/utc boolean true
d-i time/zone string Europe/Moscow
d-i clock-setup/ntp boolean true
d-i clock-setup/ntp-server string 192.168.1.3

### Users
d-i passwd/root-password-crypted password $6$0eoSvBXMw0y0mftg$11111a/RFS7wUD5tjS8lh9iuaqkcK6rp/iay72E9yr0L0IJd7kg.zv742n0yklSQ.W7F3Uk9Lh/
d-i passwd/user-fullname string cfgadmin
d-i passwd/username string cfgadmin
d-i passwd/user-password-crypted password $6$R9QSOFvUWKc816UF$cyX111116/Y4tDAOqsaF8miQdaWTwVj1hV8nlFEXK2HrVg2C3kLTw38xPoGcy5193lhGxS7aJT/
d-i passwd/user-default-groups cfgadmin sudo

### Partition
#d-i partman-auto/disk string /dev/sda
#d-i partman-auto/method string regular
#d-i partman-auto/choose_recipe select atomic
#d-i partman-partitioning/confirm_write_new_label boolean true
#d-i partman/choose_partition select finish
#d-i partman/confirm boolean true
#d-i partman/confirm_nooverwrite boolean true

#d-i partman-auto/disk string /dev/sda
#d-i partman-auto/method string lvm
#d-i partman-lvm/device_remove_lvm boolean true
#d-i partman-lvm/confirm boolean true
#d-i partman-lvm/confirm_nooverwrite boolean true
#d-i partman-auto/choose_recipe select atomic
#d-i partman-partitioning/confirm_write_new_label boolean false
#d-i partman/choose_partition select finish
#d-i partman/confirm boolean false
#d-i partman/confirm_nooverwrite boolean true
#partman-efi partman-efi/non_efi_system boolean true

### Disk partitioning
d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

d-i partman-auto-lvm/new_vg_name string debian
d-i partman-auto-lvm/guided_size string max
#d-i partman-auto/choose_recipe select custom
d-i partman-auto/expert_recipe string  \
  custom ::  \
    256 30 256 ext2  \
        $primary{ }  \
        $bootable{ }  \
        method{ format } format{ }  \
        use_filesystem{ } filesystem{ ext2 }  \
        mountpoint{ /boot }  \
    . \
    16384 30 16384 xfs  \
        $lvmok{ }  lv_name{ root }  \
        method{ format } format{ }  \
        use_filesystem{ } filesystem{ xfs }  \
        mountpoint{ / }  \
    . \
    1024 1025 -1 xfs  \
        $lvmok{ }  lv_name{ var }  \
        method{ format } format{ }  \
        use_filesystem{ } filesystem{ xfs }  \
        mountpoint{ /var }  \
    . \
    4096 30 4096 linux-swap  \
        $lvmok{ } lv_name{ swap }  \
        method{ swap } format{ } \
    . 

# This makes partman automatically partition without confirmation, provided
# that you told it what to do using one of the methods above.
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

### Apt setup
# остальные настройки apt'a - включаем contrib и non-free репозитории. 
# d-i apt-setup/non-free boolean true
# d-i apt-setup/contrib boolean true
# Local repo 
# d-i apt-setup/local0/repository string http://192.168.1.21/debian buster main contrib non-free
# d-i apt-setup/local0/comment string Local repo Debian
# Values shown below are the normal defaults.
# d-i apt-setup/services-select multiselect security, updates
# d-i apt-setup/security_host string security.debian.org

# Некоторые версии программы установки могут отсылать отчёт
# об установленных и используемых пакетах. По умолчанию данная
# возможность выключена, но отправка отчёта помогает проекту
# определить популярность программ и какие из них включать на CD.
# не отправляем данные об установленных пакетах. 
popularity-contest popularity-contest/participate boolean false

### Package selection
tasksel tasksel/first multiselect ssh-server
#tasksel tasksel/first multiselect standard, ssh-server
d-i pkgsel/include string sudo python3-apt aptitude

### Certificate
d-i preseed/late_command string \
  in-target mkdir -p /root/.ssh; \
  in-target chmod 700 /root/.ssh; \
  in-target /bin/sh -c "echo 'ssh-ed25519 AAAAC3Nz111111NTE5AAAAIFCiz11YRaT3/C7QUVOJC5klAunWtFRhHJ9k/+e94dYb ed25519-root' > /root/.ssh/authorized_keys"; \
  in-target chmod 600 /root/.ssh/authorized_keys; \
  in-target mkdir -p /home/myuser/.ssh; \
  in-target chmod 700 /home/myuser/.ssh; \
  in-target /bin/sh -c "echo 'ssh-ed25519 AAAAC3Nz11111TE5AAAAIGv4Pt+Ocj3WEW3u/p8RMlH6r4TqW7qCiTofqnmKGiEe ed25519-myuser' > /home/myuser/.ssh/authorized_keys"; \
  in-target chmod 600 /home/myuser/.ssh/authorized_keys; \
  in-target chown -R myuser:myuser /home/myuser/.ssh/; \
  in-target /bin/sh -c "echo 'myuser ALL=(ALL) NOPASSWD:ALL' | tee -a /etc/sudoers"

d-i finish-install/reboot_in_progress note

### Finish install
d-i finish-install/keep-consoles boolean true
d-i finish-install/reboot_in_progress note
d-i cdrom-detect/eject boolean true

# В случае с виртаульными машинами, лучше всего
# остановливать систему после завершения установки, а
# не перегружаться в установленную систему.
# d-i debian-installer/exit/halt boolean true
# Эта настройка позволяет выключить питание машины, а не просто остановить её.
# d-i debian-installer/exit/poweroff boolean true

### Grub
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true
d-i grub-installer/bootdev string default
```

## Разбор содержимого файла

### Первый блок. Локализация

```
### Localization
d-i debian-installer/language string ru
d-i debian-installer/locale string ru_RU.UTF-8
d-i debian-installer/country string RU
d-i localechooser/supported-locales multiselect en_US.UTF-8, ru_RU.UTF-8
```

В нем мы задаем локализацию и поддерживаемые языки

### Второй блок. Раскладка клавиатуры

```
### Keyboard
#d-i keyboard-configuration/xkb-keymap select ru
d-i keymap select ru
d-i console-setup/ask_detect boolean false
d-i console-setup/layoutcode string ru
d-i console-setup/variant   select  Россия
d-i console-setup/toggle    select  Alt+Shift
```

В нем указаны параметры раскладки клавиатуры

### Третий блок. Сеть

```
### Network configuration
d-i netcfg/choose_interface select auto
d-i netcfg/dhcp_timeout string 2
#d-i netcfg/get_hostname string unassigned-hostname
#d-i netcfg/get_domain string unassigned-domain
d-i netcfg/get_hostname string localhost
d-i netcfg/get_domain string localdomain
d-i netcfg/confirm_static boolean false
d-i netcfg/disable_dhcp boolean false

# Static
#d-i netcfg/get_nameservers  string 192.168.1.5 192.168.1.6
#d-i netcfg/get_ipaddress    string 192.168.1.10
#d-i netcfg/get_netmask      string 255.255.255.0
#d-i netcfg/get_gateway      string 192.168.1.1
#d-i netcfg/confirm_static   boolean true
#d-i netcfg/get_hostname string tempnode
#d-i netcfg/get_domain string localdomain
```

Это блок с сетевыми настройками.  
Раскомментированный - настройки для получения IP с помощью DHCP-сервера.  
Закомментированный - статический IP

### Четвертый блок. Репозиторий

```
### Repo
#d-i mirror/country string manual
#d-i mirror/http/hostname string mirror.yandex.ru
#d-i mirror/http/directory string /debian
#d-i mirror/http/proxy string
```

Указываем репозиторий и если нужно прокси-сервер

### Пятый блок. Временная зона

```
### Timezone
d-i clock-setup/utc boolean true
d-i time/zone string Europe/Moscow
d-i clock-setup/ntp boolean true
d-i clock-setup/ntp-server string 192.168.1.3
```

Настройки временной зоны, ntp-сервера

### Шестой блок. Пользователи

```
### Users
d-i passwd/root-password-crypted password $6$0eoSvBXMw1111.NfjecfghfghfghjS8lh9iuaqkcK6rp/iay72E9yr0L0IJd7kg.zv742n0yklSQ.W7F3Uk9Lh/
d-i passwd/user-fullname string myuser
d-i passwd/username string myuser
d-i passwd/user-password-crypted password $6$R9QSOFvUWK1111F$cyXMFXtfghfghfghfQdaWTwVj1hV8nlFEXK2HrVg2C3kLTw38xPoGcy5193lhGxS7aJT/
d-i passwd/user-default-groups myuser sudo
```

Задаем пароль для root-пользователя, создаем пользователя myuser и добавляем ему права sudo

Для генерации хэшированного пароля воспользуемся утилитой openssl, и сгенерируем пароль

```sh
$ openssl passwd -6
Password: pass
Verifying - Password: pass
$6$guSkbpdN4kWXQBsE$dmr6b8ZA515kaqvCRbOG2MGIIAwYTtnwARVUFPt/JHjjL4BvA3TEx6mQgmYGto14DiwCM7X82X04mxoHNkSwA0
```

### Седьмой блок. Разметка диска

```
### Partition
#d-i partman-auto/disk string /dev/sda
#d-i partman-auto/method string regular
#d-i partman-auto/choose_recipe select atomic
#d-i partman-partitioning/confirm_write_new_label boolean true
#d-i partman/choose_partition select finish
#d-i partman/confirm boolean true
#d-i partman/confirm_nooverwrite boolean true

#d-i partman-auto/disk string /dev/sda
#d-i partman-auto/method string lvm
#d-i partman-lvm/device_remove_lvm boolean true
#d-i partman-lvm/confirm boolean true
#d-i partman-lvm/confirm_nooverwrite boolean true
#d-i partman-auto/choose_recipe select atomic
#d-i partman-partitioning/confirm_write_new_label boolean false
#d-i partman/choose_partition select finish
#d-i partman/confirm boolean false
#d-i partman/confirm_nooverwrite boolean true
#partman-efi partman-efi/non_efi_system boolean true

### Disk partitioning
d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

d-i partman-auto-lvm/new_vg_name string debian
d-i partman-auto-lvm/guided_size string max
#d-i partman-auto/choose_recipe select custom
d-i partman-auto/expert_recipe string  \
  custom ::  \
    256 30 256 ext2  \
        $primary{ }  \
        $bootable{ }  \
        method{ format } format{ }  \
        use_filesystem{ } filesystem{ ext2 }  \
        mountpoint{ /boot }  \
    . \
    16384 30 16384 xfs  \
        $lvmok{ }  lv_name{ root }  \
        method{ format } format{ }  \
        use_filesystem{ } filesystem{ xfs }  \
        mountpoint{ / }  \
    . \
    1024 1025 -1 xfs  \
        $lvmok{ }  lv_name{ var }  \
        method{ format } format{ }  \
        use_filesystem{ } filesystem{ xfs }  \
        mountpoint{ /var }  \
    . \
    4096 30 4096 linux-swap  \
        $lvmok{ } lv_name{ swap }  \
        method{ swap } format{ } \
    . 

# This makes partman automatically partition without confirmation, provided
# that you told it what to do using one of the methods above.
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
```

Disk partitioning - раскомментированный блок, разметка диска:

- использовать весь диск sda

- метод распределения пространства жёсткого диска — LVM

- очистить диск, если на нем что-то было

- Volume Group - debian

- /boot - 512 Mb, ext2

- / - lv_name: root, 16 Gb, xfs

- /var - lv_name: var, все оставшееся место (параметр "-1"), xfs

- swap - 4 Gb

### Восьмой блок. Apt setup

```
### Apt setup
# остальные настройки apt'a - включаем contrib и non-free репозитории. 
# d-i apt-setup/non-free boolean true
# d-i apt-setup/contrib boolean true
# Local repo 
# d-i apt-setup/local0/repository string http://192.168.1.9/debian buster main contrib non-free
# d-i apt-setup/local0/comment string Local repo Debian
# Values shown below are the normal defaults.
# d-i apt-setup/services-select multiselect security, updates
# d-i apt-setup/security_host string security.debian.org
```

Остальные настройки apt, подключаем репозитории

### Девятый блок. Программное обеспечение

```
### Package selection
tasksel tasksel/first multiselect ssh-server
#tasksel tasksel/first multiselect standard, ssh-server
d-i pkgsel/include string sudo python3-apt aptitude
```

Указываем какие сервисы добавлять, какой софт ставить

### Десятый блок. Пользовательские скрипты

```
### Certificate
d-i preseed/late_command string \
  in-target mkdir -p /root/.ssh; \
  in-target chmod 700 /root/.ssh; \
  in-target /bin/sh -c "echo 'ssh-ed25519 AAAAC3NzaC11111AIFCiz11YRaT3/C7fghfghfghfghRhHJ9k/+e94dYb ed25519-root' > /root/.ssh/authorized_keys"; \
  in-target chmod 600 /root/.ssh/authorized_keys; \
  in-target mkdir -p /home/myuser/.ssh; \
  in-target chmod 700 /home/myuser/.ssh; \
  in-target /bin/sh -c "echo 'ssh-ed25519 AAAAC3Nz111111TEdfgrtrthrhfghfghEW3u/p8RMlH6r4TqW7qCiTofqnmKGiEe ed25519-myuser' > /home/myuser/.ssh/authorized_keys"; \
  in-target chmod 600 /home/myuser/.ssh/authorized_keys; \
  in-target chown -R myuser:myuser /home/myuser/.ssh/; \
  in-target /bin/sh -c "echo 'myuser ALL=(ALL) NOPASSWD:ALL' | tee -a /etc/sudoers"
```

Добавляю сертификаты для пользователей root и myuser, отключаю запрос пароля при выполнении sudo пользователем myuser

## Как данным файлом пользоваться?

- Загружаемся с диска

- Advanced options

- Automated install

- После подгрузки необходимых модулей прописываем url к файлу preseed.cfg

![](/assets/img/posts/2019/12/17/Screenshot_9.png){: w="300" }

![](/assets/img/posts/2019/12/17/Screenshot_10.png){: w="300" }

![](/assets/img/posts/2019/12/17/Screenshot_11.png){: w="300" }

Либо добавить пункт меню в образ диска

Устанавливаем софт:

```sh
$ sudo apt install genisoimage isolinux wget xorriso
```

[Скачиваем и выполняем скрипт](https://framagit.org/fiat-tux/hat-softwares/preseed-creator/) автоматической сборки образа из исходного образа

```sh
$ sudo ./preseed_creator.sh -i image.iso -p preseed.cfg -o image_preseed.iso
```
