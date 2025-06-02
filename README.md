# ProfHW5
Домашнее задание №5. Практические навыки работы с ZFS.  
<b>1. Определить алгоритм с наилучшим сжатием:</b>  
Устанвливаем ZFS.
```
root@UbuntuTestVirt:~# apt install zfsutils-linux
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
Уже установлен пакет zfsutils-linux самой новой версии (2.2.2-0ubuntu9.2).
Обновлено 0 пакетов, установлено 0 новых пакетов, для удаления отмечено 0 пакетов, и 6 пакетов не обновлено.
```
---
Создаём zpool
```
root@UbuntuTestVirt:~#  zpool create mypool mirror /dev/sdb /dev/sdc
root@UbuntuTestVirt:~# zpool status
  pool: mypool
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors
```
---
Создаём чтерыре файловые системы
```
root@UbuntuTestVirt:~# zfs create mypool/zfs1
root@UbuntuTestVirt:~# zfs create mypool/zfs2
root@UbuntuTestVirt:~# zfs create mypool/zfs3
root@UbuntuTestVirt:~# zfs create mypool/zfs4
```
Устанавливаем разные варианты компрессии
```
root@UbuntuTestVirt:~# zfs get compress
NAME         PROPERTY     VALUE           SOURCE
mypool       compression  on              default
mypool/zfs1  compression  on              default
mypool/zfs2  compression  on              default
mypool/zfs3  compression  on              default
mypool/zfs4  compression  on              default
root@UbuntuTestVirt:~# zfs set compress=gzip /mypool/zfs1
cannot open '/mypool/zfs1': leading slash in name
root@UbuntuTestVirt:~# zfs set compress=gzip mypool/zfs1
root@UbuntuTestVirt:~# zfs set compress=zle mypool/zfs2
root@UbuntuTestVirt:~# zfs set compress=lzjb mypool/zfs3
root@UbuntuTestVirt:~# zfs set compress=lz4 mypool/zfs4
root@UbuntuTestVirt:~# zfs get compress
NAME         PROPERTY     VALUE           SOURCE
mypool       compression  on              default
mypool/zfs1  compression  gzip            local
mypool/zfs2  compression  zle             local
mypool/zfs3  compression  lzjb            local
mypool/zfs4  compression  lz4             local
```
---
Проверяем эффективность сжатия, копируя директорию /var/log во все файловые системы
```
root@UbuntuTestVirt:~# cp -r /var/log/* /mypool/zfs1
root@UbuntuTestVirt:~# cp -r /var/log/* /mypool/zfs2
root@UbuntuTestVirt:~# cp -r /var/log/* /mypool/zfs3
root@UbuntuTestVirt:~# cp -r /var/log/* /mypool/zfs4
root@UbuntuTestVirt:~# df -hT
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  197M  1,2M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4   9,8G  5,7G  3,6G  62% /
tmpfs                             tmpfs  984M     0  984M   0% /dev/shm
tmpfs                             tmpfs  5,0M     0  5,0M   0% /run/lock
/dev/sda2                         ext4   1,7G  189M  1,4G  12% /boot
tmpfs                             tmpfs  197M   12K  197M   1% /run/user/1000
/dev/mapper/vg--hw-hw1            ext4   1,4G  442M  896M  34% /mnt/01
/dev/mapper/vg--hw-hw2            ext4   553M  244M  282M  47% /mnt/02
/dev/mapper/vg--hw-hw3            ext4   553M  244M  282M  47% /mnt/03
mypool                            zfs    772M  128K  771M   1% /mypool
mypool/zfs1                       zfs    779M  8,0M  771M   2% /mypool/zfs1
mypool/zfs2                       zfs    792M   21M  771M   3% /mypool/zfs2
mypool/zfs3                       zfs    792M   21M  771M   3% /mypool/zfs3
mypool/zfs4                       zfs    783M   12M  771M   2% /mypool/zfs4
```

Наиболее эффективное сжатие текста у алгоритма gzip

