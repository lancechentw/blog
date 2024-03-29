+++
title = "Installing Archlinux on GPT and Software RAID"
date = "2012-08-31T14:54:00+08:00"
type = "post"
tags = ["Arch Linux"]
+++

昨天組內投票決議後，我們維護的工作站由Gentoo改為Archlinux，不知道是件好事還是壞事...

在官方[捨棄AIF(the Arch Installation Framework)][1]，並以[GRUB2取代GRUB legacy][2]後，
安裝流程對我們所需的環境變得友善多了。

------------------------------------------------------------------------------------

Installation media: [archlinux-2012.08.04-dual.iso][3]

## Preparing disk
### Create partition table
因為要用GPT scheme，所以使用gdisk來切partition table，依照個人喜好，你也可以選
擇parted。因為gdisk是interactive的操作模式，所以這邊不細講分割流程，只列出幾個
要點

1. 使用`o`產生空的GPT
2. 2MB的partition for GRUB，type code為`ef02`(BIOS boot partition)
3. `/boot`, `/`, `/tmp`的partition作raid，type code要設為`fd00`(Linux RAID)
4. 把partition table apply到另一顆硬碟上，使用sgdisk或者gdisk的backup功能

到此，應該會有類似這樣的partition table
```bash
$ sgdisk -p /dev/sda
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048            6143   2.0 MiB     EF02  BIOS boot partition
   2            6144          268287   128.0 MiB   FD00  Linux RAID
   3          268288         2365439   1024.0 MiB  8200  Linux swap
   4         2365440        17045503   7.0 GiB     FD00  Linux RAID
   5        17045504        20971486   1.9 GiB     FD00  Linux RAID
```

### Create mdadm array
接下來，用mdadm建立array
```bash
$ mdadm --create /dev/md2 --level=1 --metadata=0.90 --raid-devices=2 /dev/sda2 /dev/sdb2
$ mdadm --create /dev/md4 --level=1 --metadata=0.90 --raid-devices=2 /dev/sda4 /dev/sdb4
$ mdadm --create /dev/md5 --level=0 --raid-devices=2 /dev/sda5 /dev/sdb5
# 查看array的狀態
$ cat /proc/mdstat
```

### Format partitions
`/boot`使用ext2，`/`使用xfs，`/tmp`則使用ext4
```bash
$ mkfs.ext2 /dev/md2
$ mkfs.xfs /dev/md4
$ mkfs.ext4 /dev/md5
$ mkswap /dev/sda3
$ swapon /dev/sda3
$ mkswap /dev/sdb3
$ swapon /dev/sdb3
```

### Mount partitions
```bash
$ mount /dev/md4 /mnt
# 把切成獨立partition的部份也mount上去
$ mkdir /mnt/boot
$ mkdir /mnt/tmp
$ mount /dev/md2 /mnt/boot
$ mount /dev/md5 /mnt/tmp
```
    
## Installation
首先，修改/etc/pacman.d/mirrorlist，留下linux.cs.nctu.edu.tw就好
### Base system
```bash
$ pacstrap /mnt base base-devel
# genfstab會把有mount進/mnt的partition都一並列出
$ genfstab -p /mnt >> /mnt/etc/fstab
$ mdadm --detail --scan >> /mnt/etc/mdadm.conf
```

### Chroot
```bash
# 以往需要先手動mount上dev, sys, proc，arch-chroot都幫你作好了
$ arch-chroot /mnt
```

### Configuration
基本的設定請參照

1. [Configure the base system][4]
2. [Configure the network][5]

### Create an initial ramdisk environment
修改/etc/mkinitcpio.conf，在`HOOKS`中加入`mdadm`，並在`MODULES`中加入`raid1`，
接著產生initial ramdisk
```bash
$ mkinitcpio -p linux
```

### Bootloader
我們選擇GRUB for BIOS motherboard
```bash
$ pacman -S grub-bios
# 分別裝在兩顆硬碟上
$ grub-install --target=i386-pc --recheck /dev/sda
$ grub-install --target=i386-pc --recheck /dev/sdb
```

為了要讓GRUB認的懂RAID，修改/etc/default/grub，在`GRUB_PRELOAD_MODULES`中加入`raid`
，接著產生grub.cfg
```bash
$ grub-mkconfig -o /boot/grub/grub.cfg
```

到這邊大致就完成了，設定一下root password，然後退出chroot環境，並重開

------------------------------------------------------------------------------------

### Reference
1. [Arch wiki - Beginners' Guide](https://wiki.archlinux.org/index.php/Beginners'_Guide)
2. [Arch wiki - GPT](https://wiki.archlinux.org/index.php/GPT)
3. [Arch wiki - Mkinitcpio](https://wiki.archlinux.org/index.php/Mkinitcpio#Using_RAID)
4. [Arch wiki - GRUB2](https://wiki.archlinux.org/index.php/GRUB2#RAID)

[1]: http://www.archlinux.org/news/install-media-20120715-released/
[2]: http://www.archlinux.org/news/install-media-20120804-available/
[3]: http://linux.cs.nctu.edu.tw/archlinux/iso/2012.08.04/archlinux-2012.08.04-dual.iso
[4]: https://wiki.archlinux.org/index.php/Beginners'_Guide#Configure_the_base_system
[5]: https://wiki.archlinux.org/index.php/Beginners'_Guide#Configure_the_network
