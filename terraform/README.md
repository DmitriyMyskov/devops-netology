## Добавление информации об исключениях при commit -ах

## **/.terraform/*
Исключается скрытый каталог terraform и все его содержимое в любых директориях

## *.tfstate
## *.tfstate.*
- Исключаются любые файлы с расширением -.tfstate в любых директориях
- Исключаются файлы, в названии которых присутствует строка -.tfstate. с любым расширением в любых директориях

## crash.log
## crash.*.log
- Исключается указанный файл логов находящийся в любых директориях
- Исключается файл логов, в названии которого присутствует строка -crash. с расширением -.log

## *.tfvars
## *.tfvars.json
- Исключается  файлы с расширением -.tfvars в любых директориях
- Исключаются файлы в назнавании которых присуствует строка -.tfvars и расширением -.json в любых директориях 

## override.tf
## override.tf.json
## *_override.tf
## *_override.tf.json
- Исключается указанный файл в любых директориях
- Исключается указанный файл в любых директориях
- Исключается файл, в назавнии которого присутствует строка -_override с расширением -.tf в любых директориях
- Исключается файл, в назавнии которого присутсвует строка -_override.tf с расширением -.json в любых директориях

## .terraformrc
## terraform.rc
- Исключается скрытый файл -.terraformrc в любых директориях
- Исключается указанный файл в любых директориях