---
<b>2. Импортировать zpool. Определить настройки пула.</b>  
Копируем архив с pool из инета и разархивируем его
```
root@UbuntuTestVirt:~# zpool list
NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
mypool   960M  60.9M   899M        -         -     0%     6%  1.00x    ONLINE  -
root@UbuntuTestVirt:~# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
--2025-06-02 21:30:00--  https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 216.58.212.161, 2a00:1450:4001:802::2001
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|216.58.212.161|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7275140 (6,9M) [application/octet-stream]
Saving to: ‘archive.tar.gz’

archive.tar.gz                             100%[=======================================================================================>]   6,94M   865KB/s    in 9,0s

2025-06-02 21:30:18 (790 KB/s) - ‘archive.tar.gz’ saved [7275140/7275140]
root@UbuntuTestVirt:~# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
root@UbuntuTestVirt:~# ll
total 7164
drwx------  5 root root    4096 июн  2 21:30 ./
drwxr-xr-x 25 root root    4096 июн  1 23:35 ../
-rw-r--r--  1 root root 7275140 дек  6  2023 archive.tar.gz
-rw-------  1 root root    4749 мая 18 00:14 .bash_history
-rw-r--r--  1 root root    3106 апр 22  2024 .bashrc
-rw-------  1 root root     212 мая 30 00:57 .lesshst
drwxr-xr-x  3 root root    4096 мар 14 01:48 .local/
-rw-------  1 root root      15 мая 26 00:46 .lvm_history
-rw-r--r--  1 root root     161 апр 22  2024 .profile
-rw-r--r--  1 root root     119 мая 16 17:13 raid_create.sh
-rw-r--r--  1 root root     141 мая 18 00:14 raid_create.sh.save
-rw-r--r--  1 root root      66 мар 25 23:58 .selected_editor
drwx------  2 root root    4096 ноя  3  2024 .ssh/
drwxr-xr-x  2 root root    4096 мая 15  2020 zpoolexport/
```
---
Проверяем возможность импорта и импортируем pool
```
root@UbuntuTestVirt:~# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
status: Some supported features are not enabled on the pool.
 (Note that they may be intentionally disabled if the
 'compatibility' property is set.)
 action: The pool can be imported using its name or numeric identifier, though
 some features will not be available without an explicit 'zpool upgrade'.
 config:

 otus                         ONLINE
   mirror-0                   ONLINE
     /root/zpoolexport/filea  ONLINE
     /root/zpoolexport/fileb  ONLINE
root@UbuntuTestVirt:~# zpool import -d zpoolexport/ otus
root@UbuntuTestVirt:~# zpool status
  pool: mypool
 state: ONLINE
config:

 NAME        STATE     READ WRITE CKSUM
 mypool      ONLINE       0     0     0
   mirror-0  ONLINE       0     0     0
     sdb     ONLINE       0     0     0
     sdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
 The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
 the pool may no longer be accessible by software that does not support
 the features. See zpool-features(7) for details.
config:

 NAME                         STATE     READ WRITE CKSUM
 otus                         ONLINE       0     0     0
   mirror-0                   ONLINE       0     0     0
     /root/zpoolexport/filea  ONLINE       0     0     0
     /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```
