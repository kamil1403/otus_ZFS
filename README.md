<p align="center">
  <img src="https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/lvm.jpg" alt="RAID Banner" width="800">
</p>

<h1 align="center">otus_ZFS</h1>
<p align="center">Дата: 19-05-2025<br>Автор: Kamil Ibragimov</p>

## Домашнее задание: работа с LVM
Задание:   
1. Определить алгоритм с наилучшим сжатием:   
• Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);   
• Cоздать 4 файловых системы на каждой применить свой алгоритм сжатия;   
• Для сжатия использовать либо текстовый файл, либо группу файлов.   
2. Определить настройки пула и при помощи команды zfs import собрать pool ZFS.   
Командами zfs определить настройки:     
• Размер хранилища;   
• Тип pool;   
• Значение recordsize;   
• Какое сжатие используется;   
• Какая контрольная сумма используется.   
3. Работа со снапшотами:   
• Скопировать файл из удаленной директории;   
• Сосстановить файл локально. zfs receive;   
• Найти зашифрованное сообщение в файле secret_message.   

Результат:   
• Создал Physical Volume, Volume Group и Logical Volume. Результат см. на скриншоте ["lvm_create"](https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/lvm_create.png)  
• Отформатировал том и смонтировал файловую систему в каталог /lvm. Результат см. на скриншоте ["mkfs.ext4"](https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/mkfs.ext4.png)  
• Расширил файловую систему за счет нового диска на 5Gb. Результат см. на скриншоте ["resize"](https://github.com/kamil1403/otus_LVM-1/blob/main/screenshots/resize.png) 



## 🧭 Оглавление

- [🛠️ Основные принципы LVM](#pvl)
- [⚙️ Форматирование и монтирование файловой системы](#ext4)
- [✂️ Расширение файловой системы](#resize)

---

<a id="pvl"></a>
## 🛠️ Создание Physical Volume, Volume Group и Logical Volume

```bash
# LVM (Logical Volume Manager) — это система управления дисками в Linux, позволяющая гибко объединять и перераспределять пространство на физических носителях   
# LVM использует три уровня:
# 1. PV (Physical Volume) — это "сырой" диск или раздел, подготовленный под использование в LVM   
pvcreate /dev/sdb
# 2. VG (Volume Group) — это контейнер, в который объединяются один или несколько PV   
vgcreate otus /dev/sdb
# 3. LV — это "кусок" из группы VG, который работает как обычный раздел, только гибче   
lvcreate -L 10G -n test otus
# Создание логического тома на 80% от доступного места   
lvcreate -l+80%FREE -n test otus   
# Удаление логического тома   
lvremove /dev/otus/test   
# Просмотр и управление LVM   
# Подробно:  
pvdispaly 
vgdisplay   
lvdisplay   
# Кратко:   
pvs   
vgs   
lvs   
```

---

<a id="ext4"></a>
## ⚙️ Форматирование и монтирование файловой системы

```bash
# Форматирование логического тома
mkfs.ext4 /dev/otus/test
# Монтирование логического тома
mkdir /lvm
mount /dev/otus/test /lvm
```

---

<a id="resize"></a>
## ✂️ Расширение файловой системы

```bash
# Инициализация нового PV   
pvcreate /dev/sdc   
vgextend otus /dev/sdc   
# Расширение логического тома   
lvextend -l+80%FREE /dev/otus/test
# Расширение файловой системы
resize2fs /dev/otus/test
```

---
