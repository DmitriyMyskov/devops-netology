# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

***
## На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.
  
###
```bash
Установка и весь процесс:

dmitriy@dmitriy-lin:~$ wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
dmitriy@dmitriy-lin:~$ sudo mkdir /opt/node_exporter
dmitriy@dmitriy-lin:~$ chown -R dmitriy:dmitriy /opt/node_exporter

dmitriy@dmitriy-lin:~$ tar -xvf node_exporter-1.4.0.linux-amd64.tar.gz -C /opt/node_exporter/
node_exporter-1.4.0.linux-amd64/
node_exporter-1.4.0.linux-amd64/LICENSE
node_exporter-1.4.0.linux-amd64/NOTICE
node_exporter-1.4.0.linux-amd64/node_exporter

dmitriy@dmitriy-lin:~$ sudo nano /etc/systemd/system/node_exporter.service

dmitriy@dmitriy-lin:~$ cat /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter

[Service]
ExecStart=/opt/node_exporter/node_exporter-1.4.0.linux-amd64/node_exporter $my_options
EnvironmentFile=/etc/default/node_exporter

[Install]
WantedBy=multi-user.target

dmitriy@dmitriy-lin:~$ sudo touch /etc/default/node_exporter

dmitriy@dmitriy-lin:~$ systemctl enable node_exporter
Created symlink /etc/systemd/system/multi-user.target.wants/node_exporter.service → /etc/systemd/system/node_exporter.service.

dmitriy@dmitriy-lin:~$ systemctl start node_exporter
dmitriy@dmitriy-lin:~$ systemctl status node_exporter
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor>
     Active: active (running) since Fri 2023-02-03 17:58:10 +07; 9s ago
   Main PID: 33466 (node_exporter)
      Tasks: 6 (limit: 9442)
     Memory: 2.5M
     CGroup: /system.slice/node_exporter.service
             └─33466 /opt/node_exporter/node_exporter-1.4.0.linux-amd64/node_ex>

dmitriy@dmitriy-lin:~$ systemctl restart node_exporter
dmitriy@dmitriy-lin:~$ systemctl status node_exporter
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor>
     Active: active (running) since Fri 2023-02-03 18:00:41 +07; 3s ago
   Main PID: 33497 (node_exporter)
      Tasks: 6 (limit: 9442)
     Memory: 2.5M
     CGroup: /system.slice/node_exporter.service
             └─33497 /opt/node_exporter/node_exporter-1.4.0.linux-amd64/node_ex>

/etc/default/node_exporter - в последствии можем добавить необходимые параметры
```

***
## Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

###
```bash
dmitriy@dmitriy-lin:~$ curl localhost:9100/metrics | grep cpu
...
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 189859.57
node_cpu_seconds_total{cpu="0",mode="iowait"} 29.34
node_cpu_seconds_total{cpu="0",mode="irq"} 0
node_cpu_seconds_total{cpu="0",mode="nice"} 26.69
node_cpu_seconds_total{cpu="0",mode="softirq"} 23.9
node_cpu_seconds_total{cpu="0",mode="steal"} 1.06
node_cpu_seconds_total{cpu="0",mode="system"} 352.22
node_cpu_seconds_total{cpu="0",mode="user"} 1816.7
node_cpu_seconds_total{cpu="1",mode="idle"} 189756.62
node_cpu_seconds_total{cpu="1",mode="iowait"} 29.08
node_cpu_seconds_total{cpu="1",mode="irq"} 0
...
```
```bash
dmitriy@dmitriy-lin:~$ curl localhost:9100/metrics | grep memory
...
# TYPE node_memory_MemAvailable_bytes gauge
  node_memory_MemAvailable_bytes 5.826322432e+09
# TYPE node_memory_MemFree_bytes gauge
  node_memory_MemFree_bytes 1.85735168e+09
# TYPE node_memory_MemTotal_bytes gauge
   node_memory_MemTotal_bytes 8.335654912e+09
...
```
```bash
dmitriy@dmitriy-lin:~$ curl localhost:9100/metrics | grep network
...
# TYPE node_network_receive_bytes_total counter
node_network_receive_bytes_total{device="ens18"} 1.088103451e+09
# TYPE node_network_receive_drop_total counter
node_network_receive_drop_total{device="ens18"} 99646
# TYPE node_network_receive_errs_total counter
node_network_receive_errs_total{device="ens18"} 0
# TYPE node_network_receive_packets_total counter
node_network_receive_packets_total{device="ens18"} 6.229813e+06
# TYPE node_network_transmit_packets_total counter
node_network_transmit_packets_total{device="ens18"} 1.778946e+06
...
```
```bash
dmitriy@dmitriy-lin:~$ curl localhost:9100/metrics | grep disk
...
# TYPE node_disk_io_time_seconds_total counter
node_disk_io_time_seconds_total{device="sda"} 723.8480000000001
# TYPE node_disk_write_time_seconds_total counter
node_disk_write_time_seconds_total{device="sda"} 1217.021
# TYPE node_disk_written_bytes_total counter
node_disk_written_bytes_total{device="sda"} 1.711510016e+10
# TYPE node_disk_read_time_seconds_total counter
node_disk_read_time_seconds_total{device="sda"} 89.949
# TYPE node_disk_read_bytes_total counter
node_disk_read_bytes_total{device="sda"} 3.226595328e+09
...
```

