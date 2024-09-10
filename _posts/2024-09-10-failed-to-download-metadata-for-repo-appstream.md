---
layout: post
title: "[Решено] Error: Failed to download metadata for repo appstream в AlmaLinux 9"
date: "2024-09-10"
categories:
  - Manuals
tags:
  - almalinux
image:
  path: /commons/export-desktop.webp
  alt: "Error: Failed to download metadata for repo appstream"
---

> Ошибка “Failed to download metadata for repo appstream” означает, что система не может скачать метаданные репозитория AppStream для AlmaLinux. Метаданные репозитория содержат информацию о доступных пакетах, их версиях и зависимости, необходимые для установки, обновления или удаления пакетов.
{: .prompt-tip }

При выполнении команды `sudo dnf update` появляется ошибка:

> Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: Curl error (60): SSL peer certificate or SSH remote key was not OK for https://mirrors.almalinux.org/mirrorlist/9/appstream [SSL: no alternative certificate subject name matches target host name 'mirrors.almalinux.org']
{: .prompt-danger }

Временное решение: комментируем `mirrorlist`, разкомментируем `baseurl`

```sh
$ sudo sed -i 's/mirrorlist=/#mirrorlist=/g' /etc/yum.repos.d/almalinux-*
$ sudo sed -i 's/# baseurl=/baseurl=/g' /etc/yum.repos.d/almalinux-*
```
