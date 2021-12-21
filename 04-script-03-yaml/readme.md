# Домашнее задание к занятию "4.3. Языки разметки JSON и YAML"


## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис
  
 ```json
     { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            },
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43"
            }
        ]
    }
 ```

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
import socket
import time
import yaml
import json

namesdict = {'drive.google.com': '', 'mail.google.com': '', 'google.com': ''}
for key in namesdict:
    namesdict[key] = socket.gethostbyname(key)
with open('log.json', 'w') as outf:
    outf.write(json.dumps(namesdict, indent=2))
with open('log.yaml', 'w') as outf:
    outf.write(yaml.dump(namesdict))
while True:
    for key in namesdict:
        ip = socket.gethostbyname(key)
        if namesdict[key] != ip:
            print('[ERROR] ' + key + ' IP mismatch: ' + namesdict[key] + ' -> ' + ip)
            namesdict[key] = ip
            with open('log.json', 'w') as outf:
                outf.write(json.dumps(namesdict, indent=2))
            with open('log.yaml', 'w') as outf:
                outf.write(yaml.dump(namesdict))
        else:
            print(key, namesdict[key])
    time.sleep(5)
```

### Вывод скрипта при запуске при тестировании:
```
C:\Users\admin\PycharmProjects\pythonProject4-2\venv\Scripts\python.exe C:/Users/admin/PycharmProjects/pythonProject4-2/script3.py
drive.google.com 8.8.8.8
mail.google.com 173.194.221.83
google.com 64.233.165.113
[ERROR] drive.google.com IP mismatch: 8.8.8.8 -> 142.251.1.194
mail.google.com 173.194.221.83
google.com 64.233.165.113
drive.google.com 142.251.1.194
mail.google.com 173.194.221.83
google.com 64.233.165.113
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
{
  "drive.google.com": "142.251.1.194",
  "mail.google.com": "173.194.221.83",
  "google.com": "64.233.165.113"
}
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
drive.google.com: 142.251.1.194
google.com: 64.233.165.113
mail.google.com: 173.194.221.83
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так как команды в нашей компании никак не могут прийти к единому мнению о том, какой формат разметки данных использовать: JSON или YAML, нам нужно реализовать парсер из одного формата в другой. Он должен уметь:
   * Принимать на вход имя файла
   * Проверять формат исходного файла. Если файл не json или yml - скрипт должен остановить свою работу
   * Распознавать какой формат данных в файле. Считается, что файлы *.json и *.yml могут быть перепутаны
   * Перекодировать данные из исходного формата во второй доступный (из JSON в YAML, из YAML в JSON)
   * При обнаружении ошибки в исходном файле - указать в стандартном выводе строку с ошибкой синтаксиса и её номер
   * Полученный файл должен иметь имя исходного файла, разница в наименовании обеспечивается разницей расширения файлов

### Ваш скрипт:
```python
???
```

### Пример работы скрипта:
???
