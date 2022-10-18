
1. Отключите SeLinux.
2. Отключите сетевой экран.
3. Настройте синхронизацию времени (установить и настроить chrony).
4. Настройте файл /etc/hosts на всех машинах, добавив туда dns-имена узлов.
5. Обменяйтесь ключами SSH пользователя root (ssh-keygen и ssh-copy-id root@ceph1 && ssh-copy-id root@ceph2 && ssh-copy-id root@ceph3).
6. Установите Podman и Python.
7. Скачайте cephadm https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm
8. Поместите cephadm в системное расположение бинарей (/usr/local/bin), дайте необходимые права (+х) и с его помощью добавьте репозиторий ceph - pacific (/usr/local/bin/cephadm add-repo --release pacific)

```
НА КАЖДОЙ НОДЕ

1.
# vi /etc/selinux/config -- Disabled
# setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

2.
# systemctl stop firewalld && systemctl disable firewalld

3.
# yum install chrony -y
# systemctl restart chronyd.service
# systemctl enable chronyd.service

4. 
# vim /etc/hosts
158.160.13.254 ceph1
158.160.3.28   ceph2
51.250.23.200  ceph3

5.
# ssh-keygen
# ssh-copy-id root@ceph1 && ssh-copy-id root@ceph2 && ssh-copy-id root@ceph3

6.
# yum groupinstall server -y     ## установить дополнительный софт (включая podman). В CentOS 8 все сопутствующие утилиты можно установить с помощью одной команды
# yum install python3 -y

7.
# curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm

8.
# mv cephadm /usr/local/bin/cephadm
# chmod +x /usr/local/bin/cephadm
# /usr/local/bin/cephadm add-repo --release pacific
# echo 'export PATH-"$PATH:/usr/local/bin/"' >> .bashrc     ## Записываем в bashrc (только для centos 8):

```

https://docs.ceph.com/en/quincy/install/

https://habr.com/ru/post/456446/

https://www.redhat.com/files/summit/session-assets/2018/Optimize-Ceph-object-storage-for-production-in-multisite-clouds.pdf

