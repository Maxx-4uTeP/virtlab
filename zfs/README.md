 # Домашняя работа 4 (ZFS)
 ## 1. Определить алгоритм с наилучшим сжатием
<details>
 <summary>Решение</summary>
 Создаем 5 файловых систем по названиям алгоритмов сжатия:

    [root@server ~]# zfs create storage/lzjb
    [root@server ~]# zfs create storage/gzip
    [root@server ~]# zfs create storage/zle
    [root@server ~]# zfs create storage/lz4
    [root@server ~]# zfs create storage/gzip-N
    [root@server ~]# zfs list
    NAME             USED  AVAIL     REFER  MOUNTPOINT
    storage          266K   832M     29.5K  /storage
    storage/gzip      24K   832M       24K  /storage/gzip
    storage/gzip-N    24K   832M       24K  /storage/gzip-N
    storage/lz4       24K   832M       24K  /storage/lz4
    storage/lzjb      24K   832M       24K  /storage/lzjb
    storage/zle       24K   832M       24K  /storage/zle
    [root@server ~]#

Применяем к каждой свой тип компрессии(в случае с gzip-N выбрал gzip-5):

    [root@server ~]# zfs set compression=lzjb /storage/lzjb
    cannot open '/storage/lzjb': leading slash in name
    [root@server ~]# zfs set compression=lzjb storage/lzjb
    [root@server ~]# zfs set compression=gzip storage/gzip
    [root@server ~]# zfs set compression=gzip-N storage/gzip-N
    cannot set property for 'storage/gzip-N': 'compression' must be one of 'on | off | lzjb | gzip | gzip-[1-9] | zle | lz4'
    [root@server ~]# zfs set compression=gzip-5 storage/gzip-N
    [root@server ~]# zfs set compression=lz4 storage/lz4
    [root@server ~]# zfs set compression=lzjb storage/lzjb
    [root@server ~]# zfs set compression=zle storage/zle
    [root@server ~]# zfs get compression
    NAME            PROPERTY     VALUE     SOURCE
    storage         compression  off       default
    storage/gzip    compression  gzip      local
    storage/gzip-N  compression  gzip-5    local
    storage/lz4     compression  lz4       local
    storage/lzjb    compression  lzjb      local
    storage/zle     compression  zle       local

Залил везде Войну и Мир:

    [root@server ~]# cp War_and_Peace.txt /storage/gzip/
    [root@server ~]# cp War_and_Peace.txt /storage/gzip-N
    [root@server ~]# cp War_and_Peace.txt /storage/lz4
    [root@server ~]# cp War_and_Peace.txt /storage/lzjb
    [root@server ~]# cp War_and_Peace.txt /storage/zle

Этого пока мало:

    [root@server ~]# zfs get compressratio
    NAME            PROPERTY       VALUE  SOURCE
    storage         compressratio  1.08x  -
    storage/gzip    compressratio  1.08x  -
    storage/gzip-N  compressratio  1.08x  -
    storage/lz4     compressratio  1.08x  -
    storage/lzjb    compressratio  1.07x  -
    storage/zle     compressratio  1.08x  -

Закачал на каждую ФС образ установки FreeBSD.

    [root@server ~]# zfs get compressratio
    NAME            PROPERTY       VALUE  SOURCE
    storage/gzip    compressratio  2.83x  -
    storage/gzip-N  compressratio  2.81x  -
    storage/lz4     compressratio  2.14x  -
    storage/lzjb    compressratio  1.93x  -
    storage/zle     compressratio  1.27x  -


ИТОГО: Gzip показало лучшую степень сжатия.
ps: попробовал менять Gzip-N на 1 или 9, отличия не обнаружил...
</details>

## 2. Определить настройки pool’a
<details>
 <summary>Решение</summary>

#### Скачали и распаковали файл zfs_task1.tar.gz

    [root@server ~]# ls -l
    total 7124
    -rw-------. 1 root root    5166 Jun 11  2020 anaconda-ks.cfg
    -rw-------. 1 root root    5006 Jun 11  2020 original-ks.cfg
    -rw-r--r--. 1 root root 7275140 May 15 07:39 zfs_task1.tar.gz
    drwxr-xr-x. 2 root root      32 May 15  2020 zpoolexport

#### смотрим статус и имя пула:
    [root@server ~]# zpool import -d ${PWD}/zpoolexport/
    pool: otus
        id: 6554193320433390805
    state: ONLINE
    action: The pool can be imported using its name or numeric identifier.
    config:

            otus                         ONLINE
            mirror-0                   ONLINE
                /root/zpoolexport/filea  ONLINE
                /root/zpoolexport/fileb  ONLINE

#### импортируем пул:

    [root@server ~]# zpool import -d ${PWD}/zpoolexport/ otus
    [root@server ~]# zpool list
    NAME      SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
    otus      480M  2.18M   478M        -         -     0%     0%  1.00x    ONLINE  -
    storage   960M   420K   960M        -         -     5%     0%  1.00x    ONLINE  -


    [root@server ~]# zfs list
    NAME             USED  AVAIL     REFER  MOUNTPOINT
    otus            2.04M   350M       24K  /otus
    otus/hometask2  1.88M   350M     1.88M  /otus/hometask2
    storage          328K   832M       24K  /storage


#### значение recordsize:

    [root@server ~]# zfs get recordsize
    NAME            PROPERTY    VALUE    SOURCE
    otus            recordsize  128K     local
    otus/hometask2  recordsize  128K     inherited from otus
    storage         recordsize  128K     default

#### какое сжатие используется:

    [root@server ~]# zfs get compression
    NAME            PROPERTY     VALUE     SOURCE
    otus            compression  zle       local
    otus/hometask2  compression  zle       inherited from otus
    storage         compression  off       default

#### тип pool:

    [root@server ~]# zpool status
    pool: otus
    state: ONLINE
    scan: none requested
    config:

            NAME                         STATE     READ WRITE CKSUM
            otus                         ONLINE       0     0     0
            mirror-0                   ONLINE       0     0     0
                /root/zpoolexport/filea  ONLINE       0     0     0
                /root/zpoolexport/fileb  ONLINE       0     0     0

    errors: No known data errors

#### какая контрольная сумма используется:

    [root@server ~]# zfs get checksum
    NAME            PROPERTY  VALUE      SOURCE
    otus            checksum  sha256     local
    otus/hometask2  checksum  sha256     inherited from otus
    storage         checksum  on         default



 </details>


 ## 3. Найти сообщение от преподавателей
 <details>
 <summary>Решение</summary>

получил файл otus_task2.file
Восстановил локально:

    [root@server ~]# zfs receive otus/storage/task2 < otus_task2.file -F
    [root@server ~]# zfs list -t snapshot
    NAME                       USED  AVAIL     REFER  MOUNTPOINT
    otus/storage/task2@task2     0B      -     2.83M  -


Ищем нужный файл и читаем содержимое:

    [root@server ~]# find /otus/storage/task2 -name secret_message
    /otus/storage/task2/task1/file_mess/secret_message
    [root@server ~]# more /otus/storage/task2/task1/file_mess/secret_message
    https://github.com/sindresorhus/awesome


 </details>
