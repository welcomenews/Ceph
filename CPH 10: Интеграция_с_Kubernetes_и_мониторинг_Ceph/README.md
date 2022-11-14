#### CPH 10: Интеграция с Kubernetes и мониторинг Ceph


Создайте пул на кластере ceph. Ключ доступа интегрируйте k8s c ceph - выдайте диск тестовому приложению.

1. Создайте пул rbd с именем kube и ключ для пользователя kube. на kubernetes.
2. Установите helm репо ceph-csi (helm repo add ceph-csi https://ceph.github.io/csi-charts).
3. На машине k3s поправьте файл в папке rbd/rbd.yml блок. csiConfig:
- clusterID: "6c1cc102-e97c-11ec-930c-080027c10f66" monitors:
- "v2:<локальный ip-адрес 1 машины кластера>:3300/0,v1:<локальный ip-адрес 1 машины кластера>:6789/0"
- "v2:<локальный ip-адрес 2 машины кластера>:3300/0,v1:<локальный ip-адрес 2 машины кластера>:6789/0"
- "v2:<локальный ip-адрес 3 машины кластера>:3300/0,v1:<локальный ip-адрес 3 машины кластера>:6789/0"
> Информация о вашем кластере содержится в файле /etc/ceph/ceph.conf на первой ноде кластера.
4. На машине k3s с помощью файла rbd.yaml и helm chart ceph-csi/ceph-csi-rbd установите компоненты ceph в k3s с созданием неймспейса. helm upgrade -i ceph-csi-rbd ceph-csi/ceph-csi-rbd -f rbd/rbd.yml -n ceph-csi-rbd --create-namespace
5. На машине k3s в файл в папке rbd/secret.yaml запишите полученный от ceph ключ для пользователя kube и примените его, создавая секрет. kubectl apply -f rbd/secret.yaml
6. На машине k3s в файл в папке rbd/storageclass.yaml поменяйте clusterID: 6c1cc102-e97c-11ec-930c-080027c10f66 на свой и примените его для создания storage class. kubectl apply -f rbd/storageclass.yaml
7. На машине k3s cоздайте namespace apps. kubectl create namespace apps
8. На машине k3s c помощью файла rbd/pvc.yaml создайте диск в неймспейсе apps. kubectl apply -f rbd/pvc.yaml -n apps

==========================

```
-->  Переходим в Ceph

создаем пул Kubernetes:
ceph osd pool create kube 32 32
активируем
ceph osd pool application enable kube rbd

Далее необходимо сделать пользователя kube:
ceph auth get-or-create client.kube mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=kube'
ceph auth get-key client.kube

<--


--> на сервере

apt install ceph-common

kubectl get all -A

Установите helm репо ceph-csi
helm repo add ceph-csi https://ceph.github.io/csi-charts

на машине k3s поправьте файл в папке rbd/rbd.yml блок. csiConfig
тут редактируем  clusterID: ..... и несколько значений меняем на true, а именно podSecurityPolicy на nodeplugin и на provisioner.
Информация о вашем кластере содержится в файле /etc/ceph/ceph.conf на первой ноде кластера.
93799bf4-6284-11ed-9e10-d00dd34cd9ff

helm upgrade -i ceph-csi-rbd ceph-csi/ceph-csi-rbd -f rbd/rbd.yml -n ceph-csi-rbd --create-namespace

Дальше правим “secret.yaml” (ключи должны быть те, что получили на Ceph):
vim secret.yaml
kubectl apply -f rbd/secret.yaml

Устанавливаем storageclass:
правим тут   clusterID:
kubectl apply -f rbd/storageclass.yaml

kubectl create namespace apps
kubectl apply -f rbd/pvc.yaml -n apps

kubectl get pv -A
kubectl get pvc -A

<--


-->  Переходим в Ceph

посмотреть на Ceph, что создался rbd
rbd ls -p kube

Подробная информация о созданом rbd
rbd -p kube info <имя_которое_отобразилось_в_rbd_ls>

<--


--> на сервере

kubectl apply -f carolyne/Deployment.yaml
kubectl get all -A
Создадим для нее сервис:
kubectl apply -f carolyne/Service.yaml
И создадим ингресс:
kubectl apply -f carolyne/Ingress.yaml

kubectl get pod -n apps
kubectl exec -it --namespace=apps carolyne-544f6b5bbd-prz9b -- sh

cd /disk
dd if=/dev/zero of=daygeek2.txt bs=500M count=1

ceph -s

<--

```



https://habr.com/ru/company/southbridge/blog/519130/

