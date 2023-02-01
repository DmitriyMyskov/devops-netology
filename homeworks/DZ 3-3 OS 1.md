## Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

***
## Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`.
###
```bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ strace /bin/bash -c 'cd /tmp'
...
ioctl(2, TIOCGPGRP, [4896])             = 0
rt_sigaction(SIGCHLD, {sa_handler=0x55e7a49b5a70, sa_mask=[], sa_flags=SA_RESTORER|SA_RESTART, sa_restorer=0x7f6dfc47a090}, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=SA_RESTORER|SA_RESTART, sa_restorer=0x7f6dfc47a090}, 8) = 0
prlimit64(0, RLIMIT_NPROC, NULL, {rlim_cur=31475, rlim_max=31475}) = 0
rt_sigprocmask(SIG_BLOCK, NULL, [], 8)  = 0
getpeername(0, 0x7ffc16535d20, [16])    = -1 ENOTSOCK (Операция для сокета применена к не-сокету)
rt_sigprocmask(SIG_BLOCK, NULL, [], 8)  = 0
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")                           = 0
rt_sigprocmask(SIG_BLOCK, [CHLD], [], 8) = 0
rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
exit_group(0)                           = ?
+++ exited with 0 +++

> chdir("/tmp")
```

***
## Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
###
```bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ file /dev/tty
/dev/tty: character special (5/0)
dmitriy@dmitriy-lin:~/netology/devops-netology$ file /dev/sda
/dev/sda: block special (8/0)
dmitriy@dmitriy-lin:~/netology/devops-netology$ file /bin/bash
/bin/bash: ELF 64-bit 
```

## Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.
##
```bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ strace file /dev/tty 2>&1 | grep open
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (Нет такого файла или каталога)
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
```
### Полагаю, что база file - /usr/share/misc/magic.mgc и /etc/magic

### Пользовательские файлы:
```bash
...
stat("/home/dmitriy/.magic.mgc", 0x7ffd5be9aeb0) = -1 ENOENT (Нет такого файла или каталога)
stat("/home/dmitriy/.magic", 0x7ffd5be9aeb0) = -1 ENOENT (Нет такого файла или каталога)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
```

***
## Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
###
1. По PID процесса, который держит файл, можно посмотреть его дескрипторы.
2. В дескриптор с использованием команды echo добавить пустую строку, например: ```echo '' > /proc/641740/fd/3 ```

***
## Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
### Зомби-процессы не занимают какие-либо системные ресурсы, но не освобождают PID в таблице процессов.

***
## В iovisor BCC есть утилита `opensnoop`: На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? 
###
```bash
dmitriy@dmitriy-lin:~$ sudo /usr/sbin/opensnoop-bpfcc
PID    COMM               FD ERR PATH
787    vminfo              6   0 /var/run/utmp
583    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
583    dbus-daemon        18   0 /usr/share/dbus-1/system-services
583    dbus-daemon        -1   2 /lib/dbus-1/system-services
583    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/
1      systemd            12   0 /proc/400/cgroup
```

***
## Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
### 
Он называется так же, uname()

А получить то же самое из /proc можно так:

Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.

***
## Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?
###
Оба оператора нужны чтобы последовательно выполнить несколько команд.

";" просто выполнит все команды последовательно, даже если какая-то завершится ошибкой.

"&&" остановится, если какая-то команда в последовательности завершится ошибкой.

Судя по мануалу, скрипт с "set -e" не упадёт, если ошибкой завершится команда выполненная в конструкции с оператором "&&"

The shell does not exit if the command that fails is part of the command list immediately following a while or until keyword, part of the test following the if or elif reserved words, part of any command executed in a && or || list except the command following the final && or || ...

***
## Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
###
- errexit      same as -e - скрипт завершится, если что-то в скрипте завершится ошибкой

- nounset      same as -u - скрипт завершится, если попытаться использовать незаданную переменную

- xtrace       same as -x - включит режим дебага

- -o pipefail - выведет последнюю корректно завершившуюся команду в пайплайне

Это очень удобно для отладки скриптов.

-x детально покажет как скрипт работал,

-u покажет, если не задана какая-то переменная,

-e в связке с pipefail помогут понять на какой команде скрипт упал.

***
## Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
###
```bash
dmitriy@dmitriy-lin:~$ ps -d -o stat | sort | uniq -c
     13 I
     43 I<
      1 R
      1 R+
      1 Rl
     57 S
      6 S+
     55 Sl
     22 Sl+
      3 SLl
      2 SN
      1 STAT
```
```bash
Больше всего процессов с кодами I (треды ожидания ядра) S (Спящие прерываемые процессы)
Дополнительные обозначения:
< - Высокоприоритетные процессы
N - Низкоприоритетные процессы
L - заблокированные в памяти процессы
s - лидер сеанса
l - процесс имеющий треды
+ - процесс на переднем плане
```