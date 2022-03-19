### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-03-yaml/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

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
- исправленный файл
	```json
	{ "info" : "Sample JSON output from our service\t",
      "elements": [
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
import os
import socket # модуль для получения ip
import re # обработка regexp
import yaml
import json
# открываем/создаем файлы лдля текущего запуска + предполагается, что старые ip лежат в файле old.lst
file = open("new.lst", "w")
service_list = ["drive.google.com", "mail.google.com", "google.com"]
json_dict = {}

## делаем чистые файлы
filej = open("list.json", "w")
filey = open("list.yaml", "w")
filej.close()
filey.close()


## генерация old.lst для первого запуска
if os.path.exists('old.lst') == 0:
    file_old = open("old.lst", "w")
    for host in service_list:
        current_ip = socket.gethostbyname(host)  # получили текущий ip
        current_result = [host, "-", current_ip]  # составили строку как для прошлого дз
        res1 = ' '.join(current_result)  # делаем строку старого формата
        file_old.write(res1 + '\n')
    file_old.close()

old_list = open('old.lst').read().splitlines()  # открываем построчно

for host in service_list:
    current_ip = socket.gethostbyname(host) # получили текущий ip
    current_result = [host, "-", current_ip] # составили строку как для прошлого дз
    json_dict[host] = current_ip
    res1 = ' '.join(current_result) # делаем строку старого формата
    file.write(res1 + '\n')
    print(res1)
    filtered = [x for x in old_list if re.match(host, x)]  # ищем строку с текущим хостом в старых данных
    filt_list = filtered[0].split() # разделяем первую найденную строку на слова
    if res1 not in old_list:  # если новая строка не совпадает со старым файлом
        print("[ERROR]", host, "IP mismatch:", current_ip, filt_list[2])  # выводим сообщение

### словарь для форматирования по заданию
val = {}
for k,v in json_dict.items():
    filej = open("list.json", "a")
    filey = open("list.yaml", "a")
    val[k] = v
    json.dump(val, filej)
    filej.write('\n') # каждый объект на отдельной строке
    yaml.dump([val], filey) # форматирование согласно заданию
    val.clear()


os.popen('cp new.lst old.lst')
filej.close()
filey.close()

```

### Вывод скрипта при запуске при тестировании:
```
#1 запуск
$ python 04v3.py
drive.google.com - 142.251.1.194
[ERROR] drive.google.com IP mismatch: 142.251.1.194 64.233.164.194
mail.google.com - 173.194.73.17
[ERROR] mail.google.com IP mismatch: 173.194.73.17 142.251.1.19
google.com - 74.125.205.100

#2 запуск
$ python 04v3.py
drive.google.com - 142.251.1.194
mail.google.com - 173.194.73.17
google.com - 74.125.205.100
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
{"drive.google.com": "142.251.1.194"}
{"mail.google.com": "173.194.73.17"}
{"google.com": "74.125.205.100"}
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
- drive.google.com: 142.251.1.194
- mail.google.com: 173.194.73.17
- google.com: 74.125.205.100
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
import yaml
import sys
import json
import os
import re

name = os.path.splitext(sys.argv[1])[0]  # вырезаем имя
ext = os.path.splitext(sys.argv[1])[1]  # вырезаем расширение
#print(name)
#print(ext)
# проверка препутанных расширений
full_file = open(sys.argv[1], "r")
each_line = full_file.read().replace('\n', '') # зачитаем весь файл в одну строку убирая переносы
full_file.close()
for item in each_line: #  поиск первого печатаемого (\S) символа в строке
    if re.match(r'\S', item):
       break
#print(item) #, end='')

# проверяем только первый печатный символ если он равен { а расширение не json, то меняем
if (item != '{'):
    if (ext == '.json'):
        print("change to .yml")
        if os.path.exists(f'{name}.yml'):
            os.remove(f'{name}.yml')
        os.rename(sys.argv[1],f'{name}.yml')
        ext = '.yml'
else:
    if (ext == '.yml'):
        print("change to .json")
        if os.path.exists(f'{name}.json'):
            os.remove(f'{name}.json')
        os.rename(sys.argv[1], f'{name}.json')
        ext = '.json'
#print(ext)


### конверсия
if ext == '.json':
    OUT = open(f'{name}.yml', 'w')
    with open(f'{name}.json', 'r') as IN:
        try:
            JSON = json.load(IN)
        except json.JSONDecodeError as er :
            print("JSON format error:")
            print(er)
        else:
            yaml.dump(JSON, OUT)
            OUT.close()

elif ext == '.yml':
     OUT = open(f'{name}.json', 'w')
     with open(f'{name}.yml', 'r') as IN:
         try:
             YAML = yaml.safe_load(IN)
         except yaml.YAMLError as er:
             print("YAML format error:")
             print(er)
         else:
             json.dump(YAML, OUT)
             OUT.close()
else:
    print("Format error:", ext)
```

### Пример работы скрипта:
```bash
#Ошибочные
$ python 0403_4.py test.yml
change to .json
.json
$ python 0403_4.py test.json
change to .yml
.yml
#неправильное расширение
$ python 0403_4.py 0403_4.py
Format error: .py
нормальные 
$ python 0403_4.py test.json
#test.json на входе
{"elements": [{"ip": 7175, "name": "first", "type": "server"}, {"ip": "71.78.22.43", "name": "second", "type": "proxy"}], "info": "Sample JSON output from our service\t"}
#test.yml на выходе
elements:
- ip: 7175
  name: first
  type: server
- ip: 71.78.22.43
  name: second
  type: proxy
info: "Sample JSON output from our service\t"
#ошибка синтаксиса
$ python 0403_4.py test.json
JSON format error:
Expecting ':' delimiter: line 1 column 131 (char 130)
###
$ python 0403_4.py test.yml
YAML format error:
mapping values are not allowed here
  in "test.yml", line 3, column 7

```
