#### CPH 11: Резервное копирование и логирование Ceph

1. Создайте и подготовьте пул rbd и диск на 1 гб. Скопируйте ключ ceph на клиент.
2. Примапьте и смонтируйте диск в директорию /test на клиенте.
3. Заполните директорию /test файлами для снепшот.
4. Сделайте снепшот snap01 диска на клиенте.
5. Удалите содержимое /test, отмонтируйте и отмапьте диск.
6. Восстановите снепшот snap01 и смонтируйте его в директорю /test, убедитесь, что файлы восстановились.

====================


```
На Ceph

Создаём пул
ceph osd pool create kube 32
ceph osd pool application enable kube rbd

Проверка PG
ceph osd pool get kube pg_num
ceph osd pool get kube pgp_num
ceph osd pool autoscale-status

Создаём диск
rbd create disk01 --size 1G --pool kube
rbd ls -p kube

Минимальный конфиг
ceph config generate-minimal-conf > ceph.conf
scp /etc/ceph/ceph.conf client:/etc/ceph/ceph.conf

Создаём и копируем ключ
ceph auth get-or-create client.rbd mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=kube'
ceph auth get-or-create client.rbd | ssh user@client sudo tee /etc/ceph/keyring



На Клиенте

ceph osd pool ls --name client.rbd

rbd ls -p kube --name client.rbd


Мапим
rbd map -p kube --image disk01 --name client.rbd

mkfs.xfs /dev/rbd0

Монтируем
mkdir /test
mount /dev/rbd0 /test

Заполняем диск файлами
cp –r /etc/* /test


Теперь сделаем snaphot с названием snap01:
rbd snap create kube/disk01@snap01 --name client.rbd

rbd snap ls kube/disk01 --name client.rbd

rm -rf /test/*
ls -l /test

umount /test

Размапить
rbd unmap --image kube/disk01 --name client.rbd


Восстановить снапшот
rbd snap rollback kube/disk01@snap01 --name client.rbd

rbd map -p kube --image disk01 --name client.rbd
mount /dev/rbd0 /test

ls -l /test


```

https://russianblogs.com/article/3002981758/

https://docs.ceph.com/en/latest/rados/troubleshooting/log-and-debug/



