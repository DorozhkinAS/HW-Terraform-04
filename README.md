# Домашнее задание к занятию «Продвинутые методы работы с Terraform»

### Задание 1

1. Возьмите из [демонстрации к лекции готовый код](https://github.com/netology-code/ter-homeworks/tree/main/04/demonstration1) для создания ВМ с помощью remote-модуля.
2. Создайте одну ВМ, используя этот модуль. В файле cloud-init.yml необходимо использовать переменную для ssh-ключа вместо хардкода. Передайте ssh-ключ в функцию template_file в блоке vars ={} .
Воспользуйтесь [**примером**](https://grantorchard.com/dynamic-cloudinit-content-with-terraform-file-templates/). Обратите внимание, что ssh-authorized-keys принимает в себя список, а не строку.
3. Добавьте в файл cloud-init.yml установку nginx.
4. Предоставьте скриншот подключения к консоли и вывод команды ```sudo nginx -t```.

### Решение 1

1. Склонировал код из демонстрации, изучал его.

2. Создаю одну ВМ используя модуль. Чтобы создать именно одну ВМ потребовалось указать параметр ```instance_count  = 1``` в модуле ```test-vm```.

Чтобы передать ssh-ключ используя функцию template_file пишу в блок vars следующий код:

![img_1.png](IMG/img_1.png)

Использую такую конструкцию именно потому, что в задании указано условие, что переменная authorized-keys должна принимать в себя список, а не строку.

Сама переменная ssh-authorized-keys выглядит следующим образом:

![img_2.png](IMG/img_2.png)

3. Чтобы установить nginx на хост используя cloud-init.yml, нужно в секцию ```packages``` добавить строку ``` - nginx```. В нашем случае после запуска виртуальной машины выполняются следующие действия: обновляется кэш пакетов системы, устанавливается текстовый редактор vim и устанавливается nginx.

Проверяю, что nginx установился:

![img.png](IMG/img_3.png)

------

### Задание 2

1. Напишите локальный модуль vpc, который будет создавать 2 ресурса: **одну** сеть и **одну** подсеть в зоне, объявленной при вызове модуля, например: ```ru-central1-a```.
2. Вы должны передать в модуль переменные с названием сети, zone и v4_cidr_blocks.
3. Модуль должен возвращать в root module с помощью output информацию о yandex_vpc_subnet. Пришлите скриншот информации из terraform console о своем модуле. Пример: > module.vpc_dev  
4. Замените ресурсы yandex_vpc_network и yandex_vpc_subnet созданным модулем. Не забудьте передать необходимые параметры сети из модуля vpc в модуль с виртуальной машиной.
5. Откройте terraform console и предоставьте скриншот содержимого модуля. Пример: > module.vpc_dev.
6. Сгенерируйте документацию к модулю с помощью terraform-docs.    
 
Пример вызова

```
module "vpc_dev" {
  source       = "./vpc"
  env_name     = "develop"
  zone = "ru-central1-a"
  cidr = "10.0.1.0/24"
}
```

### Решение 2

1. Написал локальный модуль с одной сетью и одной подсетью в зоне ```ru-central1-a```.

2. В модуле используются переменные с именем сети, зоны и cidr блок:

![img_4.png](IMG/img_4.png)

3. В terraform console проверю, какой output будет показан при вызове модуля ```module.vpc_dev```:

![img_5.png](IMG/img_5.png)

4. Заменил сетевые ресурсы созданным модулем:

![img_6.png](IMG/img_6.png)

5. В root модуле написал следующий output:

![img_7.png](IMG/img_7.png)

Вывод ```module.vpc_dev``` не изменился:

![img_8.png](IMG/img_8.png)

6. С помощью terraform-docs сгенерировал файл документации. Смотреть файл DOCS.md.

![img_15.png](IMG/img_15.png)

### Задание 3
1. Выведите список ресурсов в стейте.
2. Полностью удалите из стейта модуль vpc.
3. Полностью удалите из стейта модуль vm.
4. Импортируйте всё обратно. Проверьте terraform plan. Изменений быть не должно.
Приложите список выполненных команд и скриншоты процессы.

### Решение 3

1. Текущий список ресурсов в стейте:

![img_9.png](IMG/img_9.png)

2. Полностью удалил из стейта модуль vpc_dev:

![img_10.png](IMG/img_10.png)

3. Полностью удалил из стейта модуль test-vm:

![img_11.png](IMG/img_11.png)

4. Обратно импортирую удаленные стейты:

![img_12.png](IMG/img_12.png)

Аналогичными командами импортирую остальные стейты:

![img_13.png](IMG/img_13.png)

![img_14.png](IMG/img_14.png)

После импортирования модулей изменений не последовало.


# Листинг кода по заданиям 1-3

#gitignore

```
**/.terraform/*
.terraform*

# .tfstate files
*.tfstate
*.tfstate.*

# own secret vars store.
personal.auto.tfvars

#checkov
.external_modules

#ansible
*.cfg

```


#cloud-config 

```
users:
  - name: ubuntu
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_authorized_keys:
      - ${ssh-authorized-keys}
package_update: true
package_upgrade: false
packages:
 - vim
 - nginx

```

#main.tr

```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">=0.13"
}

provider "yandex" {
  token     = var.token
  cloud_id  = var.cloud_id
  folder_id = var.folder_id
  zone      = var.default_zone
}

module "vpc_dev" {
  source       = "./vpc"
  zone = "ru-central1-a"
  cidr = "10.0.1.0/24"
  env_name = "develop"
}




module "test-vm" {
  source          = "git::https://github.com/udjin10/yandex_compute_instance.git?ref=main"
  env_name        = "develop"
  network_id      = module.vpc_dev.network_id
  subnet_zones    = ["ru-central1-a"]
  subnet_ids      = [module.vpc_dev.subnet_id]
  instance_name   = "web"
  instance_count  = 1
  image_family    = "ubuntu-2004-lts"
  public_ip       = true

  metadata = {
      user-data          = data.template_file.cloudinit.rendered #Для демонстрации №3
      serial-port-enable = 1
  }

}

#Пример передачи cloud-config в ВМ для демонстрации №3
data "template_file" "cloudinit" {
  template = file("./cloud-init.yml")
  vars     = {
    ssh-authorized-keys = file(var.ssh-authorized-keys[0])
  }
}

```


#outputs.tf

```
output "dev_network_id" {
  value = module.vpc_dev.network_id
}

output "dev_subnet_id" {
  value = module.vpc_dev.subnet_id
}

```


#cloud vars
```

variable "token" {
  type        = string
  description = "OAuth-token; https://cloud.yandex.ru/docs/iam/concepts/authorization/oauth-token"
}

variable "cloud_id" {
  type        = string
  description = "https://cloud.yandex.ru/docs/resource-manager/operations/cloud/get-id"
}

variable "folder_id" {
  type        = string
  description = "https://cloud.yandex.ru/docs/resource-manager/operations/folder/get-id"
}

variable "default_zone" {
  type        = string
  default     = "ru-central1-a"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}
variable "default_cidr" {
  type        = list(string)
  default     = ["10.0.1.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "vpc_name" {
  type        = string
  default     = "develop"
  description = "VPC network&subnet name"
}

variable "ssh-authorized-keys" {
  description = "Path to public SSH key file"
  type        = list(string)
  default     = ["~/.ssh/id_ed25519.pub"]
}

```



















