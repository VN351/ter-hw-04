# Домашнее задание к занятию «Продвинутые методы работы с Terraform»

### Цели задания

1. Научиться использовать модули.
2. Отработать операции state.
3. Закрепить пройденный материал.


### Чек-лист готовности к домашнему заданию

1. Зарегистрирован аккаунт в Yandex Cloud. Использован промокод на грант.
2. Установлен инструмент Yandex CLI.
3. Исходный код для выполнения задания расположен в директории [**04/src**](https://github.com/netology-code/ter-homeworks/tree/main/04/src).
4. Любые ВМ, использованные при выполнении задания, должны быть прерываемыми, для экономии средств.

------
### Внимание!! Обязательно предоставляем на проверку получившийся код в виде ссылки на ваш github-репозиторий!
Убедитесь что ваша версия **Terraform** ~>1.8.4
Пишем красивый код, хардкод значения не допустимы!
------

## Ссылка на репозиторий с выполненой работой

[Ссылка на конфигурацию Terraform](https://github.com/VN351/dev/tree/master)

### Задание 1

1. Возьмите из [демонстрации к лекции готовый код](https://github.com/netology-code/ter-homeworks/tree/main/04/demonstration1) для создания с помощью двух вызовов remote-модуля -> двух ВМ, относящихся к разным проектам(marketing и analytics) используйте labels для обозначения принадлежности.  В файле cloud-init.yml необходимо использовать переменную для ssh-ключа вместо хардкода. Передайте ssh-ключ в функцию template_file в блоке vars ={} .
Воспользуйтесь [**примером**](https://grantorchard.com/dynamic-cloudinit-content-with-terraform-file-templates/). Обратите внимание, что ssh-authorized-keys принимает в себя список, а не строку.
3. Добавьте в файл cloud-init.yml установку nginx.
4. Предоставьте скриншот подключения к консоли и вывод команды ```sudo nginx -t```, скриншот консоли ВМ yandex cloud с их метками. Откройте terraform console и предоставьте скриншот содержимого модуля. Пример: > module.marketing_vm
------
В случае использования MacOS вы получите ошибку "Incompatible provider version" . В этом случае скачайте remote модуль локально и поправьте в нем версию template провайдера на более старую.
------
## Ответ на задание 1
1.  main.tf
    ```
    module "vms" {
    source = "./modules/vm"
    for_each = local.vm_configs
    env_name       = each.value.env_name
    network_id     = each.value.network_id
    subnet_zones   = each.value.subnet_zones
    subnet_ids     = each.value.subnet_ids
    instance_name  = each.value.instance_name
    instance_count = each.value.instance_count
    image_family   = each.value.image_family
    public_ip      = each.value.public_ip
    labels         = each.value.labels
    metadata       = each.value.metadata
    }
    ```
    locals.tf
    ```
    locals {
      vm_configs = {
        marketing = {
          env_name       = "marketing"
          network_id     = module.vpc_dev.network_id
          subnet_zones   = ["ru-central1-a"]
          subnet_ids     = [module.vpc_dev.subnet_id]
          instance_name  = "vm"
          instance_count = 1
          image_family   = "ubuntu-2004-lts"
          public_ip      = true
          labels         = { project = "marketing" }
          metadata       = {
            "user-data"           = data.template_file.cloudinit.rendered
            "serial-port-enable" = "1"
          }
        },
        analytics = {
          env_name       = "analytics"
          network_id     = module.vpc_dev.network_id
          subnet_zones   = ["ru-central1-a"]
          subnet_ids     = [module.vpc_dev.subnet_id]
          instance_name  = "vm"
          instance_count = 1
          image_family   = "ubuntu-2004-lts"
          public_ip      = true
          labels         = { project = "analytics" }
          metadata       = {
            "user-data"           = data.template_file.cloudinit. rendered
            "serial-port-enable" = "1"
          }
        }
      }
    } 
    ```
    data.tf
    ```
    data "template_file" "cloudinit" {
      template = file("./cloud-init.yml")
  
      vars = {
        username           = var.username
        ssh_public_key     = file(var.ssh_public_key)
        packages           = jsonencode(var.packages)
      }
    }
    ```
    variables.tf
    ```
    variable "username" {
      type = string
      default = "ubuntu"
    }

    variable "ssh_public_key" {
      description = "Путь к публичному SSH ключу"
      type        = string
      default     = "/home/vlad/.ssh/id_ed25519.pub"
    }

    variable packages {
      type    = list
      default = ["vim", "nginx"]
    }

    variable "public_key" {
      type    = string
      default = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC8bDGbiyUNa2k07/T9jlaRKD1gMcMT9/4wqljOvFJOD nevzorovvlad@mail.ru"
    }
    ```
    cloud-init.yml
    ```
    users:
      - name: ubuntu
        groups: sudo
        shell: /bin/bash
        sudo: ["ALL=(ALL) NOPASSWD:ALL"]
        ssh_authorized_keys:
          - ${ssh_public_key}
    package_update: true
    package_upgrade: false
    packages:
      - nginx
    ```

2. ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-1-1.png)
3. ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-1-2.png)
4. ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-1-3.png)
5. ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-1-4.png)

