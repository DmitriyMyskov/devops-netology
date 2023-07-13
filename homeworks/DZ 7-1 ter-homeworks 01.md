# Домашнее задание к занятию «Введение в Terraform»

### Цель задания

1. Установить и настроить Terrafrom.
2. Научиться использовать готовый код.

***

### Чеклист готовности к домашнему заданию

1. Скачайте и установите актуальную версию **terraform** >=1.4.X . Приложите скриншот вывода команды ```terraform --version```.
   
```shell
dmitriy@dmitriy-lin:~$ terraform version
Terraform v1.5.3
on linux_amd64
```
1. Скачайте на свой ПК данный git репозиторий. Исходный код для выполнения задания расположен в директории **01/src**.

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ ls -la
итого 20
drwxrwxr-x 2 dmitriy dmitriy 4096 июл 12 19:38 .
drwxrwxr-x 3 dmitriy dmitriy 4096 июл 12 19:38 ..
-rw-rw-r-- 1 dmitriy dmitriy  155 июл 12 19:38 .gitignore
-rw-rw-r-- 1 dmitriy dmitriy  756 июл 12 19:38 main.tf
-rw-rw-r-- 1 dmitriy dmitriy  206 июл 12 19:38 .terraformrc
```
3. Убедитесь, что в вашей ОС установлен docker.

```shell
dmitriy@dmitriy-lin:~$ docker version
Client:
 Version:           20.10.21
 API version:       1.41
 Go version:        go1.18.1
 Git commit:        20.10.21-0ubuntu1~20.04.2
 Built:             Thu Apr 27 05:56:19 2023
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true
```

***

### Задание 1

1. Перейдите в каталог [**src**](https://github.com/netology-code/ter-homeworks/tree/main/01/src). Скачайте все необходимые зависимости, использованные в проекте.

Все зависимости скачаются при выполнении команды terraform init. Выполняем:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0.1"...
- Finding latest version of hashicorp/random...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)
- Installing hashicorp/random v3.5.1...
- Installed hashicorp/random v3.5.1 (signed by HashiCorp)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!
```

Убеждаемся, что нужные дополнительные файлы появились в каталоге:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ ls -la
итого 28
drwxrwxr-x 3 dmitriy dmitriy 4096 июл 13 11:37 .
drwxrwxr-x 3 dmitriy dmitriy 4096 июл 12 19:38 ..
-rw-rw-r-- 1 dmitriy dmitriy  155 июл 12 19:38 .gitignore
-rw-rw-r-- 1 dmitriy dmitriy  756 июл 12 19:38 main.tf
drwxr-xr-x 3 dmitriy dmitriy 4096 июл 13 11:36 .terraform
-rw-r--r-- 1 dmitriy dmitriy 2384 июл 13 11:37 .terraform.lock.hcl
-rw-rw-r-- 1 dmitriy dmitriy  206 июл 12 19:38 .terraformrc
```

2. Изучите файл **.gitignore**. В каком terraform файле согласно этому `.gitignore` допустимо сохранить личную, секретную информацию?

Изучим `.gitignore`:

```shell
# Local .terraform directories and files
**/.terraform/*
.terraform*

# .tfstate files
*.tfstate
*.tfstate.*

# own secret vars store.
personal.auto.tfvars
```

Согласно содержимому файла `.gitignore` - личную и секретную информацию допустимо хранить в файле `personal.auto.tfvars`

3. Выполните код проекта. Найдите  в State-файле секретное содержимое созданного ресурса **random_password**, пришлите в качестве ответа конкретный ключ и его значение.

Выполнить код можно командой `terraform apply`:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ terraform apply

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # random_password.random_string will be created
  + resource "random_password" "random_string" {
      + bcrypt_hash = (sensitive value)
      + id          = (known after apply)
      + length      = 16
      + lower       = true
      + min_lower   = 1
      + min_numeric = 1
      + min_special = 0
      + min_upper   = 1
      + number      = true
      + numeric     = true
      + result      = (sensitive value)
      + special     = false
      + upper       = true
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

random_password.random_string: Creating...
random_password.random_string: Creation complete after 0s [id=none]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```
Рандомный пароль можно найти в State-файле в следующей строке "result":

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ cat terraform.tfstate 
{
  "version": 4,
  "terraform_version": "1.5.3",
  "serial": 1,
  "lineage": "4335b41d-41dc-68bd-1b9e-7d5b35222594",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "random_password",
      "name": "random_string",
      "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
      "instances": [
        {
          "schema_version": 3,
          "attributes": {
            "bcrypt_hash": "$2a$10$6TtRtYAo4sZ1PLihUUvgk.BHXD3asnBew0bMOxGNTzJyIuc1Dhu9S",
            "id": "none",
            "keepers": null,
            "length": 16,
            "lower": true,
            "min_lower": 1,
            "min_numeric": 1,
            "min_special": 0,
            "min_upper": 1,
            "number": true,
            "numeric": true,
            "override_special": null,
            "result": "B8yUvyMgVlSqu5j2",
            "special": false,
            "upper": true
          },
          "sensitive_attributes": []
        }
      ]
    }
  ],
  "check_results": null
}
```

4. Раскомментируйте блок кода, примерно расположенный на строчках 29-42 файла **main.tf**.
Выполните команду ```terraform validate```. Объясните в чем заключаются намеренно допущенные ошибки? Исправьте их.

Закомментирован был данный код находится между:

```shell
/*
resource "docker_image" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "1nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string_fake.resuld}"

  ports {
    internal = 80
    external = 8000
  }
}
*/
```

Убрал комментарий, выполнил команду `terraform validate`:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ terraform validate
╷
│ Error: Missing name for resource
│ 
│   on main.tf line 23, in resource "docker_image":
│   23: resource "docker_image" {
│ 
│ All resource blocks must have 2 labels (type, name).
╵
╷
│ Error: Invalid resource name
│ 
│   on main.tf line 28, in resource "docker_container" "1nginx":
│   28: resource "docker_container" "1nginx" {
│ 
│ A name must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.
```

