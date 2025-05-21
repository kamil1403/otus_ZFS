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
# Тестирование дедупликации и сжатия в ZFS

## 🧱 Подготовка пула

Создаём пул ZFS с зеркалированием:

```bash
zpool create tmp_pool mirror /dev/vdb /dev/vdd -f
```

Проверь статус пула:

```bash
zpool status
```

---

## 📁 Создание файловых систем

Создаём три независимые ZFS-файловые системы:

```bash
zfs create tmp_pool/zfs_dedup
zfs create tmp_pool/zfs_compress
zfs create tmp_pool/zfs_plain
```

---

## ⚙️ Настройка параметров

Включаем дедупликацию:

```bash
zfs set dedup=on tmp_pool/zfs_dedup
```

Включаем сжатие:

```bash
zfs set compression=on tmp_pool/zfs_compress
```

Контрольная система `zfs_plain` остаётся без изменений.

---

## 🧪 Генерация тестовых данных

Создаём файл из повторяющихся строк (хорошо сжимается и дедуплицируется):

```bash
mkdir test_data && cd test_data
yes "REPEATME" | head -c 50M > file_repeat.txt
```

---

## 📝 Копирование файла в файловые системы

```bash
cp file_repeat.txt /tmp_pool/zfs_plain/
cp file_repeat.txt /tmp_pool/zfs_dedup/
cp file_repeat.txt /tmp_pool/zfs_compress/
```

---

## 📊 Анализ результатов

Проверяем использование диска:

```bash
zfs list
```

Коэффициент сжатия:

```bash
zfs get compressratio
```

Статистика дедупликации:

```bash
zfs get dedup
zpool list
```

---

## ⚠️ Тестирование на случайных данных

Создаём несжимаемый файл:

```bash
dd if=/dev/urandom of=randfile.bin bs=1M count=50
```

Копируем его во все файловые системы:

```bash
cp randfile.bin /tmp_pool/zfs_plain/
cp randfile.bin /tmp_pool/zfs_dedup/
cp randfile.bin /tmp_pool/zfs_compress/
```

Снова анализируем:

```bash
zfs list
zfs get compressratio
zfs get dedup
```

---

## 🧼 Очистка (по желанию)

```bash
zfs destroy tmp_pool/zfs_dedup
zfs destroy tmp_pool/zfs_compress
zfs destroy tmp_pool/zfs_plain
rm -rf test_data randfile.bin
```

---

## 📌 Комментарии

- **Дедупликация** работает на уровне блоков и требует много оперативной памяти (особенно для больших файлов).
- **Сжатие** срабатывает сразу при записи и может эффективно экономить пространство на однотипных или повторяющихся данных.

---
```

---
