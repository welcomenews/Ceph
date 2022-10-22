Создайте пул для RBD, выдайте диск и смонитируйте его на клиент. Для этого:

1. Создайте пул rbd (размер 32 pg, фактор репликации по умолчанию).
2. Создайте диск (lun rbd) с именем disk01 размером 2GB.
3. Сгенерируйте минимальный конфиг и пользовательский ключ, затем скопируйте ключ на клиент.
4. На клиенте смапьте диск (rbd map).
5. Подготовьте диск через LVM (vg=test lv=test size=1G).
6. Смонтируйте диск в директорию /test как xfs, предварительно создав директорию.
7. Создайте на диске в директории test файл test с содержимым "This is an RBD - it's working"


```
Создаём пул rbd
# ceph osd pool create rbd 32 32

# ceph osd pool ls

# ceph osd pool application enable rbd rbd

Инициализируем пул rbd
# rbd pool init -p rbd

Создаем диск (lun) на 2 гигабайт
# rbd create disk01 --size 2G --pool rbd

# rbd --image disk01 -p rbd info

Генерируем минимальный конфиг для клиента
# ceph config generate-minimal-conf > ceph.conf

Копируем на клиента
# scp /etc/ceph/ceph.conf client:/etc/ceph/ceph.conf

Генерируем ключ. Он будет ограниченным, чтобы клиент ходил только на пул RBD.
# ceph auth get-or-create client.rbd mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=rbd'

# ceph auth get-or-create client.rbd | ssh user@client sudo tee /etc/ceph/keyring


На client-е

Смотрим состояние кластера
ceph -s --name client.rbd

Смотреть диск непосредственно с клиента
rbd ls --name client.rbd

$ lsblk

Так можно примапить диск на машину
$ sudo rbd map --image disk01 --name client.rbd

ДЛя настройки LVM разделов
$ sudo apt install lvm2

ИНИЦИАЛИЗАЦИЯ ФИЗИЧЕСКИХ LVM РАЗДЕЛОВ для работы LVM
$ sudo pvcreate /dev/rbd0

Чтобы посмотреть действительно ли были созданы физические тома LVM
$ sudo pvscan

Также можно посмотреть физические LVM разделы с более подробными атрибутами
$ sudo pvdisplay

После того как физические разделы инициализированы, вы можете создать из них группу томов (Volume Group, VG)
$ sudo vgcreate test /dev/rbd0

Смотреть созданные группы томов
$ sudo vgdisplay

Создать логические LVM разделы
В качестве приставки для указания размера можно использовать такие буквы:
B - байты;
K - килобайты;
M - мегабайты;
G - гигабайты;
T - терабайты.

$ sudo lvcreate -L 1G -n test test

Посмотреть список доступных логических разделов LVM
$ sudo lvdisplay

отформатируем его в файловую систему ext4, а затем примонтируем в /mnt:
$ sudo mkfs.xfs /dev/test/test

$ sudo mkdir /test
$ sudo mount /dev/test/test /test

удалить LVM раздел
$ sudo lvremove /dev/test/test

```

https://losst.pro/sozdanie-i-nastrojka-lvm-linux

https://docs.ceph.com/en/quincy/radosgw/index.html

https://archive.fosdem.org/2018/schedule/event/cephfs_gateways/attachments/slides/2636/export/events/attachments/cephfs_gateways/slides/2636/cephfs_samba_and_nfs.pdf

https://docs.openstack.org/swift/latest/s3_compat.html

