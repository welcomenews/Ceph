
1. Создайте пул rbd c именем DISK1 и в нем lun 5gb.
2. Разверните iscsi gateway (УЗ admin01 passw01 размещение ceph1 ceph2 добавить в доверенные ip и прописать их адреса).
3. На первом узле вычислите контейнер iscsi gateway и войдите в него через cephadm enter.
4. Используя gwcli, создайте таргет iqn.2022-01.local.ceph.iscsi-gw:iscsi-igw, после чего зацепите шлюзы подключения за ранее созданный диск
5. Не выходя из gwcli, создайте клиента с уз (username=iscsiadmin password=iscsipassword) и дайте ему диск.
6. На клиенте поставьте ПО icsci. Настройте клиент на подключение.
7. Настройте мультипас драйвер. Правим /etc/multipath.conf
```
   devices {
         device {
                 vendor                 "LIO-ORG"
                 product                "TCMU device"
                 hardware_handler       "1 alua"
                 path_grouping_policy   "failover"
                 path_selector          "queue-length 0"
                 failback               60
                 path_checker           tur
                 prio                   alua
                 prio_args              exclusive_pref_bit
                 fast_io_fail_tmo       25
                 no_path_retry          queue
         }
   }
```
Запустите мультипас и демона icsci.

8. Отсканируйте и примапьте Lun через iscsi.
9. Создайте директорию /test и смонтируйте в нее диск.




```
На сервере 1

Создаем пул с 32 pg:
# ceph osd pool create iscsi 32 32

Создаем application c типом RBD и диск с именем “DISK1”:
# ceph osd pool application enable iscsi rbd
# rbd --pool iscsi create DISK1 --size=5120

Разворачиваем iscsi gateway: в trusted_ip_list указываем IP кластерных хостов, в расположении — ноды 1 и 2:
# ceph orch apply iscsi iscsi admin01 passw01 --placement ceph1,ceph2 --trusted_ip_list _10.129.0.20,10.129.0.27,10.129.0.35_

воспользоваться JQ и отфильтровать вывод имя контейнера
yum install jq
# cephadm ls --no-detail | jq '.[].name'

Через ‘Cephadm shell’ можно войти в него
# cephadm enter -n iscsi.iscsi.ceph2.fnelz

Запустим оболочку управления iscsi:
# gwcli

Так мы создаем “iqn”:
/> cd iscsi-targets
 > create iqn.2022-01.local.ceph.iscsi-qw:iscsi-iqw

Теперь создадим ему два шлюза:
> cd iqn.2022-01.local.ceph.iscsi-qw:iscsi-iqw/gateways
> create ceph2.local _10.129.0.27_ skipchecks=true
> create ceph3.local _10.129.0.35_ skipchecks=true

Теперь мы идем в диски и приаттачиваем ранее созданный диск:
> cd /disks
> attach iscsi/DISK1

Создаем клиента (инициатор):

> cd /iscsi-targets/iqn.2022-01.local.ceph.iscsi-qw:iscsi-iqw/hosts
> create iqn.2022-01.ceph.iscsi:iscsi-client

зададим ему login и password (для аутентификации):

> auth username=iscsiadmin password=iscsipassword

Объявляем диск для клиента:

> disk add iscsi/DISK1

> exit

На клиене

apt install open-iscsi multipath-tools

Идем в /etc/iscsi/iscsid.conf, ставим логин и пароль, который установили, а в /etc/iscsi/initiatorname.iscsi напишем имя инициатора iqn.2022-01.ceph.iscsi:iscsi-client

vim /etc/iscsi/iscsid.conf

node.session.auth.username = iscsiadmin
node.session.auth.password = iscsipassword

vim /etc/multipath.conf

   devices {
         device {
                 vendor                 "LIO-ORG"
                 product                "TCMU device"
                 hardware_handler       "1 alua"
                 path_grouping_policy   "failover"
                 path_selector          "queue-length 0"
                 failback               60
                 path_checker           tur
                 prio                   alua
                 prio_args              exclusive_pref_bit
                 fast_io_fail_tmo       25
                 no_path_retry          queue
         }
   }


Запускаем дискавери:

iscsiadm -m discovery -t st -p 192.168.122.101

Логинимся:

iscsiadm -m node -T iscsiadm -m node -T iqn.2022-01.local.ceph.iscsi-qw:iscsi-iqw -l

Проверяем:

multipath -l

lsblk

Далее монтируем полученный диск в директорию /test.

sudo mkfs.xfs /dev/dm-0

sudo mount /dev/dm-0 /test/

sudo mkdir /test



```

https://sidmid.ru/ceph-полезные-команды/

https://docs.ceph.com/en/quincy/rbd/rbd-mirroring/

https://docs.ceph.com/en/latest/dev/cephfs-mirroring/

https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/4/html/block_device_guide/the-ceph-iscsi-gateway