---
Определяем настройки пула (all, размер, тип пула (возможность записи), значение recordsize, тип сжатия)
```
root@UbuntuTestVirt:~# zpool get all otus
NAME  PROPERTY                       VALUE                          SOURCE
otus  size                           480M                           -
otus  capacity                       0%                             -
otus  altroot                        -                              default
otus  health                         ONLINE                         -
otus  guid                           6554193320433390805            -
otus  version                        -                              default
otus  bootfs                         -                              default
otus  delegation                     on                             default
otus  autoreplace                    off                            default
otus  cachefile                      -                              default
otus  failmode                       wait                           default
otus  listsnapshots                  off                            default
otus  autoexpand                     off                            default
otus  dedupratio                     1.00x                          -
otus  free                           478M                           -
otus  allocated                      2.09M                          -
otus  readonly                       off                            -
otus  ashift                         0                              default
otus  comment                        -                              default
otus  expandsize                     -                              -
otus  freeing                        0                              -
otus  fragmentation                  0%                             -
otus  leaked                         0                              -
otus  multihost                      off                            default
otus  checkpoint                     -                              -
otus  load_guid                      2212312092743733557            -
otus  autotrim                       off                            default
otus  compatibility                  off                            default
otus  bcloneused                     0                              -
otus  bclonesaved                    0                              -
otus  bcloneratio                    1.00x                          -
otus  feature@async_destroy          enabled                        local
otus  feature@empty_bpobj            active                         local
otus  feature@lz4_compress           active                         local
otus  feature@multi_vdev_crash_dump  enabled                        local
otus  feature@spacemap_histogram     active                         local
otus  feature@enabled_txg            active                         local
otus  feature@hole_birth             active                         local
otus  feature@extensible_dataset     active                         local
otus  feature@embedded_data          active                         local
otus  feature@bookmarks              enabled                        local
otus  feature@filesystem_limits      enabled                        local
otus  feature@large_blocks           enabled                        local
otus  feature@large_dnode            enabled                        local
otus  feature@sha512                 enabled                        local
otus  feature@skein                  enabled                        local
otus  feature@edonr                  enabled                        local
otus  feature@userobj_accounting     active                         local
otus  feature@encryption             enabled                        local
otus  feature@project_quota          active                         local
otus  feature@device_removal         enabled                        local
otus  feature@obsolete_counts        enabled                        local
otus  feature@zpool_checkpoint       enabled                        local
otus  feature@spacemap_v2            active                         local
otus  feature@allocation_classes     enabled                        local
otus  feature@resilver_defer         enabled                        local
otus  feature@bookmark_v2            enabled                        local
otus  feature@redaction_bookmarks    disabled                       local
otus  feature@redacted_datasets      disabled                       local
otus  feature@bookmark_written       disabled                       local
otus  feature@log_spacemap           disabled                       local
otus  feature@livelist               disabled                       local
otus  feature@device_rebuild         disabled                       local
otus  feature@zstd_compress          disabled                       local
otus  feature@draid                  disabled                       local
otus  feature@zilsaxattr             disabled                       local
otus  feature@head_errlog            disabled                       local
otus  feature@blake3                 disabled                       local
otus  feature@block_cloning          disabled                       local
otus  feature@vdev_zaps_v2           disabled                       local
root@UbuntuTestVirt:~# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
root@UbuntuTestVirt:~# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
root@UbuntuTestVirt:~# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
root@UbuntuTestVirt:~# zfs get compression otus
NAME  PROPERTY     VALUE           SOURCE
otus  compression  zle             local
```
---
<b>Работа со снэпшотами</b>  
Копируем файл otus_task2.file из инета
```
root@UbuntuTestVirt:~# wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
[1] 29228
root@UbuntuTestVirt:~# ll
total 12476
drwx------  5 root root    4096 июн  2 22:18 ./
drwxr-xr-x 26 root root    4096 июн  2 21:50 ../
-rw-r--r--  1 root root 7275140 дек  6  2023 archive.tar.gz
-rw-------  1 root root    4749 мая 18 00:14 .bash_history
-rw-r--r--  1 root root    3106 апр 22  2024 .bashrc
-rw-------  1 root root     212 мая 30 00:57 .lesshst
drwxr-xr-x  3 root root    4096 мар 14 01:48 .local/
-rw-------  1 root root      15 мая 26 00:46 .lvm_history
-rw-r--r--  1 root root 5432736 дек  6  2023 otus_task2.file
-rw-r--r--  1 root root     161 апр 22  2024 .profile
-rw-r--r--  1 root root     119 мая 16 17:13 raid_create.sh
-rw-r--r--  1 root root     141 мая 18 00:14 raid_create.sh.save
-rw-r--r--  1 root root      66 мар 25 23:58 .selected_editor
drwx------  2 root root    4096 ноя  3  2024 .ssh/
-rw-r--r--  1 root root    1570 июн  2 22:18 wget-log
drwxr-xr-x  2 root root    4096 мая 15  2020 zpoolexport/
```
---
Восстанавливает файловую сиситему из снэпшота otus_task2.file
```
root@UbuntuTestVirt:~# zfs list
NAME             USED  AVAIL  REFER  MOUNTPOINT
mypool          61.0M   771M    24K  /mypool
mypool/zfs1     7.96M   771M  7.96M  /mypool/zfs1
mypool/zfs2     21.0M   771M  21.0M  /mypool/zfs2
mypool/zfs3     20.1M   771M  20.1M  /mypool/zfs3
mypool/zfs4     11.7M   771M  11.7M  /mypool/zfs4
otus            2.06M   350M    24K  /otus
otus/hometask2  1.88M   350M  1.88M  /otus/hometask2
root@UbuntuTestVirt:~# zfs receive otus/test@now < otus_task2.file
root@UbuntuTestVirt:~# zfs list
NAME             USED  AVAIL  REFER  MOUNTPOINT
mypool          61.0M   771M    24K  /mypool
mypool/zfs1     7.96M   771M  7.96M  /mypool/zfs1
mypool/zfs2     21.0M   771M  21.0M  /mypool/zfs2
mypool/zfs3     20.1M   771M  20.1M  /mypool/zfs3
mypool/zfs4     11.7M   771M  11.7M  /mypool/zfs4
otus            4.92M   347M    24K  /otus
otus/hometask2  1.88M   347M  1.88M  /otus/hometask2
otus/test       2.83M   347M  2.83M  /otus/test
```
---
Ищем что-то секретное и читаем его содержимое
```
root@UbuntuTestVirt:~# find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message
root@UbuntuTestVirt:~# cat /otus/test/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
```
---
Находим рекламу курса otus :)

