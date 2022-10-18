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



```

https://docs.ceph.com/en/quincy/start/hardware-recommendations/

https://yourcmc.ru/wiki/Ceph_performance
