####

1. На первой машине создайте ключ (ssh-keygen) и разошлите на остальные машины (ssh-copy-id root@ceph1 && ssh-copy-id root@ceph2 && ssh-copy-id root@ceph3).
2. Поставьте ceph-common на первый узел.
3. Поставьте CEPH через cephadm bootstrap.
4. Скопируйте ключ CEPH на остальные машины.
5. Добавьте в кластер узлы и добавьте label узлов (mon, mgr).
6. Сконфигурируйте сеть кластера.
6. Запустите мониторы (mon, mgr).
7. Добавьте диски узлов в кластер.

```
На 1-м севере
# ssh-keygen
# ssh-copy-id root@ceph1 && ssh-copy-id root@ceph2 && ssh-copy-id root@ceph3
# cephadm install ceph-common

На каждом сервере которой будет включён в кластер
# if [[ ! -d "/etc/ceph" ]]; then sudo mkdir -p /etc/ceph;fi

На 1-м севере
# /usr/local/bin/cephadm bootstrap --mon-ip <IP_1-го_сервера> --initial-dashboard-user itclife  --initial-dashboard-password itclife --dashboard-password-noupdate
# ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph2
# ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph3

на 1-м севере
# ceph orch host add ceph2 <IP_1-го_сервера>
# ceph orch host add ceph3 <IP_1-го_сервера>

# ceph orch host label add ceph1 mon
# ceph orch host label add ceph2 mon
# ceph orch host label add ceph3 mon

# ceph orch host label add ceph1 mgr
# ceph orch host label add ceph2 mgr
# ceph orch host label add ceph3 mgr

# ceph config set mon public_network 10.129.0.0/24

# ceph orch apply mon "ceph1,ceph2,ceph3"
# ceph orch apply mgr "ceph1,ceph2,ceph3"

Проверяем количество хостов:
# ceph orch host ls

Командой lsblk можно посмотреть информацию о диске

# ceph orch daemon add osd ceph1:/dev/vdb
# ceph orch daemon add osd ceph2:/dev/vdb
# ceph orch daemon add osd ceph3:/dev/vdb

# ceph -s

Копируем настройки на сервера кластера:
# scp /etc/ceph/ceph.conf ceph2:/etc/ceph/ceph.conf
# scp /etc/ceph/ceph.conf ceph3:/etc/ceph/ceph.conf
# scp /etc/ceph/ceph.client.admin.keyring ceph2:/etc/ceph/ceph.client.admin.keyring
# scp /etc/ceph/ceph.client.admin.keyring ceph3:/etc/ceph/ceph.client.admin.keyring
# scp /etc/ceph/ceph.pub  ceph2:/etc/ceph/ceph.pub
# scp /etc/ceph/ceph.pub  ceph3:/etc/ceph/ceph.pub

```

https://docs.ceph.com/en/quincy/start/hardware-recommendations/

https://yourcmc.ru/wiki/Ceph_performance

