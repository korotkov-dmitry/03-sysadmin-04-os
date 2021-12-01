# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

## 1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.

![exporte](https://user-images.githubusercontent.com/92984527/143206106-d0ef0289-7b67-49dd-af69-e5665fd070b3.png)

unit-файл:
      
      `vagrant@vagrant:~$ systemctl cat  node_exporter
      # /etc/systemd/system/node_exporter.service
      [Unit]
      Description=Prometheus Node Exporter
      Wants=network-online.target
      After=network-online.target
   
      [Service]
      User=node_exporter
      Group=node_exporter
      Type=simple
      ExecStart=/usr/local/bin/node_exporter
   
      [Install]
      WantedBy=multi-user.target`
      
Использование `systemctl`:

      `vagrant@vagrant:~$ ps -e |grep node_exporter
         621 ?        00:00:01 node_exporter
      vagrant@vagrant:~$ systemctl stop node_exporter
      ==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
      Authentication is required to stop 'node_exporter.service'.
      Authenticating as: vagrant,,, (vagrant)
      Password:
      ==== AUTHENTICATION COMPLETE ===
      vagrant@vagrant:~$ ps -e |grep node_exporter
      vagrant@vagrant:~$ systemctl start node_exporter
      ==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
      Authentication is required to start 'node_exporter.service'.
      Authenticating as: vagrant,,, (vagrant)
      Password:
      ==== AUTHENTICATION COMPLETE ===
      vagrant@vagrant:~$ ps -e |grep node_exporter
         1839 ?        00:00:00 node_exporter
      vagrant@vagrant:~$`
      
   Автостарт:
   ![exporte2](https://user-images.githubusercontent.com/92984527/143207803-2d1e2094-58f0-425f-b403-b08d3dc835de.png)   
   
**Дополнение**
   
   Установка параметров:
   
      `vagrant@vagrant:/$ sudo vim /opt/myservice
      [Service]
      MY_OPTIONS='--itsmy'`
            
      `vagrant@vagrant:/$ sudo vim /etc/systemd/system/node_exporter.service
      [Unit]
      Description=Prometheus Node Exporter
      Wants=network-online.target
      After=network-online.target

      [Service]
      EnvironmentFile=/opt/myservice
      User=node_exporter
      Group=node_exporter
      Type=simple
      ExecStart=/usr/local/bin/node_exporter

      [Install]
      WantedBy=multi-user.target`
      
Запуск: 

      `vagrant@vagrant:/$ sudo systemctl daemon-reload
      vagrant@vagrant:/$ sudo systemctl restart node_exporter
      vagrant@vagrant:/$ sudo systemctl status node_exporter
      ● node_exporter.service - Prometheus Node Exporter
      Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
      Active: active (running) since Tue 2021-11-30 03:42:12 UTC; 3s ago
      Main PID: 14973 (node_exporter)
      Tasks: 4 (limit: 1071)
      Memory: 2.2M
      CGroup: /system.slice/node_exporter.service
             └─14973 /usr/local/bin/node_exporter
      ...
      vagrant@vagrant:/$ sudo cat /proc/14973/environ
      LANG=en_US.UTF-8LANGUAGE=en_US:PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/binHOME=/home/node_exporterLOGNAME=node_exporterUSER=node_exporterINVOCATION_ID=8ec0c2a2989341549c59f700eed04924JOURNAL_STREAM=9:62130MY_OPTIONS=--itsmy`
      
**Дополнение 2**
      
Файл конфигурации:

      `vagrant@vagrant:~$ sudo vim /etc/systemd/system/node_exporter.service
      [Unit]
      Description=Prometheus Node Exporter
      Wants=network-online.target
      After=network-online.target

      [Service]
      EnvironmentFile=/opt/myservice
      User=node_exporter
      Group=node_exporter
      Type=simple
      ExecStart=/usr/local/bin/node_exporter $OPTIONS

      [Install]
      WantedBy=multi-user.target`

Определение параметров:
      
      `vagrant@vagrant:~$ sudo vim /opt/myservice
      [Service]
      MY_OPTIONS='--itsmy'
      OPTIONS='--log.level=error'`

Перезапуск:
      
      `vagrant@vagrant:~$ sudo systemctl daemon-reload
      vagrant@vagrant:~$ sudo systemctl restart node_exporter
      vagrant@vagrant:~$ sudo systemctl status node_exporter
      ● node_exporter.service - Prometheus Node Exporter
           Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; ve>
           Active: active (running) since Wed 2021-12-01 11:18:24 UTC; 4s ago
         Main PID: 15389 (node_exporter)
            Tasks: 4 (limit: 1071)
           Memory: 2.2M
           CGroup: /system.slice/node_exporter.service
                   └─15389 /usr/local/bin/node_exporter --log.level=error

      Dec 01 11:18:24 vagrant systemd[1]: Started Prometheus Node Exporter.
      lines 1-10/10 (END)`
## 2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

CPU

      `node_cpu_seconds_total{cpu="0",mode="idle"} 5026.45
      node_cpu_seconds_total{cpu="0",mode="system"} 8.13
      node_cpu_seconds_total{cpu="0",mode="user"} 4.84
      node_cpu_seconds_total{cpu="1",mode="idle"} 5025.01
      node_cpu_seconds_total{cpu="1",mode="system"} 13.46
      node_cpu_seconds_total{cpu="1",mode="user"} 1.61
      process_cpu_seconds_total 0.58`
      
Memory

      `node_memory_MemTotal_bytes 1.028694016e+09
      node_memory_MemFree_bytes 5.68066048e+08
      node_memory_MemAvailable_bytes 7.6163072e+08
      node_memory_Buffers_bytes 2.3498752e+07
      node_memory_Cached_bytes 2.90205696e+08`
      
Disk

      `node_disk_io_time_seconds_total{device="sda"} 12.012
      node_disk_read_bytes_total{device="sda"} 3.16904448e+08
      node_disk_read_time_seconds_total{device="sda"} 7.303
      node_disk_written_bytes_total{device="sda"} 7.4236928e+07
      node_disk_write_time_seconds_total{device="sda"} 7.746`
      
Network

      `node_network_receive_bytes_total{device="eth0"} 545686
      node_network_receive_errs_total{device="eth0"} 0
      node_network_transmit_bytes_total{device="eth0"} 434942
      node_network_transmit_errs_total{device="eth0"} 0`

## 3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999```


    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

Результат

      `vagrant@vagrant:~$ sudo ss -pnltu | grep 19999
      tcp    LISTEN   0        4096              0.0.0.0:19999          0.0.0.0:*      users:(("netdata",pid=781,fd=4))                                       
      vagrant@vagrant:~$ sudo lsof -i :19999
      COMMAND PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
      netdata 781 netdata    4u  IPv4  24689      0t0  TCP *:19999 (LISTEN)
      netdata 781 netdata   52u  IPv4  30887      0t0  TCP vagrant:19999->_gateway:56143 (ESTABLISHED)
      vagrant@vagrant:~$`
      
![netdata](https://user-images.githubusercontent.com/92984527/143730020-4805e999-0c8d-4d5f-840b-ab93b24cdb59.png)

## 4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
      `vagrant@vagrant:~$ dmesg |grep virtual
      [    0.005445] CPU MTRRs all blank - virtualized system.
      [    0.044650] Booting paravirtualized kernel on KVM
      [ 4609.048932] systemd[1]: Detected virtualization oracle.`
      
`KVM (Kernel-based Virtual Machine)` - программное решение, обеспечивающее виртуализацию в среде Linux.

`Detected virtualization oracle` - определение системы виртуализации.
## 5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
`fs.nr_open` - лимит на количество открытых дескрипторов
      
`vagrant@vagrant:~$ sysctl -n fs.nr_open
      1048576`
      
`vagrant@vagrant:~$ ulimit -n` - максимальное количество открытых файловых дескрипторов (большинство систем не позволяет устанавливать это значение)

`vagrant@vagrant:~$ ulimit -Hn` - жесткое ограничение после установки превосходить нельзя;
      `vagrant@vagrant:~$ ulimit -Sn` - мягкое ограничение можно превосходить вплоть до значения соответствующего жесткого ограничения.

      `vagrant@vagrant:~$ ulimit -Hn
      1048576
      vagrant@vagrant:~$ ulimit -Sn
      1024`
      
## 6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
`pts/1`
      
      `vagrant@vagrant:~$ sudo unshare -f --pid --mount-proc /bin/bash
      root@vagrant:/home/vagrant# ps aux
      USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
      root           1  0.0  0.3   9836  3912 pts/1    S    05:32   0:00 /bin/bash
      root           8  0.0  0.3  11492  3436 pts/1    R+   05:33   0:00 ps aux
      root@vagrant:/home/vagrant# sleep 1h`
      
`pts/0`
      
      `vagrant@vagrant:~$ ps aux
      ...
      root        1696  0.0  0.4   9836  4020 pts/1    S    05:32   0:00 /bin/bash
      root        1704  0.0  0.0   8076   596 pts/1    S+   05:33   0:00 sleep 1h
      vagrant     1727  0.0  0.3  11492  3320 pts/0    R+   05:35   0:00 ps aux
      vagrant@vagrant:~$ sudo nsenter --target 1704 --pid --mount
      root@vagrant:/# ps aux
      USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
      root           1  0.0  0.4   9836  4020 pts/1    S    05:32   0:00 /bin/bash
      root           9  0.0  0.0   8076   596 pts/1    S+   05:33   0:00 sleep 1h
      root          21  0.0  0.4   9836  4116 pts/0    S    05:35   0:00 -bash
      root          30  0.0  0.3  11492  3512 pts/0    R+   05:35   0:00 ps aux
      root@vagrant:/#`
## 7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?
      `:()
      {
          :|:&
      };
      :`

Определяет функцию с именем `:`, которая запускает саму себя дважды и каждый процесс запускает еще два.

![image](https://user-images.githubusercontent.com/92984527/143731170-950970f6-473d-4ead-899c-aaf7f6c7e175.png)

      `vagrant@vagrant:~$ dmesg -T
      ...
      [Sun Nov 28 05:40:53 2021] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-3.scope
      [Sun Nov 28 05:42:09 2021] rcu: INFO: rcu_sched self-detected stall on CPU
      [Sun Nov 28 05:42:09 2021] rcu:         0-...!: (1 GPs behind) idle=03e/1/0x4000000000000002 softirq=60984/60986 fqs=0
      [Sun Nov 28 05:42:09 2021]      (t=18515 jiffies g=91213 q=177)
      [Sun Nov 28 05:42:09 2021] rcu: rcu_sched kthread starved for 18515 jiffies! g91213 f0x0 RCU_GP_WAIT_FQS(5) ->state=0x0 ->cpu=1
      [Sun Nov 28 05:42:09 2021] rcu: RCU grace-period kthread stack dump:
      [Sun Nov 28 05:42:09 2021] rcu_sched       R  running task        0    10      2 0x80004000
      ...`
      
      Настройка числа процессов:
      `vagrant@vagrant:~$ ulimit -u
      3571
      vagrant@vagrant:~$ ulimit -u 50`
      
