---
title: "Перенос GitLab на другой сервер и обновление GitLab"
date: "2018-06-05"
categories: 
  - Linux
  - GitLab
tags: 
  - "centos"
  - "gitlab"
image:
  path: /commons/penguins.png
  alt: "Перенос GitLab. Обновление"
---

> **GitLab** — это веб-приложение и система управления репозиториями программного кода для Git, предназначенная для хранения кода и совместной разработки масштабных программных проектов. Она обеспечивает управление версиями, отслеживание изменений, тестирование и развертывание кода, а также предоставляет инструменты для командной разработки и управления проектами.
{: .prompt-tip }

Имеется GitLab версии 7.4.1, установленную из исходников. Необходимо обновить его до актуальной версии с переносом всех данных.

План действия следующий:

- Установить на новый сервер GitLab той же версии, что стоит на старом сервере
- На старом сервере сделать бэкап данных средствами GitLab
- Развернуть бэкап на новом сервере
- Обновить GitLab до актуальной версии

## Установить на новый сервер GitLab той же версии, что стоит на старом сервере

Добавляем репозиторий EPEL и обновляемся

```sh
$ sudo yum -y install epel-release
$ sudo yum update
```

Ставим софт

```sh
$ sudo yum -y install curl openssh-server openssh-clients postfix policycoreutils-python
$ sudo yum -y install mc nano wget htop git rsync p7zip ntpdate
```

Отключаем SELinux

```sh
$ sudo setenforce 0
$ sudo nano /etc/selinux/config
	SELINUX=disabled
```

На новом сервере ставим GitLab той же версии, что и на старом сервере