![Screen](./Screenshot_1.png)

---
Для усвоения материала. Создание snapshot. Восстановление из snapshot.
```
root@UbuntuTestVirt:~# zpool list
NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
mypool   960M  60.9M   899M        -         -     0%     6%  1.00x    ONLINE  -
otus     480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -
root@UbuntuTestVirt:~# ll /mypool/zfs1
total 560
drwxr-xr-x 10 root root     50 июн  1 22:34 ./
drwxr-xr-x  6 root root      6 июн  1 22:19 ../
-rw-r--r--  1 root root      0 июн  1 22:34 alternatives.log
-rw-r--r--  1 root root   8879 июн  1 22:34 alternatives.log.1
-rw-r--r--  1 root root   2519 июн  1 22:34 alternatives.log.2.gz
-rw-r-----  1 root root      0 июн  1 22:34 apport.log
drwxr-xr-x  2 root root     13 июн  1 22:34 apt/
-rw-r-----  1 root root   1250 июн  1 22:34 auth.log
-rw-r-----  1 root root  38839 июн  1 22:34 auth.log.1
-rw-r-----  1 root root   4424 июн  1 22:34 auth.log.2.gz
-rw-r-----  1 root root   2494 июн  1 22:34 auth.log.3.gz
-rw-r-----  1 root root   1090 июн  1 22:34 auth.log.4.gz
-rw-r--r--  1 root root  61229 июн  1 22:34 bootstrap.log
-rw-r-----  1 root root      0 июн  1 22:34 btmp
-rw-r-----  1 root root      0 июн  1 22:34 btmp.1
-rw-r-----  1 root root  85170 июн  1 22:34 cloud-init.log
-rw-r-----  1 root root   4592 июн  1 22:34 cloud-init-output.log
drwxr-xr-x  2 root root      2 июн  1 22:34 dist-upgrade/
-rw-r-----  1 root root  57444 июн  1 22:34 dmesg
-rw-r-----  1 root root  58680 июн  1 22:34 dmesg.0
-rw-r-----  1 root root  17266 июн  1 22:34 dmesg.1.gz
-rw-r-----  1 root root  17053 июн  1 22:34 dmesg.2.gz
-rw-r-----  1 root root  16990 июн  1 22:34 dmesg.3.gz
-rw-r-----  1 root root  17336 июн  1 22:34 dmesg.4.gz
-rw-r--r--  1 root root      0 июн  1 22:34 dpkg.log
-rw-r--r--  1 root root  45043 июн  1 22:34 dpkg.log.1
-rw-r--r--  1 root root   4443 июн  1 22:34 dpkg.log.2.gz
-rw-r--r--  1 root root   1642 июн  1 22:34 dpkg.log.3.gz
-rw-r--r--  1 root root  67810 июн  1 22:34 dpkg.log.4.gz
-rw-r--r--  1 root root      0 июн  1 22:34 faillog
drwxr-x---  4 root root     24 июн  1 22:34 installer/
drwxr-xr-x  3 root root      3 июн  1 22:34 journal/
-rw-r-----  1 root root    392 июн  1 22:34 kern.log
-rw-r-----  1 root root  19196 июн  1 22:34 kern.log.1
-rw-r-----  1 root root  17327 июн  1 22:34 kern.log.2.gz
-rw-r-----  1 root root  58950 июн  1 22:34 kern.log.3.gz
-rw-r-----  1 root root  14708 июн  1 22:34 kern.log.4.gz
drwxr-xr-x  2 root root      3 июн  1 22:34 landscape/
-rw-r--r--  1 root root 292292 июн  1 22:34 lastlog
drwx------  2 root root      2 июн  1 22:34 private/
lrwxrwxrwx  1 root root     39 июн  1 22:34 README -> ../../usr/share/doc/systemd/README.logs
-rw-r-----  1 root root   8313 июн  1 22:34 syslog
-rw-r-----  1 root root 166972 июн  1 22:34 syslog.1
-rw-r-----  1 root root  42748 июн  1 22:34 syslog.2.gz
-rw-r-----  1 root root 116554 июн  1 22:34 syslog.3.gz
-rw-r-----  1 root root  32725 июн  1 22:34 syslog.4.gz
drwxr-xr-x  2 root root     19 июн  1 22:34 sysstat/
-rw-r--r--  1 root root      0 июн  1 22:34 ubuntu-advantage-apt-hook.log
drwxr-x---  2 root root     13 июн  1 22:34 unattended-upgrades/
-rw-r--r--  1 root root  67584 июн  1 22:34 wtmp
root@UbuntuTestVirt:~# zfs snapshot mypool/zfs1@20250602
root@UbuntuTestVirt:~# zfs list -t snapshot
NAME                   USED  AVAIL  REFER  MOUNTPOINT
mypool/zfs1@20250602     0B      -  7.96M  -
root@UbuntuTestVirt:~# rm /mypool/zfs1/*.gz
root@UbuntuTestVirt:~# ll /mypool/zfs1
total 122
drwxr-xr-x 10 root root     33 июн  2 23:08 ./
drwxr-xr-x  6 root root      6 июн  1 22:19 ../
-rw-r--r--  1 root root      0 июн  1 22:34 alternatives.log
-rw-r--r--  1 root root   8879 июн  1 22:34 alternatives.log.1
-rw-r-----  1 root root      0 июн  1 22:34 apport.log
drwxr-xr-x  2 root root     13 июн  1 22:34 apt/
-rw-r-----  1 root root   1250 июн  1 22:34 auth.log
-rw-r-----  1 root root  38839 июн  1 22:34 auth.log.1
-rw-r--r--  1 root root  61229 июн  1 22:34 bootstrap.log
-rw-r-----  1 root root      0 июн  1 22:34 btmp
-rw-r-----  1 root root      0 июн  1 22:34 btmp.1
-rw-r-----  1 root root  85170 июн  1 22:34 cloud-init.log
-rw-r-----  1 root root   4592 июн  1 22:34 cloud-init-output.log
drwxr-xr-x  2 root root      2 июн  1 22:34 dist-upgrade/
-rw-r-----  1 root root  57444 июн  1 22:34 dmesg
-rw-r-----  1 root root  58680 июн  1 22:34 dmesg.0
-rw-r--r--  1 root root      0 июн  1 22:34 dpkg.log
-rw-r--r--  1 root root  45043 июн  1 22:34 dpkg.log.1
-rw-r--r--  1 root root      0 июн  1 22:34 faillog
drwxr-x---  4 root root     24 июн  1 22:34 installer/
drwxr-xr-x  3 root root      3 июн  1 22:34 journal/
-rw-r-----  1 root root    392 июн  1 22:34 kern.log
-rw-r-----  1 root root  19196 июн  1 22:34 kern.log.1
drwxr-xr-x  2 root root      3 июн  1 22:34 landscape/
-rw-r--r--  1 root root 292292 июн  1 22:34 lastlog
drwx------  2 root root      2 июн  1 22:34 private/
lrwxrwxrwx  1 root root     39 июн  1 22:34 README -> ../../usr/share/doc/systemd/README.logs
-rw-r-----  1 root root   8313 июн  1 22:34 syslog
-rw-r-----  1 root root 166972 июн  1 22:34 syslog.1
drwxr-xr-x  2 root root     19 июн  1 22:34 sysstat/
-rw-r--r--  1 root root      0 июн  1 22:34 ubuntu-advantage-apt-hook.log
drwxr-x---  2 root root     13 июн  1 22:34 unattended-upgrades/
-rw-r--r--  1 root root  67584 июн  1 22:34 wtmp
root@UbuntuTestVirt:~# zfs list -t snapshot -r mypool/zfs1
NAME                   USED  AVAIL  REFER  MOUNTPOINT
mypool/zfs1@20250602   462K      -  7.96M  -
root@UbuntuTestVirt:~# zfs rollback mypool/zfs1@20250602
root@UbuntuTestVirt:~# ll /mypool/zfs1
total 560
drwxr-xr-x 10 root root     50 июн  1 22:34 ./
drwxr-xr-x  6 root root      6 июн  1 22:19 ../
-rw-r--r--  1 root root      0 июн  1 22:34 alternatives.log
-rw-r--r--  1 root root   8879 июн  1 22:34 alternatives.log.1
-rw-r--r--  1 root root   2519 июн  1 22:34 alternatives.log.2.gz
-rw-r-----  1 root root      0 июн  1 22:34 apport.log
drwxr-xr-x  2 root root     13 июн  1 22:34 apt/
-rw-r-----  1 root root   1250 июн  1 22:34 auth.log
-rw-r-----  1 root root  38839 июн  1 22:34 auth.log.1
-rw-r-----  1 root root   4424 июн  1 22:34 auth.log.2.gz
-rw-r-----  1 root root   2494 июн  1 22:34 auth.log.3.gz
-rw-r-----  1 root root   1090 июн  1 22:34 auth.log.4.gz
-rw-r--r--  1 root root  61229 июн  1 22:34 bootstrap.log
-rw-r-----  1 root root      0 июн  1 22:34 btmp
-rw-r-----  1 root root      0 июн  1 22:34 btmp.1
-rw-r-----  1 root root  85170 июн  1 22:34 cloud-init.log
-rw-r-----  1 root root   4592 июн  1 22:34 cloud-init-output.log
drwxr-xr-x  2 root root      2 июн  1 22:34 dist-upgrade/
-rw-r-----  1 root root  57444 июн  1 22:34 dmesg
-rw-r-----  1 root root  58680 июн  1 22:34 dmesg.0
-rw-r-----  1 root root  17266 июн  1 22:34 dmesg.1.gz
-rw-r-----  1 root root  17053 июн  1 22:34 dmesg.2.gz
-rw-r-----  1 root root  16990 июн  1 22:34 dmesg.3.gz
-rw-r-----  1 root root  17336 июн  1 22:34 dmesg.4.gz
-rw-r--r--  1 root root      0 июн  1 22:34 dpkg.log
-rw-r--r--  1 root root  45043 июн  1 22:34 dpkg.log.1
-rw-r--r--  1 root root   4443 июн  1 22:34 dpkg.log.2.gz
-rw-r--r--  1 root root   1642 июн  1 22:34 dpkg.log.3.gz
-rw-r--r--  1 root root  67810 июн  1 22:34 dpkg.log.4.gz
-rw-r--r--  1 root root      0 июн  1 22:34 faillog
drwxr-x---  4 root root     24 июн  1 22:34 installer/
drwxr-xr-x  3 root root      3 июн  1 22:34 journal/
-rw-r-----  1 root root    392 июн  1 22:34 kern.log
-rw-r-----  1 root root  19196 июн  1 22:34 kern.log.1
-rw-r-----  1 root root  17327 июн  1 22:34 kern.log.2.gz
-rw-r-----  1 root root  58950 июн  1 22:34 kern.log.3.gz
-rw-r-----  1 root root  14708 июн  1 22:34 kern.log.4.gz
drwxr-xr-x  2 root root      3 июн  1 22:34 landscape/
-rw-r--r--  1 root root 292292 июн  1 22:34 lastlog
drwx------  2 root root      2 июн  1 22:34 private/
lrwxrwxrwx  1 root root     39 июн  1 22:34 README -> ../../usr/share/doc/systemd/README.logs
-rw-r-----  1 root root   8313 июн  1 22:34 syslog
-rw-r-----  1 root root 166972 июн  1 22:34 syslog.1
-rw-r-----  1 root root  42748 июн  1 22:34 syslog.2.gz
-rw-r-----  1 root root 116554 июн  1 22:34 syslog.3.gz
-rw-r-----  1 root root  32725 июн  1 22:34 syslog.4.gz
drwxr-xr-x  2 root root     19 июн  1 22:34 sysstat/
-rw-r--r--  1 root root      0 июн  1 22:34 ubuntu-advantage-apt-hook.log
drwxr-x---  2 root root     13 июн  1 22:34 unattended-upgrades/
-rw-r--r--  1 root root  67584 июн  1 22:34 wtmp
```
