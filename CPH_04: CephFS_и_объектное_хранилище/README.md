
Создайте пул для CEPHFS, выдайте FS и смонитируйте ее на клиент.

Для выполнения задания необходимо выполнить следующие шаги:

1. Создайте пул CEPHFS c именем volume1 и задеплойте MDS. (ceph fs volume create volume1 и ceph orch apply mds volume1 3)
2. Проверьте количество PG \ PGP в пуле и измените их на 32 (ceph osd pool get ... pgp_num pg_num).
3. Сгенерируйте ключ для volume1. (ceph fs authorize volume1 client.user / rw)
4. На клиенте 1 смонтируйте FS в директорию /test, предварительно создав директорию. (mount -t ceph ceph1,ceph2,ceph3:/ /test -o name=user,secret=<ключ>)
5. На клиенте 2 смонтируйте FS в директорию /test, предварительно создав директорию.
6. Создайте на клиенте 1 в директории test файл test с содержимым "This is a node 1".
7. На клиенте 2 допишите в конец файла /test/test "This is a node 2".


```

На гл.ноде
# ceph fs volume create volume1
# ceph orch apply mds volume1 3

# ceph mds stat

# ceph osd pool get cephfs.volume1.data pgp_num
# ceph osd pool get cephfs.volume1.data pg_num

# ceph fs authorize volume1 client.user / rw

Клиент1
sudo mkdir /test
sudo mount -t ceph ceph1,ceph2,ceph3:/ /test -o name=user,secret=AQDt...
sudo echo "This is a node 1" > /test/test

Клиент2
sudo mkdir /test
sudo mount -t ceph ceph1,ceph2,ceph3:/ /test -o name=user,secret=AQDt...
sudo echo "This is a node 2" >> /test/test


```

https://docs.ceph.com/en/quincy/radosgw/index.html

https://docs.openstack.org/swift/latest/s3_compat.html

https://archive.fosdem.org/2018/schedule/event/cephfs_gateways/attachments/slides/2636/export/events/attachments/cephfs_gateways/slides/2636/cephfs_samba_and_nfs.pdf