Архивы старых версий: [https://about.gitlab.com/downloads/archives/](https://about.gitlab.com/downloads/archives/)

Скачиваем дистрибутив и устанавливаем его

```sh
$ sudo cd /home
$ wget https://downloads-packages.s3.amazonaws.com/centos-7.0.1406/gitlab-7.4.1_omnibus-1.el7.x86_64.rpm
$ sudo rpm -ivh gitlab-7.4.1_omnibus-1.el7.x86_64.rpm
```

- `i` - Установить пакет
- `v` - показать отладочную информацию
- `h` - выводить хэш-меток при установке

Добавляем правила в firewall

```sh
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
```

Редактируем файл `gitlab.rb` для последующей генерации конфига GitLab

```sh
$ sudo cd /etc/gitlab/
$ sudo nano gitlab.rb
external_url 'http://git.sitename.ru'
```

Пример файла `gitlab.rb`, в котором подключена LDAP-авторизация и настроено подключение к почтовому серверу

```sh
$ sudo cat gitlab.rb 

## GitLab URL
##! URL on which GitLab will be reachable.
##! For more details on configuring external_url see:
##! https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab
external_url 'http://git.sitename.ru'


### Email Settings
 gitlab_rails['gitlab_email_enabled'] = true
 gitlab_rails['gitlab_email_from'] = 'noreply@sitename.ru'
 gitlab_rails['gitlab_email_display_name'] = 'noreply'
 gitlab_rails['gitlab_email_reply_to'] = 'noreply@sitename.ru'
# gitlab_rails['gitlab_email_subject_suffix'] = ''


### LDAP Settings
###! Docs: https://docs.gitlab.com/omnibus/settings/ldap.html
###! **Be careful not to break the indentation in the ldap_servers block. It is
###!   in yaml format and the spaces must be retained. Using tabs will not work.**

 gitlab_rails['ldap_enabled'] = true

###! **remember to close this block with 'EOS' below**
 gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
   main: # 'main' is the GitLab 'provider ID' of this LDAP server
     label: 'LDAP'
     host: '192.168.0.6'
     port: 389
     uid: 'sAMAccountName'
     bind_dn: 'CN=Admin,CN=Users,DC=domain,DC=local'
     password: 'password'
#     timeout: 10
     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
     active_directory: true
#     allow_username_or_email_login: false
#     lowercase_usernames: false
#     block_auto_created_users: false
     base: 'OU=Users,OU=Office,DC=domain,DC=local'
     user_filter: ''
#     attributes:
#     username: ['uid', 'userid', 'sAMAccountName']
#     email: ['mail', 'email', 'userPrincipalName']
#     name: 'cn'
#     first_name: 'givenName'
#     last_name:  'sn'
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
#
#   secondary: # 'secondary' is the GitLab 'provider ID' of second LDAP server
#     label: 'LDAP'
#     host: '_your_ldap_server'
#     port: 389
#     uid: 'sAMAccountName'
#     bind_dn: '_the_full_dn_of_the_user_you_will_bind_with'
#     password: '_the_password_of_the_bind_user'
#     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
#     active_directory: true
#     allow_username_or_email_login: false
#     lowercase_usernames: false
#     block_auto_created_users: false
#     base: ''
#     user_filter: ''
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
 EOS


### Backup Settings
###! Docs: https://docs.gitlab.com/omnibus/settings/backups.html

#gitlab_rails['manage_backup_path'] = true
#gitlab_rails['backup_path'] = "/mnt/nfs"

###! Docs: https://docs.gitlab.com/ce/raketasks/backup_restore.html#backup-archive-permissions
# gitlab_rails['backup_archive_permissions'] = 0644

# gitlab_rails['backup_pg_schema'] = 'public'

###! The duration in seconds to keep backups before they are allowed to be deleted
#gitlab_rails['backup_keep_time'] = 604800


# localhost
# gitlab_rails['smtp_enable'] = true;
# gitlab_rails['smtp_address'] = 'localhost';
# gitlab_rails['smtp_port'] = 25;
## gitlab_rails['smtp_user_name'] = "root"
## gitlab_rails['smtp_password'] = "password"
# gitlab_rails['smtp_domain'] = 'localhost';
## gitlab_rails['smtp_authentication'] = "login"
# gitlab_rails['smtp_tls'] = false;
# gitlab_rails['smtp_openssl_verify_mode'] = 'none'
# gitlab_rails['smtp_enable_starttls_auto'] = false
# gitlab_rails['smtp_ssl'] = false
# gitlab_rails['smtp_force_ssl'] = false

# mx.sitename.ru
gitlab_rails['smtp_enable'] = true;
gitlab_rails['smtp_address'] = 'mx.sitename.ru';
gitlab_rails['smtp_port'] = 587;
gitlab_rails['smtp_user_name'] = 'noreply';
gitlab_rails['smtp_password'] = 'passwopd';
gitlab_rails['smtp_domain'] = 'sitename.ru';
gitlab_rails['smtp_authentication'] = 'login';
gitlab_rails['smtp_tls'] = false;
gitlab_rails['smtp_enable_starttls_auto'] = true;
gitlab_rails['smtp_openssl_verify_mode'] = 'none';
#gitlab_rails['smtp_ssl'] = true;
#gitlab_rails['smtp_force_ssl'] = true;
```

Генерируем конфиг

```sh
$ sudo sudo gitlab-ctl reconfigure
```

Запускаем GitLab

```sh
$ sudo sudo gitlab-ctl start
```

Открываем браузер:

```
http://192.168.1.49
login: root
password: 5iveL!fe
```

Авторизуемся, меняем пароль

## На старом сервере сделать бэкап данных средствами GitLab

У меня не делался бэкап, т.к. скрипту не нравилась версия `psql`, делаем линк на версию посвежее, которая была уже установлена на сервере

```sh
$ sudo ln -s /usr/pgsql-9.3/bin/pg_dump /usr/bin/pg_dump --force
```

Делаем бэкап

```sh
$ sudo cd /home/git/gitlab
$ sudo bundle exec rake gitlab:backup:create RAILS_ENV=production;
```

- Файлы бэкапа на старом сервере: `/home/git/gitlab/tmp/backups`

Переносим файл бэкапа со старого сервера на новый

- Файлы бэкапа на новом сервере: `/var/opt/gitlab/backups`

## Развернуть бэкап на новом сервере

Останавливаем службы

```sh
$ sudo sudo gitlab-ctl stop unicorn
$ sudo sudo gitlab-ctl stop sidekiq
$ sudo sudo gitlab-ctl status
```

Развертывание резервной копии на новом сервере

```sh
$ sudo sudo gitlab-rake gitlab:backup:restore BACKUP=1526552248
```

или

```sh
$ sudo sudo gitlab-rake gitlab:backup:restore RAILS_ENV=production
```

Запускаем службы

```sh
$ sudo gitlab-ctl start unicorn
$ sudo gitlab-ctl start sidekiq
$ sudo gitlab-ctl status
```

В процессе развертывания произошла ошибка. Из бэкапа не развернулись пустые проекты, по-этому вначале их надо удалить, потом делать бэкап.

Так же можно перенести вручную все то, что не перенеслось после:

```sh
$ sudo git clone --mirror /var/opt/gitlab/backups/repositories/%username%/%project%.bundle /var/opt/gitlab/git-data/repositories/%username%/%project%.git
```

Меняем владельца директории

```sh
$ sudo chown -R git:git /var/opt/gitlab/git-data/repositories
```

## Пробуем сделать бэкап на новом сервере и развернуть его

Делаем бэкап

```sh
$ sudo sudo gitlab-rake gitlab:backup:create
```

Останавливаем службы

```sh
$ sudo gitlab-ctl stop unicorn
$ sudo gitlab-ctl stop sidekiq
$ sudo gitlab-ctl status
```

Разворачиваем бэкап

```sh
$ sudo sudo gitlab-rake gitlab:backup:restore BACKUP=1526552248
```

Запускаем службы

```sh
$ sudo gitlab-ctl start unicorn
$ sudo gitlab-ctl start sidekiq
$ sudo gitlab-ctl status
```

## Обновить GitLab до актуальной версии

Т.к. у нас версия довольно старая, обновляться будем по следующей цепочке:

```
7.4.1 -> 7.14.x -> 8.0.x -> 8.17.x -> 9.0 -> 9.5 -> 10.0 -> 10.7 -> 10.8.3
```

Скачиваем дистрибутив версии 7.14.2

```sh
$ sudo wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-7.14.2-ce.1.el7.x86_64.rpm/download.rpm
```

Обновляем

```sh
$ sudo rpm -Uvh gitlab-ce-7.14.2-ce.1.el7.x86_64.rpm
```

- `U` - обновить пакет
- `v` - показать отладочную информацию
- `h` - выводить хэш-меток при установке

Перезапускаем

```sh
$ sudo sudo gitlab-ctl restart
```

И так далее

## Открываем доступ к GitLab снаружи

Т.к. у нас ограниченное количество внешних IP-адресов, и т.к. наш GitLab крутится на отдельном виртуальном сервере, настраиваем проброс портов, что бы у нас был доступ к GitLab снаружи

На сервере, где прописан внешний IP, надо перенастроить ssh, iptables, apache

Настраиваем ssh

```sh
$ sudo cat /etc/ssh/sshd_config
ListenAddress 192.168.1.38

$ sudo service ssh restart
```

теперь доступ к этому серверу по ssh только внутри сети

Настраиваем Настраиваем iptables, пробрасываем порт 22

```sh
$ sudo iptables -A FORWARD -d 192.168.1.49 -i eth1 -p tcp -m tcp --dport 22 -j ACCEPT
$ sudo iptables -t nat -A PREROUTING -d внешний_IP -p tcp -m tcp --dport 22 -j DNAT --to-destination 192.168.1.49:22
$ sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
$ sudo service iptables save
$ sudo service iptables reload
```

Еще надо закомментировать одно правило, иначе проброс не сработает

```sh
#-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```

Настраиваем apache, пробрасываем 80 и 443 порт:

```sh
$ sudo cat /etc/httpd/virtual/git.sitename.ru.conf
    ServerName git.sitename.ru
    ServerSignature Off
    ProxyPreserveHost On

    Redirect / https://git.sitename.ru/

    ServerName git.sitename.ru
    ServerSignature Off
  
    Order deny,allow
    Allow from all

    ProxyPassReverse http://192.168.1.49:80
    ProxyPassReverse http://git.sitename.ru/
  
    SSLEngine on
    SSLCertificateFile /etc/httpd/ssl/sitename.ru/certificate.crt
    SSLCertificateKeyFile /etc/httpd/ssl/sitename.ru/private.key

    SSLProxyEngine On
    SSLProxyCheckPeerCN on
    SSLProxyCheckPeerExpire on

    RewriteEngine on
    RewriteCond %{DOCUMENT_ROOT}/%{REQUEST_FILENAME} !-f
    RewriteRule .* http://192.168.1.49:80%{REQUEST_URI} [P,QSA]
    RequestHeader set X_FORWARDED_PROTO 'https'

    ErrorLog /var/www/vhosts/git.sitename.ru/logs/error_ssl.log
    CustomLog /var/www/vhosts/git.sitename.ru/logs/access_ssl.log combined
```

Перезапускаем apache

```sh
$ sudo service httpd restart
```