### Задание 2

1. Напишите локальный модуль vpc, который будет создавать 2 ресурса: **одну** сеть и **одну** подсеть в зоне, объявленной при вызове модуля, например: ```ru-central1-a```.
2. Вы должны передать в модуль переменные с названием сети, zone и v4_cidr_blocks.
3. Модуль должен возвращать в root module с помощью output информацию о yandex_vpc_subnet. Пришлите скриншот информации из terraform console о своем модуле. Пример: > module.vpc_dev  
4. Замените ресурсы yandex_vpc_network и yandex_vpc_subnet созданным модулем. Не забудьте передать необходимые параметры сети из модуля vpc в модуль с виртуальной машиной.
5. Сгенерируйте документацию к модулю с помощью terraform-docs.
 
Пример вызова

```
module "vpc_dev" {
  source       = "./vpc"
  env_name     = "develop"
  zone = "ru-central1-a"
  cidr = "10.0.1.0/24"
}
```
------
## Ответ на задание 2

1.  main.tf (основной)
    ```
    module "vpc_dev" {
      source       = "./modules/vpc"
      network_name = var.net_name
      zone         = var.default_zone
      v4_cidr_block = var.default_cidr
    }
    ```
    variables.tf
    ```
    variable "net_name" {
      description = "Название сети"
      type        = string
      default     = "develop-network"
    }

    variable "default_zone" {
      type        = string
      default     = "ru-central1-a"
      description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
    }

    variable "default_cidr" {
      description = "CIDR блок по умолчанию для подсети"
      type        = string
      default     = "10.0.1.0/24"
    }
    ```
    outputs.tf
    ```
    output "out" {
      value = [
        module.vms["marketing"].external_ip_address,
        module.vms["analytics"].external_ip_address
      ]
    }
    ```
    main.tf (модуля)
    ```
    resource "yandex_vpc_network" "dev" {
      name = var.network_name
    }

    resource "yandex_vpc_subnet" "dev-sub" {
      name           = "${var.network_name}-${var.zone}-subnet"
      zone           = var.zone
      network_id     = yandex_vpc_network.dev.id
      v4_cidr_blocks = [var.v4_cidr_block]
    }
    ```
    variables.tf (модуля)
    ```
    variable "network_name" {
      description = "Название VPC сети."
      type        = string
    }

    variable "zone" {
      description = "Зона для подсети."
      type        = string
    }

    variable "v4_cidr_block" {
      description = "CIDR блок подсети IPv4."
      type        = string
    }
    ```
    outputs.tf (модуля)
    ```
    output "network_id" {
      description = "ID созданной сети"
      value       = yandex_vpc_network.dev.id
    }

    output "subnet_id" {
      description = "ID созданной подсети"
      value       = yandex_vpc_subnet.dev-sub.id
    }

    output "subnet_cidr" {
      description = "CIDR блока подсети"
      value       = yandex_vpc_subnet.dev-sub.v4_cidr_blocks
    }
    ```