***
## Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:

    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:
```bash
config.vm.network "forwarded_port", guest: 19999, host: 19999
```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

![DZ-3-4-netdata.png](images/DZ-3-4-netdata.png)

***
## Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

### Да, можно

```bash
dmitriy@dmitriy-lin:~$ dmesg | grep virt
[    0.074213] Booting paravirtualized kernel on KVM
[    1.729383] virtio_net virtio3 ens18: renamed from eth0
[    2.953316] systemd[1]: Detected virtualization kvm.
```

***
## Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?

###
```bash
dmitriy@dmitriy-lin:~$ sysctl -n fs.nr_open
1048576
dmitriy@dmitriy-lin:~$ ulimit -n
1024
dmitriy@dmitriy-lin:~$ cat /proc/sys/fs/file-max 
9223372036854775807
dmitriy@dmitriy-lin:~$ ulimit -Sn
1024
dmitriy@dmitriy-lin:~$ ulimit -Hn
1048576
dmitriy@dmitriy-lin:~$
```
    fs.nr_open - максимальное количество открытых файлов (файловых дескрипторов), лимит открытых файлов на пользователя не позволит достичь данного числа одним пользователем.
    cat /proc/sys/fs/file-max - максимальный предел ОС (ядра системы).
  	ulimit -Sn и ulimit -Hn - отображают soft и hard значение вышеназванного ограничения устанавливаемого на сессионном уровне.

***
## Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.

###
```bash
dmitriy@dmitriy-lin:~$ sudo -i
[sudo] пароль для dmitriy:

root@dmitriy-lin:~# unshare -f --pid --mount-proc /usr/bin/sleep 1h

root@dmitriy-lin:~# ps aux |grep sleep
root        3210  0.0  0.0  16716   580 pts/1    S    18:23   0:00 /usr/bin/sleep 1h
root        3228  0.0  0.0  17696   692 pts/1    S+   18:24   0:00 grep --color=auto sleep

root@dmitriy-lin:~# nsenter --target 3210 --pid --mount
root@dmitriy-lin:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0  16716   580 pts/1    S    18:23   0:00 /usr/bin/sleep 1h
root           2  0.4  0.0  19500  5096 pts/1    S    18:24   0:00 -bash
root          11  0.0  0.0  20160  3340 pts/1    R+   18:24   0:00 ps aux
```

***
## Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

###
```bash
vagrant@vagrant:~$ ulimit -u
7580
vagrant@vagrant:~$ :(){ :|:& };:
```
![DZ-3-4-1.png](images/DZ-3-4-1.png)
![DZ-3-4-2.png](images/DZ-3-4-2.png)

:(){ :|:& };: - Функция, которая запускает сама себя два раза, каждая запущенная в свою очередь запускают еще два экземпляра функции и т.д. 

чтобы ограничить число процессов необходимо: ulimit -u n, где n кол-во процессов.

[ 3123.191679 ] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-2.scope - механизм стабилизации.

Указав для теста unlimit -u 512 виртуалка в несознанку не ушла.

![DZ-3-4-3.png](images/DZ-3-4-3.png)