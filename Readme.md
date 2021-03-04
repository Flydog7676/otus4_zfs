# OTUS ДЗ 4. ZFS (Centos 7)
-----------------------------------------------------------------------
### Домашнее задание

    1) Определить алгоритм с наилучшим сжатием
    2) Определить настройки poola
    3) Найти сообщение от преподавателей


1. Определить алгоритм с наилучшим сжатием
- Установка ZFS
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

5. Для автоматической загрузки модуля создаем файл '''/etc/modules-load.d/zfs.conf''' и прописываем туда модуль '''zfs''' 
6. Загружаем модуль '''modprobe zfs'''и проверяем загрузку zfs '''lsmod | grep zfs'''

Создание zfs пула и разделов 
1. Проверяем имеющиеся диски  и создаем пул c именем ZFSpool
''' lsblk
    zpool create -f ZFSpool mirror /dev/sdb /dev/sdc'''

2. Проверяем статус и ошибки пула '''zpool status'''
3. Проверяем объем и точку монтирования пула '''zfs list'''
4. Создаем разделы с разным типом сжатия
'''
   zfs create ZFSpool/gzip
   zfs set compression=gzip-9 ZFSpool/gzip
   zfs create ZFSpool/lz4
   zfs set compression=lz4 ZFSpool/lz4
   zfs create ZFSpool/lzjb
   zfs set compression=lzjb ZFSpool/lzjb
   zfs create ZFSpool/zle
   zfs set compression=zle ZFSpool/zle  '''
5. Проверяем ''' zfs list -o compression '''
6. Скачиваем файл wget -O War_and_Peace.txt http://www.gutenberg.org/ebooks/2600.txt.utf-8
7. Копируем его во все созданные тома
''' echo "/ZFSpool/gzip/ /ZFSpool/lz4/ /ZFSpool/lzjb/ /ZFSpool/zle/" | xargs -n 1 cp -v War_and_Peace.txt '''
8. Проверяем степень сжатия 
'''
zfs get compression,compressratio
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
ZFSpool/zle   compressratio  1.08x     -     '''

Делаю вывод, что на на текстовых файлах малого объема не видно разницы в типах сжатия.
При копировании же ```cp -R /etc/ /ZFSpool/gzip```
zfs get compression,compressratio ZFSpool/gzip
NAME          PROPERTY       VALUE     SOURCE
ZFSpool/gzip  compression    gzip-9    local
ZFSpool/gzip  compressratio  2.38x     -

Настройки pool для переноса диска
1. Скачиваем файл
    wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg' -O zfs_task1.tar.gz
2. Подготавливаем pool для экспорта и проверяем статус pool
zpool export ZFSpool
zpool status
3.  импортируем pool в новый
zpool import ZFSpool storage
4. проверяем zpool status
5. Ставим параметры системы
    zfs set recordsize=1M storage/gzip
    zfs get recordsize

    zfs set checksum=sha256 storage/gzip
    zfs get recordsize

    zfs set compression=gzip storage/gzip
    zfs get compression

Просмотр всех настроек 
    zfs get all

восстановление Снапшота
1. Скопировать
    wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG' -O otus_task2.file
2. zfs  receive  -F storage < otus_task2.file
3. Ищем и открываем секретный файл
    find /storage -iname secret_message
видим 
    /storage/task1/file_mess/secret_message
4.  Смотрим секретный файл
    cat /storage/task1/file_mess/secret_message
там секретная ссылка:
    https://github.com/sindresorhus/awesome

