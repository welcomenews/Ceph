
#### CPH 09: Производительность Ceph

Для выполнения задания необходимо сделать следующие шаги:

1. Создайте пул rbd и диск на 10 гб. Установите параметр пула size 3 pg 8 и скопируйте ключ на клиент.
2. Подготовьте rbd к тестам.
3. Проведите тесты на ceph1 и запишите результаты в файл test№(1-3):
- с кластера запись --no-cleanup 10 циклов (цель: Average IOPS) - результат в /root/test1 (rados bench).
- с кластера чтение последовательное 10 циклов (цель: Average IOPS) - результат в /root/test2 (rados bench).
- с кластера чтение случайное 10 циклов (цель: Average IOPS) - результат в /root/test3 (rados bench).
4. Очистите пул от тестовых данных.
5. На клиенте проведите тест записи стандартными блоками --io-total 1719973000 (цель: ops/sec: bytes/sec:) - вывод &> в /root/test1 (rbd bench-write).
6. Смонтируйте диск в директорию /test (фс xfs), запустите тест записи dd на файл в ней bs=1G count=4 oflag=direct (цель: скорость записи в mb/sec) - вывод &> в /root/test2 (dd).
7. Прочитайте созданный файл c помощью dd bs=1G count=4 iflag=direct (цель: скорость чтения в mb/sec) - вывод &> в /root/test3 (dd).
8. На всех узлах кластера примените настройки:
- fs.aio-max-nr=1048576 (echo 1048576 > /proc/sys/fs/aio-max-nr)
- echo "8192" > /sys/block/vdb/queue/read_ahead_kb
9. На кластере выставьте пулу большее количество PG до 32. Подождите минуту и повторите все измерения, занося результаты в файлы /root/newtest1, /root/newtest2, /root/newtest3.

===

```
Сервер

Отключаем autoscale
# ceph osd pool set rbd pg_autoscale_mode off

Проверим pg
# ceph osd pool get rbd pg_num
# ceph osd pool get rbd pgp_num

Создаём пул
# ceph osd pool create rbd 8 8
Устанавливаем репликусет
# ceph osd pool set rbd size 3
# ceph osd pool application enable rbd rbd

# ceph osd pool autoscale-status

если не поменялись PG то
# ceph osd pool set rbd pg_num 8
# ceph osd pool set rbd pgp_num 8

ждём 1 мин.

Создаём диск
# rbd create disk01 --size 10G --pool rbd
# touch test1
# touch test2
# touch test3

Тесты
# rados bench -p rbd 10 write --no-cleanup > test1
# rados bench -p rbd 10 seq > test2
# rados bench -p rbd 10 rand > test3

Очистака после тестов
# rados -p rbd cleanup

Создаём минимальный конфиг
# ceph config generate-minimal-conf > ceph.conf
# scp /etc/ceph/ceph.conf client:/etc/ceph/ceph.conf

Даём минимум прав для клиента
# ceph auth get-or-create client.rbd mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=rbd'
# ceph auth get-or-create client.rbd | ssh user@client sudo tee /etc/ceph/keyring


Клиент

# sudo -i

Посмотреть пулы
# ceph osd pool ls --name client.rbd
touch test1
touch test2
touch test3

# rbd ls --name client.rbd

Тесты
# rbd bench --io-type write rbd/disk01 --io-total 1719973000 --name client.rbd > test1

Мапим диск
# rbd map --image disk01 --name client.rbd

Создать файловую систему
mkfs.xfs /dev/rbd0

Маунт
mkdir /test
mount /dev/rbd0 /test

Тест записи
dd if=/dev/zero of=/test/deleteme bs=1G count=4 oflag=direct &> test2
Тест чтения
dd if=/test/deleteme of=/dev/null bs=1G count=4 iflag=direct &> test3


Сервер

ceph osd pool set rbd pg_autoscale_mode on

На каждой ноде в кластере

echo 1048576 > /proc/sys/fs/aio-max-nr
echo "8192" > /sys/block/vdb/queue/read_ahead_kb

те же тесты, только в файлы newtest1, newtest2, newtest3


```

https://docs.ceph.com/en/mimic/cephfs/experimental-features/

https://tracker.ceph.com/projects/ceph/wiki/Benchmark_Ceph_Cluster_Performance#:~:text=Benchmark%20a%20Ceph%20Storage%20Cluster,command%20is%20included%20with%20Ceph.

https://tracker.ceph.com/projects/ceph/wiki/Tuning_for_All_Flash_Deployments


===