2.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-2-1.png) 
3.  main.tf
    ```
    module "vpc_prod" {
      source   = "./modules/vpc-prod"
      env_name = var.vpc_prod_env_name
      subnets = var.vpc_prod_subnets
    }
    ```
    variables.tf
    ```
    variable "vpc_prod_env_name" {
      description = "Название сети"
      type        = string
      default     = "production"
    }

    variable "vpc_prod_subnets" {
      description = "Список подсетей с указанием зоны и CIDR блока"
      type = list(object({
        zone = string
        cidr = string
      }))
      default = [
        { zone = "ru-central1-a", cidr = "10.0.1.0/24" },
        { zone = "ru-central1-b", cidr = "10.0.2.0/24" },
        { zone = "ru-central1-d", cidr = "10.0.3.0/24" },
      ]
    }
    ```
    outputs.tf
    ```
    output "dev_network_id" {
      description = "ID сети для development окружения"
      value       = module.vpc_dev.network_id
    }

    output "dev_subnet_id" {
      description = "ID подсети для development окружения"
      value       = module.vpc_dev.subnet_id
    }

    output "dev_subnet_cidr" {
      description = "CIDR подсети для development окружения"
      value       = module.vpc_dev.subnet_cidr
    }
    ```
    main.tf (модуля)
    ```
    resource "yandex_vpc_network" "prod" {
      name = "${var.env_name}-network"
    }

    resource "yandex_vpc_subnet" "prod-sub" {
      for_each = { for idx, subnet in var.subnets : "${var.env_name}-subnet-${idx}" => subnet }
      name           = "${var.env_name}-subnet-${each.key}"
      zone           = each.value.zone
      network_id     = yandex_vpc_network.prod.id
      v4_cidr_blocks = [each.value.cidr]
    }
    ```
    locals.tf (часть)
    ```
    marketing = {
      env_name       = "marketing"
      network_id     = module.vpc_prod.network_id
      subnet_zones   = module.vpc_prod.subnet_zones
      subnet_ids     = module.vpc_prod.subnet_ids
      instance_name  = "vm"
      instance_count = 1
      image_family   = "ubuntu-2004-lts"
      public_ip      = true
      labels         = { project = "marketing" }
      metadata       = {
        "user-data"           = data.template_file.cloudinit.rendered
        "serial-port-enable" = "1"
      }
    }
    ```
    variables.tf
    ```
    variable "env_name" {
      description = "Название окружения (например, production, develop)"
      type        = string
    }

    variable "subnets" {
      description = "Список подсетей с зонами и CIDR"
      type = list(object({
        zone = string
        cidr = string
      }))
    }
    ```
    outputs.tf (модуля)
    ```
    output "network_id" {
      description = "Идентификатор созданной VPC сети"
      value       = yandex_vpc_network.prod.id
    }

    output "subnet_ids" {
      description = "Идентификаторы созданных подсетей"
      value       = [for subnet in yandex_vpc_subnet.prod-sub : subnet.id]
    }

    output "subnet_names" {
      description = "Названия созданных подсетей"
      value       = [for subnet in yandex_vpc_subnet.prod-sub  : subnet.name]
    }

    output "subnet_cidrs" {
      description = "CIDR блоки подсетей"
      value       = [for subnet in yandex_vpc_subnet.prod-sub : subnet.v4_cidr_blocks]
    }

    output "subnet_zones" {
      description = "Зоны подсетей"
      value       = [for subnet in yandex_vpc_subnet.prod-sub : subnet.zone]
    }
    ```
    README.md
    ```
    ## Requirements

    | Name | Version |
    |------|---------|
    | <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >=1.8.4 |

    ## Providers

    | Name | Version |
    |------|---------|
    | <a name="provider_yandex"></a> [yandex](#provider\_yandex) | n/a |

    ## Modules

    No modules.

    ## Resources

    | Name | Type |
    |------|------|
    | [yandex_vpc_network.prod](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/vpc_network) | resource |
    | [yandex_vpc_subnet.prod-sub](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/vpc_subnet) | resource |

    ## Inputs

    | Name | Description | Type | Default | Required |
    |------|-------------|------|---------|:--------:|
    | <a name="input_env_name"></a> [env\_name](#input\_env\_name) | Название окружения (например, production, develop) | `string` | n/a | yes |
    | <a name="input_subnets"></a> [subnets](#input\_subnets) | Список подсетей с зонами и CIDR | <pre>list(object({<br/>    zone = string<br/>    cidr = string<br/>  }))</pre> | n/a | yes |

    ## Outputs

    | Name | Description |
    |------|-------------|
    | <a name="output_network_id"></a> [network\_id](#output\_network\_id) | Идентификатор созданной VPC сети |
    | <a name="output_subnet_cidrs"></a> [subnet\_cidrs](#output\_subnet\_cidrs) | CIDR блоки подсетей |
    | <a name="output_subnet_ids"></a> [subnet\_ids](#output\_subnet\_ids) | Идентификаторы созданных подсетей |
    | <a name="output_subnet_names"></a> [subnet\_names](#output\_subnet\_names) | Названия созданных подсетей |
    | <a name="output_subnet_zones"></a> [subnet\_zones](#output\_subnet\_zones) | Зоны подсетей |
    ```
