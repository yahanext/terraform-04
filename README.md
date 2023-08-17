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

### Задание 1

1. Возьмите из [демонстрации к лекции готовый код](https://github.com/netology-code/ter-homeworks/tree/main/04/demonstration1) для создания ВМ с помощью remote-модуля.
2. Создайте одну ВМ, используя этот модуль. В файле cloud-init.yml необходимо использовать переменную для ssh-ключа вместо хардкода. Передайте ssh-ключ в функцию template_file в блоке vars ={} .
Воспользуйтесь [**примером**](https://grantorchard.com/dynamic-cloudinit-content-with-terraform-file-templates/). Обратите внимание, что ssh-authorized-keys принимает в себя список, а не строку.
3. Добавьте в файл cloud-init.yml установку nginx.
4. Предоставьте скриншот подключения к консоли и вывод команды ```sudo nginx -t```.

------
![https://github.com/yahanext/terraform-04/blob/main/scr41.png](https://github.com/yahanext/terraform-04/blob/main/scr41.png)
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
```
Модуль vpc
yaha@yahawork:~/terr/terr2/devops28-terr1/src/vpc$ cat main.tf
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">=0.13"
}

resource "yandex_vpc_network" "net_name" {
  name = var.env_name
}

resource "yandex_vpc_subnet" "subnet_name" {
  name           = "${var.env_name}-${var.zone}"
  zone           = var.zone
  network_id     = yandex_vpc_network.net_name.id
  v4_cidr_blocks = [var.cidr]
}yaha@yahawork:~/terr/terr2/devops28-terr1/src/vpc$ 
```
Скрин
![https://github.com/yahanext/terraform-04/blob/main/scr42.png](https://github.com/yahanext/terraform-04/blob/main/scr42.png)

### Задание 3
1. Выведите список ресурсов в стейте.
2. Полностью удалите из стейта модуль vpc.
3. Полностью удалите из стейта модуль vm.
4. Импортируйте всё обратно. Проверьте terraform plan. Изменений быть не должно.
Приложите список выполненных команд и скриншоты процессы.
```
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform state list
data.template_file.cloudinit
module.test-vm.data.yandex_compute_image.my_image
module.test-vm.yandex_compute_instance.vm[0]
module.vpc.yandex_vpc_network.net_name
module.vpc.yandex_vpc_subnet.subnet_name
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ 
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform state rm 'module.vpc'
Removed module.vpc.yandex_vpc_network.net_name
Removed module.vpc.yandex_vpc_subnet.subnet_name
Successfully removed 2 resource instance(s).
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ 
aha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform state rm 'module.test-vm'
Removed module.test-vm.data.yandex_compute_image.my_image
Removed module.test-vm.yandex_compute_instance.vm[0]
Successfully removed 2 resource instance(s).
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform state list
data.template_file.cloudinit
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ 
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform import 'module.vpc.yandex_vpc_network.net_name' 'enp6ensqotaidjfn9dcs'
data.template_file.cloudinit: Reading...
data.template_file.cloudinit: Read complete after 0s [id=fd5f0b9b0f4ad231425e75645e2af659e9a051a411bfcdd8926a6f4e8467d341]
module.vpc.yandex_vpc_network.net_name: Importing from ID "enp6ensqotaidjfn9dcs"...
module.test-vm.data.yandex_compute_image.my_image: Reading...
module.vpc.yandex_vpc_network.net_name: Import prepared!
  Prepared yandex_vpc_network for import
module.vpc.yandex_vpc_network.net_name: Refreshing state... [id=enp6ensqotaidjfn9dcs]
module.test-vm.data.yandex_compute_image.my_image: Read complete after 2s [id=fd8mkq33tt9kvi2c10e5]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.

yaha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform import 'module.vpc.yandex_vpc_subnet.subnet_name' 'e9b8ejnj21hbm1fi2rsk'
data.template_file.cloudinit: Reading...
data.template_file.cloudinit: Read complete after 0s [id=fd5f0b9b0f4ad231425e75645e2af659e9a051a411bfcdd8926a6f4e8467d341]
module.test-vm.data.yandex_compute_image.my_image: Reading...
module.vpc.yandex_vpc_subnet.subnet_name: Importing from ID "e9b8ejnj21hbm1fi2rsk"...
module.vpc.yandex_vpc_subnet.subnet_name: Import prepared!
  Prepared yandex_vpc_subnet for import
module.vpc.yandex_vpc_subnet.subnet_name: Refreshing state... [id=e9b8ejnj21hbm1fi2rsk]
module.test-vm.data.yandex_compute_image.my_image: Read complete after 4s [id=fd8mkq33tt9kvi2c10e5]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.

yaha@yahawork:~/terr/terr2/devops28-terr1/src$ 
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform import 'module.test-vm.yandex_compute_instance.vm[0]' 'fhmg9fbgf4st7ndqhbtf'
data.template_file.cloudinit: Reading...
data.template_file.cloudinit: Read complete after 0s [id=fd5f0b9b0f4ad231425e75645e2af659e9a051a411bfcdd8926a6f4e8467d341]
module.test-vm.data.yandex_compute_image.my_image: Reading...
module.test-vm.data.yandex_compute_image.my_image: Read complete after 1s [id=fd8mkq33tt9kvi2c10e5]
module.test-vm.yandex_compute_instance.vm[0]: Importing from ID "fhmg9fbgf4st7ndqhbtf"...
module.test-vm.yandex_compute_instance.vm[0]: Import prepared!
  Prepared yandex_compute_instance for import
module.test-vm.yandex_compute_instance.vm[0]: Refreshing state... [id=fhmg9fbgf4st7ndqhbtf]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.

yaha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform state list
data.template_file.cloudinit
module.test-vm.data.yandex_compute_image.my_image
module.test-vm.yandex_compute_instance.vm[0]
module.vpc.yandex_vpc_network.net_name
module.vpc.yandex_vpc_subnet.subnet_name
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ 
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ terraform plan
data.template_file.cloudinit: Reading...
data.template_file.cloudinit: Read complete after 0s [id=fd5f0b9b0f4ad231425e75645e2af659e9a051a411bfcdd8926a6f4e8467d341]
module.test-vm.data.yandex_compute_image.my_image: Reading...
module.vpc.yandex_vpc_network.net_name: Refreshing state... [id=enp6ensqotaidjfn9dcs]
module.test-vm.data.yandex_compute_image.my_image: Read complete after 2s [id=fd8mkq33tt9kvi2c10e5]
module.vpc.yandex_vpc_subnet.subnet_name: Refreshing state... [id=e9b8ejnj21hbm1fi2rsk]
module.test-vm.yandex_compute_instance.vm[0]: Refreshing state... [id=fhmg9fbgf4st7ndqhbtf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # module.test-vm.yandex_compute_instance.vm[0] will be updated in-place
  ~ resource "yandex_compute_instance" "vm" {
      + allow_stopping_for_update = true
        id                        = "fhmg9fbgf4st7ndqhbtf"
        name                      = "develop-web-0"
        # (11 unchanged attributes hidden)

      - timeouts {}

        # (6 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
yaha@yahawork:~/terr/terr2/devops28-terr1/src$ 
```
```
Скриншоты
```
![https://github.com/yahanext/terraform-04/blob/main/scr43.png](https://github.com/yahanext/terraform-04/blob/main/scr43.png)
![https://github.com/yahanext/terraform-04/blob/main/scr44.png](https://github.com/yahanext/terraform-04/blob/main/scr44.png)
![https://github.com/yahanext/terraform-04/blob/main/scr45.png](https://github.com/yahanext/terraform-04/blob/main/scr45.png)
![https://github.com/yahanext/terraform-04/blob/main/scr46.png](https://github.com/yahanext/terraform-04/blob/main/scr46.png)
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

### Задание 5*

1. Напишите модуль для создания кластера managed БД Mysql в Yandex Cloud с одним или тремя хостами в зависимости от переменной HA=true или HA=false. Используйте ресурс yandex_mdb_mysql_cluster: передайте имя кластера и id сети.
2. Напишите модуль для создания базы данных и пользователя в уже существующем кластере managed БД Mysql. Используйте ресурсы yandex_mdb_mysql_database и yandex_mdb_mysql_user: передайте имя базы данных, имя пользователя и id кластера при вызове модуля.
3. Используя оба модуля, создайте кластер example из одного хоста, а затем добавьте в него БД test и пользователя app. Затем измените переменную и превратите сингл хост в кластер из 2-х серверов.
4. Предоставьте план выполнения и по возможности результат. Сразу же удаляйте созданные ресурсы, так как кластер может стоить очень дорого. Используйте минимальную конфигурацию.

### Задание 6*

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