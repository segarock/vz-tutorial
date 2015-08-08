# Руководство по созданию и управлению контейнерами на базе Virtuozzo

## Содержание
1. [Введение в виртуализацию](#Введение-в-виртуализацию)
  - [Эмуляция оборудования](#Эмуляция-оборудования)
  - [Полная виртуализация](#Полная-виртуализация)
  - [Паравиртуализация](#Паравиртуализация)
  - [Виртуализация уровня операционной системы](#Виртуализация-уровня-операционной-системы)
  - [Virtuozzo — объединение технологий виртуализации уровня ОС и полной виртуализации](#virtuozzo--объединение-технологий-виртуализации-уровня-ОС-и-полной-виртуализации)
2. [Краткая история проекта Virtuozzo](#Краткая-история-проекта-virtuozzo)
3. [Установка и подготовительные действия](#Установка-и-подготовительные-действия)
  - [Установка Virtuozzo с помощью ISO-образа (bare-metal installation)](#Установка-virtuozzo-с-помощью-iso-образа-bare-metal-installation)
  - [Установка Virtuozzo на заранее установленный дистрибутив](#Установка-virtuozzo-на-заранее-установленный-дистрибутив)
  - [Подготовительные действия](#Подготовительные-действия)
4. [Управление шаблонами гостевых ОС](#Управление-шаблонами-гостевых-ОС)
5. [Создание и настройка контейнеров](#Создание-и-настройка-контейнеров)
  - [Конфигурационные файлы](#Конфигурационные-файлы)
  - [Создание контейнера](#Создание-контейнера)
  - [Настройка контейнера](#Настройка-контейнера)
  - [Запуск и вход](#Запуск-и-вход)
6. [Управление контейнерами](#Управление-контейнерами)
  - [Управление состоянием контейнера](#Управление-состоянием-контейнера)
  - [Клонирование контейнера](#Клонирование-контейнера)
  - [Запуск команд в контейнере с хост-ноды](#Запуск-команд-в-контейнере-с-хост-ноды)
  - [Расширенная информация о контейнере](#Расширенная-информация-о-контейнере)
7. [Ссылки](#Ссылки)
8. [Лицензия](#Лицензия)

## Введение в виртуализацию
Виртуализация — предоставление наборов вычислительных ресурсов или их логического объединения, абстрагированное от аппаратной реализации, и обеспечивающее изоляцию вычислительных процессов.

Виртуализацию можно использовать в:
* консолидации серверов (позволяет мигрировать с физических серверов на виртуальные, тем самым увеличивается коэффициент использования аппаратуры, что позволяет существенно сэкономить на аппаратуре, электроэнергии и обслуживании)
* разработке и тестировании приложений (возможность одновременно запускать несколько различных ОС, это удобно при разработке кроссплатформенного ПО, тем самым значительно повышается качество, скорость разработки и тестирования приложений)
* бизнесе (использование виртуализации в бизнесе растет с каждым днем и постоянно находятся новые способы применения этой технологии, например, возможность безболезненно сделать снапшот)
* организации виртуальных рабочих станций (так называемых "тонких клиентов")

*Общая схема взаимодействия виртуализации с аппаратурой и программным обеспечением*
![Общая схема взаимодействия виртуализации с аппаратурой и программным обеспечением](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/virt-scheme.png)

Понятие виртуализации можно условно разделить на две категории:
* виртуализация платформ, продуктом этого вида виртуализации являются виртуальные машины
* виртуализация ресурсов преследует целью комбинирование или упрощение представления аппаратных ресурсов для пользователя и получение неких пользовательских абстракций оборудования, пространств имен, сетей

Взаимодействие приложений и операционной системы (ОС) с аппаратным обеспечением осуществляется через абстрагированный слой виртуализации.

Существует несколько подходов организации виртуализации:
* эмуляция оборудования (QEMU, Bochs, Dynamips)
* полная виртуализация (KVM, HyperV, VirtualBox)
* паравиртуализация (Xen, L4, Trango)
* виртуализация уровня ОС (LXC, Virtuozzo, Jails, Solaris Zones)

### Эмуляция оборудования
Эмуляция аппаратных средств является одним из самых сложных методов виртуализации.
В то же время главной проблемой при эмуляции аппаратных средств является низкая скорость работы, в связи с тем, что каждая команда моделируется на основных аппаратных средствах.

В эмуляции оборудования используется механизм динамической трансляции, то есть каждая из инструкций эмулируемой платформы заменяется на заранее подготовленный фрагмент инструкций физического процессора.

Однако метод позволяет использовать виртуализированные аппаратные средства еще до выхода реальных.
Например, управление неизмененной ОС, предназначенной для PowerPC на системе с ARM процессором.

*Эмуляция оборудования моделирует аппаратные средства*
![Схема эмуляции оборудования](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/emulation.png)

### Полная виртуализация
Полная виртуализация использует гипервизор, который осуществляет связь между гостевой ОС и аппаратными средствами физического сервера.
В связи с тем, что вся работа с гостевой операционной системой проходит через гипервизор, скорость работы данного типа виртуализации ниже чем в случае прямого взаимодействия с аппаратурой.
Основным преимуществом является то, что в ОС не вносятся никакие изменения, единственное ограничение — операционная система должна поддерживать основные аппаратные средства.

*Полная виртуализация использует гипервизор*
![Схема полной виртуализации](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/full-virt.png)

Полная виртуализация возможна исключительно при условии правильной комбинации оборудования и программного обеспечения.

### Паравиртуализация
Паравиртуализация имеет некоторые сходства с полной виртуализацией.
Этот метод использует гипервизор для разделения доступа к основным аппаратным средствам, но объединяет код, касающийся виртуализации, в непосредственно операционную систему, поэтому недостатком метода является то, что гостевая ОС должна быть изменена для гипервизора.
Но паравиртуализация существенно быстрее полной виртуализации, скорость работы виртуальной машины приближена к скорости реальной, это осуществляется за счет отсутствия эмуляции аппаратуры и учета существования гипервизора при выполнении системных вызовов в коде ядра.
Вместо привилегированных операций совершаются гипервызовы обращения ядра гостевой ОС к гипервизору с просьбой о выполнении операции.

*Паравиртуализация разделяет процесс с гостевой ОС*
![Схема паравиртуализации](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/paravirt.png)

В паравиртуальном режиме (PV) оборудование не эмулируется, и гостевая операционная система должна быть специальным образом модифицирована для работы в таком окружении.
Начиная с версии 3.0, ядро Linux поддерживает запуск в паравиртуальном режиме без перекомпиляции со сторонними патчами.
Преимущество режима паравиртуализации состоит в том, что он не требует поддержки аппаратной виртуализации со стороны процессора, а также не тратит вычислительные ресурсы для эмуляции оборудования на шине PCI.

Режим аппаратной виртуализации (HVM), который появился в Xen, начиная с версии 3.0 гипервизора требует поддержки со стороны оборудования.
В этом режиме для эмуляции виртуальных устройств используется QEMU, который весьма медлителен несмотря на паравиртуальные драйвера.
Однако со временем поддержка аппаратной виртуализации в оборудовании получила настолько широкое распространение, что используется даже в современных процессорах лэптопов.

### Виртуализация уровня операционной системы
Виртуализация уровня операционной системы отличается от других.
Она использует технику, при которой сервера виртуализируются непосредственно над ОС.
Недостатком метода является то, что поддерживается одна единственная операционная система на физическом сервере, которая изолирует контейнеры друг от друга.
Преимуществом виртуализации уровня ОС является "родная" производительность.

*Виртуализация уровня ОС изолирует серверы*
![Схема виртуализации уровня ОС](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/cont-virt.png)

Виртуализация уровня ОС — метод виртуализации, при котором ядро операционной системы поддерживает несколько изолированных экземпляров пространства пользователя вместо одного.
Эти экземпляры с точки зрения пользователя полностью идентичны реальному серверу.
Для систем на базе UNIX эта технология может рассматриваться как улучшенная реализация механизма chroot.
Ядро обеспечивает полную изолированность контейнеров, поэтому программы из разных контейнеров не могут воздействовать друг на друга.

### Virtuozzo — объединение технологий виртуализации уровня ОС и полной виртуализации
Virtuozzo позволяет создавать множество защищенных, изолированных друг от друга контейнеров на одном узле.
Помимо этого возможно создание виртуальных машин на базе QEMU/KVM.
Управление контейнерами и виртуальными машинами происходит с помощью специализированных утилит.

Каждый контейнер ведет себя так же, как автономный сервер и имеет собственные файлы, процессы, сеть (IP адреса, правила маршрутизации).
В отличие от KVM или Xen, Virtuozzo использует одно ядро, которое является общим для всех виртуальных сред.

Контейнеры можно разделить на две составляющие:
* ядро (namespaces, cgroups, CRIU, ploop, vcmmd...)
* пользовательские утилиты (prlctl, vzpkg, vzquota, vzdump...)

Namespaces — пространства имен.
Это механизм ядра, который позволяет изолировать процессы друг от друга. Изоляция может быть выполнена в шести контекстах (пространствах имен):
* mount — предоставляет процессам собственную иерархию файловой системы и изолирует ее от других таких же иерархий по аналогии с chroot
* PID — изолирует идентификаторы процессов (PID) одного пространства имен от процессов с такими же идентификаторами другого пространства
* network — предоставляет отдельным процессам логически изолированный от других стек сетевых протоколов, сетевой интерфейс, IP-адрес, таблицу маршрутизации, ARP и прочие реквизиты
* IPC — обеспечивает разделяемую память и взаимодействие между процессами
* UTS — изоляция идентификаторов узла, таких как имя хоста (hostname) и домена (domain)
* user — позволяет иметь один и тот же набор пользователей и групп в рамках разных пространств имен, в каждом контейнере могут быть свой
root и любые другие пользователи и группы

CGroups (Control Groups) — позволяет ограничить аппаратные ресурсы некоторого набора процессов.
Под аппаратными ресурсами подразумеваются: процессорное время, память, дисковая и сетевая подсистемы.
Набор или группа процессов могут быть определены различными критериями.
Например, это может быть целая иерархия процессов, получающая все лимиты родительского процесса.
Кроме этого возможен подсчет расходуемых группой ресурсов, заморозка (freezing) групп, создание контрольных точек (checkpointing) и их перезагрузка.
Для управления этим полезным механизмом существует специальная библиотека libcgroups, в состав которой входят такие утилиты, как cgcreate, cgexec и некоторые другие.

CRIU (Checkpoint/Restore In Userspace) — обеспечивает создание контрольной точки для произвольного приложения, а также возобновления работы приложения с этой точки.
Основной целью CRIU является поддержка миграции контейнеров.
Уже поддерживаются такие объекты как процессы, память приложений, открытые файлы, конвейеры, IPC сокеты, TCP/IP и UDP сокеты, таймеры, сигналы, терминалы, файловые дескрипторы.
В разработке также находится миграция TCP соединений.

vcmmd (Virtuozzo containers memory management daemon) — сервис управления механизмом memory cgroups в пространстве пользователя.
Менеджер памяти 4 поколения управляет memory cgroups.
memory cgroups присутстсвует в ванильном ядре, поэтому не требует сторонних патчей со стороны Virtuozzo.

Новый менеджер памяти полностью задействует механизм memory cgroups,
который есть в ванильном ядре. Поэтомы для этой версии нам не необходимости
держать какие-то свои патчи в ядре. Управление реализовано с помощью сервиса
vcmmd в пространстве пользователя.


Проведенные тестирования показывают, что OpenVZ (ныне Virtuozzo) является одним из наиболее актуальных решений на рынке виртуализации, так как показывает внушительные результаты в различных тестированиях.

*График времени отклика системы*
![Время отклика системы](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/response-time.png)

На графике времени отклика системы можно наблюдать результаты трех тестов — с нагрузкой на систему и виртуальную машину, без нагрузки на систему и ВМ, с нагрузкой на ВМ и без нагрузки на систему.
Во всех тестах OpenVZ показал результаты наименьшего времени отклика, в то время, когда ESXi и Hyper-V показывают оверхед 700-3000%, когда у OpenVZ всего 1-3%.

*График пропускной способности сети*
![Пропускная способность сети](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/network.png)

На втором графике — результаты тестирования пропускной способности сети.
На графике можно наблюдать, что OpenVZ обеспечивает практическую нативную пропускную способность 10G сети (9.7G отправка и 9.87G прием).

## Краткая история проекта Virtuozzo
В 1999 году возникла идея создания Linux контейнеров, а уже в 2002 году компания SWsoft представила первый релиз коммерческой версии Virtuozzo. В том же 2002 году появились первые клиенты в Кремниевой долине.

В 2004 году выпуск Virtuozzo для Windows.
В 2005 году было принято решение о разделении Virtuozzo на два отдельных проекта, свободный OpenVZ (под лицензией GNU GPL) и проприетарный Virtuozzo.

В 2006 году OpenVZ стал доступен для Debian Linux, переход к ядру RHEL4.
В 2007 году портирован на RHEL5.

В 2011 году появилась идея создания проекта CRIU, OpenVZ портирован на RHEL6.
В 2012 году стала доступна CRIU v0.1.

В конце 2014 года компания Odin анонсировала открытие кодовой базы Parallels Cloud Server и объединение ее с открытым OpenVZ.

В апреле 2015 года был открыт репозиторий с ядром RHEL7 (3.10), в мае были открыты исходные коды пользовательских утилит, а в июне выложены тестовые сборки ISO-образов и RPM-пакеты.

## Установка и подготовительные действия
Существует два способа установки Virtuozzo:
* с помощью ISO-образа дистрибутива
* с помощью установки пакетов и ядра на заранее установленный дистрибутив

### Установка Virtuozzo с помощью ISO-образа (bare-metal installation)
Дистрибутив Virtuozzo основан на операционной системе [CloudLinux](https://www.cloudlinux.com/) с патчами для ядра RHEL7, утилитами управления и модифицированным установщиком.
Рекомендуется именно этот способ установки Virtuozzo.

Текущая последняя версия ISO-образа доступна по адресу: https://download.openvz.org/virtuozzo/releases/7.0/x86_64/iso/

После записи дистрибутива на носитель, можно приступать к настройке Virtuozzo.
Для этого необходимо загрузиться с носителя.

*Экран установки Virtuozzo после загрузки с носителя*
![Экран установки Virtuozzo](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/vz-install/install-vz.png)

Установка Virtuozzo ничем не отличается от установки обычного Linux-дистрибутива.
Установщик Anaconda предложит установить дату и время, раскладку клавиатуры, языковые параметры.
Также необходимо будет произвести разметку диска и настроить сеть.
По умолчанию включен kdump, который позволяет в будущем выяснить причины сбоев в ядре, поэтому рекомендуется его не отключать.

*Экран установки параметров системы*
![Настройки системы](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/vz-install/anaconda.png)

*Пример разметки для 20GB диска*
![Разметка диска](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/vz-install/partitioning.png)

Необходимо для раздела `/` выделить не менее 8GB доступного дискового пространства.
Размер раздела `swap` равен примерно половине объема оперативной памяти.
Все остальное дисковое пространство выделяется под раздел `/vz` с файловой системой ext4.

*Настройки сетевого интерфейса и имени хоста*
![Настройки сети](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/vz-install/network.png)

Также необходимо задать пароль пользователя `root` и создать локального пользователя, например `vzuser`.

*Задание пароля суперпользователя и создание локального пользователя*
![Настройки пользователей](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/vz-install/user.png)

После установки необходимо перезагрузиться.

На этом установка Virtuozzo с помощью ISO-образа завершена.

*Меню загрузчика Grub*
![Grub](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/vz-install/grub.png)

Первый вход в систему осуществляется от пользователя `vzuser`, по SSH.

Пример получения прав суперпользователя на сервере:
```
amet13@mint-17 ~ $ ssh vzuser@192.168.0.150
vzuser@192.168.0.150's password: пароль_пользователя_vzuser
[vzuser@virtuozzo ~]$ su -
Password: пароль_пользователя_root
[root@virtuozzo ~]#
```

### Установка Virtuozzo на заранее установленный дистрибутив

Поддерживаемые дистрибутивы:
* CloudLinux 7
* CentOS 7
* Scientific Linux 7
* прочие дистрибутивы, основанные на RHEL7

Установка пакетов на примере дистрибутива CentOS 7.

Пакет `virtuozzo-release` содержит метаинформацию и yum-репозитории, необходимые для установки пакетов:
```
[root@virtuozzo ~]# yum localinstall https://download.openvz.org/virtuozzo/releases/7.0/x86_64/os/Packages/v/virtuozzo-release-7.0.0-10.vz7.x86_64.rpm
```

Установка необходимых RPM-пакетов:
```
[root@virtuozzo ~]# yum install prlctl prl-disp-service vzkernel ploop
```

В качестве зависимостей также установятся такие пакеты как `criu`, `libvirt`, `lvm2`, `nfs-utils`, `quota`, `vcmmd`, `vzctl`, `vztt` и многие другие.

По окончании установки пакетов необходимо перезагрузиться:
```
[root@virtuozzo ~]# reboot
```

В меню загрузчика должны появиться новые пункты `Virtuozzo 7`.
Необходимо загрузиться с этого ядра.

*Меню загрузчика Grub после установки Virtuozzo*
![Grub](https://raw.githubusercontent.com/Amet13/virtuozzo-tutorial/master/images/vz-install/grub-vz.png)

### Подготовительные действия
На сервере важно всегда обновлять программное обеспечение, так как в новых версиях не только могут добавлять новые возможности, но и исправлять уязвимости.
Указанная ниже команда обновляет все существующие в системе пакеты:
```
[root@virtuozzo ~]# yum update
```
Для сервера очень важно, чтобы было установлено правильное время.
Чтобы синхронизировать время с интернетом необходимо установить пакет `ntp`.

Установка корректной временной зоны:
```
[root@virtuozzo ~]# timedatectl set-timezone Europe/Moscow
[root@virtuozzo ~]# date
Tue Aug  4 14:52:54 MSK 2015
```
Установка `ntp` и синхронизация времени с удаленными серверами:
```
[root@virtuozzo ~]# yum install ntp
[root@virtuozzo ~]# systemctl start ntpd
[root@virtuozzo ~]# systemctl enable ntpd
[root@virtuozzo ~]# ntpdate -q  0.pool.ntp.org  1.pool.ntp.org
server 91.236.251.5, stratum 2, offset 0.002229, delay 0.05281
server 82.193.117.90, stratum 1, offset -0.020269, delay 0.04845
server 78.26.196.124, stratum 2, offset 0.003866, delay 0.05913
server 217.175.0.36, stratum 3, offset -0.003749, delay 0.06514
server 79.142.192.4, stratum 2, offset 0.006668, delay 0.05772
server 195.138.69.242, stratum 2, offset 0.005080, delay 0.05731
server 91.236.251.12, stratum 2, offset 0.002247, delay 0.05368
server 91.198.10.20, stratum 2, offset 0.003745, delay 0.05481
 4 Aug 14:54:56 ntpdate[2804]: adjust time server 91.236.251.5 offset 0.002229 sec
```

## Управление шаблонами гостевых ОС
На данный момент Virtuozzo поддерживает такие гостевые ОС:
* CentOS 7 (x86_64)
* CentOS 6 (x86_64)
* Ubuntu 14.04 LTS (x86_64)
* Debian 8 (x86_64)

Просмотр списка уже имеющихся локальных шаблонов:
```
[root@virtuozzo ~]# vzpkg list -O --with-summary
centos-6-x86_64                    :Centos 6 (for AMD64/Intel EM64T) Virtuozzo Template
centos-5-x86                       :Centos 5 (for ix86) Virtuozzo Template
```

Установка всех доступных шаблонов:
```
[root@virtuozzo ~]# vzpkg list --available --with-summary | xargs vzpkg install template
```

После этого можно увидеть список доступных локально шаблонов гостевых ОС:
```
[root@virtuozzo ~]# vzpkg list -O --with-summary
centos-6-x86_64                    :Centos 6 (for AMD64/Intel EM64T) Virtuozzo Template
centos-5-x86                       :Centos 5 (for ix86) Virtuozzo Template
centos-7-x86_64                    :Centos 7 (for AMD64/Intel EM64T) Virtuozzo Template
ubuntu-14.04-x86_64                :Ubuntu 14.04 (for AMD64/Intel EM64T) Virtuozzo Template
debian-8.0-x86_64                  :Debian 8.0 (for AMD64/Intel EM64T) Virtuozzo Template
```

Обновление кэша шаблонов:
```
[root@virtuozzo ~]# vzpkg update cache
```

## Создание и настройка контейнеров
### Конфигурационные файлы
В старых версиях OpenVZ основным идентификатором контейнера является CTID, который вручную указывался при создании контейнера.
Сейчас в этом нет необходимости, на смену CTID пришел UUID, который создается автоматически.
Однако существует возможность указания UUID вручную.

Каждый контейнер имеет свой конфигурационный файл, который хранится в каталоге `/etc/vz/conf/`.
Именуются конфиги по UUID контейнера.
Например, для контейнера с `UUID = {3d32522a-80af-4773-b9fa-ea4915dee4b3}`, конфиг будет называться `3d32522a-80af-4773-b9fa-ea4915dee4b3.conf`.

При создании контейнера можно использовать типовую конфигурацию.
Типовые файлы конфигураций находятся в том же каталоге `/etc/vz/conf/`:
```
[root@virtuozzo ~]# ls /etc/vz/conf/ | grep sample
ve-basic.conf-sample
ve-confixx.conf-sample
ve-vswap.1024MB.conf-sample
ve-vswap.2048MB.conf-sample
ve-vswap.256MB.conf-sample
ve-vswap.512MB.conf-sample
ve-vswap.plesk.conf-sample
vps.vzpkgtools.conf-sample
```

В этих конфигурационных файлах описаны контрольные параметры ресурсов, выделенное дисковое пространство, оперативная память и т.д.
Например, при использовании конфига `ve-vswap.512MB.conf-sample`, создается контейнер с дисковым пространством 10GB, оперативной памятью 512MB и swap 512MB:
```
[root@virtuozzo ~]# cat /etc/vz/conf/ve-vswap.512MB.conf-sample | grep "DISKSPACE\|PHYSPAGES\|SWAPPAGES"
PHYSPAGES="131072:131072"
SWAPPAGES="131072"
DISKSPACE="10485760:10485760"
```

Это удобно, так как существует возможность создавать свои конфигурационные файлы для различных вариаций контейнеров.
Cоздадим свой конфигурационный файл, на базе уже существующего `vswap.512MB`.
Исправим в нем только значения `PHYSPAGES`, `SWAPPAGES`, `DISKSPACE`, `DISKINODES`:
```
[root@virtuozzo ~]# cp /etc/vz/conf/ve-vswap.512MB.conf-sample /etc/vz/conf/ve-vswap.1GB.conf-sample
[root@virtuozzo ~]# vim /etc/vz/conf/ve-vswap.1GB.conf-sample
PHYSPAGES="262144:262144"
SWAPPAGES="262144"
DISKSPACE="20971520:20971520"
DISKINODES="1310720:1310720"
```
Таким образом, при использовании этого конфигурационного файла, будет создаваться контейнер, которому будет доступно 20GB выделенного дискового пространства, 1GB оперативной памяти и 1GB swap.

### Создание контейнера
В качестве параметра к идентификатору контейнера может использоваться любое имя:
```
[root@virtuozzo ~]# CT=first
[root@virtuozzo ~]# prlctl create $CT --ostemplate debian-8.0-x86_64 --vmtype=ct
Creating the Virtuozzo Container...
The Container has been successfully created.
```

Таким образом был создан контейнер с именем `first` на базе шаблона `debian-8.0-x86_64`.

Теперь можно просмотреть список имеющихся в системе контейнеров:
```
[root@virtuozzo ~]# prlctl list -a
UUID                                    STATUS       IP_ADDR         T  NAME
{3d32522a-80af-4773-b9fa-ea4915dee4b3}  stopped      -               CT first
```

Если же при создании контейнера не указывать желаемый шаблон, то Virtuozzo будет использовать шаблон по умолчанию.
Конфигурационный файл, в котором указаны директивы по умолчанию `/etc/vz/vz.conf`.
По умолчанию, используется шаблон `centos-6` и конфигурационный файл `basic`:
```
[root@virtuozzo ~]# cat /etc/vz/vz.conf | grep "CONFIGFILE\|DEF_OSTEMPLATE"
CONFIGFILE="basic"
DEF_OSTEMPLATE=".centos-6"
```

Если планируется создание большого количества однотипных контейнеров, основываясь на одном и том же конфиге, то значения можно исправить на нужные.

### Настройка контейнера
Контейнер создан, его можно запускать.
Но перед первым запуском необходимо установить его IP адреса, hostname, указать DNS сервера и задать пароль суперпользователя.

Добавление IP адресов:
```
[root@virtuozzo ~]# prlctl set first --ipadd 192.168.0.161/24
[root@virtuozzo ~]# prlctl set first --ipadd fe80::20c:29ff:fe01:fb08
Enable automatic reconfiguration for this network adapter.
```

Установка DNS сервера и hostname:
```
[root@virtuozzo ~]# prlctl set first --nameserver 192.168.0.1
[root@virtuozzo ~]# prlctl set first --hostname first.virtuozzo.localhost
```

Установка пароля суперпользователя:
```
[root@virtuozzo ~]# prlctl set first --userpasswd root:p0oT
```

Пароль будет установлен в контейнер, в файл `/etc/shadow` и не будет сохранен в конфигурационный файл контейнера.
Если же пароль будет утерян или забыт, то можно будет просто задать новый.

Для запуска контейнера при старте хост-ноды добавляем:
```
[root@virtuozzo ~]# prlctl set first --onboot yes
```

### Запуск и вход
После настроек нового контейнера. его можно запустить:
```
[root@virtuozzo ~]# prlctl start first
Starting the CT...
The CT has been successfully started.
```

Проверяем сетевые интерфейсы внутри гостевой ОС:
```
[root@virtuozzo ~]# prlctl exec first ifconfig | grep "lo\|venet" -A 1
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
--
venet0    Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
          inet addr:127.0.0.1  P-t-P:127.0.0.1  Bcast:0.0.0.0  Mask:255.255.255.255
--
venet0:0  Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
          inet addr:192.168.0.161  P-t-P:192.168.0.161  Bcast:192.168.0.255  Mask:255.255.255.0
```

Также проверим корректность hostname:
```
[root@virtuozzo ~]# prlctl exec first hostname
first.virtuozzo.localhost
```

Проверяем доступность контейнера в сети и корректность пароля суперпользователя:
```
[root@virtuozzo ~]# ssh root@192.168.0.161
root@192.168.0.161's password: p0oT
root@first:~#
```

Вход в контейнер напрямую с хост-ноды:
```
[root@virtuozzo ~]# prlctl enter first
entered into CT
root@first:/#
```

Выход из контейнера:
```
root@first:/# exit
logout
[root@virtuozzo ~]#
```

## Управление контейнерами
### Управление состоянием контейнера
Статус контейнера:
```
[root@virtuozzo ~]# prlctl status first
CT first exist running
[root@virtuozzo ~]# prlctl status second
CT second exist stopped
```

По выводу команды можно наблюдать, что контейнер `first` существует и запущен, а контейнер `second` существует и остановлен.

Остановка контейнера:
```
[root@virtuozzo ~]# prlctl stop first
Stopping the CT...
The CT has been successfully stopped.
```

Иногда нужно выключить контейнер как можно быстрее, например если контейнер был подвержен взлому.
Для того чтобы срочно выключить контейнер, нужно использовать ключ `--kill`:
```
[root@virtuozzo ~]# prlctl stop first --kill
Stopping the CT...
The CT has been forcibly stopped
```

Перезапуск контейнера:
```
[root@virtuozzo ~]# prlctl restart first
Restarting the CT...
The CT has been successfully restarted.
```

Для удаление контейнера, его нужно сначала остановить:
```
[root@virtuozzo ~]# prlctl stop
Stopping the CT...
The CT has been successfully stopped.
[root@virtuozzo ~]# prlctl delete second
Removing the CT...
The CT has been successfully removed.
```

Команда выполняет удаление частной области сервера и переименовывает файл конфигурации, дописывая к нему `.destroyed`.

### Клонирование контейнера
Virtuozzo позволяет клонировать контейнеры:
```
[root@virtuozzo ~]# prlctl clone first --name second
Clone the first CT to CT second...
The CT has been successfully cloned.
[root@virtuozzo ~]# prlctl list -a
UUID                                    STATUS       IP_ADDR         T  NAME
{3d32522a-80af-4773-b9fa-ea4915dee4b3}  running      192.168.0.161   CT first
{54bc2ba6-b040-469e-9fda-b0eabda822d4}  stopped      192.168.0.161   CT second
```

При клонировании контейнера необходимо помнить о смене IP адреса, иначе при попытке запуска будет наблюдаться ошибка:
```
[root@virtuozzo ~]# prlctl start second
Starting the CT...
Failed to start the CT: PRL_ERR_VZCTL_OPERATION_FAILED
```

Сначала нужно удалить старые IP адреса:
```
[root@virtuozzo ~]# prlctl set second --ipdel 192.168.0.161/24
[root@virtuozzo ~]# prlctl set second --ipdel fe80::20c:29ff:fe01:fb08
```

Затем добавить новые:
```
[root@virtuozzo ~]# prlctl set second --ipadd 192.168.0.162/24
[root@virtuozzo ~]# prlctl set second --ipadd fe80::20c:29ff:fe01:fb09
```

После этого контейнер можно запустить:
```
[root@virtuozzo ~]# prlctl start second
Starting the CT...
The CT has been successfully started.
```

### Запуск команд в контейнере с хост-ноды
Пример запуска команды в контейнере:
```
[root@virtuozzo ~]# prlctl exec first cat /etc/issue
Debian GNU/Linux 8 \n \l
```

Иногда бывает нужно выполнить команду на нескольких контейнерах.
Для этого можно использовать команду:
```
[root@virtuozzo ~]# for i in `prlctl list -o name -H`; do echo "CT $i"; prlctl exec $i cat /etc/issue; done
CT first
Debian GNU/Linux 8 \n \l

CT second
Debian GNU/Linux 8 \n \l
```

### Расширенная информация о контейнере
```
[root@virtuozzo ~]# prlctl list -i first
Autostop: suspend
Autocompact: on
Undo disks: off
Boot order:
EFI boot: off
Allow select boot device: off
External boot device:
Remote display: mode=off address=0.0.0.0
Remote display state: stopped
Hardware:
  cpu cpus=unlimited VT-x accl=high mode=32 cpuunits=1000 ioprio=4
  memory 512Mb
  video 0Mb 3d acceleration=highest vertical sync=yes
  memory_quota auto
  hdd0 (+) image='/vz/private/3d32522a-80af-4773-b9fa-ea4915dee4b3/root.hdd' type='expanded' 10240Mb mnt=/
  venet0 (+) type='routed' ips='192.168.0.161/255.255.255.0 FE80:0:0:0:20C:29FF:FE01:FB08/64 '
Host Shared Folders: (-)
Features:
Encrypted: no
Faster virtual machine: on
Adaptive hypervisor: off
Disabled Windows logo: on
Auto compress virtual disks: on
Nested virtualization: off
PMU virtualization: off
Offline management: (-)
Hostname: first.virtuozzo.localhost
DNS Servers: 192.168.0.1
```

## Ссылки
* https://openvz.org/History
* https://openvz.org/Quick_installation
* https://openvz.org/OpenVZ_with_upstream_kernel
* https://openvz.org/Packages
* https://openvz.org/Roadmap
* https://openvz.org/Category:HOWTO
* http://docs.openvz.org/virtuozzo_7_users_guide.webhelp/

## Лицензия
![CC BY-SA 4.0](https://licensebuttons.net/l/by-sa/4.0/88x31.png)

[Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/deed.ru)