### Задание 3
1. Выведите список ресурсов в стейте.
2. Полностью удалите из стейта модуль vpc.
3. Полностью удалите из стейта модуль vm.
4. Импортируйте всё обратно. Проверьте terraform plan. Значимых(!!) изменений быть не должно.
Приложите список выполненных команд и скриншоты процессы.
------
## Ответ на задание 3
1.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-3-1.png)
2.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-3-2.png)
3.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-3-3.png)
4.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-3-4.png)
5.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-3-5.png)
6.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-3-6.png)
7.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-3-7.png)

## Дополнительные задания (со звёздочкой*)

**Настоятельно рекомендуем выполнять все задания со звёздочкой.**   Они помогут глубже разобраться в материале.   
Задания со звёздочкой дополнительные, не обязательные к выполнению и никак не повлияют на получение вами зачёта по этому домашнему заданию. 


### Задание 4*

1. Измените модуль vpc так, чтобы он мог создать подсети во всех зонах доступности, переданных в переменной типа list(object) при вызове модуля.  
  
Пример вызова
```
module "vpc_prod" {
  source       = "./vpc"
  env_name     = "production"
  subnets = [
    { zone = "ru-central1-a", cidr = "10.0.1.0/24" },
    { zone = "ru-central1-b", cidr = "10.0.2.0/24" },
    { zone = "ru-central1-c", cidr = "10.0.3.0/24" },
  ]
}

module "vpc_dev" {
  source       = "./vpc"
  env_name     = "develop"
  subnets = [
    { zone = "ru-central1-a", cidr = "10.0.1.0/24" },
  ]
}
```

Предоставьте код, план выполнения, результат из консоли YC.
------
## Ответ на задание 4
1.  variables.tf
    ```
    variable "vpc_prod_subnets" {
      description = "Список подсетей с указанием зоны и CIDR блока"
      type = list(object({
        zone = string
        cidr = string
      }))
      default = [
        { zone = "ru-central1-a", cidr = "10.0.1.0/24" },
        { zone = "ru-central1-b", cidr = "10.0.2.0/24" },
        { zone = "ru-central1-d", cidr = "10.0.3.0/24" },
      ]
    }
    ```
2.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-4-1.png)
3.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-4-2.png)

### Задание 5*

1. Напишите модуль для создания кластера managed БД Mysql в Yandex Cloud с одним или несколькими(2 по умолчанию) хостами в зависимости от переменной HA=true или HA=false. Используйте ресурс yandex_mdb_mysql_cluster: передайте имя кластера и id сети.
2. Напишите модуль для создания базы данных и пользователя в уже существующем кластере managed БД Mysql. Используйте ресурсы yandex_mdb_mysql_database и yandex_mdb_mysql_user: передайте имя базы данных, имя пользователя и id кластера при вызове модуля.
3. Используя оба модуля, создайте кластер example из одного хоста, а затем добавьте в него БД test и пользователя app. Затем измените переменную и превратите сингл хост в кластер из 2-х серверов.
4. Предоставьте план выполнения и по возможности результат. Сразу же удаляйте созданные ресурсы, так как кластер может стоить очень дорого. Используйте минимальную конфигурацию.

