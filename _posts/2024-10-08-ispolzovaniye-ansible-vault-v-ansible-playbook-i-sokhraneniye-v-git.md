---
layout: post
title: "Использование Ansible-vault в Ansible-playbook и сохранение в Git"
date: "2024-10-08"
categories:
  - DevOps
  - Development
tags:
  - ansible
  - ansible-playbook
  - ansible-vault
  - git
image:
  path: /commons/1687994872_bogatyr.jpg
  alt: "Использование Ansible-vault в Ansible-playbook и сохранение в Git"
---

> **Ansible-vault** — это утилита командной строки, которая позволяет создавать и управлять шифрованными файлами. Она поставляется вместе с Ansible. В качестве алгоритма шифрования Ansible Vault использует алгоритм симметричного шифрования AES256.
{: .prompt-tip }

К примеру, у нас есть проект, расположенный в Gitlab.
Для того, что бы не хранить секреты в открытом виде будем использовать Ansible-vault

Клонируем наш проект
```sh
$ git clone ssh://git@gitlab.itdraft.ru/ansible/test-project.git
```

Переходим в проект
```sh
$ cd test-project
```

Допустим, в проекте пароли ранее хранились в открытом виде. Устраняем это недоразумение, удаляем файл
```sh
$ rm user_passwords.yml
```

Создаем шифрованный файл используя Ansible-vault, заполним его необходимыми данными
```sh
$ ansible-vault create user_passwords.yml
New Vault password: superpass
Confirm New Vault password: superpass
```

В данном случае структура будет следующая
```yaml
---
user_passwords:
  m.makarov: MySuperPassw0rd
```

Если посмотрим содержимое файла стандартными средствами, мы увидим что-то вроде
```sh
$ cat user_passwords.yml 
$ANSIBLE_VAULT;1.1;AES256
62666362383931393966363735623861303739383936663266613864643930306237656163356263
3634366531653030323663353733336363383337643733630a346165653135366337646432353865
35636363613464303264746462313165383963653834343036303962363063363163376538366234
6334373435366434310a373663313638383961313563373838336433636661386434313963663132
33396562326431303836663938353935323161656435303632353263393737336536316563366436
37656266346166303261343138353163383811353930613134613739303033353633663963663634
366261376134616137306438363632323865
```

В дальнейшем, чтобы посмотреть или изменить данные в файле, необходимо использовать команды `ansible-vault view` и `ansible-vault edit`
```sh
$ ansible-vault view user_passwords.yml
Vault password: superpass

$ ansible-vault edit user_passwords.yml
Vault password: superpass
```

Что бы сохранить изменения в репозитории, выполняем `git commit` и `git push`
```sh
$ git add .
$ git commit -m "add ansible-vault"
$ git push origin main
```

Запускаем `ansible-playbook`
```sh
$ ansible-playbook -i hosts -l srv playbook.yml --ask-vault-pass
Vault password:
```

В данном примере я добавил параметр `--ask-vault-pass`. В результате, при выполнении `ansible-playbook` запросился пароль.
Ключ так же можно сохранить в отдельный файл (`$ echo "MySuperPass" > ~/.ansible_pass.txt`) и использовать параметр `--vault-password-file="~/.ansible_pass.txt"`

## UPD 2024-10-08

Полезные команды для работы с Ansible-vault:

- `$ ansible-vault encrypt user_passwords.yml`  - зашифровать имеющийся файл
- `$ ansible-vault decrypt user_passwords.yml`  - расшифровать файл, вернуть в plaintext
- `$ ansible-vault rekey user_passwords.yml`  - поменять пароль шифрования
