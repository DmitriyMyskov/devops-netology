# Домашнее задание к занятию «3.2. Работа в терминале, лекция 2»
***
## Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.
### dmitriy@dmitriy-lin:~/netology/devops-netology$ type cd
cd — это встроенная команда bash

Комманда cd - является встроенной командой в оболучку bash и меняет текущую папку только для оболочки, в которой выполняется. Внешние команды запускаются и при запуске создают отдельный процесс. Внешний вызов, будет работать со своим окружением и менять текущий каталог внутри своего окружения.

***
## Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.
### grep <some_string> <some_file> -c

***
## Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
### dmitriy@dmitriy-lin:~/netology/devops-netology$ ps -p 1
    PID TTY          TIME CMD
      1 ?        00:00:35 systemd

### dmitriy@dmitriy-lin:~/netology/devops-netology$ pstree -p
```bash
systemd(1)─┬─ModemManager(757)─┬─{ModemManager}(789)
           │                   └─{ModemManager}(802)
           ├─NetworkManager(680)─┬─{NetworkManager}(758)
           │                     └─{NetworkManager}(759)
           ├─accounts-daemon(669)─┬─{accounts-daemon}(677)
           │                      └─{accounts-daemon}(745)
           ├─acpid(670)
           ├─apache2(841)─┬─apache2(63626)─┬─{apache2}(63632)
```
***

## Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?
### 
```bash
ИМЯ   ЛИНИЯ   ВРЕМЯ       КОММЕНТАРИЙ
dmitriy  pts/1        2023-01-31 16:47 (82.200.86.154)
dmitriy@dmitriy-lin:~$ ls /root 2>/dev/pts/2
dmitriy@dmitriy-lin:~$
```
```bash
ИМЯ   ЛИНИЯ   ВРЕМЯ       КОММЕНТАРИЙ
dmitriy  pts/1        2023-01-31 16:47 (82.200.86.154)
dmitriy  pts/2        2023-01-31 16:51 (82.200.86.154)
dmitriy@dmitriy-lin:~$ ls: невозможно открыть каталог '/root': Отказано в доступе
```
```bash
dmitriy@dmitriy-lin:~$ who -H
ИМЯ   ЛИНИЯ   ВРЕМЯ       КОММЕНТАРИЙ
dmitriy  pts/1        2023-01-31 16:47 (82.200.86.154)
dmitriy  pts/2        2023-01-31 16:51 (82.200.86.154)
dmitriy@dmitriy-lin:~$ ls /root 2>/dev/pts/1
dmitriy@dmitriy-lin:~$ ls /root 2>/dev/pts/1
dmitriy@dmitriy-lin:~$
```
```bash
dmitriy@dmitriy-lin:~$ who -H
ИМЯ   ЛИНИЯ   ВРЕМЯ       КОММЕНТАРИЙ
dmitriy  pts/1        2023-01-31 16:47 (82.200.86.154)
dmitriy  pts/2        2023-01-31 16:51 (82.200.86.154)
dmitriy@dmitriy-lin:~$ ls: невозможно открыть каталог '/root': Отказано в доступе
```

***

## Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
###
``` bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ nano test_file.txt
dmitriy@dmitriy-lin:~/netology/devops-netology$ cat test_file.txt
Hello user!!!

how are you?

user: i`m fine!
```
```bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ cat <test_file.txt >test_file.md
dmitriy@dmitriy-lin:~/netology/devops-netology$ cat test_file.md
Hello user!!!

how are you?

user: i`m fine!
```

***
## Получится ли вывести находясь в графическом режиме данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?
###
```bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ tty
/dev/pts/0
---
dmitriy@dmitriy-lin:~$ tty
/dev/pts/1
---
dmitriy@dmitriy-lin:~/netology/devops-netology$ echo "hello user" >/dev/pts/1
---
dmitriy@dmitriy-lin:~$ tty
/dev/pts/1
dmitriy@dmitriy-lin:~$ hello user
```

***
## Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?
###
```bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ bash 5>&1
dmitriy@dmitriy-lin:~/netology/devops-netology$ echo netology >/proc/$$/fd/5
netology

bash 5>&1 - создаем новый дескриптор 5 и перенаправляем его в STDOUT
echo netology > /proc/$$/fd/5 - перенаправляем результат команды в дескриптор 5

dmitriy@dmitriy-lin:~$ echo netology >/proc/$$/fd/5
-bash: /proc/77708/fd/5: Нет такого файла или каталога

Если запустить из другой сесии, получим ошибку, так как такого дескриптора нет на данный момент в текущей сесии.
```