------
## Ответ на задание 5
1.  main.tf (модуля)
    ```
    resource "yandex_mdb_mysql_cluster" "mysql_cluster" {
      name                = var.name
      environment         = var.mysql_resources.data.environment
      network_id          = var.network_id
      version             = var.mysql_resources.data.version
      deletion_protection = var.mysql_resources.data.deletion_protection

      resources {
        resource_preset_id = var.mysql_resources.data.resource_preset_id
        disk_type_id       = var.mysql_resources.data.disk_type_id
        disk_size          = var.mysql_resources.data.disk_size
      }

      dynamic "host" {
        for_each = var.HA ? [for i in range(var.host_count) : i] : [1]
        content {
          subnet_id       = var.subnet_id
          zone            = var.mysql_resources.data.zone
          assign_public_ip = var.mysql_resources.data.assign_public_ip
        }
      }
    }
    ```
    variables.tf (модуля)
    ```
    variable "name" {
      type    = string
      default = null
    }

    variable "network_id" {
      type    = string
      default = null
    }

    variable "subnet_id" {
      type    = string
      default = null
    }

    variable "HA" {
      type        = bool
      default     = null
    }

    variable "host_count" {
      type        = number
      default     = 2
    }

    variable "mysql_resources" {
      type = map(object({
        environment = string
        version = string
        deletion_protection = bool
        resource_preset_id = string
        disk_type_id  = string
        disk_size = number
        zone  = string
        assign_public_ip  = bool
      }))
      default = {
        "data" = {
          environment = "PRESTABLE"
          version = "8.0"
          deletion_protection = false
          resource_preset_id = "c3-c2-m4"
          disk_type_id  = "network-hdd"
          disk_size = 20
          zone  = "ru-central1-a"
          assign_public_ip  = true
        }
      }
    }
    ```
    outputs.tf (модуля)
    ```
    output "yandex_mdb_mysql_cluster" {
      value= yandex_mdb_mysql_cluster.mysql_cluster
    }
    ```
2.  main.tf (модуля)
    ```
    resource "yandex_mdb_mysql_database" "db" {
      cluster_id = var.cluster_id
      name       = var.db_name
    }

    resource "yandex_mdb_mysql_user" "testuser" {
      cluster_id = var.cluster_id
      name       = var.db_user
      password   = "qwerty321"
      permission {
        database_name = yandex_mdb_mysql_database.db.name
        roles         = ["ALL"]
      }
    }
    ```
    variables.tf
    ```
    variable "cluster_id" {
      type        = string
      default     = null
    }

    variable "db_name" {
      type        = string
      default     = null
    }

    variable "db_user" {
      type        = string
      default     = null
    }
    ```
 3. main.tf
    ```
    module "mysql" {
      source = "./modules/mysql"
      name = var.cluster_name
      network_id = module.vpc_dev.network_id
      subnet_id = module.vpc_dev.subnet_id
      HA = true  
    }

    module "mysql_db" {
      source = "./modules/user_db"
      cluster_id = module.mysql.yandex_mdb_mysql_cluster.id
      db_name = var.db_managed_name
      db_user = var.db_managed_user
    }
    ```   
    variables.tf
    ```
    variable "cluster_name" {
      type = string
      default = "example"
    }

    variable "db_managed_name" {
      type = string
      default = "test"
    }

    variable "db_managed_user" {
      type = string
      default = "app"
    }
    ```  
4.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-5-1.png)
5.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-5-2.png) 
6.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-5-3.png)
7.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-5-4.png)
8.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-5-5.png)
9.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-5-6.png)   
### Задание 6*
1. Используя готовый yandex cloud terraform module и пример его вызова(examples/simple-bucket): https://github.com/terraform-yc-modules/terraform-yc-s3 .
Создайте и не удаляйте для себя s3 бакет размером 1 ГБ(это бесплатно), он пригодится вам в ДЗ к 5 лекции.
----
## Ответ на задание 6
1.  main.tf
    ```
    module "s3" {
      source = "git::https://github.com/terraform-yc-modules/terraform-yc-s3.git?ref=master"

      bucket_name = "simple-bucket-${random_string.unique_id.result}"
      versioning = {
        enabled = true
      }
      max_size = var.max_size
    }
    ```
    variables.tf
    ```
    variable "max_size" {
      type = string
      default = "1073741824"
    }
    ```
2.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-6-1.png)  

### Задание 7*

