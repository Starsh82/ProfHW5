# ProfHW5
Домашнее задание №5. Практические навыки работы с ZFS.
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
