---
title: "HashiCorp Vault как центр сертификации (CA) / Vault PKI"
date: "2020-12-02"
categories: 
  - Linux
  - HashiCorp-Vault
tags: 
  - "hashicorp-vault"
  - "pki"
image:
  path: /commons/1400240_c61f_3.jpg
  alt: "HashiCorp Vault как центр сертификации"
---

> **PKI** (Public Key Infrastructure, инфраструктура открытых ключей) -  набор средств, распределённых служб и компонентов, в совокупности используемых для поддержки криптозадач на основе закрытого и открытого ключей.
{: .prompt-tip }

- Список статей из категории [HashiCorp Vault](/categories/hashicorp-vault/)

## Подготовка

Устанавливаем в систему утилиту "jq"

```sh
$ sudo dnf -y install jq
```

## Настройка PKI

Авторизуемся в Vault

```sh
$ vault login
```

Активируем PKI тип секрета для корневого центра сертификации

```sh
$ vault secrets enable \
    -path=pki_root_ca \
    -description="PKI Root CA" \
    -max-lease-ttl="262800h" \
    pki
```

Создаем корневой центра сертификации (CA). 262800h = 30 лет

```sh
$ vault write -format=json pki_root_ca/root/generate/internal \
    common_name="Root Certificate Authority" \
    country="Russian Federation" \
    locality="Moscow" \
    street_address="Red Square 1" \
    postal_code="101000" \
    organization="Horns and Hooves LLC" \
    ou="IT" \
    ttl="262800h" > pki-root-ca.json
```

Сохраняем корневой сертификат. В дальнейшем именно его надо распространять в организации и делать доверенным

```sh
$ cat pki-root-ca.json | jq -r .data.certificate > rootCA.pem
```

Публикуем URL'ы для корневого центра сертификации

```sh
$ vault write pki_root_ca/config/urls \
    issuing_certificates="http://vault.example.com:8200/v1/pki_root_ca/ca" \
    crl_distribution_points="http://vault.example.com:8200/v1/pki_root_ca/crl"
```

Активируем PKI тип секрета для промежуточного центра сертификации

```sh
$ vault secrets enable \
    -path=pki_int_ca \
    -description="PKI Intermediate CA" \
    -max-lease-ttl="175200h" \
    pki
```

Генерируем запрос на выдачу сертификата для промежуточного центра сертификации

```sh
$ vault write -format=json pki_int_ca/intermediate/generate/internal \
   common_name="Intermediate CA" \
   country="Russian Federation" \
   locality="Moscow" \
   street_address="Red Square 1" \
   postal_code="101000" \
   organization="Horns and Hooves LLC" \
   ou="IT" \
   ttl="175200h" | jq -r '.data.csr' > pki_intermediate_ca.csr
```

Отправляем полученный CSR-файл в корневой центр сертификации, получаем сертификат для промежуточного центра сертификации. 175200h = 20 лет

```sh
$ vault write -format=json pki_root_ca/root/sign-intermediate csr=@pki_intermediate_ca.csr \
   country="Russia Federation" \
   locality="Moscow" \
   street_address="Red Square 1" \
   postal_code="101000" \
   organization="Horns and Hooves LLC" \
   ou="IT" \
   format=pem_bundle \
   ttl="175200h" | jq -r '.data.certificate' > intermediateCA.cert.pem
```

Публикуем подписанный сертификат промежуточного центра сертификации

```sh
$ vault write pki_int_ca/intermediate/set-signed \
    certificate=@intermediateCA.cert.pem
```

Публикуем URL'ы для промежуточного центра сертификации

```sh
$ vault write pki_int_ca/config/urls \
    issuing_certificates="http://vault.example.com:8200/v1/pki_int_ca/ca" \
    crl_distribution_points="http://vault.example.com:8200/v1/pki_int_ca/crl"
```

Создаем роль, с помощью которой будем выдавать сертификаты для серверов

```sh
$ vault write pki_int_ca/roles/example-dot-com-server \
    country="Russia Federation" \
    locality="Moscow" \
    street_address="Red Square 1" \
    postal_code="101000" \
    organization="Horns and Hooves LLC" \
    ou="IT" \
    allowed_domains="example.com" \
    allow_subdomains=true \
    max_ttl="87600h" \
    key_bits="2048" \
    key_type="rsa" \
    allow_any_name=false \
    allow_bare_domains=false \
    allow_glob_domain=false \
    allow_ip_sans=true \
    allow_localhost=false \
    client_flag=false \
    server_flag=true \
    enforce_hostnames=true \
    key_usage="DigitalSignature,KeyEncipherment" \
    ext_key_usage="ServerAuth" \
    require_cn=true
```

Создаем роль, с помощью которой будем выдавать сертификаты для клиентов

```sh
$ vault write pki_int_ca/roles/example-dot-com-client \
    country="Russia Federation" \
    locality="Moscow" \
    street_address="Red Square 1" \
    postal_code="101000" \
    organization="Horns and Hooves LLC" \
    ou="IT" \
    allow_subdomains=true \
    max_ttl="87600h" \
    key_bits="2048" \
    key_type="rsa" \
    allow_any_name=true \
    allow_bare_domains=false \
    allow_glob_domain=false \
    allow_ip_sans=false \
    allow_localhost=false \
    client_flag=true \
    server_flag=false \
    enforce_hostnames=false \
    key_usage="DigitalSignature" \
    ext_key_usage="ClientAuth" \
    require_cn=true
```

Создаем сертификат на 5 лет для домена vault.example.com

```sh
$ vault write -format=json pki_int_ca/issue/example-dot-com-server \
    common_name="vault.example.com" \
    alt_names="vault.example.com" \
    ttl="43800h" > vault.example.com.crt
```

Сохраняем сертификат в правильном формате

```sh
$ cat vault.example.com.crt | jq -r .data.certificate > vault.example.com.crt.pem
$ cat vault.example.com.crt | jq -r .data.issuing_ca >> vault.example.com.crt.pem
$ cat vault.example.com.crt | jq -r .data.private_key > vault.example.com.crt.key
```

## Работаем с сертификатами

Посмотреть список сертификатов

```sh
$ vault list pki_int_ca/certs
```

Прочитать сертификат (вывести содержимое)

```sh
$ vault read pki_int_ca/cert/<serial number>
```

Отозвать сертификат

```sh
$ vault write pki_int_ca/revoke serial_number=<serial number>
```

Очистить просроченные / отозванные сертификаты

```sh
$ vault write pki_int_ca/tidy \
    safety_buffer=5s \
    tidy_cert_store=true \
    tidy_revocation_list=true
```

## Bash-скрипт

Для удобства разворачивания PKI подготовил bash-скрипт в своем репозитории на [GitHub](https://github.com/bullvinkl/vault-pki)
