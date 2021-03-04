# OTUS ДЗ 4. ZFS (Centos 7)
-----------------------------------------------------------------------
### Домашнее задание

    1) Определить алгоритм с наилучшим сжатием
    2) Определить настройки pool’a
    3) Найти сообщение от преподавателей


Установка zfs
1. Проверяем версию установленной системы ```cat /etc/redhat-release```
2. Подключаем репозиторий и ключ pgp 
    ```yum install https://zfsonlinux.org/epel/zfs-release.el7_8.noarch.rpm
    gpg --quiet --with-fingerprint /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux```
3. Вклчаем KMOD в настройках и устанавливаем zfs
''' sudo yum-config-manager --disable zfs
    sudo yum-config-manager --enable zfs-kmod
    sudo yum -y install zfs '''

4. Перезагружаемся '''reboot now'''

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
7. Копируем его на все созданные тома
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




[root@lvm ~]# df -h
Filesystem                          Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00     8.0G  778M  7.3G  10% /
devtmpfs                            110M     0  110M   0% /dev
tmpfs                               118M     0  118M   0% /dev/shm
tmpfs                               118M  4.6M  114M   4% /run
tmpfs                               118M     0  118M   0% /sys/fs/cgroup
/dev/sda2                          1014M   61M  954M   6% /boot
/dev/mapper/VolGroup00-LogVol_Home  2.0G   33M  2.0G   2% /home
/dev/mapper/vg_var-lv_var           922M  143M  716M  17% /var
tmpfs                                24M     0   24M   0% /run/user/1000
ZFSpool                             880M     0  880M   0% /ZFSpool
tmpfs                                24M     0   24M   0% /run/user/0

Далее можно переходить в директорию cd /ZFSpool и можем создавать там файлы.

Еще мы можем создавать снэпшоты.

11. zfs snapshot /ZFSpool@friday - Создается снэпшот /ZFSpool с именем friday.

12. [root@lvm ~]# zfs list -t snapshot
NAME             USED  AVAIL  REFER  MOUNTPOINT
ZFSpool@friday     0B      -    24K  -

Видим наши снапшоты. Их всего 1.

Снимки файловых систем будут доступны в каталоге .zfs/snapshot в корне файловой системы. 
Но чтобы увидеть эту папку нужно выполнить команду: zfs set snapdir=visible ZFSpool (ZFSpool это имя нашего пула)

Далее переходим в /ZFSpool/.zfs/ и видим папку для снэпшотов:
[root@lvm ~]# ls -lah /ZFSpool/.zfs/
total 512
drwxrwxrwx 1 root root 0 Aug 12 14:43 .
drwxr-xr-x 2 root root 2 Aug 12 14:43 ..
drwxrwxrwx 2 root root 2 Aug 12 15:21 shares
drwxrwxrwx 2 root root 2 Aug 12 15:12 snapshot

13. zfs set mountpoint=/opt ZFSpool - Монтируем наш пул в каталог /opt.
[root@lvm ~]# zfs list
NAME      USED  AVAIL  REFER  MOUNTPOINT
ZFSpool   100K   880M    24K  /opt

14. zfs destroy ZFSpool@friday - Удалить снэпшот мы можем этой командой
