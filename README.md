## Работа с lvm


### Цель:
Практика c LVM

### Ход работы:
Были добавлены в систему 5 виртуальных дисков по 512 Mb каждый.

Установлена *lvm* командой `sudo apt install lvm2 -y`

Смотрим текущее состояние:
```
lsblk
sudo lvmdiskscan
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/1.png)

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/2.png)

Добавляем диск как PV:

```
sudo pvcreate /dev/sdb
sudo pvdisplay
sudo lvmdiskscan
sudo pvs
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/3.png)

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/4.png)

Создаём VG на базе PV

```
sudo vgcreate mai /dev/sdb
sudo vgdisplay -v mai
sudo vgs
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/5.png)

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/6.png)

Создаём LV на базе VG

```
sudo lvcreate -l+100%FREE -n first mai
sudo lvdisplay
sudo lvs
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/7.png)


Создаём файловую систему, монтируем её и проверяем

```
sudo mkfs.ext4 /dev/mai/first
sudo mount /dev/mai/first /mnt
sudo mount
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/8.png)
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/24.png)




Создаём файл на весь размер точки монтирования
```
sudo dd if=/dev/zero of=/mnt/test.file bs=1M count=1500 status=progress
sudo df -h
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/9.png)

Расширяем LV за счёт нового PV в VG
```
sudo pvcreate /dev/sdc
sudo vgextend mai /dev/sdc
sudo lvextend -l+100%FREE /dev/mai/first
sudo lvdisplay
sudo lvs
sudo df -h
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/10.png)

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/11.png)
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/12.png)

Расширяем файловую систему
```
sudo resize2fs /dev/mai/first
sudo df -h
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/13.png)

Уменьшаем файловую систему и LV
```
sudo umount /mnt
sudo fsck -fy /dev/mai/first
sudo resize2fs -f /dev/mai/first 650M
sudo mount /dev/mai/first /mnt
sudo df -h
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/14.png)

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/15.png)

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/16.png)

Создаём несколько файлов и делаем снимок
```
sudo touch /mnt/file{1..5}
ls /mnt
sudo lvcreate -L 100M -s -n snapsh /dev/mai/first
sudo lvs
sudo lsblk
```
![](https://github.com/Pashayam/LinuxLab5/blob/main/images/17.png)

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/18.png)

Удаляем несколько файлов
```
sudo rm -f /mnt/file{1..3}
ls /mnt
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/19.png)


Монтируем снимок и проверяем, что файлы там есть. Отмонтируем.
```
sudo mkdir /snap
sudo mount /dev/mai/snapsh /snap
ls /snap
sudo umount /snap
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/20.png)

Отмонтируем файловую систему и производим слияние. Проверяем, что файлы на месте.
```
sudo umount /mnt
sudo lvconvert --merge /dev/mai/snapsh
sudo mount /dev/mai/first /mnt
ls /mnt
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/21.png)

Добавляем ещё PV, VG и создаём LV-зеркало.
```
sudo pvcreate /dev/sd{d,e}
sudo vgcreate vgmirror /dev/sd{d,e}
sudo lvcreate -l+80%FREE -m1 -n mirror1 vgmirror
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/22.png)


Наблюдаем синхронизацию.
```
sudo lvs
```

![](https://github.com/Pashayam/LinuxLab5/blob/main/images/23.png)
