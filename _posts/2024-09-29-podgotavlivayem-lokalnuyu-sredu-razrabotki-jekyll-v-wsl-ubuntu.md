---
layout: post
title: "Подготавливаем локальную среду разработки Jekyll в WSL Ubuntu"
date: "2024-09-29"
categories:
  - Development
tags:
  - jekyll
  - wsl
  - ubuntu
image:
  path: /commons/export-desktop.webp
  alt: "Подготавливаем локальную среду разработки Jekyll в WSL Ubuntu"
---

> Jekyll - это генератор статических сайтов, написанный на языке Ruby. Он позволяет создавать веб-сайты на основе Markdown-файлов, Sass/CSS и JavaScript, а также интегрировать их с сервисами GitHub Pages.
> Jekyll обеспечивает автоматическую сборку и публикацию сайта из Markdown-файлов, что позволяет создавать контент без необходимости в знаниях HTML и CSS. Он также поддерживает использование тем и плагинов для расширения функциональности.
{: .prompt-tip }

## Разворачиваем Ubuntu 22.04 LTS в WSL

> WSL (Windows Subsystem for Linux) - это функция операционной системы Windows, которая позволяет запускать среду Linux на компьютере Windows без необходимости в отдельной виртуальной машине или двойной загрузке.
{: .prompt-tip }

Смотрим список доступных дистрибутивов

```powershell
PS> wsl -l -o
...
NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Debian                          Debian GNU/Linux
kali-linux                      Kali Linux Rolling
Ubuntu-18.04                    Ubuntu 18.04 LTS
Ubuntu-20.04                    Ubuntu 20.04 LTS
Ubuntu-22.04                    Ubuntu 22.04 LTS
Ubuntu-24.04                    Ubuntu 24.04 LTS
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_7                 Oracle Linux 8.7
OracleLinux_9_1                 Oracle Linux 9.1
openSUSE-Leap-15.6              openSUSE Leap 15.6
SUSE-Linux-Enterprise-15-SP5    SUSE Linux Enterprise 15 SP5
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
openSUSE-Tumbleweed             openSUSE Tumbleweed
```

Устанавливаем дистрибутив Ubuntu 22.04 LTS

```powershell
PS> wsl --install -d Ubuntu-22.04
Запуск Ubuntu 22.04 LTS...
Installing, this may take a few minutes...
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: user
New password:
Retype new password:
passwd: password updated successfully
Операция успешно завершена.
Installation successful!
```

Обновляемся

```sh
$ sudo apt update && sudo apt upgrade -y
```

## Подготовка к установки Ruby 3.3.5 при помощи RBENV

> **RBENV** - это инструмент управления версиями Ruby, который позволяет разработчикам легко установить и переключаться между разными версиями Ruby на их системе.
{: .prompt-tip }

Устанавливаем необходимые пакеты

```sh
$ sudo apt install git curl autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm6 libgdbm-dev libdb-dev -y
```

Устанавливаем RBENV

```sh
$ curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ source ~/.bashrc
```

Проверяем

```sh
$ rbenv -v
rbenv 1.3.0-4-gc335ab8
```

Смотрим доступные версии Ruby

```sh
$ rbenv install -l
3.1.6
3.2.5
3.3.5
jruby-9.4.8.0
mruby-3.3.0
picoruby-3.0.0
truffleruby-24.1.0
truffleruby+graalvm-24.1.0
...
```

Устанавливаем Ruby 3.3.5

```sh
$ rbenv install 3.3.5
```

Назначаем версию по умолчанию

```sh
$ rbenv global 3.3.5
```

Проверяем

```sh
$ ruby --version
ruby 3.3.5 (2024-09-03 revision ef084cc8f4) [x86_64-linux]
```

## Запускаем проект на Jekyll локально

> Основные функции Jekyll:
> - Генерация статических HTML-файлов из Markdown-файлов и шаблонов.
> - Поддержка тем и дизайна сайта через использование шаблонов Liquid.
> - Интеграция с GitHub Pages, что позволяет хранить исходники сайта в репозитории на GitHub и автоматически обновлять сайт при изменении кода.
> - Поддержка Sass и CoffeeScript для компиляции CSS и JavaScript.
{: .prompt-tip }

Клонируем репозиторий с проектом на Jekyll

```sh
$ cd ~
$ git clone https://github.com/cotes2020/chirpy-starter.git
```

Переходим в каталог
```sh
$ cd chirpy-starter
```

Устанавливаем Ruby библиотеки
```sh
$ gem install jekyll bundler
```

Обновляем RubyGems
```sh
$ gem update --system 3.5.20
```

Устанавливаем все необходимые Gem’ы (библиотеки) для клонированного приложения (указанны в файле Gemfile)
```sh
$ bundle install
```

Запускаем наше приложение на Jekyll локально
```sh
$ bundle exec jekyll s
...
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

Теперь можно запустить браузер и посмотреть результат по адресу `http://127.0.0.1:4000/`
