
Заменить сбойный диск.

Для решения задачи необходимо сделать следующее:

1. Подготовьте резервную машину к вводу в кластер:
- Добавьте ssh от кластера ceph на машину;
- Поменяйте hostname на ceph4, дополните файл /etc/hosts на ceph4 и ceph1, чтобы машины могли обращаться друг к другу по имени хоста.
2. Добавьте диск с ceph4 в кластер.
3. Удалите сбойный диск:
- Определите его имя (ceph orch ps);
- Удалите запись из crushmap;
- Удалите диск;
- Удалите daemon, отвечающий за сбойный диск.

======

```
обменялись с ней ключиками
ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph4

На Сервер 4

# cephadm install ceph-common


На Сервер 1

дополните файл /etc/hosts
ceph4

Добавляем хост
# ceph orch host add ceph4 10.129.0.5

Добавляем с Ceph 4 его диск
# ceph orch daemon add osd ceph4:/dev/vdb

# ceph -s

# ceph orch ps

удаляем OSD2 диск, который уже “поломан”
# ceph osd rm osd.2

удалить диск из карты crush
# ceph osd crush remove osd.2
# ceph orch daemon rm osd.2 --force

# ceph -s

Выключаем третью ноду, добавляем лейбл mon на четвертую. Добавляем новый монитор.
# shutdown -h now (на нода3)
или
# systemctl stop ceph-mon.target (на нода3)

# ceph orch host label add ceph4 mon
# ceph orch apply mon "ceph1,ceph2,ceph4"

Удаляем старый монитор:
# ceph mon ok-to-stop mon.ceph3
# ceph mon ok-to-rm mon.ceph3
# ceph mon rm mon.ceph3

Удаляем демона и получаем ошибку SSH:
ceph orch daemon rm mon.ceph3 --force

Пытаемся удалить хост:
# ceph orch host label rm ceph3 mon
# ceph orch host rm ceph3 --offline --force

Удаляем демона
# ceph orch daemon rm mon.ceph3 --force

# ceph -s

Перегенерируем конфиг:
ceph config generate-minimal-conf > ceph.conf

```

https://habr.com/ru/company/oleg-bunin/blog/431536/

https://habr.com/ru/company/southbridge/blog/536884/

https://habr.com/ru/company/southbridge/blog/534912/

https://habr.com/ru/company/flant/blog/495870/

https://docs.ceph.com/en/nautilus/install/upgrading-ceph/

https://cloud.garr.it/support/kb/ceph/restore-ceph-from-mon-disaster/




