---
title: "Entering emergency mode. Exit the shell to continue"
date: "2018-06-06"
categories: 
  - Linux
tags: 
  - "centos"
  - "xfs_mdrestore"
  - "xfs_metadump"
  - "xfs_repair"
image:
  path: /commons/1361114_fab1_2.jpg
  alt: "Entering emergency mode. Exit the shell to continue"
---

> **Entering emergency mode** (аварийный режим) - это состояние, при котором система или устройство переходят в режим ограниченной функциональности, чтобы обеспечить минимальную работоспособность и предотвратить дальнейшее повреждение или уничтожение.
{: .prompt-tip }

При загрузке виртуальной машины с Centos 7 появляется надпись:

> Entering emergency mode. Exit the shell to continue.
> Type "journalctl" to view system logs.
> You might want to save "/run/initramfs/sosreport.txt" to a USB stick or /boot after mounting them and attach it to a bug report.
{: .prompt-danger }

Грузимся с LiveCD (Я использова `CentOS-7-LiveGNOME`)

Создаем копию метаданных раздела

```sh
# xfs_metadump /dev/mapper/centos-root /tmp/centos-root.metadump
```

Создаем образ метаданных

```sh
# xfs_mdrestore /tmp/centos-root.metadump /tmp/centos-root.img
```

И восстанавливаем его

```sh
# xfs_repair -L /tmp/centos-root.img
# xfs_repair -L /dev/mapper/centos-root
```
