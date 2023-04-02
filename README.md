# Задание
+ Попасть в систему без пароля несколькими способами
+ Установить систему с LVM, после чего переименовать VG
+ Добавить модуль в initrd

# Попасть в систему без пароля несколькими способами
Чтобы попасть в параметр загрузки, нобходимо при выборе ядра нажать **e**. 
![Image alt](/image/boot2.png)

![Image alt](/image/boot1.png)

## Способ 1. init=/bin/sh
+ В конце строки начинающейся с **linux16** добавляем **init=/bin/sh** и нажимаем **ctrl+x** для загрузки в систему
+ Мы попали в систему, но рутовая файловая система монтируется в режиме **Read-Only**. Для перемонтирования ee в режим **Read-Write** можно воспользоваться командной:

**mount -o remount,rw /**

После чего убеждаемся, что рутовая фаловая система примонтирована в режим **Read-Write** командой:

**mount | grep root**

![Image alt](/image/boot3.png)

## Способ 2. rd.break
+ В конце строки начинающейся с **linux16** добавляем **rd.break** и нажимаем **ctrl+x** для загрузки в систему
+ Попадаем в emergency mode. Наша в корневая файловая система смонтирована. Но она смотирована в режиме **Read-Only** и мы не в ней. Далее будет пример как попасть в нее и поменять пароль администратора:

**mount -o remount,rw /sysroot**

**chroot /sysroot**

**passwd root**

**touch /.autorelabel**

![Image alt](/image/boot4.png)

Перезагружаемся и заходим в систему с новым паролем.

## Способ 3. rw init=/sysroot/bin/sh
+ В строке начинающейся с **linux16** заменяем ro на **rw init=/sysroot/bin/sh** и нажимаем **ctrl+x** для загрузки в систему

![Image alt](/image/boot5.png)

![Image alt](/image/boot6.png)

# Различие в способах
В первом способе мы оказываемся в рутовой файловой системе, во втором и третьем изначально попадаем в mergency mode. В третьем способе файловая система сразу смонтирована в Read-Write режиме, в первых двух способах можно заменить ro на rw и получить тот же режим.

# Установить систему с LVM, после чего переименовать VG

Посмотрим текущее состояние системы
```
[root@boot vagrant]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree
  centos   1   2   0 wz--n- <19,00g    0 
```
Приступим к переименованию:
```
[root@boot vagrant]# vgrename centos OtusRoot
  Volume group "centos" successfully renamed to "OtusRoot"
```
Далее правим /etc/fstab/, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое название на новое.

![Image alt](/image/boot7.png)

![Image alt](/image/boot8.png)

![Image alt](/image/boot9.png)

Пересоздаем initrd image, чтобý он знал новое название Volume Group
```
[root@boot vagrant]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
После перезагружаемся и если все сделано правильно успешно грузимся с новым именем.
```
[root@boot vagrant]# vgs 
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <19,00g    0 
```
# Добавить модуль в initrd

Скрипты модулей  находяться в директории /usr/lib/dracut/modules.d/. Для того чтобы добавить свой модуль создадим там папку с именем **01test**:

```
[root@boot vagrant]# mkdir /usr/lib/dracut/modules.d/01test
```
В эту директорию поместим два скрипта:
+ module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh
+ test.sh - собственно сам вызываемый скрипт, в нем нарисован пингвин.

Пересобираем образ initrd
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
Проверим какие модули загружены в образ: 
```
[root@boot 01test]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
Отредактируем файл /boot/grub2/grub.cfg, уберем опции rghb и quiet

![Image alt](/image/boot10.png)

Перезагружаемся

![Image alt](/image/boot11.png)





