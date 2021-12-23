# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`. Обратите внимание, что `strace` выдаёт результат своей работы в поток stderr, а не в stdout.

   - /tmp содержат 2 вызова, cd делает chdir 
> stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0

> chdir("/tmp")


2. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.
  - пытается прочитать файл magic в нескольких местах:
> /etc/magic.mgc, /etc/magic, /usr/share/misc/magic.mgc

  3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
> cat /dev/null > /proc/PID/fd/FD
  4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
> только идентифкатор в таблице процессов, их количество ограничено, и могут возникнуть проблеммы при создании дочерниих процессов
  5. В iovisor BCC есть утилита `opensnoop`:
      ```bash
      root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
      /usr/sbin/opensnoop-bpfcc
      ```
      На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
>sudo strace -o strc_opensnoop.txt /usr/sbin/opensnoop-bpfcc
```
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libdl.so.2", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libutil.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libm.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libexpat.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/bin/pyvenv.cfg", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/pyvenv.cfg", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/localtime", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/python3.8", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
```
  7. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
* системный вызов uname
>  Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.

  8. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
      ```bash
      root@netology1:~# test -d /tmp/some_dir; echo Hi
      Hi
      root@netology1:~# test -d /tmp/some_dir && echo Hi
      root@netology1:~#
      ```

>  Commands separated by a ; are executed sequentially; the shell waits for each        command to terminate in turn.  The return status is the exit status of the last command executed  

> command1 && command2     #  command2 is executed if, and only if, command1 returns an exit status of zero (success).

то есть разница в условии выполнения второй команды, в случае && она будет исполнена только при успешном завершении первой.
 > help set #      -e  Exit immediately if a command exits with a non-zero status.
      Есть ли смысл использовать в bash `&&`, если применить `set -e`?
в однострочном скрипте смысла нет, а в многострочных есть , поскольку условий && vj;tn ,snm несколько 

  9. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
- e - немедленный выход при ненулевом коде возврата
- u - считать ошибкой обращение к пременным без присвоенного значения
- x печать команд и аргументов, как они фактически были исполнены
- o pipefail - возвращаемое значения pipe - статус завершения последней команды, отличный от 0, или 0 если нет команд с ненулевым кодом (ошибкой)
такой режим позволяет отлаживать скрипты и очень нетерпим к неявным ошибкам  
  10. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
  - у меня Ss и  R+
  - ps ax -o stat | uniq -c
- s  is a session leader, процесс у котрого id совпадает c id сессии, в котором может быть много процессов-потомков
- \+ группа процессов исполняется в активном режиме (foreground)  


---
