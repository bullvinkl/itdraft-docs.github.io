---
title: "Перенос виртуальной машины из VirtualBox в ESXi"
date: "2017-10-05"
categories: 
  - Linux
tags: 
  - "esxi"
  - "ovftool"
  - "ubuntu"
  - "virtualbox"
image:
  path: /commons/987847_cbc9_2.jpg
  alt: "Перенос виртуальной машины из VirtualBox в ESXi"
---

> **VMware ESXi** - это гипервизор, не зависящий от операционной системы, который позволяет легко развертывать решения для виртуализации. Он занимает минимальное количество дискового пространства (около 32 МБ) и может быть встроен в стандартные серверы с архитектурой x86.
{: .prompt-tip }

Исходные данные:

- VirtualBox 2.2
- ESXi 6.5
- Ubuntu 17.04

Экспортируем виртуальную машину в формат OVA

Скачиваем утилиту [Open Virtualization Format Tool (ovftool)](https://code.vmware.com/web/dp/tool/ovf/4.1.0)

Изменяем атрибуты скаченного файла, сделав его исполняемым, и устанавливаем утилиту

```sh
$ cd /home/download
$ chmod a+x VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle
$ sudo ./VMware-ovftool-4.1.0-2459827-lin.x86_64.bundle
```

Конвертируем нашу виртуальную машину с помощью установленной утилиты (Без этого, при добавлении ovf в ESXi вылезает ошибка: `Unsupported hardware family 'virtualbox-2.2'`.)

```sh
$ ovftool --lax testmachine_vitrualbox.ova result.ovf
```

- `testmachine_vitrualbox.ova` - наша виртуальная машина, которую мы экспортировали из VirtualBox
- `result.ovf` - результат конвертации

На выходе получаем 3 файла

- `result.mf`
- `result.ovf`
- `result-disk1.vmdk`

Открываем файл `result.ovf` в текстовом редакторе  и меняем `virtualbox-2.2` на `vmx-07`

Далее меняем следующие строки:

```
0
sataController0
SATA Controller
sataController0
5
AHCI
20
```

на

```
0
SCSIController
SCSI Controller
SCSIController
5
lsilogic
6
```

Удаляем строки:

```
3
false
sound
Sound Card
sound
7
ensoniq1371
35
```

Иначе при импорте в ESXi появится ошибка: `No support for the virtual hardware device type '35'`

Корректируем SHA1-сумму для измененного файла `result.ovf`

Узнаем SHA1-сумму:

```sh
$ sha1sum result.ovf
cbbe4462a9ba0be06ffe2359cf89a5ade6f85fd6  result.ovf
```

Открываем в текстовом редакторе `result.mf` и меняем старое значение на новое

Теперь можно импортировать Виртуальную машину в ESXi

Чтобы завершить импорт, включите машину и удалите дополнение VirtualBox Guest Additions, затем установите VMware Guest Tools.
