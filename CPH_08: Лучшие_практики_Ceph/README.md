
Вам дан кластер с 120 OSD. В кластере создается один пул для RDB с названием rbd и один пул для CEPHFS с названием cephfs. 
Надо посчитать, сколько PG нужно этим пулам при равноправном распределении, если фактор репликации каждого пула равен 3. 
PG расчитываются только для пулов с данными. Пулы с метасодержимым, технические пулы, в виду их незначительного размера используют минимальное число PG. 
В ответ в файл /home/user/answer добавьте команды создания данных пулов (создание пула с полученным значением pg, установление для пула фактора репликации, равного 3). 
Подсказка: искомое значение pg, общее для двух пулов, будет равно ближайшей степени 2ки от полученного в расчетах значения.


ceph osd pool create rbd 2048 2048

ceph osd pool set rbd size 3

ceph osd pool application enable rbd rbd

ceph osd pool create cephfs_data 2048 2048

ceph osd pool create cephfs_metadata 32 32

ceph osd pool set cephfs_data size 3

ceph osd pool set cephfs_metadata size 3

=== 

https://documentation.suse.com/sbp/all/html/SBP-rook-ceph-kubernetes/index.html