Проверка валидности кода сообщает нам о двух ошибках. Во-первых, для первого resource блока задан только один label (type), а должно быть два (type, name), во-вторых, во втором resource блоке допущена опечатка - 1nginx.

Исправленный код:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ cat main.tf 
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
  required_version = ">=0.13" /*Многострочный комментарий.
 Требуемая версия terraform */
}
provider "docker" {}

#однострочный комментарий

resource "random_password" "random_string" {
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 8000
  }
}
```

Выполним валидацию заново:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ terraform validate
Success! The configuration is valid.
```

Как видно, после исправлений код стал абсолютно валидным и готов к применению.

5. Выполните код. В качестве ответа приложите вывод команды ```docker ps```

Выполняем код:

```shell
docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Creation complete after 14s [id=sha256:021283c8eb95be02b23db0de7f609d603553c6714785e7a673c6594a624ffbdanginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 5s [id=b9bcd36c2638072f81172f072faf11febb10d6da0735cc8be80041b53532a2c2]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Проверяем:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                       NAMES
b9bcd36c2638   021283c8eb95   "/docker-entrypoint.…"   21 seconds ago   Up 16 seconds   0.0.0.0:8000->80/tcp                        example_B8yUvyMgVlSqu5j2
```

6. Замените имя docker-контейнера в блоке кода на ```hello_world```, выполните команду ```terraform apply -auto-approve```.

Заменил имя контейнера на `hello_world`, выполнил команду `terraform apply -auto-approve`.

Результат выполнения:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                       NAMES
76b0b5dd4f49   021283c8eb95   "/docker-entrypoint.…"   18 seconds ago   Up 17 seconds   0.0.0.0:8000->80/tcp                        hello_world
```

Объясните своими словами, в чем может быть опасность применения ключа  ```-auto-approve``` ? В качестве ответа дополнительно приложите вывод команды ```docker ps```

Команда `terraform apply -auto-approve` исключает необходимость ввода подтверждения выполнения от пользователя. Таким образом, действия Terraform переходят от этапа Plan на этап Apply без всякого воздействия пользователя.

Опасность данного ключа заключается в том, что не требуется подтверждение администратора на создание или изменение инфраструктуры, а это значит, что в процессе применения инфраструктуры можно что-нибудь потерять из важного или насоздавать лишнего, если в процессе создания кода были допущены ошибки в инфраструктуре.

7. Уничтожьте созданные ресурсы с помощью **terraform**. Убедитесь, что все ресурсы удалены. Приложите содержимое файла **terraform.tfstate**.

```shell
random_password.random_string: Destroying... [id=none]
random_password.random_string: Destruction complete after 0s
docker_container.nginx: Destroying... [id=76b0b5dd4f49cd34ed46bc13b9b7f3b27987aea07a095b4b4fb340be211a88cd]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:021283c8eb95be02b23db0de7f609d603553c6714785e7a673c6594a624ffbdanginx:latest]
docker_image.nginx: Destruction complete after 0s
```

Содержимое файла `terraform.tfstate`:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ cat terraform.tfstate
{
  "version": 4,
  "terraform_version": "1.5.3",
  "serial": 11,
  "lineage": "4335b41d-41dc-68bd-1b9e-7d5b35222594",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```

8. Объясните, почему при этом не был удален docker образ **nginx:latest** ? Ответ подкрепите выдержкой из документации провайдера.

Для начала убедимся, что образ действительно не был удален:

```shell
dmitriy@dmitriy-lin:~/netology/ter-homeworks/01/src$ sudo docker image list
REPOSITORY                    TAG       IMAGE ID       CREATED         SIZE
nginx                         latest    021283c8eb95   8 days ago      187MB
```

Да, образ действительно остался в локальном хранилище. Это произошло потому что в файле `main.tf` указано - Хранить образ локально:

```shell
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}
```