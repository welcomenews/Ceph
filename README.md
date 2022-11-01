## Ceph


```
## Установка cephadm

# yum install python3 -y
# curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm
# mv cephadm /usr/local/bin/cephadm
# chmod +x /usr/local/bin/cephadm
# /usr/local/bin/cephadm add-repo --release pacific
# echo 'export PATH-"$PATH:/usr/local/bin/"' >> .bashrc     ## Записываем в bashrc (только для centos 8):
```

#### Commands
```

Смотерь состояние кластера
# ceph health detail

Отслеживание событий в кластере
# ceph -w

Ключи аутентификации кластера
# ceph auth list

Состояние MON
# ceph mon stat
# ceph mon dump

Состояние кворума MON
# ceph quorum_status

Просмотр дерева OSD
# ceph osd tree

Отобразить список дисков
# ceph-disk list

Отобразить статистику кластера
# ceph df

Для получения детализированной информации о кластере Ceph и OSD
# ceph osd dump

Наблюдение за MDS
# ceph mds stat

просмотр карты CRUSH
# ceph osd crush dump

Для просмотра правил карты CRUSH
# ceph osd crush rule list

Для подробного просмотра правил CRUSH.  crush_rule_name берётся из ceph osd crush rule list
# ceph osd crush rule dump <crush_rule_name>

Наблюдение за группами размещения
Группы размещения могут демонстрировать множество состояний:

Peering: При пиринге (равноправном информационном обмене) группы размещения OSD, которые находятся в действующем наборе, сохраняют реплики этих групп размещения, приходят к соглашению о состоянии объекта и его метаданных в PG. После того, как пиринг завершен, OSD, которые хранят PG договариваются об их текущем состоянии.

Active: После завершения выполнения пиринга Ceph делает данную PG активной. При активном состоянии данные PG доступны в ее первичной PG, а также в ее репликах для операций ввода/ вывода.

Clean: В чистом состоянии первичное и вторичные OSD успешно размещены, никакие PG не перемещаются со своего правильного местоположения, а также все объекты реплицированы необходимое количество раз.

Degraded: Если OSD отключается (down), Ceph изменяет состояние всех своих PG, которые назначены на это OSD как деградировавшие. После того, как OSD переходит в UP, оно должно выполнить пиринг вновь чтобы сделать деградировавшие PG чистыми. Если OSD остается отключенным и прошло более 300 секунд, Ceph восстанавливает все испытавшие деградацию PG из их реплик для поддержания необходимого числа реплик. Клиенты могут выполнять операции ввода/ вывода даже после того, как PG находятся в состоянии деградации. Может существовать еще одна причина по которой группа размещения может стать деградировавшей; а именно, когда один или более объектов внутри PG становятся недоступными. Ceph предполагает, что объекты должны быть внутри данной PG, однако они в реальности не доступны. В этом случае Ceph помечает PG как деградировавшую и пытается восстановить PG из ее реплик.

Recovering: Когда OSD выключается (down). содержимое его групп размещения отстает от содержимого его реплик на других OSD. Как только OSD возвращается в рабочее состояние (UP), Ceph инициализирует операцию восстановления на данной PG для приведения ее в соответствие с реплицированными PG в других OSD.

Backfiling: Как только новое OSD добавлено в кластер, Ceph пытается выполнить балансировку данных перемещая некоторые PG с других OSD на это новое OSD; этот процесс называется заливкой (backfiling). После выполнения заливки для этих групп размещения, OSD может участвовать в клиентском вводе/ выводе. Ceph выполняет заливку гладко в фоновом режиме и гарантирует отсутствие перегрузки данного кластера.

Remapped: Каждый раз, когда возникают изменения в PG действующего набора, происходит миграция со старого действующего набора OSD на новый действующий набор OSD. Такая операция может занять некоторое время, причем продолжительность зависит от размера данных, которые должны быть перенесены на новое OSD. В течение этого времени старое первичное OSD и старая действующая группа обслуживают запросы клиентов. Как только операция по перемещению данных завершена, Ceph начинает использовать новое первичное OSD из действующей группы.

Stale: Ceph выдает свои статистические данные монитору Ceph каждые 0.5 секунды; может оказаться, что если первичное OSD группы размещения действующего набора отказывает в выдаче своих статистических данных своим мониторам, или если другие OSD сообщают об отключении (down) своего первичного OSD, монитор будет рассматривать эту PG как утратившую силу (stale).

# ceph pg stat

vNNNN: X pgs: Y active+clean; R bytes data, U MB used, F GB / T GB avail
Значения переменных здесь:
vNNNN: Это номер версии карты PG
X: Это общее число групп размещения
Y: Это значение устанавливает число PG с их состояниями
R: Это значение задает объем сохраненных первичных данных
U: Это значение описывает объем реально сохраненных данных после репликации
F: Это размер оставшегося свободного пространства
T: Это общая емкость


Отобразить черный список клиентов
# ceph osd blacklist ls

Получить список групп размещения
# ceph pg dump

Чтобы получить список удерживаемых (stuck) групп размещения
# ceph pg dump_stuck unclean

Проверяем количество хостов:
# ceph orch host ls

Инфа о кластере
# ceph -s

Смотреть пул
# ceph osd pool ls

Смотреть инфу о соданном rbd диске disk01
# rbd --image disk01 -p rbd info

Смотреть количество PG\PGP
# ceph osd pool get <имя волума> pgp_num (например cephfs.volume1.data)
# ceph osd pool get <имя волума> pg_num (например cephfs.volume1.data)



Какая именно placement group в неконсистентном состоянии
# ceph health detail

Чтобы узнать, какую именно ошибку выдал Scrub
# rados list-inconsistent-obj {PG} | jq

Когда есть primary-копия, и она корректная, можно запустить восстановление командой
# ceph pg repair {PG}

Отключить Scrub с помощью флагов:
Обычный:
      # ceph osd set noscrub
Глубокий:
      # ceph osd set nodeep-scrub

Если у вас несколько пулов, и вы хотите для конкретного пула отключить скраб, то это можно сделать командой:
# ceph osd pool set {name} noscrub 1
# ceph osd pool set {name} nodeep-scrub 1
Эти флаги блокируют новый Scrub, но уже запущенные проверки не отклоняются и будут завершены.


Останавливать процессы восстановления и перемещения данных с флагами.  флаг nobackfill и norecover работают одинаково.
# ceph osd set nobackfill и ceph osd set norecover
# ceph osd set norebalance

снимем флаг norecover
# ceph osd unset norecover

остановим одну из OSD и отправим её в out, чтобы запустился процесс recovery.
# systemctl stop ceph-osd@0
# ceph osd out 0

Флаг pause по сути останавливает клиентское io. Никто из клиентов не сможет ни читать, ни писать
# ceph osd set pause

используется в случаях, когда требуется ограничить скорость ребаланса. В итоге, ребаланс будет выполняться медленнее и вызовет меньшую сетевую нагрузку.
osd_max_backfill


переведем нужную OSD в состояние down
# ceph osd down osd.${ID}

Остановим сервис OSD
# systemctl stop ceph-osd@${ID}




```
https://sidmid.ru/ceph-полезные-команды/

https://docs.ceph.com/en/quincy/install/

http://onreader.mdl.ru/LearningCeph/content/Ch08.html

https://old.ceph.com/pgcalc/



