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



```

https://docs.ceph.com/en/quincy/install/
