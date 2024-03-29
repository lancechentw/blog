+++
title = "Manually Install FreeBSD 9.0 Release on GPT and Gmirror"
date = "2012-08-22T17:50:00+08:00"
type = "post"
tags = ["FreeBSD"]
+++

新學期、新組別、新測試機

FreeBSD branch: [FreeBSD 9.0 Release][1]

## 準備安裝環境
依照你的環境，把installation media開起來，第一個畫面應該會像這樣
![](http://www.freebsd.org/doc/handbook/bsdinstall/bsdinstall-choose-mode.png)

因為標題寫"manually"，所以選擇**Shell**，接下來一切就靠自己了


## Preparing disk
我們使用GPT([GUID Partition Table][2]) scheme，GPT相較於MBR有以下兩個好處，

1. 可使用於大於2TB的硬碟
2. 不再受限於四個Primary partitions

### Create partition table
```bash
# 查看有沒有存在的partition table，應該要是空白的
$ gpart show

# 產生gpt scheme
$ gpart create -s gpt da0
$ gpart create -s gpt da1

# 切boot
$ gpart add -t freebsd-boot -s 128 -l boot0 da0
$ gpart add -t freebsd-boot -s 128 -l boot1 da1

# 把/boot/pmbr塞到硬碟的第一個sector去，OS就會去找type是freebsd-boot的partition
# 把/boot/gptboot塞到freebsd-boot partition去，OS就會去找type是freebsd-ufs的partition
# 並跑/boot/loader
# (for root on zfs, use gptzfsboot instead)
$ gpart bootcode -b /boot/pmbr -p /boot/gptboot -i 1 da0
$ gpart bootcode -b /boot/pmbr -p /boot/gptboot -i 1 da1

# 切swap
$ gpart add -t freebsd-swap -s 512M -l swap0 da0
$ gpart add -t freebsd-swap -s 512M -l swap1 da1

# 切root
$ gpart add -t freebsd-ufs -s 4G -l root0 da0
$ gpart add -t freebsd-ufs -s 4G -l root1 da1

# 剩下的部份切一塊給zfs
$ gpart add -t freebsd-zfs -l zpool0 da0
$ gpart add -t freebsd-zfs -l zpool1 da1

# 檢查一下結果
$ gpart show
=>      34  20971453  da0  GPT  (10G)
        34       128    1  freebsd-boot  (64k)
       162   1048576    2  freebsd-swap  (512M)
   1048738   8388608    3  freebsd-ufs  (4.0G)
   9437346  11534141    4  freebsd-zfs  (5.5G)

=>      34  20971453  da1  GPT  (10G)
        34       128    1  freebsd-boot  (64k)
       162   1048576    2  freebsd-swap  (512M)
   1048738   8388608    3  freebsd-ufs  (4.0G)
   9437346  11534141    4  freebsd-zfs  (5.5G)
```

### 使用gmirror作raid1，並建立filesystem
```bash
$ gmirror label -v -b round-robin gm0 /dev/gpt/root*
$ newfs -O2 -Uj /dev/mirror/gm0
```


## 安裝FreeBSD
### 安裝root filesystem
```bash
$ cd /usr/freebsd-dist
$ mount /dev/mirror/gm0 /mnt
$ tar xpf base.txz -C /mnt
$ tar xpf kernel.txz -C /mnt
$ tar xpf lib32.txz -C /mnt
```

### 基本設定
#### chroot && 設定root密碼
```bash
$ chroot /mnt
$ passwd
```

#### 新增第一個使用者
```bash
$ vipw
$ passwd lance
```

#### /etc/rc.conf
{{< gist Lance0312 3428614 "rc.conf" >}}

#### /etc/fstab
{{< gist Lance0312 3428614 "fstab" >}}

#### /boot/loader.conf
{{< gist Lance0312 3428614 "loader.conf" >}}


## 大功告成
```bash
$ exit
$ reboot
```

大致步驟如上，至於zfs以及ports的部份，再說!

[1]: http://ftp.tw.freebsd.org/pub/releases/ISO-IMAGES/9.0/FreeBSD-9.0-RELEASE-amd64-disc1.iso
[2]: http://en.wikipedia.org/wiki/GUID_Partition_Table
