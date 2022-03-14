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
import os
import socket
import pickle

host_dict_old = {} # словарь старых занчений
host_dict = {} # текущих

service_list = ['drive.google.com', 'mail.google.com', 'google.com']

## генерация file.pkl для первого запуска
if os.path.exists('file.pkl') == 0:
    file_old = open("file.pkl", "wb")
    for host in service_list:
        current_ip = socket.gethostbyname(host)
        host_dict[host] = current_ip
    pickle.dump(host_dict, file_old)
    file_old.close()
host_dict.clear()

# загрузка старых значений
f = open("file.pkl","rb")
host_dict_old = pickle.load(f)
f.close()

for host in service_list:
    current_ip = socket.gethostbyname(host)
    host_dict[host] = current_ip
    current_result = [host, "-", current_ip]
    res1 = ' '.join(current_result)
    print(res1)
    if host_dict_old[host] != host_dict[host]:
        print("[ERROR]", host, "IP mismatch:", current_ip, host_dict_old[host])

# сохраним старые значения
f = open("file.pkl","wb")
pickle.dump(host_dict,f)
f.close()
```
### Вывод скрипта при запуске при тестировании:
```
$ python 42v2.py
drive.google.com - 64.233.165.194
mail.google.com - 142.251.1.83
[ERROR] mail.google.com IP mismatch: 142.251.1.83 142.250.150.83
google.com - 74.125.131.102


# сразу второй запуск
$ python 42v2.py
drive.google.com - 64.233.165.194
mail.google.com - 142.251.1.83
google.com - 74.125.131.102

```


## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так получилось, что мы очень часто вносим правки в конфигурацию своей системы прямо на сервере. Но так как вся наша команда разработки держит файлы конфигурации в github и пользуется gitflow, то нам приходится каждый раз переносить архив с нашими изменениями с сервера на наш локальный компьютер, формировать новую ветку, коммитить в неё изменения, создавать pull request (PR) и только после выполнения Merge мы наконец можем официально подтвердить, что новая конфигурация применена. Мы хотим максимально автоматизировать всю цепочку действий. Для этого нам нужно написать скрипт, который будет в директории с локальным репозиторием обращаться по API к github, создавать PR для вливания текущей выбранной ветки в master с сообщением, которое мы вписываем в первый параметр при обращении к py-файлу (сообщение не может быть пустым). При желании, можно добавить к указанному функционалу создание новой ветки, commit и push в неё изменений конфигурации. С директорией локального репозитория можно делать всё, что угодно. Также, принимаем во внимание, что Merge Conflict у нас отсутствуют и их точно не будет при push, как в свою ветку, так и при слиянии в master. Важно получить конечный результат с созданным PR, в котором применяются наши изменения. 

### Ваш скрипт:  
```python
import requests
import os
import sys
import json

username = "korsh84"
token = "...." # здесь стоит настоящий token
reponame = "0402py" # из задания не очень понятно фиксировано ли имя репо
pull_url = f"https://api.github.com/repos/{username}/{reponame}/pulls"
push_url = f"https://github.com/{username}/{reponame}.git"
# .read() чтобы дождаться окончания команды 
os.popen(f'git add .').read()
os.popen(f'git commit -m {sys.argv[1]}').read()
os.popen(f'git push {push_url}').read()


branch_name = os.popen('git branch --show-current').read()
br = branch_name.replace('\n', '')
branch = print(f"\"{br}\"")


headers = {
    "Authorization": f'token {token}',
    "Accept": "application/vnd.github.v3+json"
}

data = {
    "title": sys.argv[1],
    "body": "commit message test",
    "head": br,
    "base": "main"
}
r = requests.post(pull_url, headers=headers, data=json.dumps(data))
```


---

### Как сдавать задания?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
