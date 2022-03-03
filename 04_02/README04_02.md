# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательные задания

1. Есть скрипт:
	```python
    #!/usr/bin/env python3
	a = 1
	b = '2'
	c = a + b
	```
	* Какое значение будет присвоено переменной c?
		- ошибка исполнения:  TypeError: unsupported operand type(s) for +: 'int' and 'str'
	* Как получить для переменной c значение 12?
		- объявить как строку  a = '1'
	* Как получить для переменной c значение 3?
		- объявить числом b = 2

1. Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?
### Ваш скрипт:   
    ```python
    #!F:\Docs\2021_devops\pythonProject\venv\Scripts\python.exe
    
    import os
    bash_command = ["cd ../git02", "git status"]
    path_command = ["cd ../git02", "pwd"]
    result_os = os.popen(' && '.join(bash_command)).read()
    result_path = os.popen(' && '.join(path_command)).read()
    is_change = False
    for result in result_os.split('\n'):
        if result.find('modified') != -1:
            prepare_result = result.replace('\tmodified:   ', '')
            #print('../git02', prepare_result, sep='/')
            print(result_path.rstrip(), prepare_result, sep='/')
	```
### Вывод скрипта при запуске при тестировании:
```
Andrey@korshun2 MINGW64 /f/Docs/2021_devops/pythonProject/hw04-02
$ python.exe 0402a.py
/f/Docs/2021_devops/pythonProject/git02/index.php
/f/Docs/2021_devops/pythonProject/git02/second.php
```

1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:   
```python
     #!F:\Docs\2021_devops\pythonProject\venv\Scripts\python.exe
     import sys
     import os
     
     bash_cd = ["cd", sys.argv[1]]
     bash_command = [(' '.join(bash_cd)), "git status 2>&1"]
     path_command = [(' '.join(bash_cd)), "pwd"]
     result_os = os.popen(' && '.join(bash_command)).read()
     result_path = os.popen(' && '.join(path_command)).read()
     for result in result_os.split('\n'):
         if result.find('fatal:') == 0:
             print('Not repo')
             break
         elif result.find('modified') != -1:
             prepare_result = result.replace('\tmodified:   ', '')
             print(result_path.rstrip(), prepare_result, sep='/')
```
### Вывод скрипта при запуске при тестировании:
```
Andrey@korshun2 MINGW64 /f/Docs/2021_devops/pythonProject/hw04-02
$ python.exe 0402_2.py ../../netology.devops/git03_hw
/f/Docs/2021_devops/netology.devops/git03_hw/1cs.txt

$ python.exe 0402_2.py ../../netology.devops
Not repo
```

1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: drive.google.com, mail.google.com, google.com.
### Ваш скрипт:  
```python
#!F:\Docs\2021_devops\pythonProject\venv\Scripts\python.exe
import os
import socket
import re
file = open("new.lst", "w")
old_list = open('old.lst').read().splitlines()
service_list = ['drive.google.com', 'mail.google.com', 'google.com']
for host in service_list:
    current_ip = socket.gethostbyname(host)
    current_result = [host, "-", current_ip]
    res1 = ' '.join(current_result)
    file.write(res1+ '\n')
    print(res1)
    filtered = [x for x in old_list if re.match(host, x)]
    filt_list = filtered[0].split()
    if res1 not in old_list:
       print("[ERROR]", host, "IP mismatch:", current_ip, filt_list[2])
os.popen('cp new.lst old.lst')
```
### Вывод скрипта при запуске при тестировании:
```
$ python.exe 0402_4.py
drive.google.com - 209.85.233.194
[ERROR] drive.google.com IP mismatch: 209.85.233.194 64.233.162.194
mail.google.com - 142.251.1.17
[ERROR] mail.google.com IP mismatch: 142.251.1.17 74.125.131.83
google.com - 74.125.205.102

# сразу второй запуск
$ python.exe 0402_4.py
drive.google.com - 209.85.233.194
mail.google.com - 142.251.1.17
google.com - 74.125.205.102
```


## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так получилось, что мы очень часто вносим правки в конфигурацию своей системы прямо на сервере. Но так как вся наша команда разработки держит файлы конфигурации в github и пользуется gitflow, то нам приходится каждый раз переносить архив с нашими изменениями с сервера на наш локальный компьютер, формировать новую ветку, коммитить в неё изменения, создавать pull request (PR) и только после выполнения Merge мы наконец можем официально подтвердить, что новая конфигурация применена. Мы хотим максимально автоматизировать всю цепочку действий. Для этого нам нужно написать скрипт, который будет в директории с локальным репозиторием обращаться по API к github, создавать PR для вливания текущей выбранной ветки в master с сообщением, которое мы вписываем в первый параметр при обращении к py-файлу (сообщение не может быть пустым). При желании, можно добавить к указанному функционалу создание новой ветки, commit и push в неё изменений конфигурации. С директорией локального репозитория можно делать всё, что угодно. Также, принимаем во внимание, что Merge Conflict у нас отсутствуют и их точно не будет при push, как в свою ветку, так и при слиянии в master. Важно получить конечный результат с созданным PR, в котором применяются наши изменения. 


---

### Как сдавать задания?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
