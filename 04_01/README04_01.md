
# Домашнее задание к занятию "4.1. Командная оболочка Bash: Практические навыки"

## Обязательная задача 1

Есть скрипт:
```bash
a=1
b=2
c=a+b
d=$a+$b
e=$(($a+$b))

echo "$a\n $b\n $c\n $d\n $e\n"
```

Какие значения переменным c,d,e будут присвоены? Почему?

| Переменная  | Значение | Обоснование |
| ------------- | ------------- | ------------- |
| `c`  | a+b  | это просто строка |
| `d`  | 1+2  | строка с подставленными значениями переменных |
| `e`  | 3  | вычисления через через обращение к переменной с двойными скобками |


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

### Ваш скрипт:
```bash
while ((1==1)) #потерялась скобка
do
	curl https://localhost:4757
	if (($? != 0))
	then
		date >> curl.log
	else  # добавим условие выхода из цикла при успешном выполнении команды
		break
	fi
done
```

## Обязательная задача 3
Необходимо написать скрипт, который проверяет доступность трёх IP: `192.168.0.1`, `173.194.222.113`, `87.250.250.242` по `80` порту и записывает результат в файл `log`. Проверять доступность необходимо пять раз для каждого узла.

### Ваш скрипт:
```bash
#!/bin/bash
array_ip=("192.168.0.1" "173.194.222.113" "87.250.250.242")
for i in ${array_ip[@]}
do
	a=5
	echo "----$i test---------" >>curl.log
	while (($a > 0))
	do
		let "a -= 1"
		curl --connect-timeout 1 https://${i}:80   >>curl.log 2>&1
	done
done


```

## Обязательная задача 4
Необходимо дописать скрипт из предыдущего задания так, чтобы он выполнялся до тех пор, пока один из узлов не окажется недоступным. Если любой из узлов недоступен - IP этого узла пишется в файл error, скрипт прерывается.

### Ваш скрипт:
```bash
#!/bin/bash
array_ip=("173.194.222.113" "87.250.250.242" "192.168.0.1")
while ((1==1))
do
	for i in ${array_ip[@]}
	do
	echo "----$i test---------" >>curl.log
	curl --connect-timeout 1 http://${i}:80   >>curl.log 2>&1
	if (($? != 0))
		then
			echo "Error on $i" >> error.log
			date >> error.log
			break 2 #прерывание внешнего цикла
	fi
	done
done

```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Мы хотим, чтобы у нас были красивые сообщения для коммитов в репозиторий. Для этого нужно написать локальный хук для git, который будет проверять, что сообщение в коммите содержит код текущего задания в квадратных скобках и количество символов в сообщении не превышает 30. Пример сообщения: \[04-script-01-bash\] сломал хук.

### Ваш скрипт:
```bash
???
```