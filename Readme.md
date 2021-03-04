# OTUS ДЗ 4. ZFS (Centos 7)
-----------------------------------------------------------------------
### Домашнее задание

    1) Определить алгоритм с наилучшим сжатием
    2) Определить настройки poola
    3) Найти сообщение от преподавателей


1. Определить алгоритм с наилучшим сжатием
- Проверяем версию установленной системы ```cat /etc/redhat-release```
- Подключаем репозиторий и ключ pgp:
```
yum install https://zfsonlinux.org/epel/zfs-release.el7_8.noarch.rpm
gpg --quiet --with-fingerprint /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
```
- Включаем KMOD в настройках и устанавливаем ZFS:
```
yum-config-manager --disable zfs
    yum-config-manager --enable zfs-kmod
    sudo yum -y install zfs
```
- Перезагружаемся ```reboot now```

- Для автоматической загрузки модуля создаем файл ```/etc/modules-load.d/zfs.conf``` и прописываем туда модуль ```zfs``` 
- Загружаем модуль ```modprobe zfs```и проверяем загрузку ZFS ```lsmod | grep zfs```
- Проверяем имеющиеся диски  и создаем pool c именем ZFSpool:
```
lsblk
zpool create -f ZFSpool mirror /dev/sdb /dev/sdc
```
- Проверяем что pool создался и есть ли ошибки ```zpool status```
- Проверяем объем и точку монтирования pool ```zfs list```
- Создаем на pool доп разделы с разным типом сжатия:
```
zfs create ZFSpool/gzip
zfs set compression=gzip-9 ZFSpool/gzip
zfs create ZFSpool/lz4
zfs set compression=lz4 ZFSpool/lz4
zfs create ZFSpool/lzjb
zfs set compression=lzjb ZFSpool/lzjb
zfs create ZFSpool/zle
zfs set compression=zle ZFSpool/zle
```
- Скачиваем файл ```wget -O War_and_Peace.txt http://www.gutenberg.org/ebooks/2600.txt.utf-8```
- Копируем этот файл во все созданные разделы ```echo "/ZFSpool/gzip/ /ZFSpool/lz4/ /ZFSpool/lzjb/ /ZFSpool/zle/" | xargs -n 1 cp -v War_and_Peace.txt```
- Проверяем степень сжатия:
    ```zfs get compression,compressratio```
```
AME          PROPERTY       VALUE     SOURCE
ZFSpool       compression    off       default
ZFSpool       compressratio  1.08x     -
ZFSpool/gzip  compression    gzip-9    local
ZFSpool/gzip  compressratio  1.08x     -
ZFSpool/lz4   compression    lz4       local
ZFSpool/lz4   compressratio  1.08x     -
ZFSpool/lzjb  compression    lzjb      local
ZFSpool/lzjb  compressratio  1.07x     -
ZFSpool/zle   compression    zle       local
ZFSpool/zle   compressratio  1.08x     -     
```
- Можно сделать вывод, что на на текстовых файлах малого объема не видно разницы в типах сжатия.
- При копировании же по папкам большего объема ```echo "/ZFSpool/gzip/ /ZFSpool/lz4/ /ZFSpool/lzjb/ /ZFSpool/zle/" | xargs -n 1 cp -R /etc/```
    ```zfs get compression,compressratio ZFSpool/gzip```
```
AME          PROPERTY       VALUE     SOURCE
ZFSpool       compression    off       default
ZFSpool       compressratio  1.70x     -
ZFSpool/gzip  compression    gzip-9    local
ZFSpool/gzip  compressratio  2.45x     -
ZFSpool/lz4   compression    lz4       local
ZFSpool/lz4   compressratio  1.78x     -
ZFSpool/lzjb  compression    lzjb      local
ZFSpool/lzjb  compressratio  1.84x     -
ZFSpool/zle   compression    zle       local
ZFSpool/zle   compressratio  1.25x     -     
```
- Можно сделать вывод что максимальное сжатие дает gzip

2. Определить настройки poola
- Скачиваем файл
    ```wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg' -O zfs_task1.tar.gz``
- Копируем в один из разделов: ```cp -v  zfs_task1.tar.gz /ZFSpool/gzip/ ```
- Подготавливаем pool для экспорта и проверяем статус pool
```
zpool export ZFSpool
zpool status
```
- импортируем pool в новое место: ```zpool import ZFSpool storage```
- проверяем импортированный pool и наличие файла:
```
 zpool status
 ls /storage/gzip
```
- Изменяем параметры файловой системы:
``` 
zfs set recordsize=1M storage/gzip #изменение размера блока 
zfs get recordsize
zfs set checksum=sha256 storage/gzip #изменение вида контрольной суммы
zfs get checksum
zfs set compression=gzip storage/gzip #изменение вида компрессии
zfs get compression
```
- Просмотр всех настроек ```zfs get all```

3. Найти сообщение от преподавателей
- Копируем файл снапшота:
```
    wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG' -O otus_task2.file
```
- восстанавливаем снапшот в нашей системе в уже готовый раздел: ```zfs  receive  -F storage < otus_task2.file```
- проверяем в разделе storage ``` ls -l /storage``` и видим что там появились дополнтельные файлы которых небыло
- Мы знаем имя искомого секретного файла, поэтому ищем: ```find /storage -iname secret_message```
```
видим 
    /storage/task1/file_mess/secret_message
```
- Смотрим секретный файл ```cat /storage/task1/file_mess/secret_message```
и видим там:
###    https://github.com/sindresorhus/awesome

