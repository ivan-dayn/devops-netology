
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

`Проще начать работу, т.к. использует ssh и не требутся разворачивать pki инфраструктуру`
- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?

`На мой взгляд более надёжный push т.к. он более контролируем и нагляден, поскольку сам определяю куда, когда и какую конфигурацию отправить`

## Задача 3

Установить на личный компьютер:

`Выполнял на свежеустановленной Ubuntu 20.04`
- VirtualBox
```bash
get -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
sudo apt-get update
sudo apt install virtualbox virtualbox-ext-pack

$ virtualbox --help
Oracle VM VirtualBox VM Selector v6.1.26_Ubuntu
(C) 2005-2021 Oracle Corporation
All rights reserved.
```
- Vagrant
```bash
sudo apt install curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install vagrant

$ vagrant --version
Vagrant 2.2.19
```
- Ansible
```bash
sudo add-apt-repository universe multiverse
sudo add-apt-repository universe
sudo add-apt-repository multiverse
sudo apt update
sudo apt install ansible


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

```bash

sudo apt install software-properties-common
sudo apt install python 3.8
sudo apt install python3.8

$ssh-keygen
```
- Создать виртуальную машину.
```bash
mkdir vagrant
mkdir ansible ansible/inventory
cd vagrant
vagrant init 
```
`Разложил файлы из репозитория по каталогам`
```bash
vagrant up

получил ошибку, т.к. файл provosion.yml расположил неверно, переложил

vagrant provision

==> server1.netology: Running provisioner: ansible...
    server1.netology: Running ansible-playbook...

PLAY [nodes] *******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [server1.netology]

TASK [Create directory for ssh-keys] *******************************************
ok: [server1.netology]

TASK [Adding rsa-key in /root/.ssh/authorized_keys] ****************************
changed: [server1.netology]

TASK [Checking DNS] ************************************************************
changed: [server1.netology]

TASK [Installing tools] ********************************************************
ok: [server1.netology] => (item=['git', 'curl'])

TASK [Installing docker] *******************************************************
changed: [server1.netology]

TASK [Add the current user to docker group] ************************************
ok: [server1.netology]

PLAY RECAP *********************************************************************
server1.netology           : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- Зайти внутрь ВМ, убедиться, что Docker установлен с помощью команды
```
vagarnt ssh
docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```
