---
title: "VirtualBox: cannot register the hard disk already exists"
date: "2018-05-17"
categories: 
  - Virtualization
tags: 
  - "linux"
  - "uuid"
  - "virtualbox"
  - "windows"
image:
  path: /commons/1353848_735f_3.jpg
  alt: "cannot register the hard disk already exists"
---

> Ошибка “cannot register the hard disk already exists” в VirtualBox возникает при попытке добавить файл виртуального жёсткого диска (VHD или VDI) к виртуальной машине, если файл с таким же UUID (Universally Unique Identifier) уже зарегистрирован в VirtualBox.
{: .prompt-tip }

Данная ошибка появляется в результате того, что мы скопировали файл образа диска виртуальной машины, а `UUID` остался прежним.

## Решение для Linux (Ubuntu 18.04):

Переходим в каталог, где лежит файл образа диска виртуальной машины

```sh
$ cd /mnt/b419095b-b487-4c19-99b2-9c57ac9f544b/vms/
```

Генерируем новый `UUID`

```sh
$ VBoxManage internalcommands sethduuid gitlab_7.4.1.vdi
```

### Решение для Windows

```
> "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" internalcommands sethduuid "D:\VirtualBox VMs\gitlab_7.4.1.vdi"
```

на примере `VirtualBox версии 5.2.10`