***
## Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
###
```bash
Меняем местами STDOUT и STDERR, N>&1 1>&2 2>&N (где N - промежуточный дескриптор)
5>&2 - новый дескриптор перенаправили в stderr
2>&1 - stderr перенаправили в stdout 
1>&5 - stdout - перенаправили в в новый дескриптор
grep denied -c - скрываем отказ в доступе

dmitriy@dmitriy-lin:~/netology/devops-netology$ ls -l /root 5>&2 2>&1 1>&5 | grep denied -c
0
```
###
```bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ ls /usr
bin  games  include  lib  lib32  lib64  libexec  libx32  local  sbin  share  src
dmitriy@dmitriy-lin:~/netology/devops-netology$ ls /var
backups  cache  crash  lib  local  lock  log  mail  metrics  opt  run  snap  spool  tmp  www
dmitriy@dmitriy-lin:~/netology/devops-netology$ ls
 branching  'file from IDE.md'   has_been_moved.txt   homeworks   README.md   terraform   test_file.md   test_file.txt
dmitriy@dmitriy-lin:~/netology/devops-netology$ (ls /usr && ls /var && ls) 3>&1 1>&2 2>&3 | wc -l
bin  games  include  lib  lib32  lib64  libexec  libx32  local  sbin  share  src
backups  cache  crash  lib  local  lock  log  mail  metrics  opt  run  snap  spool  tmp  www
 branching  'file from IDE.md'   has_been_moved.txt   homeworks   README.md   terraform   test_file.md   test_file.txt
0
```

***
## Что выведет команда cat /proc/$$/environ? Как еще можно получить аналогичный по содержанию вывод?
### Выводится список переменных окружения для процесса, под которым выполняется текущая оболочка bash
Аналогичный вывод только построчно можно получить с помощью команд printenv и env

***
## Используя man, опишите что доступно по адресам /proc/<PID>/cmdline, /proc/<PID>/exe.
### /proc/<PID>/cmdline - путь до исполняемого файла процесса [PID]
/proc/<PID>/exe - сожержит полное имя выполняемого файла для процесса [PID]

***
## Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью /proc/cpuinfo.
###
```bash
dmitriy@dmitriy-lin:~/netology/devops-netology$ cat /proc/cpuinfo | grep sse
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology cpuid tsc_known_freq pni pclmulqdq vmx ssse3 cx16 pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes hypervisor lahf_lm cpuid_fault pti tpr_shadow vnmi flexpriority ept vpid tsc_adjust arat umip arch_capabilities
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology cpuid tsc_known_freq pni pclmulqdq vmx ssse3 cx16 pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes hypervisor lahf_lm cpuid_fault pti tpr_shadow vnmi flexpriority ept vpid tsc_adjust arat umip arch_capabilities
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology cpuid tsc_known_freq pni pclmulqdq vmx ssse3 cx16 pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes hypervisor lahf_lm cpuid_fault pti tpr_shadow vnmi flexpriority ept vpid tsc_adjust arat umip arch_capabilities
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology cpuid tsc_known_freq pni pclmulqdq vmx ssse3 cx16 pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes hypervisor lahf_lm cpuid_fault pti tpr_shadow vnmi flexpriority ept vpid tsc_adjust arat umip arch_capabilities

sse4_2
```

***
## При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

    ```bash
	vagrant@netology1:~$ ssh localhost 'tty'
	not a tty
    ```

	Почитайте, почему так происходит, и как изменить поведение.
###
```bash
По умолчанию при запуске команды через SSH не выделяется TTY.
Добавление ключа -t (переназначение псевдотерминала).

dmitriy@dmitriy-lin:~$ ssh localhost 'tty'
dmitriy@localhost's password:
не телетайп
dmitriy@dmitriy-lin:~$ ssh -t localhost 'tty'
dmitriy@localhost's password:
/dev/pts/2
Connection to localhost closed.
dmitriy@dmitriy-lin:~$ tty
/dev/pts/1
```

***
## Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись reptyr. Например, так можно перенести в screen процесс, который вы запустили по ошибке в обычной SSH-сессии.
###
```bash
Во втором терминале открыл файл через nano и свернул ctrl+z.
Затем выполнил jobs -l чтоб узнать идентификатор процесса.
Затем выолнил sudo reptyr -T 4052
* -T - означает забрать всю сессию, что видно из лога ниже.

dmitriy@dmitriy-lin:~$ sudo reptyr -T 4052
[sudo] пароль для dmitriy:
dmitriy@dmitriy-lin:~$ jobs -l
[1]+  4052 Остановлено (сигнал)             nano file.txt
```
***
## `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.
### Будет работать т.к. в данном случае запись в файл будет осуществляться командой tee запущенной от рута (и соответственно имеющей необходимые права).