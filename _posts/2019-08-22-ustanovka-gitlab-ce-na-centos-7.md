---
title: "Установка GitLab CE на Centos 7"
date: "2019-08-22"
categories: 
  - Linux
  - GitLab
tags: 
  - "centos"
  - "git"
  - "gitlab"
image:
  path: /commons/1387054_7b12_2.jpg
  alt: "Установка GitLab"
---

> **GitLab** — это веб-приложение и система управления репозиториями программного кода для Git, предназначенная для хранения кода и совместной разработки масштабных программных проектов. Она обеспечивает управление версиями, совместную работу над проектами, тестирование, отладку и развертывание программного обеспечения.
{: .prompt-tip }

Чуть ранее была рассмотрена статья [по переносу GitLab на другой сервер и обновлению GitLab]({% post_url 2018-06-05-perenos-gitlab-na-drugoj-server-i-obnovlenie-gitlab %})

Добавляем репозиторий EPEL и обновляемся

```sh
$ sudo yum -y install epel-release
$ sudo yum -y update
```

Устанавливаем необходимый софт

```sh
$ sudo yum -y install curl openssh-server openssh-clients postfix policycoreutils-python mc nano wget htop git rsync p7zip ntpdate
```

Отключаем SELinux

```sh
$ sudo setenforce 0
$ sudo nano /etc/selinux/config
SELINUX=disabled
```

Добавляем правила в файерволл

```sh
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
```

Скачиваем релизную версию GitLab и устанавливаем ее

```sh
$ sudo cd /home
$ wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-12.1.6-ce.0.el7.x86_64.rpm/download.rpm
$ sudo rpm -ivh gitlab-ce-12.1.6-ce.0.el7.x86_64.rpm
```

Правим конфиг `gitlab.rb`

```sh
$ sudo nano /etc/gitlab/gitlab.rb
$ grep -v "^#\|^$" /etc/gitlab/gitlab.rb
 external_url 'http://gitlab.itdraft.ru'
 gitlab_rails['gitlab_email_enabled'] = true
 gitlab_rails['gitlab_email_from'] = 'admin@itdraft.ru'
 gitlab_rails['gitlab_email_display_name'] = 'Admin'
 gitlab_rails['gitlab_email_reply_to'] = 'admin@itdraft.ru'
 gitlab_rails['smtp_enable'] = true
 gitlab_rails['smtp_address'] = "smtp.itdraft.ru"
 gitlab_rails['smtp_port'] = 465
 gitlab_rails['smtp_user_name'] = "admin"
 gitlab_rails['smtp_password'] = "%password%"
 gitlab_rails['smtp_domain'] = "itdraft.ru"
 gitlab_rails['gitlab_email_from'] = 'admin@itdraft.ru'
 gitlab_rails['smtp_authentication'] = "login"
 gitlab_rails['smtp_enable_starttls_auto'] = true
 gitlab_rails['smtp_tls'] = false
 gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
```

Генерируем конфиг и запускаем GitLab

```sh
$ sudo sudo gitlab-ctl reconfigure
$ sudo sudo gitlab-ctl start
```

Открываем GitLab в браузере и задаем новый пароль для пользователя `root`
