# Домашнее задание к занятию "4.1. Командная оболочка Bash: Практические навыки" - Дайн Иван

## Обязательная задача 1

Есть скрипт:
```bash
a=1
b=2
c=a+b
d=$a+$b
e=$(($a+$b))
```

Какие значения переменным c,d,e будут присвоены? Почему?

| Переменная  | Значение | Обоснование |
| ------------- | ------------- | ------------- |
| `c`  | a+b  | нет $ перед a и b, поэтому баш воспринимает их как строку, а не переменную |
| `d`  | 1+2  | без явного указания типа переменной баш считает переменные как строки |
| `e`  | 3  | двойные скобки указывают, что операция арифметическая и баш преобразует строковые переменные  в числовые |


## Обязательная задача 2
На нашем локальном сервере упал сервис и мы написали скрипт, который постоянно проверяет его доступность, записывая дату проверок до тех пор, пока сервис не станет доступным (после чего скрипт должен завершиться). В скрипте допущена ошибка, из-за которой выполнение не может завершиться, при этом место на Жёстком Диске постоянно уменьшается. Что необходимо сделать, чтобы его исправить:
```bash
while ((1==1)
do
	curl https://localhost:4757
	if (($? != 0))
	then
		date >> curl.log
	fi
done
```
Исправленный вариант
```bash
while ((1==1))
do
	curl https://localhost:4757
	if (($? != 0))
	then
		date >> curl.log
	else break
	fi
done
```
Необходимо написать скрипт, который проверяет доступность трёх IP: `192.168.0.1`, `173.194.222.113`, `87.250.250.242` по `80` порту и записывает результат в файл `log`. Проверять доступность необходимо пять раз для каждого узла.

### Ваш скрипт:
```bash
#!/usr/bin/env bash
arrHosts=(192.168.0.1 173.194.222.113 87.250.250.242)
for i in  {1..5}
do
        date >>status.log
        for h in ${arrHosts[@]}
        do
                curl -s -m 1 $h:80 >/dev/null
                if [ "$?" -eq 0 ]
                then state="up"
                else state=$?
                fi
                echo "  " $h "  " is $state >>status.log
        done
done
```

## Обязательная задача 3
Необходимо дописать скрипт из предыдущего задания так, чтобы он выполнялся до тех пор, пока один из узлов не окажется недоступным. Если любой из узлов недоступен - IP этого узла пишется в файл error, скрипт прерывается.

### Ваш скрипт:
```bash
#!/usr/bin/env bash
arrHosts=(192.168.1.1 173.194.222.113 87.250.250.242)
f=0
while (($f == 0))
do
        date >>status.log
        for h in ${arrHosts[@]}
        do
                curl -s -m 1 $h:80 >/dev/null
                if (("$?" == 0))
                then state="up"
                else
                        echo $h >> error.log
                        f=1
                fi
                echo "  " $h "  " is $state >>status.log
        done
done
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Мы хотим, чтобы у нас были красивые сообщения для коммитов в репозиторий. Для этого нужно написать локальный хук для git, который будет проверять, что сообщение в коммите содержит код текущего задания в квадратных скобках и количество символов в сообщении не превышает 30. Пример сообщения: \[04-script-01-bash\] сломал хук.

### Ваш скрипт:
```bash
???
```