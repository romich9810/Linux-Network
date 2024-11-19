# Сети в Linux

Настройка сетей в Linux на виртуальных машинах.


## Contents

1. [Chapter I](#chapter-i)
2. [Chapter II](#chapter-ii) \
   2.1. [Стек протоколов TCP IP](#стек-протоколов-tcp-ip) \
   2.2. [Адресация](#адресация) \
   2.3. [Маршрутизация](#маршрутизация)
3. [Chapter III](#chapter-iii) \
   3.1. [Инструмент ipcalc](#part-1-инструмент-ipcalc) \
   3.2. [Статическая маршрутизация между двумя машинами](#part-2-статическая-маршрутизация-между-двумя-машинами) \
   3.3. [Утилита iperf3](#part-3-утилита-iperf3) \
   3.4. [Сетевой экран](#part-4-сетевой-экран) \
   3.5. [Статическая маршрутизация сети](#part-5-статическая-маршрутизация-сети) \
   3.6. [Динамическая настройка IP с помощью DHCP](#part-6-динамическая-настройка-ip-с-помощью-dhcp) \
   3.7. [NAT](#part-7-nat) \
   3.8. [Допополнительно. Знакомство с SSH Tunnels](#part-8-дополнительно-знакомство-с-ssh-tunnels)



## Chapter I



## Chapter II

### Стек протоколов TCP IP

Собственно, что есть сеть? Сеть - это более двух компьютеров, объединенных между собой какими-то каналами связи, в более сложном примере - каким-то сетевым
оборудованием и обменивающиеся между собой информацией по определенным правилам. Эти правила «диктуются» стеком протоколов **TCP/IP**.

Transmission Control Protocol/Internet Protocol (Стек протоколов **TCP/IP**) - если говорить простым языком, это набор взаимодействующих протоколов разных уровней (каждый уровень взаимодействует с соседним, то есть состыковывается, поэтому и стек), согласно которым происходит обмен данными в сети. 
Итого, стек протоколов **TCP/IP** - это набор наборов правил :) Тут может возникнуть 
резонный вопрос: а зачем же иметь много протоколов? Неужели нельзя обмениваться всем по одному протоколу?

Все дело в том, что каждый протокол описывает строго отведенные ему правила. Кроме того, протоколы разделены по уровням функциональности, что позволяет работе 
сетевого оборудования и программного обеспечения становиться гораздо проще, прозрачнее и выполнять «свой» круг задач. Для разделения данного набора протоколов 
по уровням была разработана модель сетевого взаимодействия **OSI** (англ. Open Systems Interconnection Basic Reference Model, 1978 года, она же - базовая эталонная 
модель взаимодействия открытых систем). Модель **OSI** состоит из семи различных уровней. Уровень отвечает за отдельный участок в работе коммуникационных систем, 
не зависит от рядом стоящих уровней – он только предоставляет определённые услуги. Каждый уровень выполняет свою задачу в соответствии с набором правил, называемым протоколом.

### Адресация

В сети, построенной на стеке протоколов **TCP/IP**, каждому хосту (компьютеру или устройству подключенному к сети) присвоен IP-адрес. 
IP-адрес представляет собой 32-битовое двоичное число. 
Удобной формой записи IP-адреса (**IPv4**) является запись в виде четырёх десятичных чисел (от 0 до 255), разделённых точками, например, *192.168.0.1*. 
В общем случае, IP-адрес делится на две части: адрес сети (подсети) и адрес хоста:


Как видно из иллюстрации, есть такое понятие, как сеть и подсеть. 
Думаю, из значений слов понятно, что IP адреса делятся на сети, а сети, в свою очередь, делятся на подсети с помощью маски подсети (корректнее будет сказать: адрес хоста может быть разбит на подсети).

Кроме адреса хоста в сети **TCP/IP** есть такое понятие, как порт. Порт является числовой характеристикой какого-то системного ресурса. 
Порт выделяется приложению, выполняемому на некотором сетевом хосте, для связи с приложениями, выполняемыми на других сетевых хостах (в том числе с другими приложениями на этом же хосте). С программной точки зрения, порт - это область памяти, которая контролируется каким-либо сервисом.

IP протокол находится ниже **TCP** и **UDP** в иерархии протоколов и отвечает за передачу и маршрутизацию информации в сети. 
Для этого протокол IP заключает каждый блок информации (пакет **TCP** или **UDP**) в другой пакет - IP пакет или дейтаграмма IP, который хранит заголовок об источнике, получателе и маршруте.

Если провести аналогию с реальным миром, то сеть **TCP/IP** - это город. Названия улиц и проулков - это сети и подсети. Номера строений - это адреса хостов. 
В строениях, номера кабинетов/квартир - это порты. Точнее, порты - это почтовые ящики, в которые ожидают прихода корреспонденции получатели (службы). 
Соответственно, номера портов кабинетов 1,2 и т.п. обычно отдаются директорам и руководителям, как привилегированным, а рядовым сотрудникам достаются номера кабинетов с большими цифрами. При отправке и доставке корреспонденции, информация упаковывается в конверты (ip-пакеты), на которых указывается адрес отправителя (ip и порт) и адрес получателя (ip и порт).

Следует отметить, что протокол IP не имеет представления о портах, за интерпретацию портов отвечает **TCP** и **UDP**, по аналогии **TCP** и **UDP** не обрабатывают IP-адреса.

### Маршрутизация

![network_route](misc/images/network_route.png)

Может возникнуть вопрос, а как же один компьютер соединится с другим? 
Откуда он знает, куда посылать пакеты?

Для разрешения этого вопроса сети между собой соединены шлюзами (маршрутизаторами). 
Шлюз - это тот же хост, но имеющий соединение с двумя и более сетями, который может передавать информацию между сетями и направлять пакеты в другую сеть. 
На рисунке роль шлюза выполняет pineapple и papaya, имеющих по 2 интерфейса, подключенные к разным сетям.

Чтобы определить маршрут передачи пакетов, IP использует сетевую часть адреса (маску подсети). 
Для определения маршрута, на каждой машине в сети имеется таблица маршрутизации (routing table), которая хранит список сетей и шлюзов для этих сетей. 
IP «просматривает» сетевую часть адреса назначения в проходящем пакете и, если для этой сети есть запись в таблице маршрутизации, то пакет отправляется на соответствующий шлюз.

В Linux ядро операционной системы хранит таблицу маршрутизации в файле */proc/net/route*. 
Просмотреть текущую таблицу маршрутизации можно командой `netstat -rn` (r - routing table, n - не преобразовывать IP в имена), `route` или `ip r`.

Пример таблицы маршрутизации для хоста eggplant:
```
[root@eggplant ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
128.17.75.0      128.17.75.20   255.255.255.0   UN        1500 0          0 eth0
default          128.17.75.98   0.0.0.0         UGN       1500 0          0 eth0
127.0.0.1        127.0.0.1      255.0.0.0       UH        3584 0          0 lo
128.17.75.20     127.0.0.1      255.255.255.0   UH        3584 0          0 lo
```

Значения колонок:
- Destination - адреса сетей (хостов) назначения. При этом, при указании сети, адрес обычно заканчивается на ноль;
- Gateway - адрес шлюза для указанного в первой колонке хоста/сети. Третья колонка - маска подсети, для которой работает данный маршрут;
- Flags - информация об адресе назначения (U - маршрут работает, N - маршрут для сети, H - маршрут для хоста и т.п.);
- MSS - число байтов, которое может быть отправлено за 1 раз; 
- Window - количество фреймов, которое может быть отправлено до получения подтверждения;
- irtt - статистика использования маршрута;
- Iface - указывает сетевой интерфейс, используемый для маршрута (eth0, eth1 и т.п.).

\> *Как и в прошлый раз, ещё больше полезной информации ты сохраняешь в папке materials.*


## Chapter III

В качестве результата работы ты должен представить отчет по выполненным задачам. В каждой части задания указано, что должно быть помещено в отчёт после её выполнения. Это могут быть ответы на вопросы, скриншоты и т.д.
- В репозиторий, в папку src, должен быть загружен отчёт с расширением .md.
- В отчёте должны быть выделены все части задания, как заголовки 2-го уровня.
- В рамках одной части задания всё, что помещается в отчёт, должно быть оформлено в виде списка.
- Каждый скриншот в отчёте должен быть кратко подписан (что показано на скриншоте).
- Все скриншоты обрезаны так, чтобы была видна только нужная часть экрана.
- На одном скриншоте допускается отображение сразу нескольких пунктов задания, но они все должны быть описаны в подписи к скриншоту.
- На все виртуальные машины, созданные в процессе выполнения задания, устанавливай **Ubuntu 20.04 Server LTS**.

## Part 1. Инструмент **ipcalc**
`-` Итак, начнём наше погружение в удивительный мир сетей со знакомства с IP адресами. А использовать для этого мы будем инструмент **ipcalc**.


##### Подними виртуальную машину (далее -- ws1)

#### 1.1. Сети и маски
##### Определи и запиши в отчёт:
##### 1) Адрес сети *192.167.38.54/13*\
![1-1](Screenshoots/1-1.png)
##### 2) Перевод маски *255.255.255.0* в префиксную и двоичную запись, */15* в обычную и двоичную, *11111111.11111111.11111111.11110000* в обычную и префиксную\
- /24 и 11111111.11111111.11111111.0
- 255.254.0.0 и 11111111.11111110.0.0
- 255.255.255.240 и /28
##### 3) Минимальный и максимальный хост в сети *12.167.38.4* при масках: */8*, *11111111.11111111.00000000.00000000*, *255.255.254.0* и */4*\
- /8: 12.0.0.1 min, 12.255.255.254 max
- /16: 12.167.0.1 min, 12.167.255.254 max
- /23: 12.167.38.1 min, 12.167.39.254 max
- /4: 0.0.0.1 min, 15.255.255.254 max

#### 1.2. localhost
##### Определи и запиши в отчёт, можно ли обратиться к приложению, работающему на localhost, со следующими IP: *194.34.23.100*, *127.0.0.2*, *127.1.0.1*, *128.0.0.1*
Существуют стандарты диапазонов по использованию частных сетей:\
![1-3](Screenshoots/1-3.png)
- Cоответственно для обращения внутри хоста:
    - 194.34.23.100 нет
    - 127.0.0.2 да
    - 127.1.0.1 - да
    - 128.0.0.1 - нет
#### 1.3. Диапазоны и сегменты сетей
##### Определи и запиши в отчёт:
##### 1) Какие из перечисленных IP можно использовать в качестве публичного, а какие только в качестве частных: *10.0.0.45*, *134.43.0.2*, *192.168.4.2*, *172.20.250.4*, *172.0.2.1*, *192.172.0.1*, *172.68.0.2*, *172.16.255.255*, *10.10.10.10*, *192.169.168.1*
- 10.0.0.45 - частная
- 134.43.0.2 - публичная
- 192.168.4.2 - частная
- 172.20.250.4 - частная
- 172.0.2.1 - публичная
- 192.172.0.1 - публичная
- 172.68.0.2 - публичная
- 172.16.255.255 - частная
- 10.10.10.10 - частная
- 192.169.168.1 - публичная
##### 2) Какие из перечисленных IP адресов шлюза возможны у сети *10.10.0.0/18*: *10.0.0.1*, *10.10.0.2*, *10.10.10.10*, *10.10.100.1*, *10.10.1.255*
- 10.0.0.1 - нет
- 10.10.0.2 - да
- 10.10.10.10 - да
- 10.10.100.1 нет
- 10.10.1.255 - да

## Part 2. Статическая маршрутизация между двумя машинами

##### Подними две виртуальные машины (далее -- ws1 и ws2).
##### С помощью команды `ip a` посмотри существующие сетевые интерфейсы.
- В отчёт помести скрин с вызовом и выводом использованной команды.

- Интерфейсы ws1
![2-1](Screenshoots/2-1.png)

- Интерфейсы ws2
![2-2](Screenshoots/2-2.png)

##### Опиши сетевой интерфейс, соответствующий внутренней сети, на обеих машинах и задать следующие адреса и маски: ws1 - *192.168.100.10*, маска */16*, ws2 - *172.24.116.8*, маска */12*.
- В отчёт помести скрины с содержанием изменённого файла *etc/netplan/00-installer-config.yaml* для каждой машины.

![2-3-1](Screenshoots/2-3-1.png)
![2-3-2](Screenshoots/2-3-2.png)
##### Выполни команду `netplan apply` для перезапуска сервиса сети.

#### 2.1. Добавление статического маршрута вручную
##### Добавь статический маршрут от одной машины до другой и обратно при помощи команды вида `ip r add`.
##### Пропингуй соединение между машинами.
- В отчёт помести скрин с вызовом и выводом использованных команд.

![2-4-1](Screenshoots/2-4-1.png)
![2-4-2](Screenshoots/2-4-2.png)

> Пинговаться машины будут только при совместном добавлении статической маршрутизации на машины по адрессам. Использовал тип соединения для сетевого адаптера на машинах "Сетевой мост" в VirtualBox.

#### 2.2. Добавление статического маршрута с сохранением
##### Перезапусти машины.
##### Добавь статический маршрут от одной машины до другой с помощью файла *etc/netplan/00-installer-config.yaml*.
- В отчёт помести скрин с содержанием изменённого файла *etc/netplan/00-installer-config.yaml*.

![2-5-1](Screenshoots/2-5-1.png)
![2-5-2](Screenshoots/2-5-2.png)


##### Пропингуй соединение между машинами.
- В отчёт помести скрин с вызовом и выводом использованной команды.

![2-6-1](Screenshoots/2-6-1.png)
![2-6-2](Screenshoots/2-6-2.png)

## Part 3. Утилита **iperf3**

#### 3.1. Скорость соединения
##### Переведи и запиши в отчёт: 8 Mbps в MB/s, 100 MB/s в Kbps, 1 Gbps в Mbps.
- 8 Mbps = 1 MB/s
- 100 MB/s = 819200 Kbps
- 1 Gbps = 1024 Mbps
> Также для наглядности перевода можно использовать команду virsh

#### 3.2. Утилита **iperf3**
##### Измерь скорость соединения между ws1 и ws2.
- В отчёт помести скрины с вызовом и выводом использованных команд.

![3-1](Screenshoots/3-1.png)
![3-2](Screenshoots/3-2.png)

## Part 4. Сетевой экран

`-` После соединения машин перед нами стоит следующая задача: контролировать информацию, проходящую по соединению. Для этого используются сетевые экраны.

**== Задание ==**

*В данном задании используются виртуальные машины ws1 и ws2 из Части 2*

#### 4.1. Утилита **iptables**
##### Создай файл */etc/firewall.sh*, имитирующий фаерволл, на ws1 и ws2:
```shell
#!/bin/sh

# Удаление всех правил в таблице «filter» (по-умолчанию).
iptables -F 
iptables -X
```
##### Нужно добавить в файл подряд следующие правила:
##### 1) На ws1 примени стратегию, когда в начале пишется запрещающее правило, а в конце пишется разрешающее правило (это касается пунктов 4 и 5).
##### 2) На ws2 примени стратегию, когда в начале пишется разрешающее правило, а в конце пишется запрещающее правило (это касается пунктов 4 и 5).
##### 3) Открой на машинах доступ для порта 22 (ssh) и порта 80 (http).
##### 4) Запрети *echo reply* (машина не должна «пинговаться», т.е. должна быть блокировка на OUTPUT).
##### 5) Разреши *echo reply* (машина должна «пинговаться»).
##### Запусти файлы на обеих машинах командами `chmod +x /etc/firewall.sh` и `/etc/firewall.sh`.
- В отчёте опиши разницу между стратегиями, применёнными в первом и втором файлах.
- В отчёт помести скрины с содержанием файла */etc/firewall* для каждой машины.

**== Решение ==**
- WS1
![4-1-1](Screenshoots/4-1-1.png)
- WS2
![4-1-2](Screenshoots/4-1-2.png)
> *iptables* - утиллита для формирования правил обработки пакетов, в которой существует несколько цепочек обработок: INPUT, OUTPUT, FORWARD(проходящие пакеты). Немного пояснений по синтаксису команд:
> - -A - добавляет правило в конец таблицы цепочки фаервола
> - -i/-o <название интерфейса в системе>- флаг интерфейса на прием, отдачу, через который будут идти пакеты
> - -p <название протокола> --<название протокола/службы> - флаг направленный на работу с пакетами от конкретной службы/порта
> - -j <ACCEPT/DROP> - флаг джампа(действия) и название самого действия.

- Команды в таблице правил фаервола исполняются сверху-вниз, поэтому мы можем пропинговать WS1 с WS2, но не наоборот: срабатывает DROP пакетов.

#### 4.2. Утилита **nmap**
##### Командой **ping** найди машину, которая не «пингуется», после чего утилитой **nmap** покажи, что хост машины запущен.
*Проверка: в выводе nmap должно быть сказано: `Host is up`*.
- В отчёт помести скрины с вызовом и выводом использованных команд **ping** и **nmap**.

- WS1
![4-2-1](Screenshoots/4-2-1.png)
- WS2
![4-2-2](Screenshoots/4-2-2.png)


##### Сохрани дампы образов виртуальных машин
**P.S. Ни в коем случае не сохраняй дампы в гит!**


## Part 5. Статическая маршрутизация сети

`-` Пока что мы соединяли всего две машины, но теперь пришло время для статической маршрутизации целой сети.

**== Задание ==**

Сеть: \
![route](Screenshoots/route.png)

##### Подними пять виртуальных машин (3 рабочие станции (ws11, ws21, ws22) и 2 роутера (r1, r2)).

#### 5.1. Настройка адресов машин
##### Настрой конфигурации машин в *etc/netplan/00-installer-config.yaml* согласно сети на рисунке.
- В отчёт помести скрины с содержанием файла *etc/netplan/00-installer-config.yaml* для каждой машины.

- WS21
![5-1-1](Screenshoots/5-1-1.png)

- WS22
![5-1-2](Screenshoots/5-1-2.png)

- R2
![5-1-3](Screenshoots/5-1-3.png)

- WS11
![5-1-4](Screenshoots/5-1-4.png)

- R1
![5-1-5](Screenshoots/5-1-5.png)

##### Перезапусти сервис сети. Если ошибок нет, то командой `ip -4 a` проверь, что адрес машины задан верно. Также пропингуй ws22 с ws21. Аналогично пропингуй r1 с ws11.

- R1 с WS11
![5-2-1](Screenshoots/5-2-1.png)

- W22 с W21
![5-2-2](Screenshoots/5-2-2.png)

#### 5.2. Включение переадресации IP-адресов
##### Для включения переадресации IP, выполни команду на роутерах:
`sysctl -w net.ipv4.ip_forward=1`
> Флаг *-w* указывает на то, что мы хотим поменять системный параметр(в нашем случае - параметр переадресации) при помощи утилиты sysctl

*При таком подходе переадресация не будет работать после перезагрузки системы.*
- В отчёт помести скрин с вызовом и выводом использованной команды.

- R1
![5-3-1](Screenshoots/5-3-1.png)

- R2
![5-3-2](Screenshoots/5-3-2.png)

##### Открой файл */etc/sysctl.conf* и добавь в него следующую строку:
`net.ipv4.ip_forward = 1`
*При использовании этого подхода, IP-переадресация включена на постоянной основе.*
- В отчёт помести скрин с содержанием изменённого файла */etc/sysctl.conf*.

-R1
![5-3-3](Screenshoots/5-3-3.png)

-R2
![5-3-4](Screenshoots/5-3-4.png)

#### 5.3. Установка маршрута по-умолчанию
Пример вывода команды `ip r` после добавления шлюза:
```
default via 10.10.0.1 dev eth0
10.10.0.0/18 dev eth0 proto kernel scope link src 10.10.0.2
```
##### Настрой маршрут по-умолчанию (шлюз) для рабочих станций. Для этого добавь `default` перед IP роутера в файле конфигураций.
- В отчёт помести скрин с содержанием файла *etc/netplan/00-installer-config.yaml*;

- WS11 с маршрутом по-умолчанию
![5-4-1](Screenshoots/5-4-1.png)

- WS21 с маршрутом по-умолчанию
![5-4-2](Screenshoots/5-4-2.png)

- WS22 с маршрутом по-умолчанию
![5-4-3](Screenshoots/5-4-3.png)

> Маршрут по умолчанию - это путь к шлюзу. Шлюз - это выход во внешние сети данной подсети

##### Вызови `ip r` и покажи, что добавился маршрут в таблицу маршрутизации.
- В отчёт помести скрин с вызовом и выводом использованной команды.

![5-4-4-1](Screenshoots/5-4-4-1.png)

##### Пропингуй с ws11 роутер r2 и покажи на r2, что пинг доходит. Для этого используй команду:
`tcpdump -tn -i eth0`
- В отчёт помести скрин с вызовом и выводом использованных команд.
> tcpdump - утилита для контроля трафика пакетов. В нашем случае - на определенном интерфейсе.

- WS11 пингуется c R2
![5-4-4](Screenshoots/5-4-4.png)

- пакеты пересылаются с R2 в ответ
![5-4-5](Screenshoots/5-4-5.png)

#### 5.4. Добавление статических маршрутов
##### Добавь в роутеры r1 и r2 статические маршруты в файле конфигураций. Пример для r1 маршрута в сетку 10.20.0.0/26:
```shell
# Добавь в конец описания сетевого интерфейса eth1:
- to: 10.20.0.0
  via: 10.100.0.12
```
- В отчёт помести скрины с содержанием изменённого файла *etc/netplan/00-installer-config.yaml* для каждого роутера.

**== Решение ==**

- R1
![5-5-1](Screenshoots/5-5-1.png)

- R2
![5-5-2](Screenshoots/5-5-2.png)

##### Вызови `ip r` и покажи таблицы с маршрутами на обоих роутерах. Пример таблицы на r1:
```
10.100.0.0/16 dev eth1 proto kernel scope link src 10.100.0.11
10.20.0.0/26 via 10.100.0.12 dev eth1
10.10.0.0/18 dev eth0 proto kernel scope link src 10.10.0.1
```
- В отчёт помести скрин с вызовом и выводом использованной команды.

**== Решение ==**
- R1
![5-5-3](Screenshoots/5-5-3.png)

- R2
![5-5-4](Screenshoots/5-5-4.png)

##### Запусти команды на ws11:
`ip r list 10.10.0.0/[маска сети]` и `ip r list 0.0.0.0/0`
- В отчёт помести скрин с вызовом и выводом использованных команд;
- В отчёте объясни, почему для адреса 10.10.0.0/\[маска сети\] был выбран маршрут, отличный от 0.0.0.0/0, хотя он попадает под маршрут по-умолчанию.

**== Решение ==**

- WS1
![5-5-5](Screenshoots/5-5-5.png)
> При наличии нескольких маршрутов одинаковой длины, используется более точный. Так, 10.10.0.0/18 задан точнее (через 10.0.0.2), чем 0.0.0.0/0.

#### 5.5. Построение списка маршрутизаторов
Пример вывода утилиты **traceroute** после добавления шлюза:
```
1 10.10.0.1 0 ms 1 ms 0 ms
2 10.100.0.12 1 ms 0 ms 1 ms
3 10.20.0.10 12 ms 1 ms 3 ms
```
##### Запусти на r1 команду дампа:
`tcpdump -tnv -i eth0`
##### При помощи утилиты **traceroute** построй список маршрутизаторов на пути от ws11 до ws21.
- В отчёт помести скрины с вызовом и выводом использованных команд (tcpdump и traceroute);

**== Решение ==**

- WS11
![5-6-1](Screenshoots/5-6-1.png)

- R1
![5-6-2](Screenshoots/5-6-2.png)

- В отчёте, опираясь на вывод, полученный из дампа на r1, объясни принцип работы построения пути при помощи **traceroute**.

>traceroute использует прыжки (хопы) для определения трассировки пути. Утилита отправляет 3 UDP-пакета с разным time-to-live (ttl), которые двигаются между маршрутизаторами. При достижении конечной точки, он отправляет ответ запросившему адресу, что по порту 34434 (порт по умолчанию. в большинстве случаев, он не используется) ничего не подключено. Благодаря использованию протокола UDP, в каждом пакете есть порт отправителя и порт получателя.

#### 5.6. Использование протокола **ICMP** при маршрутизации
##### Запусти на r1 перехват сетевого трафика, проходящего через eth0 с помощью команды:
`tcpdump -n -i eth0 icmp`
##### Пропингуй с ws11 несуществующий IP (например, *10.30.0.111*) с помощью команды:
`ping -c 1 10.30.0.111`
- В отчёт помести скрин с вызовом и выводом использованных команд.

- WS11
![5-6-3](Screenshoots/5-6-3.png)

-R1
![5-6-4](Screenshoots/5-6-4.png)

##### Сохрани дампы образов виртуальных машин.
**P.S. Ни в коем случае не сохраняй дампы в гит!**

## Part 6. Динамическая настройка IP с помощью **DHCP**
**== Задание ==**

*В данном задании используются виртуальные машины из Части 5.*

##### Для r2 настрой в файле */etc/dhcp/dhcpd.conf* конфигурацию службы **DHCP**:
##### 1) Укажи адрес маршрутизатора по-умолчанию, DNS-сервер и адрес внутренней сети. Пример файла для r2:
```shell
subnet 10.100.0.0 netmask 255.255.0.0 {}

subnet 10.20.0.0 netmask 255.255.255.192
{
    range 10.20.0.2 10.20.0.50;
    option routers 10.20.0.1;
    option domain-name-servers 10.20.0.1;
}
```
##### 2) В файле *resolv.conf* пропиши `nameserver 8.8.8.8`.
- В отчёт помести скрины с содержанием изменённых файлов.
##### Перезагрузи службу **DHCP** командой `systemctl restart isc-dhcp-server`. Машину ws21 перезагрузи при помощи `reboot` и через `ip a` покажи, что она получила адрес. Также пропингуй ws22 с ws21.
- В отчёт помести скрины с вызовом и выводом использованных команд.

**== Решение ==**

- Для вступление в силу файла dhcpd.conf, необходимо указать интерфейс и путь до этого файла в основном конфиге
![6-1-5](Screenshoots/6-1-5.png)

- Затем включаем динамическую типизацию на WS21. 
![6-1-3](Screenshoots/6-1-3.png)

Чтобы понять, что адрес арендован, заходим в файл /var/lib/dhcp/dhcp.leases и видим наименование хостов наших машин:
![6-1-6](Screenshoots/6-1-6.png)

- На R2 внесли конфиг DHCP и поменяли nameserver
![6-1-1](Screenshoots/6-1-1.png)
![6-1-2](Screenshoots/6-1-2.png)

- Пингуем WS22 c WS21

![6-1-4](Screenshoots/6-1-4.png)

##### Укажи MAC адрес у ws11, для этого в *etc/netplan/00-installer-config.yaml* надо добавить строки: `macaddress: 10:10:10:10:10:BA`, `dhcp4: true`.
- В отчёт помести скрин с содержанием изменённого файла *etc/netplan/00-installer-config.yaml*.

**== Решение ==**
- конфиг dhcp на R1
![6-2-1](Screenshoots/6-2-1.png)

- netplan на WS11
![6-2-2](Screenshoots/6-2-2.png)

##### Для r1 настрой аналогично r2, но сделай выдачу адресов с жесткой привязкой к MAC-адресу (ws11). Проведи аналогичные тесты.
- В отчёте этот пункт опиши аналогично настройке для r2.

**== Решение ==**
![6-2-3](Screenshoots/6-2-3.png)

- настройка с привязкой аналогична предыдущим пунктам, помимо одного отличия - в настройках сети необходимо указать имя хотса, его оборудование с мак адресом и сам ip, который мы будем присваивать.

- видим, что ip адрес присвоен хосту WS11
![6-2-4](Screenshoots/6-2-4.png)

##### Запроси с ws22 обновление ip адреса.
- В отчёте помести скрины ip до и после обновления.
- В отчёте опиши, какими опциями **DHCP** сервера пользовался в данном пункте.

**== Решение ==**
- WS22 до обновления
![6-2-5](Screenshoots/6-2-5.png)

- затем вводим *dhckient -r enp0s3* (интерфейс), чтобы удалить ip. Следом запрашиваем новый:
![6-2-6](Screenshoots/6-2-6.png)

##### Сохрани дампы образов виртуальных машин.
**P.S. Ни в коем случае не сохраняй дампы в гит!**

## Part 7. **NAT**
**== Задание ==**

*В данном задании используются виртуальные машины из Части 5.*
##### В файле */etc/apache2/ports.conf* на ws22 и r1 измени строку `Listen 80` на `Listen 0.0.0.0:80`, то есть сделай сервер Apache2 общедоступным.
- В отчёт помести скрин с содержанием изменённого файла.
##### Запусти веб-сервер Apache командой `service apache2 start` на ws22 и r1.
- В отчёт помести скрины с вызовом и выводом использованной команды.
##### Добавь в фаервол, созданный по аналогии с фаерволом из Части 4, на r2 следующие правила:
##### 1) Удаление правил в таблице filter - `iptables -F`;
##### 2) Удаление правил в таблице "NAT" - `iptables -F -t nat`;
##### 3) Отбрасывать все маршрутизируемые пакеты - `iptables --policy FORWARD DROP`.
##### Запусти файл также, как в Части 4.
##### Проверь соединение между ws22 и r1 командой `ping`.
*При запуске файла с этими правилами, ws22 не должна «пинговаться» с r1.*
- В отчёт помести скрины с вызовом и выводом использованной команды.

**== Решение ==**
- Сделал сервер общедоступным на R1 и WS22
![7-1-1](Screenshoots/7-1-1.png)

- Прописал правила для фаервола на R2
![7-1-2](Screenshoots/7-1-2.png)

- С WS22 R1 не пингуется
![7-1-3](Screenshoots/7-1-3.png)

##### Добавь в файл ещё одно правило:
##### 4) Разрешить маршрутизацию всех пакетов протокола **ICMP**.
##### Запусти файл также, как в Части 4.
##### Проверь соединение между ws22 и r1 командой `ping`.
*При запуске файла с этими правилами, ws22 должна «пинговаться» с r1.*
- В отчёт помести скрины с вызовом и выводом использованной команды.

**== Решение ==**
- Добавил правило на R2
![7-2-1](Screenshoots/7-2-1.png)

- Пришлось убрать прямой маршрут до подсети и оставить шлюз для R2
![7-2-2](Screenshoots/7-2-2.png)

- R1 пингуется с WS22
![7-2-3](Screenshoots/7-2-3.png)

##### Добавь в файл ещё два правила:
##### 5) Включи **SNAT**, а именно маскирование всех локальных ip из локальной сети, находящейся за r2 (по обозначениям из Части 5 - сеть 10.20.0.0).
*Совет: стоит подумать о маршрутизации внутренних пакетов, а также внешних пакетов с установленным соединением.*
##### 6) Включи **DNAT** на 8080 порт машины r2 и добавить к веб-серверу Apache, запущенному на ws22, доступ извне сети.
*Совет: стоит учесть, что при попытке подключения возникнет новое tcp-соединение, предназначенное ws22 и 80 порту.*
- В отчёт помести скрин с содержанием изменённого файла.
##### Запусти файл также, как в Части 4.
*Перед тестированием рекомендуется отключить сетевой интерфейс **NAT** (его наличие можно проверить командой `ip a`) в VirtualBox, если он включен.*
##### Проверь соединение по TCP для **SNAT**: для этого с ws22 подключиться к серверу Apache на r1 командой:
`telnet [адрес] [порт]`
##### Проверь соединение по TCP для **DNAT**: для этого с r1 подключиться к серверу Apache на ws22 командой `telnet` (обращаться по адресу r2 и порту 8080).
- В отчёт помести скрины с вызовом и выводом использованных команд.

##### Сохрани дампы образов виртуальных машин.
**P.S. Ни в коем случае не сохраняй дампы в гит!**

## Part 8. Дополнительно. Знакомство с **SSH Tunnels**

`-` Пожалуй, на этом у меня всё. Может, у тебя появились ещё какие-то вопросы?

`-` Да, я хотел спросить ещё об одной вещи. На работе я краем уха услышал, что в моей компании есть некие проекты по обучению. Подробностей я не знаю, но очень хочется взглянуть... Вдруг будет полезно.

`-` Действительно интересно, но как в этом помогу тебе я?

`-` Дело в том, что, чтобы добраться до этих проектов, нужно получить доступ к закрытой сети. Можешь посоветовать что-нибудь по этому поводу?

`-` Ну ты, конечно, даёшь... Не уверен на все сто, что это поможет, но могу рассказать тебе про **SSH Tunnels**.

**== Задание ==**

*В данном задании используются виртуальные машины из Части 5.*

##### Запусти на r2 фаервол с правилами из Части 7.
##### Запусти веб-сервер **Apache** на ws22 только на localhost (то есть в файле */etc/apache2/ports.conf* измени строку `Listen 80` на `Listen localhost:80`).
##### Воспользуйся *Local TCP forwarding* с ws21 до ws22, чтобы получить доступ к веб-серверу на ws22 с ws21.
##### Воспользуйся *Remote TCP forwarding* c ws11 до ws22, чтобы получить доступ к веб-серверу на ws22 с ws11.
##### Для проверки, сработало ли подключение в обоих предыдущих пунктах, перейди во второй терминал (например, клавишами Alt + F2) и выполни команду:
`telnet 127.0.0.1 [локальный порт]`
- В отчёте опиши команды, необходимые для выполнения этих четырёх пунктов, а также приложи скриншоты с их вызовом и выводом.

##### Сохрани дампы образов виртуальных машин.
**P.S. Ни в коем случае не сохраняй дампы в гит!**
