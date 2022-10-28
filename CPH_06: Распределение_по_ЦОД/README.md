
Для этого Вам предстоит выполнить следующие действия:

1. Поднимите RGW realm=ceph.local, rgw-zone=ceph.local, все остальное defaults, машины для размещения шлюзов — ceph1 и ceph2.
- создайте реалм;
- создайте одну зону;
- создайте одну группу;
- коммит изменений и разворот шлюзов;
2. Создайте пользователя test, скопируйте блок с его УЗ на клиентскую машину в файл /root/answer. Формат:
     "user": "test",
     "access_key": "84UL...",
     "secret_key": "mCglxWp..." 
3. На клиенте установите необходимый софт (awscli boto boto3 через pip install).
4. Под рутом сконфигурируйте клиент, используя полученные от кластера ключи с профилем ceph (--profile=ceph).
5. Под рутом проверьте настройки и доступ к гейтвею. Если все работает, создайте бакет test.
6. Создайте файл testfile, запишите в него строку "S3 is Working". Скопируйте файл в бакет test.

===

```
Сервер 1

создайте реалм
# radosgw-admin realm create --rgw-realm=ceph.local --default

создайте одну группу
# radosgw-admin zonegroup create --rgw-zonegroup=default  --master --default

создайте одну зону
# radosgw-admin zone create --rgw-zonegroup=default --rgw-zone=ceph.local --master --default

коммит изменений и развёртывание шлюзов
# radosgw-admin period update --rgw-realm=ceph.local --commit

# ceph orch apply rgw ceph --realm=ceph.local --zone=ceph.local --placement="2 ceph1 ceph2"

Делаем пользователя
# radosgw-admin user create --display-name="test admin" --uid=test


"user": "test",
     "access_key": "Z9M8...",
     "secret_key": "OHEZ..." 

Их копируем в отдельный файл /root/answer


Клиент

sudo apt update
sudo apt install python3-pip

# pip3 install awscli boto boto3

# sudo aws configure --profile=ceph

AWS Access Key ID
AWS Secret Access Key
Default region name [None]: _(не нужно заполнять)_
Default output format [None]: json

# cat ~/.aws/credentials

Создаем bucket
# aws --profile=ceph --endpoint=http://ceph1 s3api create-bucket --bucket test

Проверяем, создан ли бакет:
# aws --profile=ceph --endpoint=http://ceph1 s3 ls

Теперь давайте создадим файл testfile
# echo "S3 is Working" > testfile

Копия файла
# aws --profile=ceph --endpoint=http://ceph1 s3 cp testfile s3://test/

Проверяем, скопировался ли файл:
# aws --profile=ceph --endpoint=http://ceph1 s3 ls test

```


