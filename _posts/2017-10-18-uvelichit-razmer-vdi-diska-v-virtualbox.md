---
title: "Увеличить размер VDI диска в VirtualBox"
date: "2017-10-18"
categories: 
  - Virtualisation
tags: 
  - "linux"
  - "vdi"
  - "virtualbox"
  - "windows"
image:
  path: /commons/1032198_11a3_4.jpg
  alt: "Увеличить размер VDI"
---

> **VirtualBox** - это программное решение для виртуализации с открытым кодом, позволяющее запускать несколько операционных систем на одном устройстве и легко развертывать приложения в облаке.
{: .prompt-tip }

Первоначально необходимо остановить виртуальную машину

## VitrualBox под Windows

Чтобы посмотреть информацию о текущем виртуальном диске, в командной строке набираем:

```
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showhdinfo "C:\Users\test\Win10\disk1.vdi"
```

Чтобы увеличить размер виртуального жесткого диска, в командной строке набираем:

```
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyhd "C:\Users\test\Win10\disk1.vdi" --resize 25000
```

## VitrualBox под Linux

Чтобы посмотреть информацию о текущем виртуальном диске, в терминале набираем:

```sh
$ VBoxManage showhdinfo /home/VirtualBox/Win10/disk1.vdi
```

Чтобы увеличить размер виртуального жесткого диска, в терминале набираем:

```sh
$ VBoxManage modifyhd /home/VirtualBox/Win10/disk1.vdi --resize 40000
```

Далее запускаем Виртуальную машину, где должна появилась не размеченная область
