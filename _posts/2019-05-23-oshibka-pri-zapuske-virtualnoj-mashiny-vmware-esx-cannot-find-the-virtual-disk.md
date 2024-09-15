---
title: "Ошибка при запуске виртуальной машины - vmware esx cannot find the virtual disk"
date: "2019-05-23"
categories: 
  - Virtualization
tags: 
  - "esxi"
  - "vsphere"
  - "wmvare"
image:
  path: /commons/1387342_082d_2.jpg
  alt: "vmware esx cannot find the virtual disk"
---

> **VMware ESXi** - это программное обеспечение для виртуализации серверов, которое позволяет консолидировать приложения и уменьшить затраты на управление инфраструктурой IT. 
{: .prompt-tip }

Перестала запускаться виртуальная машина. В процессе гугления выяснилось, что в каталоге с виртуальной машиной отсутствует файл с конфигурацией. Восстановим (создадим) его.

В VMware vSphere включаем SSH, и подключаемся. Затем переходим в каталог с нашей виртуальной машиной

```sh
# cd /vmfs/volumes/datastore2/srv-virtual-01
```

Смотрим размер flat-диска

```sh
# ls -l srv-virtual-010-flat.vmdk
-rw------- 1 root     root     85899345920 Jul 29  2016 srv-virtual-010-flat.vmdk
```

При помощи утилиты `vmkfstools` создаем новый vmdk-файл

```sh
# vmkfstools -c 85899345920 -d thin -a lsilogic new.vmdk
```

где
- `85899345920` - размер (узнали из предыдущего шага)
- `thin` - тип диска
- `lsilogic` - адаптер


После выполнения этого действия в каталоге появится 2 файла:

- `new.vmdk` - файл конфигурации, он нам и нужен
- `new-flat.vmdk` - файл с данными


Удаляем файл с данными

```sh
# rm new-flat.vmdk
```

Переименовываем `new.vmdk` в нужный нам и редактируем его

```sh
# mv new.vmdk srv-virtual-010.vmdk
# vi srv-virtual-010.vmdk
```

Находим строку и меняем:

```
RW 167772160 VMFS "new-flat.vmdk"
```

на

```
RW 167772160 VMFS "srv-virtual-010-flat.vmdk"
```

Так же, у меня в каталоге присутствовали файлы

```
-rw------- 1 root     root     18035675136 May 23 09:14 srv-virtual-010-000002-delta.vmdk
-rw------- 1 root     root           330 May 23 09:13 srv-virtual-010-000002.vmdk
```

Как мне удалось выяснить, это файл с данными и файл конфигурации, которые появились после того, как сделали snapshot

Смотрим содержимое файла конфигурации `srv-virtual-010-000002.vmdk`

```sh
# cat srv-virtual-010-000002.vmdk
# Disk DescriptorFile
version=1
encoding="UTF-8"
CID=a02b6973
parentCID=e650b626
isNativeSnapshot="no"
createType="vmfsSparse"
parentFileNameHint="srv-virtual-010-000001.vmdk"
# Extent description
RW 167772160 VMFSSPARSE "srv-virtual-010-000002-delta.vmdk"
...
```

Нас интересуют строки:

```
parentCID=e650b626
parentFileNameHint="srv-virtual-010-000001.vmdk"
```

Значение `parentCID` файла `srv-virtual-010-000002.vmdk` соответствует значению CID файла `srv-virtual-010-000001.vmdk`  
Но файл `srv-virtual-010-000001.vmdk` у нас отсутствует, и мы в начале статьи создали новый файл конфигурации `srv-virtual-010.vmdk`  
Смотрим его содержимое:

```sh
# сat srv-virtual-010.vmdk
# Disk DescriptorFile
version=1
encoding="UTF-8"
CID=5d2981c6
parentCID=ffffffff
isNativeSnapshot="no"
createType="vmfs"

# Extent description
RW 167772160 VMFS "srv-virtual-010-flat.vmdk"
...
```

Запоминаем строку

```
CID=5d2981c6
```

Теперь редактируем файл конфигурации `srv-virtual-010-000002.vmdk`, в котором меняем значение `parentCID` и `parentFileNameHint`

```sh
# vi srv-virtual-010-000002.vmdk
# Disk DescriptorFile
version=1
encoding="UTF-8"
CID=a02b6973
parentCID=5d2981c6 ## значение CID файла srv-virtual-010.vmdk
isNativeSnapshot="no"
createType="vmfsSparse"
parentFileNameHint="srv-virtual-010.vmdk" ## наш вновь созданный файл конфигурации
# Extent description
RW 167772160 VMFSSPARSE "srv-virtual-010-000002-delta.vmdk"
```

После всех этих манипуляций моя виртуальная машина запустилась

При составлении материала были использованы следующие статьи:  
[http://geckich.blogspot.com/2011/12/cannot-open-disk-or-vmware-esx-cannot.html](https://geckich.blogspot.com/2011/12/cannot-open-disk-or-vmware-esx-cannot.html)  
[https://kb.vmware.com/s/article/1026353](https://kb.vmware.com/s/article/1026353)
