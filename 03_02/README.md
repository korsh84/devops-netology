# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.
  - cd is a shell builtin, то есть встроенная утилита
  - по стандарту она не относится к специальным встроенным утилитам, поэтому должна быть для внешнего доступна исполнения через exec, однако она влияет на окружение текущего shell, поэтому также реализована и как встроенная
  - думаю, что ее функционал мог бы быть реализован установкой специальной переменной, от которой shell отсчитывал бы относительные пути, поэтому допустима вариативность в реализации
  - в некоторых ОС реализована как отдельный скрипт для эмуляции регистронезависимости  
2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.
- grep -c <шаблон>
4. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
-  pstree -p -> systemd(1)
5. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?
- tty  
- >/dev/pts/0
- ls 2>/dev/pts/1
6. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
- cat <1.txt  >2.txt
7. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?
- ctrl+alt+F2 - открывает ссесию tty2, логинимся
- обратно переключаемся на pty ctrl+alt+F1
- ls >/dev/tty2
- опять переключаемся ctrl+alt+F2 и смотрим вывод
8. Выполните команду `bash 5>&1`. К чему она приведет? 
- запускает новую оболочку (другой PID), но ее вывод направляется в текущее окно
Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?
- создался новый дескриптор для потока вывода (5), который указывает на текущий терминал
- выведется netology в текущее окно терминала
 10. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.
``` bash 3>&1
command1 2>&1 >&3 3>&- | command2 3>&-
bash 3>&1
ls -R /ust 2>&1 >&3 3>&- | less 3>&-
```
 11. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?
- настройки окружения для текущей сессии shell
- ```ps wwe $$```

 12. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.
 - ```man proc```
 - cmdline - файл содержащий команду для запуска соотвествущего  процесса, за исключением  зомби процесса, для которых файл пустой
 - exe - символьная ссылка на исполняемый файл процесса  
 14. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.
`grep -we 'sse\w*' /proc/cpuinfo`
- sse4_2
 15. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

      ```bash
      vagrant@netology1:~$ ssh localhost 'tty'
      not a tty
     ```
Почитайте, почему так происходит, и как изменить поведение.
- ssh -t localhost 'tty' - форсированное включение псевдотерминала, когда нет локального tty

 16. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.
  ``vagrant@vagrant:~$ reptyr -L 1237``
 
 19. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.
 - tee читает из стандартного ввода (то есть результат  работы echo) и выводит в стандартный вывод
 - в перврм случае с правами запущен echo, а запись с обычными, а во втором запись вс правами root

 
 ---

---