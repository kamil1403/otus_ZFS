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

- [💡 Различные команды](#other)
- [⚙️ Форматирование и монтирование файловой системы](#ext4)
- [✂️ Расширение файловой системы](#resize)

---

<a id="other"></a>
## 💡 Различные команды

```bash
# Установка zpool   
sudo apt install zfsutils-linux   
# Показывает список всех ZFS-пулов, их размер, доступное пространство, состояние   
zpool list   
# Создает новый пул режиме mirror (RAID1 - данные дублируются на оба диска) из дисков /dev/vdb и /dev/vdd   
zpool create tmp_pool mirror /dev/vdb /dev/vdd -f   
# Детализированная информация о состоянии пула: устройства, ошибки, деградация   
zpool status   
# Открывает manual по команде zpool. /слово и стрелками искать впреред или назад. Также можно лисать результаты клавишей N. Клавишка q - выход   
man zpool   
# Создает ZFS файловые системы   
zfs create tmp_pool/zfs01   
zfs create tmp_pool/zfs02   
zfs create tmp_pool/zfs03   
# Показывает все файловые системы ZFS с их размером, использованием и статусом   
zfs list   
# Показывает статус дедупликации (удаление дубликатов блоков данных)   
zfs get dedup   
# Включает дедупликацию на файловой системе zfs02   
zfs set dedup=on tmp_pool/zfs02   
# Показывает все параметры ZFS с выводом ошибок в стандартный поток
zfs get all 2>&1 | less






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

