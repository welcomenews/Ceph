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
Смотреть пул
# ceph osd pool ls


```

https://docs.ceph.com/en/quincy/install/
