
1. Отключите SeLinux.
2. Отключите сетевой экран.
3. Настройте синхронизацию времени (установить и настроить chrony).
4. Настройте файл /etc/hosts на всех машинах, добавив туда dns-имена узлов.
5. Обменяйтесь ключами SSH пользователя root (ssh-keygen и ssh-copy-id root@ceph1 && ssh-copy-id root@ceph2 && ssh-copy-id root@ceph3).
6. Установите Podman и Python.
7. Скачайте cephadm https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm
8. Поместите cephadm в системное расположение бинарей (/usr/local/bin), дайте необходимые права (+х) и с его помощью добавьте репозиторий ceph - pacific (/usr/local/bin/cephadm add-repo --release pacific)

```



```

https://docs.ceph.com/en/quincy/install/

https://habr.com/ru/post/456446/

https://www.redhat.com/files/summit/session-assets/2018/Optimize-Ceph-object-storage-for-production-in-multisite-clouds.pdf