1. Разверните у себя локально vault, используя docker-compose.yml в проекте.
2. Для входа в web-интерфейс и авторизации terraform в vault используйте токен "education".
3. Создайте новый секрет по пути http://127.0.0.1:8200/ui/vault/secrets/secret/create
Path: example  
secret data key: test 
secret data value: congrats!  
4. Считайте этот секрет с помощью terraform и выведите его в output по примеру:
```
provider "vault" {
 address = "http://<IP_ADDRESS>:<PORT_NUMBER>"
 skip_tls_verify = true
 token = "education"
}
data "vault_generic_secret" "vault_example"{
 path = "secret/example"
}

output "vault_example" {
 value = "${nonsensitive(data.vault_generic_secret.vault_example.data)}"
} 

Можно обратиться не к словарю, а конкретному ключу:
terraform console: >nonsensitive(data.vault_generic_secret.vault_example.data.<имя ключа в секрете>)
```
5. Попробуйте самостоятельно разобраться в документации и записать новый секрет в vault с помощью terraform. 

----
## Ответ на задание 7
1.  docker-compose.yml
    ```
    services:
      vault:
        image: hashicorp/vault:latest
        container_name: vault
        cap_add:
          - IPC_LOCK
        environment:
          VAULT_DEV_ROOT_TOKEN_ID: "education" 
          VAULT_DEV_LISTEN_ADDRESS: "0.0.0.0:8200"
        ports:
          - "8200:8200"
        volumes:
          - ./vault-data:/vault/file
        command: "server -dev -dev-root-token-id=education -dev-listen-address=0.0.0.0:8200"
    ```
2.  main.tf
    ```
    data "vault_generic_secret" "vault_example"{
      path = var.secret_path
    }

    resource "vault_generic_secret" "vault_example2" {
      path = var.secret_path1
      data_json = jsonencode(var.secret_data)
    }

    data "vault_generic_secret" "vault_example3"{
    path = var.secret_path1
    }
    ```
    provider.tf
    ```
    terraform {
      required_providers {
        yandex = {
          source = "yandex-cloud/yandex"
        }
      }
      required_version = ">=1.8.4"
    }

    provider "vault" {
      address = var.vault_address
      token   = var.vault_token
    }
    ```
    variables.tf
    ```
    variable "vault_address" {
      description = "Адрес Vault сервера"
      type        = string
      default     = "http://127.0.0.1:8200"
    }

    variable "vault_token" {
      description = "Токен доступа Vault"
      type        = string
      sensitive   = true
    }

    variable "secret_path" {
      description = "Путь, по которому будет записан секрет в Vault"
      type        = string
      default     = "secret/example"
    }

    variable "secret_path1" {
      description = "Путь, по которому будет записан секрет в Vault"
      type        = string
      default     = "secret/testnv/"
    }

    variable "secret_data" {
      description = "Данные секрета для записи в Vault"
      type        = map(string)
      default = {
        username = "root"
        password = "P@ssw0rd"
      }
    }
    ```
    outputs.tf
    ```
    output "vault_example" {
      value = "${nonsensitive(data.vault_generic_secret.vault_example.data)}"
    } 

    output "vault_example3" {
      value = nonsensitive(data.vault_generic_secret.vault_example3.data)
    }
    ```
2.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-7-1.png)
3.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-7-2.png) 
4.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-7-3.png)
5.  ![alt text](https://github.com/VN351/ter-hw-4/raw/main/images/task-7-4.png)

### Задание 8*
Попробуйте самостоятельно разобраться в документаци и с помощью terraform remote state разделить root модуль на два отдельных root-модуля: создание VPC , создание ВМ . 

### Правила приёма работы

В своём git-репозитории создайте новую ветку terraform-04, закоммитьте в эту ветку свой финальный код проекта. Ответы на задания и необходимые скриншоты оформите в md-файле в ветке terraform-04.

В качестве результата прикрепите ссылку на ветку terraform-04 в вашем репозитории.

**Важно.** Удалите все созданные ресурсы.

### Критерии оценки

Зачёт ставится, если:

* выполнены все задания,
* ответы даны в развёрнутой форме,
* приложены соответствующие скриншоты и файлы проекта,
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку работу отправят, если:

* задание выполнено частично или не выполнено вообще,
* в логике выполнения заданий есть противоречия и существенные недостатки. 




