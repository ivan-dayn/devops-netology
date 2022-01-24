
# Домашнее задание к занятию "5.2. Применение принципов IaaC в работе с виртуальными машинами" - Дайн Иван

---

## Задача 1

- Опишите своими словами основные преимущества применения на практике IaaC паттернов.

`
Не происходит дрейф конфигураций в инфраструктуре, упрощается и ускоряется создание большого количества сред разработки и тестовых сред это приводит к тому, что развертывание и доставка кода становятся непрерывными, что в итоге даёт более быструю разработку и вывод готового продукта.
`
- Какой из принципов IaaC является основополагающим?

`
Новое слово - идемпотентность, а точнее уверенность в том, что создаваемая инфраструктура идемпотентна.
`

## Задача 2

- Чем Ansible выгодно отличается от других систем управление конфигурациями?
- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?

## Задача 3

Установить на личный компьютер:

- VirtualBox
```bash
$ virtualbox --help
Oracle VM VirtualBox VM Selector v6.1.26_Ubuntu
(C) 2005-2021 Oracle Corporation
All rights reserved.
```
- Vagrant
```bash
$ vagrant --version
Vagrant 2.2.19
```
- Ansible
```bash
$ ansible --version
ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/ubuntu/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 26 2021, 20:14:08) [GCC 9.3.0]
```

*Приложить вывод команд установленных версий каждой из программ, оформленный в markdown.*

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

- Создать виртуальную машину.

`
под управлением Ubuntu 20.04 всё
`
- Зайти внутрь ВМ, убедиться, что Docker установлен с помощью команды
```
docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```
