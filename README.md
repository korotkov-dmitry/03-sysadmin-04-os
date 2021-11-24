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
## 2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.
## 3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

## 4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
## 5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
## 6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
## 7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?
